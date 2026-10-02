# Part 89: CI/CD for Android
## หลักสูตร Java & Android Development - ระดับ World-Class

---

## 89.1 GitHub Actions Workflow

```yaml
# .github/workflows/android.yml
name: Android CI/CD

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    name: Run Tests
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: 'gradle'
      
      - name: Grant execute permission for gradlew
        run: chmod +x gradlew
      
      - name: Cache Gradle packages
        uses: actions/cache@v4
        with:
          path: |
            ~/.gradle/caches
            ~/.gradle/wrapper
          key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}
          restore-keys: |
            ${{ runner.os }}-gradle-
      
      - name: Run lint
        run: ./gradlew lint
      
      - name: Run unit tests
        run: ./gradlew testDebugUnitTest
      
      - name: Upload test results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: test-results
          path: '**/build/reports/tests/'
      
      - name: Upload lint results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: lint-results
          path: '**/build/reports/lint-results*.html'

  build-debug:
    name: Build Debug APK
    runs-on: ubuntu-latest
    needs: test
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: 'gradle'
      
      - run: chmod +x gradlew
      
      - name: Build Debug APK
        run: ./gradlew assembleDebug
      
      - name: Upload Debug APK
        uses: actions/upload-artifact@v4
        with:
          name: debug-apk
          path: app/build/outputs/apk/debug/*.apk

  build-release:
    name: Build & Sign Release APK
    runs-on: ubuntu-latest
    needs: test
    if: github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: 'gradle'
      
      - run: chmod +x gradlew
      
      - name: Decode keystore
        run: |
          echo "${{ secrets.KEYSTORE_BASE64 }}" | base64 -d > release.jks
      
      - name: Build Release APK
        env:
          KEYSTORE_PATH: release.jks
          KEY_ALIAS: ${{ secrets.KEY_ALIAS }}
          KEY_PASSWORD: ${{ secrets.KEY_PASSWORD }}
          STORE_PASSWORD: ${{ secrets.STORE_PASSWORD }}
        run: ./gradlew assembleRelease
      
      - name: Upload Release APK
        uses: actions/upload-artifact@v4
        with:
          name: release-apk
          path: app/build/outputs/apk/release/*.apk

  deploy-to-firebase:
    name: Deploy to Firebase App Distribution
    runs-on: ubuntu-latest
    needs: build-debug
    if: github.ref == 'refs/heads/develop'
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Download debug APK
        uses: actions/download-artifact@v4
        with:
          name: debug-apk
          path: app/build/outputs/apk/debug/
      
      - name: Upload to Firebase App Distribution
        uses: wzieba/Firebase-Distribution-Github-Action@v1
        with:
          appId: ${{ secrets.FIREBASE_APP_ID }}
          serviceCredentialsFileContent: ${{ secrets.FIREBASE_CREDENTIALS }}
          groups: testers
          file: app/build/outputs/apk/debug/app-debug.apk
          releaseNotesFile: CHANGELOG.md
```

---

## 89.2 Signing Config in build.gradle

```groovy
// build.gradle (app) - read from environment variables or local.properties
android {
    signingConfigs {
        release {
            // From environment (CI)
            storeFile file(System.getenv("KEYSTORE_PATH") ?: "release.jks")
            storePassword System.getenv("STORE_PASSWORD") ?: keystoreProperties['storePassword']
            keyAlias System.getenv("KEY_ALIAS") ?: keystoreProperties['keyAlias']
            keyPassword System.getenv("KEY_PASSWORD") ?: keystoreProperties['keyPassword']
        }
    }
    
    buildTypes {
        release {
            signingConfig signingConfigs.release
            minifyEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'),
                'proguard-rules.pro'
        }
    }
}

// Read from local.properties (never commit this file!)
def keystoreProperties = new Properties()
def keystoreFile = rootProject.file('keystore.properties')
if (keystoreFile.exists()) {
    keystoreFile.withInputStream { keystoreProperties.load(it) }
}
```

```properties
# keystore.properties (in .gitignore!)
storePassword=yourStorePassword
keyAlias=yourKeyAlias
keyPassword=yourKeyPassword
```

---

## 89.3 Fastlane Setup

```ruby
# fastlane/Fastfile
default_platform(:android)

platform :android do
  
  desc "Run all tests"
  lane :test do
    gradle(task: "test")
  end
  
  desc "Build debug APK"
  lane :build_debug do
    gradle(task: "assemble", build_type: "Debug")
  end
  
  desc "Build and sign release APK"
  lane :build_release do
    gradle(
      task: "assemble",
      build_type: "Release",
      properties: {
        "android.injected.signing.store.file" => ENV["KEYSTORE_PATH"],
        "android.injected.signing.store.password" => ENV["STORE_PASSWORD"],
        "android.injected.signing.key.alias" => ENV["KEY_ALIAS"],
        "android.injected.signing.key.password" => ENV["KEY_PASSWORD"],
      }
    )
  end
  
  desc "Deploy to Firebase App Distribution"
  lane :distribute do
    build_debug
    firebase_app_distribution(
      app: ENV["FIREBASE_APP_ID"],
      testers: "tester1@example.com, tester2@example.com",
      release_notes: File.read("CHANGELOG.md"),
      firebase_cli_token: ENV["FIREBASE_TOKEN"]
    )
  end
  
  desc "Deploy to Google Play (internal track)"
  lane :deploy_internal do
    build_release
    upload_to_play_store(
      track: "internal",
      apk: "app/build/outputs/apk/release/app-release.apk",
      json_key: ENV["PLAY_STORE_JSON_KEY"]
    )
  end
  
  desc "Promote internal → production"
  lane :promote_to_prod do
    upload_to_play_store(
      track: "internal",
      track_promote_to: "production",
      json_key: ENV["PLAY_STORE_JSON_KEY"]
    )
  end
  
end
```

---

## 89.4 สรุป Part 89

ในบทนี้คุณได้เรียนรู้:

✅ GitHub Actions workflow (test, build, deploy)  
✅ Gradle caching in CI  
✅ Decode keystore from GitHub secret  
✅ Sign release APK in CI  
✅ Firebase App Distribution deployment  
✅ Signing config with environment variables  
✅ Fastlane: test, build, distribute, deploy lanes  
✅ Google Play internal → production promotion  

---

*[← Part 88: Java Memory](./part-88-java-memory.md) | [Part 90: Android Release & Play Store →](./part-90-android-release.md)*
