# Part 54: CI/CD สำหรับ Android
## หลักสูตร Java & Android Development - ระดับ Professional Android

---

## 54.1 CI/CD คืออะไร

```
CI = Continuous Integration  - auto build & test บน every push
CD = Continuous Delivery     - auto deploy to testing/production

ประโยชน์:
- ค้นพบ bug เร็วขึ้น
- Deploy อัตโนมัติ
- Consistent build environment
- Code quality enforcement
```

---

## 54.2 GitHub Actions - Basic Workflow

```yaml
# .github/workflows/android.yml
name: Android CI

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
      
      - name: Setup JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
      
      - name: Cache Gradle
        uses: actions/cache@v3
        with:
          path: |
            ~/.gradle/caches
            ~/.gradle/wrapper
          key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*') }}
          restore-keys: ${{ runner.os }}-gradle-
      
      - name: Grant Gradle permissions
        run: chmod +x gradlew
      
      - name: Run Unit Tests
        run: ./gradlew test
      
      - name: Upload test results
        uses: actions/upload-artifact@v3
        if: failure()
        with:
          name: test-results
          path: app/build/reports/tests/
  
  lint:
    name: Lint Check
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
      
      - name: Run Lint
        run: ./gradlew lint
      
      - name: Upload lint results
        uses: actions/upload-artifact@v3
        if: failure()
        with:
          name: lint-results
          path: app/build/reports/lint-results-*.html
  
  build:
    name: Build Release APK
    runs-on: ubuntu-latest
    needs: [test, lint]
    if: github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
      
      # Decode keystore from secret
      - name: Decode Keystore
        env:
          KEYSTORE_BASE64: ${{ secrets.KEYSTORE_BASE64 }}
        run: echo "$KEYSTORE_BASE64" | base64 -d > app/keystore.jks
      
      - name: Build Release APK
        env:
          KEYSTORE_PASSWORD: ${{ secrets.KEYSTORE_PASSWORD }}
          KEY_ALIAS: ${{ secrets.KEY_ALIAS }}
          KEY_PASSWORD: ${{ secrets.KEY_PASSWORD }}
        run: |
          ./gradlew assembleRelease \
            -Pandroid.injected.signing.store.file=keystore.jks \
            -Pandroid.injected.signing.store.password=$KEYSTORE_PASSWORD \
            -Pandroid.injected.signing.key.alias=$KEY_ALIAS \
            -Pandroid.injected.signing.key.password=$KEY_PASSWORD
      
      - name: Upload APK
        uses: actions/upload-artifact@v3
        with:
          name: release-apk
          path: app/build/outputs/apk/release/*.apk
```

---

## 54.3 Signing Configuration

```groovy
// build.gradle - release signing
android {
    signingConfigs {
        release {
            if (project.hasProperty('KEYSTORE_FILE')) {
                storeFile file(KEYSTORE_FILE)
                storePassword KEYSTORE_PASSWORD
                keyAlias KEY_ALIAS
                keyPassword KEY_PASSWORD
            }
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
        
        debug {
            applicationIdSuffix ".debug"
            versionNameSuffix "-DEBUG"
            debuggable true
        }
    }
}
```

```properties
# local.properties (NEVER commit this!)
KEYSTORE_FILE=../keystore.jks
KEYSTORE_PASSWORD=your_keystore_password
KEY_ALIAS=your_key_alias
KEY_PASSWORD=your_key_password
```

---

## 54.4 Deploy to Firebase App Distribution

```yaml
# .github/workflows/distribute.yml
name: Firebase Distribution

on:
  push:
    branches: [ develop ]

jobs:
  distribute:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
      
      - name: Build Debug APK
        run: ./gradlew assembleDebug
      
      - name: Upload to Firebase App Distribution
        uses: wzieba/Firebase-Distribution-Github-Action@v1
        with:
          appId: ${{ secrets.FIREBASE_APP_ID }}
          serviceCredentialsFileContent: ${{ secrets.CREDENTIAL_FILE_CONTENT }}
          groups: testers
          file: app/build/outputs/apk/debug/app-debug.apk
          releaseNotes: |
            Build from branch: ${{ github.ref_name }}
            Commit: ${{ github.sha }}
            Message: ${{ github.event.head_commit.message }}
```

---

## 54.5 Version Management

```groovy
// build.gradle - automatic version from git
def getVersionCode = {
    def code = 1
    try {
        def stdout = new ByteArrayOutputStream()
        exec {
            commandLine 'git', 'rev-list', '--count', 'HEAD'
            standardOutput = stdout
        }
        code = Integer.parseInt(stdout.toString().trim())
    } catch (ignored) {}
    return code
}

def getVersionName = {
    def name = "1.0.0"
    try {
        def stdout = new ByteArrayOutputStream()
        exec {
            commandLine 'git', 'describe', '--tags', '--always', '--dirty'
            standardOutput = stdout
        }
        name = stdout.toString().trim()
    } catch (ignored) {}
    return name
}

android {
    defaultConfig {
        versionCode getVersionCode()
        versionName getVersionName()
    }
}
```

---

## 54.6 Fastlane (Optional - Mac/Linux)

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
    gradle(
      task: "assemble",
      build_type: "Debug"
    )
  end
  
  desc "Build and sign release"
  lane :build_release do
    gradle(
      task: "bundle",
      build_type: "Release",
      properties: {
        "android.injected.signing.store.file" => ENV["KEYSTORE_FILE"],
        "android.injected.signing.store.password" => ENV["KEYSTORE_PASSWORD"],
        "android.injected.signing.key.alias" => ENV["KEY_ALIAS"],
        "android.injected.signing.key.password" => ENV["KEY_PASSWORD"],
      }
    )
  end
  
  desc "Deploy to Play Store (internal track)"
  lane :deploy_internal do
    build_release
    upload_to_play_store(
      track: "internal",
      aab: "app/build/outputs/bundle/release/app-release.aab"
    )
  end
  
  desc "Promote internal to production"
  lane :promote_to_production do
    upload_to_play_store(
      track: "internal",
      track_promote_to: "production",
      rollout: "0.1"  # 10% rollout
    )
  end
end
```

---

## 54.7 Code Quality Gates

```yaml
# .github/workflows/quality.yml
name: Code Quality

on: [pull_request]

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
      
      # Lint
      - name: Lint
        run: ./gradlew lint
      
      # Unit tests with coverage
      - name: Test with coverage
        run: ./gradlew testDebugUnitTestCoverage
      
      # Check coverage threshold
      - name: Check coverage
        run: |
          COVERAGE=$(cat app/build/reports/jacoco/testDebugUnitTestCoverage/index.html \
            | grep -o 'Total[^%]*%' | tail -1 | grep -o '[0-9]*%' | head -1 | tr -d '%')
          echo "Coverage: $COVERAGE%"
          if [ "$COVERAGE" -lt 60 ]; then
            echo "Coverage $COVERAGE% is below 60% threshold"
            exit 1
          fi
```

---

## 54.8 สรุป Part 54

ในบทนี้คุณได้เรียนรู้:

✅ CI/CD concept  
✅ GitHub Actions workflow (test + build)  
✅ Keystore signing in CI (secrets)  
✅ Firebase App Distribution  
✅ Auto version code from git  
✅ Fastlane for automation  
✅ Code quality gates (lint + coverage)  

---

*[← Part 53: Security](./part-53-android-security.md) | [Part 55: Play Store Publishing →](./part-55-android-publish.md)*
