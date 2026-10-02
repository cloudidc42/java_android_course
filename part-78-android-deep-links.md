# Part 78: Deep Linking & Android App Links
## หลักสูตร Java & Android Development - ระดับ World-Class Android

---

## 78.1 Deep Links vs App Links

```
Deep Link:              myapp://product/123
Web Link (HTTP):        https://example.com/product/123
Android App Link:       https://example.com/.well-known/assetlinks.json (verified)

App Link (Android 6+):
- ต้องยืนยัน domain ownership ผ่าน assetlinks.json
- ผู้ใช้ไม่เห็น dialog "Open with..." → เปิด app โดยตรง
```

---

## 78.2 Setup Deep Links in Navigation

```xml
<!-- res/navigation/nav_graph.xml -->
<navigation ...>
    
    <fragment android:id="@+id/productDetailFragment"
        android:name=".presentation.product.ProductDetailFragment">
        
        <!-- Custom scheme deep link -->
        <deepLink
            app:uri="myapp://product/{productId}"
            app:action="android.intent.action.VIEW" />
        
        <!-- HTTP deep link (App Link) -->
        <deepLink
            app:uri="https://shopapp.example.com/product/{productId}" />
        
        <!-- Multiple patterns -->
        <deepLink app:uri="myapp://product/{productId}?ref={referralCode}" />
        
        <argument android:name="productId"
            app:argType="string" />
        <argument android:name="referralCode"
            app:argType="string"
            android:defaultValue="none" />
    </fragment>
    
    <fragment android:id="@+id/orderDetailFragment"
        android:name=".presentation.order.OrderDetailFragment">
        <deepLink app:uri="myapp://order/{orderId}" />
        <argument android:name="orderId" app:argType="string" />
    </fragment>
    
    <fragment android:id="@+id/categoryFragment"
        android:name=".presentation.home.CategoryFragment">
        <deepLink app:uri="myapp://category/{categoryName}" />
        <argument android:name="categoryName" app:argType="string" />
    </fragment>
    
</navigation>
```

```xml
<!-- AndroidManifest.xml -->
<activity android:name=".presentation.MainActivity"
    android:exported="true"
    android:launchMode="singleTop">
    
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
    
    <!-- Custom scheme - handled by NavController -->
    <intent-filter android:autoVerify="false">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="myapp" />
    </intent-filter>
    
    <!-- HTTPS App Links - requires assetlinks.json -->
    <intent-filter android:autoVerify="true">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="https"
              android:host="shopapp.example.com" />
    </intent-filter>
    
</activity>
```

---

## 78.3 Handle Deep Link in Activity

```java
// MainActivity.java
@AndroidEntryPoint
public class MainActivity extends AppCompatActivity {
    
    private NavController navController;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        NavHostFragment navHostFragment = (NavHostFragment)
            getSupportFragmentManager().findFragmentById(R.id.navHostFragment);
        navController = navHostFragment.getNavController();
        
        // NavController handles deep links automatically
        // But we can also handle manually:
        handleDeepLink(getIntent());
    }
    
    @Override
    protected void onNewIntent(Intent intent) {
        super.onNewIntent(intent);
        setIntent(intent);
        handleDeepLink(intent);
    }
    
    private void handleDeepLink(Intent intent) {
        if (intent == null) return;
        
        Uri data = intent.getData();
        if (data == null) return;
        
        String scheme = data.getScheme();
        String host   = data.getHost();
        String path   = data.getPath();
        
        Log.d("DeepLink", "Received: " + data.toString());
        
        // Let NavController handle it (reads nav_graph deep links)
        if (!navController.handleDeepLink(intent)) {
            // Manual handling if nav graph doesn't cover it
            handleCustomDeepLink(data);
        }
    }
    
    private void handleCustomDeepLink(Uri uri) {
        String path = uri.getPath();
        if (path == null) return;
        
        if (path.startsWith("/promo/")) {
            String promoCode = path.substring(7);
            Bundle args = new Bundle();
            args.putString("promo_code", promoCode);
            navController.navigate(R.id.promoFragment, args);
        }
    }
}
```

---

## 78.4 Create Deep Links Programmatically

```java
// Create deep links for sharing
public class DeepLinkHelper {
    
    // Build product share link
    public static Uri buildProductLink(String productId) {
        return Uri.parse("https://shopapp.example.com/product/" + productId);
    }
    
    // Build dynamic link with referral
    public static String buildReferralLink(String productId, String userId) {
        return "myapp://product/" + productId + "?ref=" + userId;
    }
    
    // Share product via Intent
    public static void shareProduct(Activity activity, Product product) {
        String shareUrl = "https://shopapp.example.com/product/" + product.getId();
        String shareText = "ดูสินค้านี้: " + product.getName() + "\n" + shareUrl;
        
        Intent shareIntent = new Intent(Intent.ACTION_SEND);
        shareIntent.setType("text/plain");
        shareIntent.putExtra(Intent.EXTRA_TEXT, shareText);
        shareIntent.putExtra(Intent.EXTRA_SUBJECT, product.getName());
        
        activity.startActivity(Intent.createChooser(shareIntent, "แชร์สินค้า"));
    }
    
    // Navigate to a deep link in the same app
    public static void navigateToDeepLink(NavController navController, String deepLinkUri) {
        NavDeepLinkRequest request = NavDeepLinkRequest.Builder
            .fromUri(Uri.parse(deepLinkUri))
            .build();
        navController.navigate(request);
    }
}
```

---

## 78.5 assetlinks.json (App Links Verification)

```json
// Place at: https://example.com/.well-known/assetlinks.json
[
  {
    "relation": ["delegate_permission/common.handle_all_urls"],
    "target": {
      "namespace": "android_app",
      "package_name": "com.myapp.shopapp",
      "sha256_cert_fingerprints": [
        "AA:BB:CC:DD:EE:FF:00:11:22:33:44:55:66:77:88:99:AA:BB:CC:DD:EE:FF:00:11:22:33:44:55:66:77:88:99"
      ]
    }
  }
]
```

```bash
# Get your app's SHA256 fingerprint:
keytool -list -v -keystore release.jks -alias release | grep SHA256

# Test App Link verification:
adb shell pm get-app-links com.myapp.shopapp
```

---

## 78.6 สรุป Part 78

ในบทนี้คุณได้เรียนรู้:

✅ Deep Links vs App Links  
✅ Deep link setup in nav_graph.xml  
✅ Intent filter in AndroidManifest  
✅ autoVerify for App Links  
✅ Handle deep link in Activity (onCreate, onNewIntent)  
✅ NavController.handleDeepLink  
✅ Create share links programmatically  
✅ assetlinks.json setup  

---

*[← Part 77: Architecture Deep](./part-77-android-architecture-deep.md) | [Part 79: Gradle Build System →](./part-79-android-gradle.md)*
