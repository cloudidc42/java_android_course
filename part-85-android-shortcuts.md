# Part 85: Android App Shortcuts
## หลักสูตร Java & Android Development - ระดับ World-Class

---

## 85.1 Static Shortcuts (XML)

```xml
<!-- res/xml/shortcuts.xml -->
<?xml version="1.0" encoding="utf-8"?>
<shortcuts xmlns:android="http://schemas.android.com/apk/res/android">
    
    <shortcut
        android:shortcutId="open_cart"
        android:enabled="true"
        android:icon="@drawable/ic_cart"
        android:shortcutShortLabel="@string/shortcut_cart_short"
        android:shortcutLongLabel="@string/shortcut_cart_long"
        android:shortcutDisabledMessage="@string/shortcut_cart_disabled">
        
        <intent
            android:action="android.intent.action.VIEW"
            android:targetPackage="com.myapp.shopapp"
            android:targetClass="com.myapp.shopapp.presentation.MainActivity">
            <extra android:name="destination" android:value="cart" />
        </intent>
        
    </shortcut>
    
    <shortcut
        android:shortcutId="search_products"
        android:enabled="true"
        android:icon="@drawable/ic_search"
        android:shortcutShortLabel="@string/shortcut_search_short"
        android:shortcutLongLabel="@string/shortcut_search_long">
        
        <!-- Multiple intents = back stack -->
        <intent
            android:action="android.intent.action.MAIN"
            android:targetPackage="com.myapp.shopapp"
            android:targetClass="com.myapp.shopapp.presentation.MainActivity" />
        
        <intent
            android:action="android.intent.action.VIEW"
            android:targetPackage="com.myapp.shopapp"
            android:targetClass="com.myapp.shopapp.presentation.search.SearchActivity" />
        
    </shortcut>
    
</shortcuts>
```

```xml
<!-- AndroidManifest.xml -->
<activity android:name=".presentation.MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
    
    <!-- Link shortcuts to this launcher activity -->
    <meta-data
        android:name="android.app.shortcuts"
        android:resource="@xml/shortcuts" />
</activity>
```

---

## 85.2 Dynamic Shortcuts (Java)

