# Part 80: Android App Monetization
## หลักสูตร Java & Android Development - ระดับ World-Class

---

## 80.1 In-App Purchases (Google Play Billing)

```groovy
// build.gradle
implementation 'com.android.billingclient:billing:6.2.1'
```

```java
// BillingManager.java
public class BillingManager {
    
    private final Context context;
    private BillingClient billingClient;
    private final List<PurchaseListener> listeners = new ArrayList<>();
    
    public interface PurchaseListener {
        void onPurchaseSuccess(Purchase purchase);
        void onPurchaseFailed(int responseCode);
    }
    
    public BillingManager(Context context) {
        this.context = context;
        setupBillingClient();
    }
    
    private void setupBillingClient() {
        billingClient = BillingClient.newBuilder(context)
            .setListener(this::handlePurchasesUpdated)
            .enablePendingPurchases()
            .build();
        
        billingClient.startConnection(new BillingClientStateListener() {
            @Override
            public void onBillingSetupFinished(@NonNull BillingResult result) {
                if (result.getResponseCode() == BillingClient.BillingResponseCode.OK) {
                    Log.d("Billing", "Connected to Google Play Billing");
                    queryExistingPurchases();
                }
            }
            
            @Override
            public void onBillingServiceDisconnected() {
                // Retry after delay
                new Handler(Looper.getMainLooper()).postDelayed(
                    () -> setupBillingClient(), 3000);
            }
        });
    }
    
    // Query available products
    public void queryProducts(List<String> productIds, Consumer<List<ProductDetails>> callback) {
        List<QueryProductDetailsParams.Product> products = productIds.stream()
            .map(id -> QueryProductDetailsParams.Product.newBuilder()
                .setProductId(id)
                .setProductType(BillingClient.ProductType.INAPP)
                .build())
            .collect(Collectors.toList());
        
        QueryProductDetailsParams params = QueryProductDetailsParams.newBuilder()
            .setProductList(products)
            .build();
        
        billingClient.queryProductDetailsAsync(params, (result, details) -> {
            if (result.getResponseCode() == BillingClient.BillingResponseCode.OK) {
                callback.accept(details);
            }
        });
    }
    
    // Launch purchase flow
    public void launchPurchase(Activity activity, ProductDetails productDetails) {
        BillingFlowParams params = BillingFlowParams.newBuilder()
            .setProductDetailsParamsList(Collections.singletonList(
                BillingFlowParams.ProductDetailsParams.newBuilder()
                    .setProductDetails(productDetails)
                    .build()))
            .build();
        
        BillingResult result = billingClient.launchBillingFlow(activity, params);
        if (result.getResponseCode() != BillingClient.BillingResponseCode.OK) {
            notifyFailed(result.getResponseCode());
        }
    }
    
    // Subscription products
    public void querySubscriptions(List<String> subIds, Consumer<List<ProductDetails>> callback) {
        List<QueryProductDetailsParams.Product> products = subIds.stream()
            .map(id -> QueryProductDetailsParams.Product.newBuilder()
                .setProductId(id)
                .setProductType(BillingClient.ProductType.SUBS)
                .build())
            .collect(Collectors.toList());
        
        QueryProductDetailsParams params = QueryProductDetailsParams.newBuilder()
            .setProductList(products)
            .build();
        
        billingClient.queryProductDetailsAsync(params, (result, details) -> {
            if (result.getResponseCode() == BillingClient.BillingResponseCode.OK) {
                callback.accept(details);
            }
        });
    }
    
    private void handlePurchasesUpdated(@NonNull BillingResult result,
            @Nullable List<Purchase> purchases) {
        if (result.getResponseCode() == BillingClient.BillingResponseCode.OK
                && purchases != null) {
            for (Purchase purchase : purchases) {
                handlePurchase(purchase);
            }
        } else {
            notifyFailed(result.getResponseCode());
        }
    }
    
    private void handlePurchase(Purchase purchase) {
        if (purchase.getPurchaseState() == Purchase.PurchaseState.PURCHASED) {
            // Acknowledge purchase (required within 3 days)
            if (!purchase.isAcknowledged()) {
                AcknowledgePurchaseParams ackParams = AcknowledgePurchaseParams.newBuilder()
                    .setPurchaseToken(purchase.getPurchaseToken())
                    .build();
                billingClient.acknowledgePurchase(ackParams, ackResult -> {
                    if (ackResult.getResponseCode() == BillingClient.BillingResponseCode.OK) {
                        notifySuccess(purchase);
                    }
                });
            } else {
                notifySuccess(purchase);
            }
        }
    }
    
    private void queryExistingPurchases() {
        billingClient.queryPurchasesAsync(
            QueryPurchasesParams.newBuilder()
                .setProductType(BillingClient.ProductType.INAPP)
                .build(),
            (result, purchases) -> {
                // Restore purchases (e.g., after reinstall)
                for (Purchase p : purchases) {
                    if (p.getPurchaseState() == Purchase.PurchaseState.PURCHASED) {
                        notifySuccess(p);
                    }
                }
            });
    }
    
    public void addListener(PurchaseListener l)    { listeners.add(l); }
    public void removeListener(PurchaseListener l) { listeners.remove(l); }
    
    private void notifySuccess(Purchase p) {
        for (PurchaseListener l : listeners) l.onPurchaseSuccess(p);
    }
    private void notifyFailed(int code) {
        for (PurchaseListener l : listeners) l.onPurchaseFailed(code);
    }
    
    public void destroy() {
        if (billingClient != null && billingClient.isReady()) {
            billingClient.endConnection();
        }
    }
}
```

