# Part 98: Production Checklist & Best Practices
## หลักสูตร Java & Android Development - ระดับ World-Class

---

## 98.1 Code Quality Checklist

```
✅ CODE QUALITY
□ ไม่มี TODO / FIXME ที่ยังค้างอยู่ใน production code
□ ไม่มี hardcoded strings (ใช้ strings.xml ทั้งหมด)
□ ไม่มี hardcoded magic numbers (ใช้ constants)
□ ไม่มี commented-out code ที่ไม่ได้ใช้
□ ทุก public method มี meaningful name
□ ทุก class ทำสิ่งเดียว (Single Responsibility)
□ ไม่มี method ที่ยาวเกิน 50 บรรทัด
□ ไม่มี class ที่ยาวเกิน 300 บรรทัด

✅ ANDROID-SPECIFIC
□ ไม่มี static reference ไปยัง Activity/Fragment/View
□ ทุก listener มีการ unregister ใน onDestroy/onStop
□ ทุก Cursor ถูก close ใน finally หรือ try-with-resources
□ ทุก Handler ใช้ WeakReference หรือ static inner class
□ ทุก background thread ถูก cancel ใน onDestroy
□ ViewBinding reference เป็น null ใน onDestroyView
□ ไม่มี network call ใน main thread
□ ไม่มี disk I/O ใน main thread
□ RecyclerView.ViewHolder ไม่เก็บ context reference แบบ direct
□ Bitmap ถูก recycle หรือปล่อยให้ Glide จัดการ

✅ PERFORMANCE
□ RecyclerView ใช้ setHasFixedSize(true) เมื่อขนาดไม่เปลี่ยน
□ RecyclerView ใช้ DiffUtil แทน notifyDataSetChanged()
□ Layout depth ไม่เกิน 5 ชั้น (ใช้ ConstraintLayout)
□ ไม่มีการสร้าง object ใน onDraw() หรือ onBindViewHolder()
□ Image ถูก resize ก่อน load (ไม่ load รูป 4K สำหรับ thumbnail)
□ Database query ไม่ทำบน main thread
□ ใช้ LruCache สำหรับ in-memory cache
□ ใช้ StrictMode ใน debug build เพื่อตรวจสอบ

✅ SECURITY
□ API keys ไม่อยู่ใน source code (ใช้ BuildConfig หรือ environment)
□ Token เก็บใน EncryptedSharedPreferences หรือ Keystore
□ Certificate pinning เปิดใช้งานสำหรับ production API
□ cleartext traffic disabled
□ ProGuard/R8 เปิดใช้งานใน release build
□ Export ของ Activity/Service/BroadcastReceiver ที่ไม่จำเป็น = false
□ WebView ปิด JavaScript ถ้าไม่ได้ใช้
□ Input validation ทุกจุดที่รับ user input

✅ TESTING
□ Unit tests coverage ≥ 70% สำหรับ business logic
□ Integration tests สำหรับ Room DAO
□ Instrumented tests สำหรับ UI ที่สำคัญ
□ Test migration (MigrationTestHelper)
□ ทุก ViewModel test มี InstantTaskExecutorRule
□ Mock dependencies ด้วย Mockito

✅ ACCESSIBILITY
□ ทุก ImageButton/ImageView มี contentDescription
□ Touch target ≥ 48dp x 48dp
□ Color contrast ratio ≥ 4.5:1
□ ทดสอบด้วย TalkBack
□ ทุก form field มี label ที่ชัดเจน
□ Error messages อ่านออกเสียงได้

✅ LOCALIZATION
□ ทุก string อยู่ใน strings.xml
□ ไม่มี string concatenation ที่อาจผิด grammar ในภาษาอื่น
□ ใช้ plurals สำหรับ "1 item", "2 items"
□ ทดสอบกับ RTL layout (Arabic, Hebrew)
□ Date/Number format ใช้ locale-aware

✅ RELEASE
□ versionCode เพิ่มขึ้น
□ versionName อัปเดต
□ release keystore สำรองไว้ในที่ปลอดภัย
□ ProGuard mapping file เก็บไว้ (สำหรับ crash deobfuscation)
□ Firebase Crashlytics เปิดใช้งาน
□ Firebase Analytics เปิดใช้งาน
□ ทดสอบบน physical device ก่อน release
□ ทดสอบบน Android version ต่ำสุดที่ support
□ ทดสอบ fresh install + upgrade install
```

