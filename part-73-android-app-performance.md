# Part 73: Android App Performance (ขั้นสูง)
## หลักสูตร Java & Android Development - ระดับ World-Class Android

---

## 73.1 Rendering Performance

```
16ms budget per frame (60fps):
- Overdraw: วาด pixel ซ้ำหลายชั้นโดยไม่จำเป็น
- Layout pass: ยิ่ง layout ลึก ยิ่งช้า
- Measure/Draw: ลด custom views ที่ซับซ้อน

Tools:
- GPU Overdraw (Developer Options → Debug GPU overdraw)
- Layout Inspector (Android Studio)
- Systrace / Perfetto
- Method Profiler (Android Studio Profiler)
```

---

## 73.2 Layout Optimization

```xml
<!-- BAD: Deeply nested LinearLayouts -->
<LinearLayout>
  <LinearLayout>
    <LinearLayout>
      <TextView/>
      <TextView/>
    </LinearLayout>
  </LinearLayout>
</LinearLayout>

<!-- GOOD: Flat ConstraintLayout -->
<androidx.constraintlayout.widget.ConstraintLayout>
    <TextView
        android:id="@+id/tvTitle"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintStart_toStartOf="parent" />
    <TextView
        android:id="@+id/tvSubtitle"
        app:layout_constraintTop_toBottomOf="@id/tvTitle"
        app:layout_constraintStart_toStartOf="@id/tvTitle" />
</androidx.constraintlayout.widget.ConstraintLayout>

<!-- ViewStub: inflate only when needed -->
<ViewStub
    android:id="@+id/stubEmptyState"
    android:layout="@layout/layout_empty_state"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

```java
// Inflate ViewStub only when needed
ViewStub stub = findViewById(R.id.stubEmptyState);
if (isEmpty) {
    stub.inflate();  // inflates and replaces stub
}

// merge tag: reduces hierarchy by 1 level
// When included via <include>, merge's children go directly to parent
// In layout_empty_state.xml:
// <merge xmlns:android="...">
//   <TextView .../>
//   <Button .../>
// </merge>
```

---

## 73.3 Bitmap Memory Management

```java
// BitmapUtils.java
public class BitmapUtils {
    
    // Decode with sample size to avoid OOM
    public static Bitmap decodeSampledBitmap(String filePath, int reqWidth, int reqHeight) {
        BitmapFactory.Options options = new BitmapFactory.Options();
        options.inJustDecodeBounds = true;  // get size without allocating
        BitmapFactory.decodeFile(filePath, options);
        
        options.inSampleSize = calculateSampleSize(options, reqWidth, reqHeight);
        options.inJustDecodeBounds = false;
        options.inPreferredConfig = Bitmap.Config.RGB_565;  // 2 bytes/px (vs ARGB_8888 = 4)
        
        return BitmapFactory.decodeFile(filePath, options);
    }
    
    private static int calculateSampleSize(BitmapFactory.Options options,
            int reqWidth, int reqHeight) {
        int height = options.outHeight;
        int width  = options.outWidth;
        int inSampleSize = 1;
        
        while (height / inSampleSize > reqHeight || width / inSampleSize > reqWidth) {
            inSampleSize *= 2;
        }
        return inSampleSize;
    }
    
    // Recycle bitmaps explicitly
    public static void safeRecycle(Bitmap bitmap) {
        if (bitmap != null && !bitmap.isRecycled()) bitmap.recycle();
    }
    
    // Use LruCache for caching decoded bitmaps
    private static LruCache<String, Bitmap> bitmapCache;
    
    public static void initCache() {
        long maxMemory = Runtime.getRuntime().maxMemory();
        int cacheSize = (int) (maxMemory / 8);  // use 1/8 of available memory
        
        bitmapCache = new LruCache<String, Bitmap>(cacheSize) {
            @Override
            protected int sizeOf(String key, Bitmap bitmap) {
                return bitmap.getByteCount();
            }
        };
    }
    
    public static void cacheImage(String key, Bitmap bitmap) {
        if (bitmapCache.get(key) == null) bitmapCache.put(key, bitmap);
    }
    
    public static Bitmap getCachedImage(String key) {
        return bitmapCache.get(key);
    }
}
```

---

## 73.4 Memory Leaks Prevention

```java
// WeakReference for context
public class ImageLoader {
    private final WeakReference<Context> contextRef;
    
    public ImageLoader(Context context) {
        this.contextRef = new WeakReference<>(context);
    }
    
    public void loadImage(String url, ImageView imageView) {
        Context context = contextRef.get();
        if (context == null) return;  // Activity destroyed
        
        Glide.with(context)
            .load(url)
            .into(new WeakReference<>(imageView).get());
    }
}

// Avoid static references to Activity/Fragment/View
// BAD:
public class BadActivity extends Activity {
    static BadActivity instance;  // MEMORY LEAK!
    
    @Override
    public void onCreate(Bundle state) {
        instance = this;  // prevents GC
    }
}

// GOOD: Use Application context when Activity context not needed
Glide.with(context.getApplicationContext())...

// Handler leak fix
public class SafeHandler extends Handler {
    private final WeakReference<Activity> actRef;
    
    SafeHandler(Activity activity) {
        super(Looper.getMainLooper());
        this.actRef = new WeakReference<>(activity);
    }
    
    @Override
    public void handleMessage(Message msg) {
        Activity activity = actRef.get();
        if (activity != null && !activity.isFinishing()) {
            // handle
        }
    }
}

// In Activity.onDestroy: remove all callbacks
@Override
protected void onDestroy() {
    super.onDestroy();
    handler.removeCallbacksAndMessages(null);
    disposables.clear();
}
```

---

## 73.5 Profiler Usage Guide

```java
// Add trace markers to profile specific code
@Override
protected void onCreate(Bundle state) {
    Trace.beginSection("MainActivity.onCreate");
    super.onCreate(state);
    
    Trace.beginSection("setupUI");
    setupUI();
    Trace.endSection();
    
    Trace.beginSection("loadData");
    loadInitialData();
    Trace.endSection();
    
    Trace.endSection();  // MainActivity.onCreate
}

// Detect ANR: never do this on main thread
// BAD - blocks main thread for 2 seconds → ANR!
mainThread.sleep(2000);
networkCall();
databaseQuery();

// GOOD - move to background
AppThreadPool.IO_EXECUTOR.execute(() -> {
    List<User> users = networkCall();
    runOnUiThread(() -> updateUI(users));
});

// StrictMode for development: catch policy violations
if (BuildConfig.DEBUG) {
    StrictMode.setThreadPolicy(new StrictMode.ThreadPolicy.Builder()
        .detectAll()
        .penaltyLog()
        .build());
    
    StrictMode.setVmPolicy(new StrictMode.VmPolicy.Builder()
        .detectLeakedSqlLiteObjects()
        .detectLeakedClosableObjects()
        .detectActivityLeaks()
        .penaltyLog()
        .build());
}
```

---

## 73.6 สรุป Part 73

ในบทนี้คุณได้เรียนรู้:

✅ 16ms frame budget  
✅ Overdraw detection  
✅ Layout optimization (flat hierarchy, ViewStub, merge)  
✅ Bitmap sampling to prevent OOM  
✅ LruCache for bitmap caching  
✅ Memory leak prevention (WeakReference, static)  
✅ Handler leak fix  
✅ Trace markers for profiling  
✅ StrictMode for development  

---

*[← Part 72: Design Patterns](./part-72-java-design-patterns-advanced.md) | [Part 74: Advanced Testing →](./part-74-android-advanced-testing.md)*