```java
// ShortcutManager.java
public class AppShortcutManager {
    
    private final Context context;
    private final ShortcutManagerCompat shortcutManager;
    
    public AppShortcutManager(Context context) {
        this.context = context;
        this.shortcutManager = ShortcutManagerCompat.getInstance(context);
    }
    
    // Add dynamic shortcut for recently viewed product
    public void addProductShortcut(Product product) {
        if (!shortcutManager.isRequestPinShortcutSupported()) return;
        
        Intent intent = new Intent(context, MainActivity.class);
        intent.setAction(Intent.ACTION_VIEW);
        intent.putExtra("destination", "product_detail");
        intent.putExtra("product_id", product.getId());
        
        ShortcutInfoCompat shortcut = new ShortcutInfoCompat.Builder(context, "product_" + product.getId())
            .setShortLabel(product.getName())
            .setLongLabel("ดูสินค้า: " + product.getName())
            .setIcon(IconCompat.createWithResource(context, R.drawable.ic_product))
            .setIntent(intent)
            .setRank(0)  // Higher rank = appears first
            .build();
        
        // Push dynamic shortcut
        shortcutManager.pushDynamicShortcut(shortcut);
        
        // Remove oldest if exceeded limit (typically 5)
        trimShortcuts();
    }
    
    // Remove specific shortcut
    public void removeProductShortcut(String productId) {
        shortcutManager.removeDynamicShortcuts(
            Collections.singletonList("product_" + productId));
    }
    
    // Report usage (for ranking/suggestions)
    public void reportShortcutUsed(String shortcutId) {
        shortcutManager.reportShortcutUsed(shortcutId);
    }
    
    // Get all dynamic shortcuts
    public List<ShortcutInfoCompat> getDynamicShortcuts() {
        return shortcutManager.getDynamicShortcuts();
    }
    
    private void trimShortcuts() {
        int maxCount = shortcutManager.getMaxShortcutCountPerActivity();
        List<ShortcutInfoCompat> shortcuts = getDynamicShortcuts();
        
        if (shortcuts.size() >= maxCount) {
            // Remove lowest-ranked (last) shortcut
            List<String> toRemove = shortcuts.stream()
                .sorted(Comparator.comparingInt(ShortcutInfoCompat::getRank).reversed())
                .limit(shortcuts.size() - maxCount + 1)
                .map(ShortcutInfoCompat::getId)
                .collect(Collectors.toList());
            shortcutManager.removeDynamicShortcuts(toRemove);
        }
    }
    
    // Request to pin shortcut on home screen
    public void requestPinShortcut(ShortcutInfoCompat shortcut) {
        if (!shortcutManager.isRequestPinShortcutSupported()) return;
        
        PendingIntent successCallback = PendingIntent.getBroadcast(
            context, 0,
            new Intent(context, ShortcutPinnedReceiver.class),
            PendingIntent.FLAG_IMMUTABLE);
        
        shortcutManager.requestPinShortcut(shortcut, successCallback.getIntentSender());
    }
    
    // Update shortcut (e.g., change label/icon)
    public void updateShortcut(String shortcutId, String newLabel) {
        List<ShortcutInfoCompat> shortcuts = getDynamicShortcuts().stream()
            .filter(s -> s.getId().equals(shortcutId))
            .collect(Collectors.toList());
        
        if (!shortcuts.isEmpty()) {
            ShortcutInfoCompat updated = new ShortcutInfoCompat.Builder(context, shortcutId)
                .setShortLabel(newLabel)
                .setLongLabel("ดู: " + newLabel)
                .setIntent(shortcuts.get(0).getIntent())
                .setIcon(IconCompat.createWithResource(context, R.drawable.ic_product))
                .build();
            
            shortcutManager.updateShortcuts(Collections.singletonList(updated));
        }
    }
    
    // Disable shortcut (e.g., product out of stock)
    public void disableShortcut(String shortcutId, String message) {
        shortcutManager.disableShortcuts(
            Collections.singletonList(shortcutId), message);
    }
    
    // Re-enable shortcut
    public void enableShortcut(String shortcutId) {
        shortcutManager.enableShortcuts(Collections.singletonList(shortcutId));
    }
    
    // Clear all dynamic shortcuts
    public void clearAll() {
        shortcutManager.removeAllDynamicShortcuts();
    }
}
```

---

## 85.3 Handle Shortcut Intent in Activity

```java
// MainActivity.java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    
    // Handle shortcut intent
    if (getIntent() != null) {
        handleShortcutIntent(getIntent());
    }
}

@Override
protected void onNewIntent(Intent intent) {
    super.onNewIntent(intent);
    handleShortcutIntent(intent);
}

private void handleShortcutIntent(Intent intent) {
    String destination = intent.getStringExtra("destination");
    if (destination == null) return;
    
    // Report usage for ML-based suggestions
    if (intent.getStringExtra("shortcut_id") != null) {
        shortcutManager.reportShortcutUsed(intent.getStringExtra("shortcut_id"));
    }
    
    switch (destination) {
        case "cart":
            navController.navigate(R.id.cartFragment);
            break;
        case "product_detail":
            String productId = intent.getStringExtra("product_id");
            if (productId != null) {
                Bundle args = new Bundle();
                args.putString("productId", productId);
                navController.navigate(R.id.productDetailFragment, args);
            }
            break;
    }
}
```

---

## 85.4 สรุป Part 85

ในบทนี้คุณได้เรียนรู้:

✅ Static shortcuts (XML + AndroidManifest)  
✅ Multiple intents for back stack  
✅ Dynamic shortcuts with ShortcutManagerCompat  
✅ pushDynamicShortcut, removeDynamicShortcuts  
✅ reportShortcutUsed (ML ranking)  
✅ Request to pin shortcut on home screen  
✅ Update, disable, enable shortcuts  
✅ Handle shortcut intent in Activity  

---

*[← Part 84: Java Streams](./part-84-java-streams-collectors.md) | [Part 86: Jetpack Compose Interop →](./part-86-compose-interop.md)*
