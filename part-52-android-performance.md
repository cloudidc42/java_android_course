# Part 52: Android Performance Optimization
## หลักสูตร Java & Android Development - ระดับ Professional Android

---

## 52.1 ปัญหา Performance ที่พบบ่อย

```
1. Jank (frame drops < 60fps)  - heavy work on main thread
2. Memory leaks               - references holding Activities
3. Slow startup               - too much work in onCreate
4. Battery drain              - excessive background work
5. Large APK size             - unused resources, duplicate code
6. Slow images                - not resizing, not caching
7. Slow database queries      - no indexes, querying main thread
```

---

## 52.2 StrictMode - ตรวจจับปัญหา

```java
// MyApplication.java
@HiltAndroidApp
public class MyApplication extends Application {
    
    @Override
    public void onCreate() {
        super.onCreate();
        
        if (BuildConfig.DEBUG) {
            // Detect slow operations on main thread
            StrictMode.setThreadPolicy(new StrictMode.ThreadPolicy.Builder()
                .detectAll()          // disk read/write, network, slow calls
                .penaltyLog()         // log violation
                .penaltyFlashScreen() // flash screen on violation
                .build());
            
            // Detect memory leaks and Activity leaks
            StrictMode.setVmPolicy(new StrictMode.VmPolicy.Builder()
                .detectActivityLeaks()
                .detectLeakedClosableObjects()
                .detectLeakedSqlLiteObjects()
                .penaltyLog()
                .build());
        }
    }
}
```

---

## 52.3 Memory Leak Prevention

```java
// BAD: Activity leak via static reference
public class BadActivity extends AppCompatActivity {
    static BadActivity instance;  // NEVER do this!
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        instance = this;  // prevents GC!
    }
}

// BAD: Anonymous Runnable holding Activity ref
public class BadActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        
        new Thread(new Runnable() {
            @Override
            public void run() {
                // BAD: this Runnable holds a reference to BadActivity
                runOnUiThread(() -> updateUI());
            }
        }).start();
    }
}

// GOOD: WeakReference
public class GoodActivity extends AppCompatActivity {
    
    private WeakReference<GoodActivity> weakRef;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        weakRef = new WeakReference<>(this);
        
        new Thread(() -> {
            GoodActivity activity = weakRef.get();
            if (activity != null && !activity.isFinishing()) {
                activity.runOnUiThread(activity::updateUI);
            }
        }).start();
    }
    
    private void updateUI() { /* update UI */ }
}

// GOOD: Cancel async work on destroy
public class GoodAsyncActivity extends AppCompatActivity {
    
    private CompositeDisposable disposables = new CompositeDisposable();
    
    @Override
    protected void onDestroy() {
        super.onDestroy();
        disposables.clear();  // cancel all Rx subscriptions
    }
}
```

---

## 52.4 RecyclerView Performance

```java
// RecyclerView optimization
public class OptimizedAdapter extends RecyclerView.Adapter<OptimizedAdapter.ViewHolder> {
    
    public OptimizedAdapter() {
        // Enable stable IDs for better animations
        setHasStableIds(true);
    }
    
    @Override
    public long getItemId(int position) {
        return items.get(position).getId();
    }
    
    // ViewHolder pattern (already required, but best practices)
    public static class ViewHolder extends RecyclerView.ViewHolder {
        // Pre-fetch all views in constructor, not in bind
        final TextView tvTitle;
        final ImageView ivIcon;
        
        ViewHolder(View view) {
            super(view);
            tvTitle = view.findViewById(R.id.tvTitle);
            ivIcon  = view.findViewById(R.id.ivIcon);
        }
    }
    
    @Override
    public void onBindViewHolder(ViewHolder holder, int position) {
        Item item = items.get(position);
        holder.tvTitle.setText(item.getTitle());
        
        // Use Glide for image loading (async, cached)
        Glide.with(holder.itemView)
            .load(item.getImageUrl())
            .thumbnail(0.1f)    // show 10% size first
            .placeholder(R.drawable.placeholder)
            .into(holder.ivIcon);
    }
}

// Setup RecyclerView properly
void setupRecyclerView(RecyclerView rv) {
    rv.setHasFixedSize(true);     // items don't change size
    rv.setItemViewCacheSize(20);  // cache more items off screen
    rv.setDrawingCacheEnabled(true);
    rv.setDrawingCacheQuality(View.DRAWING_CACHE_QUALITY_HIGH);
    
    // Use DiffUtil instead of notifyDataSetChanged
    ListAdapter<Item, ViewHolder> adapter = new ListAdapter<Item, ViewHolder>(DIFF_CALLBACK) {
        @Override
        public void onBindViewHolder(ViewHolder holder, int position) {
            holder.bind(getItem(position));
        }
        @Override
        public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
            View view = LayoutInflater.from(parent.getContext())
                .inflate(R.layout.item_view, parent, false);
            return new ViewHolder(view);
        }
    };
    
    rv.setAdapter(adapter);
    adapter.submitList(dataList);  // DiffUtil runs on background thread
}

static final DiffUtil.ItemCallback<Item> DIFF_CALLBACK = new DiffUtil.ItemCallback<Item>() {
    @Override
    public boolean areItemsTheSame(Item oldItem, Item newItem) {
        return oldItem.getId() == newItem.getId();
    }
    
    @Override
    public boolean areContentsTheSame(Item oldItem, Item newItem) {
        return oldItem.equals(newItem);  // compare all fields
    }
};
```

