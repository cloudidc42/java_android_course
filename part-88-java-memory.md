# Part 88: Java Memory Management & JVM Internals
## หลักสูตร Java & Android Development - ระดับ World-Class

---

## 88.1 JVM Memory Structure

```
JVM Memory (Android = Dalvik/ART):
┌─────────────────────────────────────────────────────────────┐
│  HEAP                                                       │
│  ┌───────────────────┐  ┌──────────────────────────────┐  │
│  │  Young Generation  │  │  Old Generation (Tenured)    │  │
│  │  ┌────┐ ┌────┐    │  │                              │  │
│  │  │Eden│ │S0/S1│   │  │  Long-lived objects          │  │
│  │  └────┘ └────┘    │  │  Large objects               │  │
│  └───────────────────┘  └──────────────────────────────┘  │
├─────────────────────────────────────────────────────────────┤
│  NON-HEAP                                                   │
│  Metaspace: Class metadata, static fields, method bytecode │
│  Stack:     Each thread has its own stack (local vars)     │
│  Code Cache: JIT-compiled native code                      │
└─────────────────────────────────────────────────────────────┘

GC Events:
- Minor GC: Young generation → fast, frequent
- Major GC: Old generation → slower, less frequent
- Full GC:  All → pause app (avoid!)
```

---

## 88.2 Memory Leak Patterns & Fixes

```java
// ❌ WRONG: Static reference to Activity (leaks Activity)
public class ImageLoader {
    private static Activity activity;  // holds Activity forever!
    
    public static void setActivity(Activity a) { activity = a; }
    
    public static void load(String url) {
        Glide.with(activity).load(url).into(imageView);  // leaked!
    }
}

// ✅ CORRECT: Use ApplicationContext or WeakReference
public class ImageLoader {
    private final WeakReference<Activity> activityRef;
    
    public ImageLoader(Activity activity) {
        activityRef = new WeakReference<>(activity);
    }
    
    public void load(String url, ImageView imageView) {
        Activity activity = activityRef.get();
        if (activity == null || activity.isDestroyed()) return;
        Glide.with(activity).load(url).into(imageView);
    }
}

// ❌ WRONG: Non-static inner class (holds reference to outer class)
public class MyActivity extends Activity {
    private Handler handler = new Handler() {  // non-static = holds Activity!
        @Override
        public void handleMessage(Message msg) {
            updateUI(msg.obj.toString());
        }
    };
}

// ✅ CORRECT: Static inner class + WeakReference
public class MyActivity extends Activity {
    
    private final Handler handler = new SafeHandler(this);
    
    static class SafeHandler extends Handler {
        private final WeakReference<MyActivity> activityRef;
        
        SafeHandler(MyActivity activity) {
            super(Looper.getMainLooper());
            this.activityRef = new WeakReference<>(activity);
        }
        
        @Override
        public void handleMessage(@NonNull Message msg) {
            MyActivity activity = activityRef.get();
            if (activity != null && !activity.isDestroyed()) {
                activity.updateUI(msg.obj.toString());
            }
        }
    }
    
    @Override
    protected void onDestroy() {
        super.onDestroy();
        handler.removeCallbacksAndMessages(null);  // cancel pending messages
    }
}

// ❌ WRONG: Register listener but never unregister
public class EventFragment extends Fragment {
    @Override
    public void onViewCreated(View view, Bundle savedInstanceState) {
        EventBus.getInstance().register(this);  // registers but...
    }
    // ...never unregisters → Fragment held alive by EventBus!
}

// ✅ CORRECT: Always pair register/unregister with lifecycle
public class EventFragment extends Fragment {
    @Override
    public void onStart() {
        super.onStart();
        EventBus.getInstance().register(this);
    }
    
    @Override
    public void onStop() {
        super.onStop();
        EventBus.getInstance().unregister(this);
    }
}

// ❌ WRONG: Cursor not closed
Cursor cursor = db.query(...);
// process cursor
// forgot to close!

// ✅ CORRECT: Always close in finally or try-with-resources
try (Cursor cursor = db.query(...)) {
    if (cursor.moveToFirst()) {
        // process
    }
}  // auto-closed
```

---

## 88.3 Garbage Collection Tuning

```java
// Force GC hint (not guaranteed, just a hint)
System.gc();  // avoid calling this in production

// Understand object eligibility:
// 1. Unreachable objects → collected
// 2. Objects reachable only via WeakReference → collected on next GC
// 3. Objects reachable via SoftReference → collected when memory pressure
// 4. Objects reachable via PhantomReference → collected after finalization

// WeakReference use case: caches
public class PhotoCache {
    private final Map<String, WeakReference<Bitmap>> cache = new HashMap<>();
    
    public void put(String key, Bitmap bitmap) {
        cache.put(key, new WeakReference<>(bitmap));
    }
    
    public Bitmap get(String key) {
        WeakReference<Bitmap> ref = cache.get(key);
        if (ref == null) return null;
        Bitmap bitmap = ref.get();
        if (bitmap == null) {
            cache.remove(key);  // clean up dead reference
        }
        return bitmap;
    }
}

// SoftReference: GC reclaims ONLY under memory pressure
// Good for memory-sensitive caches
public class SoftCache<K, V> {
    private final Map<K, SoftReference<V>> map = new LinkedHashMap<>();
    
    public void put(K key, V value) {
        map.put(key, new SoftReference<>(value));
    }
    
    public V get(K key) {
        SoftReference<V> ref = map.get(key);
        return ref != null ? ref.get() : null;
    }
}
```

---

## 88.4 Android Memory Profiler Checklist

```
1. Heap Dump Analysis:
   - Open Android Studio → Profiler → Memory
   - Click "Dump Java Heap"
   - Look for: Bitmap objects taking huge space
   - Look for: Activity instances > 1 (leak!)
   - Filter by: Retained size descending

2. Allocation Tracking:
   - Record allocations during a UI action
   - Look for: repeated large allocations in hot paths
   - Avoid: new objects inside onDraw() or RecyclerView.onBindViewHolder()

3. LeakCanary (library):
   - Automatic memory leak detection in debug builds
   - Notifies when Activity/Fragment leaks detected

// LeakCanary setup (debugImplementation only)
debugImplementation 'com.squareup.leakcanary:leakcanary-android:2.14'

4. Android-specific rules:
   - Bitmap: always recycle() when done (or let Glide manage it)
   - Drawable: don't store Context-bound drawables as static
   - View: don't store in static fields
   - Thread: always stop background threads in onDestroy/onStop
```

---

## 88.5 สรุป Part 88

ในบทนี้คุณได้เรียนรู้:

✅ JVM Heap structure (Young/Old Generation)  
✅ Memory leak: static Activity reference  
✅ Fix: WeakReference + SafeHandler pattern  
✅ Fix: always unregister listeners  
✅ Fix: try-with-resources for Cursor  
✅ WeakReference vs SoftReference vs PhantomReference  
✅ SoftCache pattern  
✅ Memory Profiler checklist  
✅ LeakCanary setup  

---

*[← Part 87: Advanced Networking](./part-87-advanced-networking.md) | [Part 89: CI/CD for Android →](./part-89-android-cicd.md)*