---

## 80.2 AdMob Integration

```groovy
// build.gradle
implementation 'com.google.android.gms:play-services-ads:23.5.0'
```

```xml
<!-- AndroidManifest.xml -->
<meta-data
    android:name="com.google.android.gms.ads.APPLICATION_ID"
    android:value="ca-app-pub-XXXXXXXXXXXXXXXX~XXXXXXXXXX"/>
```

```java
// AdManager.java
public class AdManager {
    
    private InterstitialAd interstitialAd;
    private RewardedAd rewardedAd;
    
    // Test IDs
    private static final String BANNER_ID       = "ca-app-pub-3940256099942544/6300978111";
    private static final String INTERSTITIAL_ID = "ca-app-pub-3940256099942544/1033173712";
    private static final String REWARDED_ID     = "ca-app-pub-3940256099942544/5224354917";
    
    public static void initialize(Context context) {
        MobileAds.initialize(context, initStatus -> {
            Log.d("AdMob", "Initialized");
        });
    }
    
    // Banner Ad
    public void setupBannerAd(AdView adView) {
        AdRequest request = new AdRequest.Builder().build();
        adView.setAdListener(new AdListener() {
            @Override
            public void onAdLoaded() { Log.d("AdMob", "Banner loaded"); }
            
            @Override
            public void onAdFailedToLoad(@NonNull LoadAdError error) {
                Log.e("AdMob", "Banner failed: " + error.getMessage());
            }
        });
        adView.loadAd(request);
    }
    
    // Interstitial Ad
    public void loadInterstitial(Activity activity, Runnable onLoaded) {
        InterstitialAd.load(activity, INTERSTITIAL_ID,
            new AdRequest.Builder().build(),
            new InterstitialAdLoadCallback() {
                @Override
                public void onAdLoaded(@NonNull InterstitialAd ad) {
                    interstitialAd = ad;
                    if (onLoaded != null) onLoaded.run();
                }
                
                @Override
                public void onAdFailedToLoad(@NonNull LoadAdError error) {
                    Log.e("AdMob", "Interstitial failed: " + error.getMessage());
                    interstitialAd = null;
                }
            });
    }
    
    public void showInterstitial(Activity activity, Runnable onDismissed) {
        if (interstitialAd != null) {
            interstitialAd.setFullScreenContentCallback(new FullScreenContentCallback() {
                @Override
                public void onAdDismissedFullScreenContent() {
                    interstitialAd = null;
                    loadInterstitial(activity, null);  // preload next
                    if (onDismissed != null) onDismissed.run();
                }
                
                @Override
                public void onAdFailedToShowFullScreenContent(@NonNull AdError error) {
                    interstitialAd = null;
                }
            });
            interstitialAd.show(activity);
        } else {
            if (onDismissed != null) onDismissed.run();
        }
    }
    
    // Rewarded Ad
    public void loadRewardedAd(Activity activity) {
        RewardedAd.load(activity, REWARDED_ID,
            new AdRequest.Builder().build(),
            new RewardedAdLoadCallback() {
                @Override
                public void onAdLoaded(@NonNull RewardedAd ad) {
                    rewardedAd = ad;
                }
                
                @Override
                public void onAdFailedToLoad(@NonNull LoadAdError error) {
                    rewardedAd = null;
                }
            });
    }
    
    public void showRewardedAd(Activity activity, Consumer<RewardItem> onRewarded) {
        if (rewardedAd != null) {
            rewardedAd.show(activity, reward -> {
                Log.d("AdMob", "Reward: " + reward.getAmount() + " " + reward.getType());
                onRewarded.accept(reward);
                rewardedAd = null;
                loadRewardedAd(activity);
            });
        }
    }
}
```

---

## 80.3 สรุป Part 80

ในบทนี้คุณได้เรียนรู้:

✅ Google Play Billing setup  
✅ Query products (INAPP / SUBS)  
✅ Launch purchase flow  
✅ Acknowledge purchase (required)  
✅ Restore purchases on reinstall  
✅ AdMob Banner, Interstitial, Rewarded Ads  
✅ Preload ads for smooth UX  

---

*[← Part 79: Gradle Build System](./part-79-android-gradle.md) | [Part 81: Android Security →](./part-81-android-security.md)*