---

## 52.5 App Startup Optimization

```java
// Use App Startup library for lazy initialization
// build.gradle: implementation "androidx.startup:startup-runtime:1.1.1"

// MyInitializer.java
public class MyInitializer implements Initializer<MyLibrary> {
    
    @Override
    public MyLibrary create(Context context) {
        // Initialize library lazily
        return MyLibrary.init(context);
    }
    
    @Override
    public List<Class<? extends Initializer<?>>> dependencies() {
        return Collections.emptyList();
    }
}

// AndroidManifest.xml
// <provider
//     android:name="androidx.startup.InitializationProvider"
//     android:authorities="${applicationId}.androidx-startup"
//     android:exported="false"
//     tools:node="merge">
//     <meta-data android:name="com.example.MyInitializer"
//         android:value="androidx.startup" />
// </provider>

// Baseline profile for faster startup (Android 7+)
// Create app/src/main/baseline-prof.txt
// Generated by: ./gradlew generateBaselineProfile

// Splash screen API (Android 12+)
// In themes.xml:
// <item name="android:windowSplashScreenBackground">@color/primary</item>
// <item name="android:windowSplashScreenAnimatedIcon">@drawable/ic_splash</item>
```

---

## 52.6 Background Work Optimization

```java
// BAD: Long work on main thread
button.setOnClickListener(v -> {
    // NEVER do heavy work on main thread!
    String result = heavyComputation();  // blocks UI!
    textView.setText(result);
});

// GOOD: Coroutines-style with ExecutorService
ExecutorService executor = Executors.newSingleThreadExecutor();
Handler mainHandler = new Handler(Looper.getMainLooper());

button.setOnClickListener(v -> {
    executor.execute(() -> {
        String result = heavyComputation();  // background thread
        mainHandler.post(() -> textView.setText(result));  // back to UI
    });
});

// GOOD: AsyncTask alternative - CompletableFuture
CompletableFuture.supplyAsync(() -> heavyComputation())
    .thenAcceptAsync(result -> textView.setText(result),
        ContextCompat.getMainExecutor(context));

// GOOD: For ViewModel
viewModel.getResult().observe(this, result -> {
    textView.setText(result);
});

// In ViewModel:
private MutableLiveData<String> result = new MutableLiveData<>();
void computeResult() {
    executor.execute(() -> {
        String computed = heavyComputation();
        result.postValue(computed);
    });
}
```

---

## 52.7 APK Size Reduction

```groovy
// build.gradle
android {
    buildTypes {
        release {
            minifyEnabled true       // shrink code (R8/ProGuard)
            shrinkResources true     // remove unused resources
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'),
                'proguard-rules.pro'
        }
    }
    
    // Remove unused density resources
    defaultConfig {
        resConfigs "th", "en"        // only Thai and English strings
    }
    
    // Use App Bundle instead of APK
    bundle {
        language { enableSplit = true }
        density { enableSplit = true }
        abi { enableSplit = true }
    }
}

// Split APK by ABI (reduces size per device)
splits {
    abi {
        enable true
        reset()
        include 'x86', 'x86_64', 'armeabi-v7a', 'arm64-v8a'
        universalApk false
    }
}
```

```
# proguard-rules.pro
# Keep model classes for Gson/Retrofit
-keep class com.example.app.model.** { *; }
-keep class com.example.app.api.** { *; }

# Keep Firebase
-keep class com.google.firebase.** { *; }

# Keep Parcelable
-keep class * implements android.os.Parcelable { *; }
```

---

## 52.8 Profiling Tools

```
Android Studio Profilers:
1. CPU Profiler    - Find slow methods (flame chart, method traces)
2. Memory Profiler - Find memory leaks, heap dumps
3. Network Profiler - Analyze API calls
4. Energy Profiler  - Battery usage

Usage:
- Run > Profile app
- Click CPU/Memory/Network/Energy panel
- Record trace, analyze

Key metrics to watch:
- Startup time < 1 second
- Frame render < 16ms (60fps)
- Memory < 50MB normal usage  
- No leaked Activities
```

---

## 52.9 สรุป Part 52

ในบทนี้คุณได้เรียนรู้:

✅ StrictMode สำหรับ Debug  
✅ Memory leak prevention (WeakReference)  
✅ RecyclerView optimization (DiffUtil, stable IDs)  
✅ App startup optimization  
✅ Background threading (ExecutorService, CompletableFuture)  
✅ APK size reduction (R8, App Bundle, resConfigs)  
✅ ProGuard/R8 rules  
✅ Android Studio Profilers  

---

*[← Part 51: Testing](./part-51-android-testing.md) | [Part 53: ProGuard and Security →](./part-53-android-security.md)*
