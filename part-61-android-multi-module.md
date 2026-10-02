# Part 61: Multi-Module Android Architecture
## หลักสูตร Java & Android Development - ระดับ World-Class Android

---

## 61.1 ทำไมต้อง Multi-Module

```
ข้อดีของ Multi-Module:
✅ Build speed: Gradle compile เฉพาะ module ที่เปลี่ยน
✅ Reusability: ใช้ module ร่วมกันระหว่างโปรเจกต์
✅ Clear boundaries: แต่ละ module มีหน้าที่ชัดเจน
✅ Independent development: ทีมทำงาน module แยกกัน
✅ Feature flags: เปิด/ปิด feature module ได้

โครงสร้างแนะนำ:
:app                      ← Main app module
:core:common              ← Utils, extensions, base classes
:core:network             ← Retrofit, OkHttp, API clients
:core:database            ← Room, DAOs, entities
:core:ui                  ← Shared views, themes, styles
:feature:home             ← Home screen feature
:feature:profile          ← Profile feature
:feature:settings         ← Settings feature
:feature:auth             ← Authentication feature
```

---

## 61.2 Project Structure Setup

```groovy
// settings.gradle
include ':app'
include ':core:common'
include ':core:network'
include ':core:database'
include ':core:ui'
include ':feature:home'
include ':feature:auth'
include ':feature:profile'
include ':feature:settings'
```

```groovy
// build.gradle (root project)
buildscript {
    ext {
        // Shared versions
        kotlinVersion  = '1.9.0'
        composeVersion = '2024.02.00'
        roomVersion    = '2.6.1'
        retrofitVersion= '2.9.0'
    }
}

// buildSrc/build.gradle - for shared build logic
plugins {
    id 'groovy-gradle-plugin'
}
```

---

## 61.3 core:common Module

```groovy
// core/common/build.gradle
plugins {
    id 'com.android.library'
}

android {
    namespace 'com.myapp.core.common'
    compileSdk 34
    
    defaultConfig {
        minSdk 26
        targetSdk 34
    }
}

dependencies {
    implementation 'androidx.core:core-ktx:1.12.0'
    implementation 'androidx.lifecycle:lifecycle-livedata-ktx:2.7.0'
}
```

```java
// core/common/src/main/java/.../util/Resource.java
// Shared Result wrapper used by all modules
public class Resource<T> {
    public enum Status { LOADING, SUCCESS, ERROR }
    
    public final Status status;
    public final T data;
    public final String message;
    
    private Resource(Status status, T data, String message) {
        this.status = status;
        this.data = data;
        this.message = message;
    }
    
    public static <T> Resource<T> loading() {
        return new Resource<>(Status.LOADING, null, null);
    }
    
    public static <T> Resource<T> success(T data) {
        return new Resource<>(Status.SUCCESS, data, null);
    }
    
    public static <T> Resource<T> error(String message) {
        return new Resource<>(Status.ERROR, null, message);
    }
}

// core/common/src/main/java/.../util/ViewUtils.java
public class ViewUtils {
    public static void setVisible(View view, boolean visible) {
        view.setVisibility(visible ? View.VISIBLE : View.GONE);
    }
    
    public static void showLoading(ProgressBar pb, View content) {
        pb.setVisibility(View.VISIBLE);
        content.setVisibility(View.GONE);
    }
    
    public static void showContent(ProgressBar pb, View content) {
        pb.setVisibility(View.GONE);
        content.setVisibility(View.VISIBLE);
    }
}
```

---

## 61.4 core:network Module

```groovy
// core/network/build.gradle
plugins {
    id 'com.android.library'
}

android {
    namespace 'com.myapp.core.network'
}

dependencies {
    api 'com.squareup.retrofit2:retrofit:2.9.0'
    api 'com.squareup.retrofit2:converter-gson:2.9.0'
    api 'com.squareup.okhttp3:okhttp:4.12.0'
    api 'com.squareup.okhttp3:logging-interceptor:4.12.0'
    implementation project(':core:common')
}
```

