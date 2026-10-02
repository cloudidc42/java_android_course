# Part 67: Firebase Analytics & Crashlytics
## หลักสูตร Java & Android Development - ระดับ World-Class Android

---

## 67.1 Firebase Analytics Setup

```groovy
// build.gradle
implementation platform('com.google.firebase:firebase-bom:32.7.0')
implementation 'com.google.firebase:firebase-analytics'
implementation 'com.google.firebase:firebase-crashlytics'
implementation 'com.google.firebase:firebase-perf'  // Performance monitoring
```

```java
// AnalyticsHelper.java - wrapper สำหรับ Firebase Analytics
public class AnalyticsHelper {
    
    private final FirebaseAnalytics analytics;
    
    // Singleton
    private static AnalyticsHelper instance;
    
    public static void init(Context context) {
        instance = new AnalyticsHelper(context);
    }
    
    public static AnalyticsHelper get() {
        if (instance == null) throw new IllegalStateException("Not initialized");
        return instance;
    }
    
    private AnalyticsHelper(Context context) {
        analytics = FirebaseAnalytics.getInstance(context);
    }
    
    // Track screen view
    public void trackScreen(String screenName, String screenClass) {
        Bundle params = new Bundle();
        params.putString(FirebaseAnalytics.Param.SCREEN_NAME, screenName);
        params.putString(FirebaseAnalytics.Param.SCREEN_CLASS, screenClass);
        analytics.logEvent(FirebaseAnalytics.Event.SCREEN_VIEW, params);
    }
    
    // Track button click
    public void trackButtonClick(String buttonName, String screen) {
        Bundle params = new Bundle();
        params.putString("button_name", buttonName);
        params.putString("screen", screen);
        analytics.logEvent("button_click", params);
    }
    
    // Track purchase
    public void trackPurchase(String itemId, String itemName, double price, String currency) {
        Bundle params = new Bundle();
        params.putString(FirebaseAnalytics.Param.ITEM_ID, itemId);
        params.putString(FirebaseAnalytics.Param.ITEM_NAME, itemName);
        params.putDouble(FirebaseAnalytics.Param.PRICE, price);
        params.putString(FirebaseAnalytics.Param.CURRENCY, currency);
        params.putLong(FirebaseAnalytics.Param.QUANTITY, 1);
        analytics.logEvent(FirebaseAnalytics.Event.PURCHASE, params);
    }
    
    // Track search
    public void trackSearch(String query, int resultCount) {
        Bundle params = new Bundle();
        params.putString(FirebaseAnalytics.Param.SEARCH_TERM, query);
        params.putInt("result_count", resultCount);
        analytics.logEvent(FirebaseAnalytics.Event.SEARCH, params);
    }
    
    // Track login
    public void trackLogin(String method) {
        Bundle params = new Bundle();
        params.putString(FirebaseAnalytics.Param.METHOD, method);
        analytics.logEvent(FirebaseAnalytics.Event.LOGIN, params);
    }
    
    // Track signup
    public void trackSignUp(String method) {
        Bundle params = new Bundle();
        params.putString(FirebaseAnalytics.Param.METHOD, method);
        analytics.logEvent(FirebaseAnalytics.Event.SIGN_UP, params);
    }
    
    // Track content view
    public void trackContentView(String contentType, String contentId) {
        Bundle params = new Bundle();
        params.putString(FirebaseAnalytics.Param.CONTENT_TYPE, contentType);
        params.putString(FirebaseAnalytics.Param.ITEM_ID, contentId);
        analytics.logEvent(FirebaseAnalytics.Event.SELECT_CONTENT, params);
    }
    
    // Custom event
    public void trackEvent(String eventName, Bundle params) {
        analytics.logEvent(eventName, params);
    }
    
    // Set user properties
    public void setUserId(String userId) {
        analytics.setUserId(userId);
    }
    
    public void setUserProperty(String key, String value) {
        analytics.setUserProperty(key, value);
    }
    
    // Opt out (for GDPR)
    public void setCollectionEnabled(boolean enabled) {
        analytics.setAnalyticsCollectionEnabled(enabled);
    }
}
```

---

## 67.2 Automatic Screen Tracking

```java
// AnalyticsActivity.java - base class for auto screen tracking
public abstract class AnalyticsActivity extends AppCompatActivity {
    
    @Override
    protected void onResume() {
        super.onResume();
        
        String screenName = getScreenName();
        if (screenName != null) {
            AnalyticsHelper.get().trackScreen(screenName, getClass().getSimpleName());
        }
    }
    
    // Override in subclasses to provide screen name
    protected String getScreenName() {
        // Default: use class name
        return getClass().getSimpleName().replace("Activity", "");
    }
}

// Usage
public class ProductListActivity extends AnalyticsActivity {
    @Override
    protected String getScreenName() { return "product_list"; }
}
```

---

## 67.3 Firebase Crashlytics

