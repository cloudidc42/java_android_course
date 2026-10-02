# Part 79: Gradle Build System ขั้นสูง
## หลักสูตร Java & Android Development - ระดับ World-Class

---

## 79.1 Build Variants

```groovy
// build.gradle (app)
android {
    
    // Product flavors: different versions of the app
    flavorDimensions "environment", "audience"
    
    productFlavors {
        dev {
            dimension "environment"
            applicationIdSuffix ".dev"
            versionNameSuffix "-DEV"
            buildConfigField "String", "API_BASE_URL", '"https://dev-api.example.com/"'
            buildConfigField "boolean", "ENABLE_LOGGING", "true"
        }
        
        staging {
            dimension "environment"
            applicationIdSuffix ".staging"
            versionNameSuffix "-STAGING"
            buildConfigField "String", "API_BASE_URL", '"https://staging-api.example.com/"'
            buildConfigField "boolean", "ENABLE_LOGGING", "true"
        }
        
        prod {
            dimension "environment"
            buildConfigField "String", "API_BASE_URL", '"https://api.example.com/"'
            buildConfigField "boolean", "ENABLE_LOGGING", "false"
        }
        
        free {
            dimension "audience"
            applicationIdSuffix ".free"
            buildConfigField "boolean", "IS_PREMIUM", "false"
        }
        
        premium {
            dimension "audience"
            buildConfigField "boolean", "IS_PREMIUM", "true"
        }
    }
    
    // Build types
    buildTypes {
        debug {
            minifyEnabled false
            debuggable true
            applicationIdSuffix ".debug"
            signingConfig signingConfigs.debug
        }
        
        release {
            minifyEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'),
                'proguard-rules.pro'
            signingConfig signingConfigs.release
        }
    }
}
```

```java
// Use BuildConfig in code
String apiUrl = BuildConfig.API_BASE_URL;
boolean isPremium = BuildConfig.IS_PREMIUM;

if (BuildConfig.ENABLE_LOGGING) {
    Timber.plant(new Timber.DebugTree());
}
```

---

## 79.2 BuildConfig Fields & ResValues

```groovy
// Custom BuildConfig fields
android {
    defaultConfig {
        buildConfigField "int",    "MAX_RETRY_COUNT", "3"
        buildConfigField "long",   "CACHE_TTL_MS",    "300000L"
        buildConfigField "String", "APP_VERSION",     '"1.0.0"'
        
        // String resources (accessible via R.string)
        resValue "string", "app_display_name", "My App"
        resValue "bool",   "is_debug_build",   "false"
    }
}
```

---

## 79.3 Custom Gradle Tasks

```groovy
// Task: print all dependencies
task printDependencies {
    doLast {
        configurations.releaseRuntimeClasspath.resolvedConfiguration
            .resolvedArtifacts.each { artifact ->
                println "${artifact.moduleVersion.id.module}: ${artifact.moduleVersion.id.version}"
            }
    }
}

// Task: generate version from git
def getGitVersion = {
    def tag = ""
    def count = 0
    try {
        tag = 'git describe --tags --abbrev=0'.execute().text.trim()
        count = 'git rev-list --count HEAD'.execute().text.trim().toInteger()
    } catch (e) {
        tag = "0.1.0"
    }
    return tag
}

// Task: copy APK to specific folder
task copyApk(type: Copy, dependsOn: assembleRelease) {
    from("${buildDir}/outputs/apk/release/")
    into("${rootDir}/release/")
    include("*.apk")
    rename { filename -> "app-release-${getGitVersion()}.apk" }
}

// Task: validate release build
task validateRelease {
    doLast {
        File proguardFile = file('proguard-rules.pro')
        assert proguardFile.exists(), "ProGuard rules file missing!"
        
        File keystoreFile = file('../keystore/release.jks')
        assert keystoreFile.exists(), "Keystore missing!"
        
        println "✓ Release build validation passed"
    }
}

assembleRelease.dependsOn validateRelease
```

---

## 79.4 Dependency Management

```groovy
// Exclude transitive dependencies
implementation("com.squareup.okhttp3:okhttp:4.12.0") {
    exclude group: "com.squareup.okio", module: "okio"
}

// Force specific version
configurations.all {
    resolutionStrategy {
        force 'com.google.guava:guava:32.1.3-android'
        
        // Fail if conflicting versions
        failOnVersionConflict()
        
        // Substitute dependency
        dependencySubstitution {
            substitute module("org.apache.commons:commons-lang")
                .using module("commons-lang:commons-lang:2.6")
        }
    }
}

// Check for dependency updates (plugin)
// plugins { id 'com.github.ben-manes.versions' version '0.50.0' }
// ./gradlew dependencyUpdates
```

---

## 79.5 ProGuard Rules

```pro
# proguard-rules.pro

# Keep model classes (Gson/Retrofit need them)
-keep class com.myapp.data.remote.dto.** { *; }
-keep class com.myapp.domain.model.** { *; }

# Keep Room entities
-keep class com.myapp.data.local.entity.** { *; }

# Keep Parcelable
-keep class * implements android.os.Parcelable { *; }

# Retrofit
-keepattributes Signature
-keepattributes Exceptions
-keep class retrofit2.** { *; }
-keepclassmembernames interface * {
    @retrofit2.http.* <methods>;
}

# Gson
-keep class com.google.gson.** { *; }
-keep class * implements com.google.gson.TypeAdapterFactory
-keep class * implements com.google.gson.JsonSerializer
-keep class * implements com.google.gson.JsonDeserializer

# Glide
-keep public class * implements com.bumptech.glide.module.GlideModule
-keep class * extends com.bumptech.glide.module.AppGlideModule {
 <init>(...);
}
-keep public enum com.bumptech.glide.load.ImageHeaderParser$** { *; }

# OkHttp
-dontwarn okhttp3.**
-dontwarn okio.**
-keep class okhttp3.** { *; }

# Hilt
-keep class dagger.hilt.** { *; }
-keep @dagger.hilt.android.HiltAndroidApp class * { *; }
-keep @dagger.hilt.android.AndroidEntryPoint class * { *; }

# Firebase
-keep class com.google.firebase.** { *; }

# Remove logging in release
-assumenosideeffects class android.util.Log {
    public static *** d(...);
    public static *** v(...);
    public static *** i(...);
}
```

---

## 79.6 สรุป Part 79

ในบทนี้คุณได้เรียนรู้:

✅ Product flavors (dev/staging/prod, free/premium)  
✅ BuildConfig fields  
✅ Custom Gradle tasks  
✅ Dependency conflict resolution  
✅ ProGuard rules (Retrofit, Gson, Hilt, Firebase)  
✅ Remove logging in release build  

---

*[← Part 78: Deep Links](./part-78-android-deep-links.md) | [Part 80: Android App Monetization →](./part-80-android-monetization.md)*