---

## 98.2 Performance Benchmarks

```java
// Macrobenchmark - measure app startup
// build.gradle (:macrobenchmark)
// plugins { id 'androidx.benchmark.macro' }

@RunWith(AndroidJUnit4::class)
class StartupBenchmark {
    
    @get:Rule
    val benchmarkRule = MacrobenchmarkRule()
    
    @Test
    fun coldStartup() = benchmarkRule.measureRepeated(
        packageName = "com.myapp.shopapp",
        metrics = listOf(StartupTimingMetric()),
        iterations = 5,
        startupMode = StartupMode.COLD
    ) {
        pressHome()
        startActivityAndWait()
    }
    
    @Test
    fun warmStartup() = benchmarkRule.measureRepeated(
        packageName = "com.myapp.shopapp",
        metrics = listOf(StartupTimingMetric()),
        iterations = 10,
        startupMode = StartupMode.WARM
    ) {
        startActivityAndWait()
    }
}

// App startup optimization
// 1. Use App Startup library to lazy-init components
// 2. Defer non-critical initialization to background thread
// 3. Use SplashScreen API (Android 12+)
// 4. Preload critical data in Application.onCreate on background thread

public class ShopApplication extends Application {
    
    @Override
    public void onCreate() {
        super.onCreate();
        
        // Init analytics synchronously (needed immediately)
        FirebaseApp.initializeApp(this);
        
        // Init non-critical libs asynchronously
        AppThreadPool.IO_EXECUTOR.execute(() -> {
            Timber.plant(new Timber.DebugTree());
            LeakCanary.install(this);
            // ... other non-critical init
        });
    }
}
```

---

## 98.3 App Size Optimization

```groovy
// build.gradle
android {
    buildTypes {
        release {
            minifyEnabled true
            shrinkResources true  // remove unused resources
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'),
                'proguard-rules.pro'
        }
    }
    
    // Split APKs by ABI (architecture)
    splits {
        abi {
            enable true
            reset()
            include 'armeabi-v7a', 'arm64-v8a', 'x86', 'x86_64'
            universalApk false  // don't include universal APK
        }
    }
    
    // WebP conversion (smaller than PNG)
    // Android Studio: right-click res folder → Convert to WebP
    
    // Limit language resources
    defaultConfig {
        resConfigs "en", "th"  // only include Thai and English strings
    }
}
```

```bash
# Analyze APK size
./gradlew assembleRelease
# Open Android Studio → Build → Analyze APK

# Check what's taking space:
# - resources/ (images, layouts)
# - classes.dex (code)
# - lib/ (native .so files)
# - assets/

# Tools:
# - Android Size Analyzer plugin
# - bundletool: java -jar bundletool.jar get-size total --bundle=app.aab
```

---

## 98.4 สรุป Part 98

ในบทนี้คุณได้เรียนรู้:

✅ Code quality checklist (static references, listeners, cursor)  
✅ Android performance checklist  
✅ Security checklist (tokens, pinning, ProGuard)  
✅ Testing checklist (coverage, migrations)  
✅ Accessibility checklist  
✅ Release checklist  
✅ Macrobenchmark for startup time  
✅ App size optimization (splits, shrinkResources, resConfigs)  

---

*[← Part 97: Kotlin Coroutines Interop](./part-97-kotlin-coroutines-interop.md) | [Part 99: Career Path & Next Steps →](./part-99-career-path.md)*