```java
// Provide Retrofit via Hilt, usable by all feature modules
@Module
@InstallIn(SingletonComponent.class)
public class NetworkModule {
    
    @Provides
    @Singleton
    public OkHttpClient provideOkHttpClient() {
        return new OkHttpClient.Builder()
            .addInterceptor(new HttpLoggingInterceptor()
                .setLevel(HttpLoggingInterceptor.Level.BODY))
            .connectTimeout(30, TimeUnit.SECONDS)
            .readTimeout(30, TimeUnit.SECONDS)
            .build();
    }
    
    @Provides
    @Singleton
    public Retrofit provideRetrofit(OkHttpClient client) {
        return new Retrofit.Builder()
            .baseUrl("https://api.example.com/")
            .client(client)
            .addConverterFactory(GsonConverterFactory.create())
            .build();
    }
}
```

---

## 61.5 feature:auth Module

```groovy
// feature/auth/build.gradle
plugins {
    id 'com.android.library'
    id 'com.google.dagger.hilt.android'
}

android {
    namespace 'com.myapp.feature.auth'
}

dependencies {
    implementation project(':core:common')
    implementation project(':core:network')
    implementation project(':core:database')
    
    implementation 'com.google.dagger:hilt-android:2.48'
    annotationProcessor 'com.google.dagger:hilt-android-compiler:2.48'
}
```

```java
// feature/auth/src/main/.../AuthActivity.java
@AndroidEntryPoint
public class AuthActivity extends AppCompatActivity {
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_auth);
        // Only auth flow lives here
    }
}
```

---

## 61.6 Navigation Between Modules

```java
// Cross-module navigation via deep links or contracts

// Define navigation contract in core:common
public interface NavigationContract {
    // Deep link URIs
    String AUTH_SCREEN   = "myapp://auth/login";
    String HOME_SCREEN   = "myapp://home";
    String PROFILE_SCREEN = "myapp://profile/{userId}";
    
    // Navigate via Intent
    static void toAuth(Context context) {
        context.startActivity(new Intent(Intent.ACTION_VIEW, 
            Uri.parse(AUTH_SCREEN)));
    }
    
    static void toProfile(Context context, int userId) {
        context.startActivity(new Intent(Intent.ACTION_VIEW,
            Uri.parse("myapp://profile/" + userId)));
    }
}

// AndroidManifest.xml in feature:auth
<activity android:name=".AuthActivity">
    <intent-filter>
        <action android:name="android.intent.action.VIEW"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <category android:name="android.intent.category.BROWSABLE"/>
        <data android:scheme="myapp" android:host="auth"/>
    </intent-filter>
</activity>
```

---

## 61.7 Gradle Version Catalog

```toml
# gradle/libs.versions.toml
[versions]
agp       = "8.2.0"
java      = "17"
core-ktx  = "1.12.0"
lifecycle = "2.7.0"
room      = "2.6.1"
retrofit  = "2.9.0"
hilt      = "2.48"

[libraries]
androidx-core       = { module = "androidx.core:core-ktx",                 version.ref = "core-ktx"  }
lifecycle-viewmodel = { module = "androidx.lifecycle:lifecycle-viewmodel", version.ref = "lifecycle" }
room-runtime        = { module = "androidx.room:room-runtime",             version.ref = "room"      }
room-compiler       = { module = "androidx.room:room-compiler",            version.ref = "room"      }
retrofit            = { module = "com.squareup.retrofit2:retrofit",        version.ref = "retrofit"  }
hilt-android        = { module = "com.google.dagger:hilt-android",        version.ref = "hilt"      }

[plugins]
android-application = { id = "com.android.application", version.ref = "agp" }
android-library     = { id = "com.android.library",     version.ref = "agp" }
hilt                = { id = "com.google.dagger.hilt.android", version.ref = "hilt" }
```

```groovy
// Use in build.gradle
dependencies {
    implementation libs.androidx.core
    implementation libs.room.runtime
    annotationProcessor libs.room.compiler
    implementation libs.hilt.android
}
```

---

## 61.8 สรุป Part 61

ในบทนี้คุณได้เรียนรู้:

✅ เหตุผลที่ต้องใช้ Multi-Module  
✅ โครงสร้าง module (core/feature)  
✅ core:common (shared utilities)  
✅ core:network (Retrofit module)  
✅ feature module (auth, home, profile)  
✅ Cross-module navigation (deep links)  
✅ Gradle Version Catalog (libs.versions.toml)  

---

*[← Part 60: Advanced Room](./part-60-android-advanced-room.md) | [Part 62: App Widgets →](./part-62-android-widgets.md)*