```java
// CrashReporter.java
public class CrashReporter {
    
    private final FirebaseCrashlytics crashlytics;
    
    public CrashReporter() {
        crashlytics = FirebaseCrashlytics.getInstance();
    }
    
    // Set user context for crash reports
    public void setUser(String userId, String email) {
        crashlytics.setUserId(userId);
        crashlytics.setCustomKey("user_email", email);
    }
    
    // Add custom key to crash reports
    public void setContext(String key, String value) {
        crashlytics.setCustomKey(key, value);
    }
    
    public void setContext(String key, boolean value) {
        crashlytics.setCustomKey(key, value);
    }
    
    public void setContext(String key, int value) {
        crashlytics.setCustomKey(key, value);
    }
    
    // Log breadcrumb (shows in crash report)
    public void log(String message) {
        crashlytics.log(message);
    }
    
    // Report non-fatal exception
    public void reportError(Throwable throwable) {
        crashlytics.recordException(throwable);
    }
    
    public void reportError(String message, Throwable cause) {
        crashlytics.log("Error: " + message);
        crashlytics.recordException(cause);
    }
    
    // Test crash (for testing only - REMOVE in production!)
    public void testCrash() {
        throw new RuntimeException("Test crash from CrashReporter");
    }
}

// Global exception handler
public class MyApplication extends Application {
    
    @Override
    public void onCreate() {
        super.onCreate();
        
        AnalyticsHelper.init(this);
        
        // Catch uncaught exceptions for extra logging
        Thread.setDefaultUncaughtExceptionHandler((thread, throwable) -> {
            FirebaseCrashlytics.getInstance().log(
                "Uncaught exception on thread: " + thread.getName());
            FirebaseCrashlytics.getInstance().recordException(throwable);
            
            // Let Android's default handler crash the app
            System.exit(1);
        });
    }
}
```

---

## 67.4 Firebase Performance Monitoring

```java
// PerformanceHelper.java
public class PerformanceHelper {
    
    // Custom trace for measuring performance
    public static Trace startTrace(String traceName) {
        Trace trace = FirebasePerformance.getInstance().newTrace(traceName);
        trace.start();
        return trace;
    }
    
    public static void stopTrace(Trace trace) {
        if (trace != null) trace.stop();
    }
    
    public static void stopTrace(Trace trace, String metric, long value) {
        if (trace != null) {
            trace.putMetric(metric, value);
            trace.stop();
        }
    }
    
    // Measure a block of code
    public static <T> T measure(String traceName, Supplier<T> block) {
        Trace trace = startTrace(traceName);
        try {
            return block.get();
        } finally {
            stopTrace(trace);
        }
    }
}

// Usage
Trace trace = PerformanceHelper.startTrace("image_processing");
// ... processing ...
PerformanceHelper.stopTrace(trace, "items_processed", 10);

// HTTP monitoring is automatic via OkHttp integration
```

---

## 67.5 A/B Testing with Remote Config

```java
// RemoteConfigHelper.java
public class RemoteConfigHelper {
    
    private final FirebaseRemoteConfig remoteConfig;
    
    // Feature flag keys
    public static final String KEY_NEW_CHECKOUT_FLOW = "new_checkout_flow";
    public static final String KEY_SHOW_PROMOTIONS   = "show_promotions";
    public static final String KEY_MAX_ITEMS_CART    = "max_items_cart";
    public static final String KEY_WELCOME_MESSAGE   = "welcome_message";
    
    public RemoteConfigHelper() {
        remoteConfig = FirebaseRemoteConfig.getInstance();
        
        // Default values (shown before fetch)
        Map<String, Object> defaults = new HashMap<>();
        defaults.put(KEY_NEW_CHECKOUT_FLOW, false);
        defaults.put(KEY_SHOW_PROMOTIONS, true);
        defaults.put(KEY_MAX_ITEMS_CART, 10);
        defaults.put(KEY_WELCOME_MESSAGE, "ยินดีต้อนรับ!");
        remoteConfig.setDefaultsAsync(defaults);
        
        // Fetch interval (1 hour in production, 0 in debug)
        remoteConfig.setConfigSettingsAsync(
            new FirebaseRemoteConfigSettings.Builder()
                .setMinimumFetchIntervalInSeconds(BuildConfig.DEBUG ? 0 : 3600)
                .build());
    }
    
    public Task<Boolean> fetchAndActivate() {
        return remoteConfig.fetchAndActivate();
    }
    
    public boolean isNewCheckoutEnabled() {
        return remoteConfig.getBoolean(KEY_NEW_CHECKOUT_FLOW);
    }
    
    public boolean showPromotions() {
        return remoteConfig.getBoolean(KEY_SHOW_PROMOTIONS);
    }
    
    public int getMaxCartItems() {
        return (int) remoteConfig.getLong(KEY_MAX_ITEMS_CART);
    }
    
    public String getWelcomeMessage() {
        return remoteConfig.getString(KEY_WELCOME_MESSAGE);
    }
}

// Usage in Activity
remoteConfig.fetchAndActivate().addOnCompleteListener(task -> {
    if (task.isSuccessful()) {
        boolean newFlow = remoteConfig.isNewCheckoutEnabled();
        if (newFlow) {
            // Show new checkout UI
        } else {
            // Show old checkout UI
        }
    }
});
```

---

## 67.6 สรุป Part 67

ในบทนี้คุณได้เรียนรู้:

✅ Firebase Analytics setup  
✅ AnalyticsHelper wrapper  
✅ Standard events (screen, purchase, login)  
✅ Auto screen tracking (base Activity)  
✅ Firebase Crashlytics (setUser, log, recordException)  
✅ Global exception handler  
✅ Firebase Performance (custom traces)  
✅ Remote Config + A/B testing  

---

*[← Part 66: WebSocket](./part-66-android-websocket.md) | [Part 68: Advanced Notifications →](./part-68-android-advanced-notifications.md)*
