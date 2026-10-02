# Part 31: Android Development - บทนำและการติดตั้ง
## หลักสูตร Java & Android Development - ระดับ Beginner Android

---

## 31.1 Android คืออะไร

Android คือระบบปฏิบัติการ (OS) สำหรับมือถือที่พัฒนาโดย Google  
ใช้ Linux kernel และ Java/Kotlin เป็น programming language หลัก

**สถาปัตยกรรม Android:**
```
Application Layer     (Apps: Gmail, Maps, Chrome)
        ↓
Application Framework (Activity Manager, Content Providers, Views)
        ↓
Android Runtime       (ART, Core Libraries)
        ↓
HAL + Native Libs     (OpenGL, SQLite, WebKit)
        ↓
Linux Kernel          (Drivers, Memory, Processes)
```

---

## 31.2 ติดตั้ง Android Studio

1. ดาวน์โหลด Android Studio จาก https://developer.android.com/studio
2. ติดตั้งและเปิด Android Studio
3. ทำตาม Setup Wizard:
   - ติดตั้ง Android SDK
   - ติดตั้ง Android Virtual Device (AVD)
   - เลือก Standard installation

**System Requirements:**
```
Windows: Windows 8/10/11, 64-bit, 8GB RAM (แนะนำ 16GB)
Mac:     macOS 10.14+, 8GB RAM
Linux:   64-bit, 8GB RAM

Disk: 8GB+ (เพื่อ SDK, emulator images)
```

---

## 31.3 สร้าง Android Project ใหม่

```
File → New → New Project → Empty Views Activity

Project settings:
- Name: MyFirstApp
- Package name: com.example.myfirstapp
- Save location: ~/AndroidProjects/MyFirstApp
- Language: Java
- Minimum SDK: API 24 (Android 7.0) - covers ~95% devices
```

**โครงสร้าง Project:**
```
MyFirstApp/
├── app/
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/com/example/myfirstapp/
│   │   │   │   └── MainActivity.java    ← main code
│   │   │   ├── res/
│   │   │   │   ├── layout/
│   │   │   │   │   └── activity_main.xml   ← UI layout
│   │   │   │   ├── values/
│   │   │   │   │   ├── strings.xml         ← text resources
│   │   │   │   │   ├── colors.xml          ← color resources
│   │   │   │   │   └── themes.xml          ← app theme
│   │   │   │   ├── drawable/               ← images, icons
│   │   │   │   └── mipmap/                 ← app launcher icons
│   │   │   └── AndroidManifest.xml         ← app configuration
│   │   └── test/
│   │       └── java/com/example/myfirstapp/
│   │           └── ExampleUnitTest.java
│   └── build.gradle    ← app dependencies
├── build.gradle        ← project-level build
├── gradle.properties
└── settings.gradle
```

---

## 31.4 AndroidManifest.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <!-- Permissions -->
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.CAMERA" />
    
    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.MyFirstApp">
        
        <!-- Main Activity (LAUNCHER = app entry point) -->
        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        
        <!-- Other activities -->
        <activity
            android:name=".DetailActivity"
            android:exported="false" />
        
        <!-- Services -->
        <service
            android:name=".MyService"
            android:exported="false" />
            
        <!-- Broadcast receivers -->
        <receiver
            android:name=".MyReceiver"
            android:exported="false">
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED" />
            </intent-filter>
        </receiver>
        
    </application>

</manifest>
```

---

## 31.5 Activity พื้นฐาน

```java
// MainActivity.java
package com.example.myfirstapp;

import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;
import android.widget.Toast;
import androidx.appcompat.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {
    
    private static final String TAG = "MainActivity";
    
    // View references
    private EditText editTextName;
    private Button buttonGreet;
    private TextView textViewResult;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // Set the layout XML
        setContentView(R.layout.activity_main);
        
        Log.d(TAG, "onCreate called");
        
        // Find views by their XML id
        editTextName = findViewById(R.id.editTextName);
        buttonGreet = findViewById(R.id.buttonGreet);
        textViewResult = findViewById(R.id.textViewResult);
        
        // Set click listener
        buttonGreet.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                handleGreetClick();
            }
        });
        
        // Lambda style (Java 8+)
        buttonGreet.setOnClickListener(v -> handleGreetClick());
    }
    
    private void handleGreetClick() {
        String name = editTextName.getText().toString().trim();
        
        if (name.isEmpty()) {
            Toast.makeText(this, "กรุณาใส่ชื่อ", Toast.LENGTH_SHORT).show();
            return;
        }
        
        String greeting = "สวัสดี, " + name + "! 🎉";
        textViewResult.setText(greeting);
        textViewResult.setVisibility(View.VISIBLE);
        
        Log.d(TAG, "Greeted: " + name);
    }
    
    // Activity Lifecycle
    @Override
    protected void onStart() {
        super.onStart();
        Log.d(TAG, "onStart");
    }
    
    @Override
    protected void onResume() {
        super.onResume();
        Log.d(TAG, "onResume");
    }
    
    @Override
    protected void onPause() {
        super.onPause();
        Log.d(TAG, "onPause");
    }
    
    @Override
    protected void onStop() {
        super.onStop();
        Log.d(TAG, "onStop");
    }
    
    @Override
    protected void onDestroy() {
        super.onDestroy();
        Log.d(TAG, "onDestroy");
    }
}
```

---

## 31.6 Layout XML

```xml
<!-- res/layout/activity_main.xml -->
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp"
    android:gravity="center_horizontal">

    <!-- App title -->
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="My First App"
        android:textSize="24sp"
        android:textStyle="bold"
        android:layout_marginBottom="32dp" />

    <!-- Input field -->
    <EditText
        android:id="@+id/editTextName"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="ใส่ชื่อของคุณ"
        android:inputType="textPersonName"
        android:layout_marginBottom="16dp" />

    <!-- Button -->
    <Button
        android:id="@+id/buttonGreet"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="ทักทาย"
        android:layout_marginBottom="16dp" />

    <!-- Result text (hidden initially) -->
    <TextView
        android:id="@+id/textViewResult"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="18sp"
        android:textColor="#4CAF50"
        android:visibility="gone" />

</LinearLayout>
```

---

## 31.7 Activity Lifecycle

```
App launched
     ↓
  onCreate()     ← Initialize UI, setup
     ↓
  onStart()      ← Becoming visible
     ↓
  onResume()     ← In foreground, user can interact
     ↓
[User uses app]
     ↓
  onPause()      ← Lost focus (dialog, another app)
     ↓
  onStop()       ← No longer visible
     ↓
  onDestroy()    ← Activity destroyed
     
Restart (e.g. phone rotated):
onPause → onStop → onDestroy → onCreate → onStart → onResume

Background/Foreground:
Going to background:  onPause → onStop
Coming to foreground: onRestart → onStart → onResume
```

---

## 31.8 app/build.gradle

```groovy
plugins {
    id 'com.android.application'
}

android {
    namespace 'com.example.myfirstapp'
    compileSdk 34

    defaultConfig {
        applicationId "com.example.myfirstapp"
        minSdk 24
        targetSdk 34
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'),
                          'proguard-rules.pro'
        }
    }
    
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}

dependencies {
    implementation 'androidx.appcompat:appcompat:1.6.1'
    implementation 'com.google.android.material:material:1.9.0'
    implementation 'androidx.constraintlayout:constraintlayout:2.1.4'
    
    testImplementation 'junit:junit:4.13.2'
    androidTestImplementation 'androidx.test.ext:junit:1.1.5'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.5.1'
}
```

---

## 31.9 รัน App

```
1. Create AVD (Emulator):
   Tools → AVD Manager → Create Virtual Device
   เลือก: Pixel 4, API 30+

2. Run App:
   คลิกปุ่ม ▶ (Run) หรือ Shift+F10
   เลือก emulator หรือ physical device

3. Debug:
   Logcat window: View → Tool Windows → Logcat
   กรอง: "MainActivity" หรือ "D/MyTag"

4. Build APK:
   Build → Build Bundle(s)/APK(s) → Build APK(s)
   ไฟล์อยู่ที่: app/build/outputs/apk/debug/app-debug.apk
```

---

## 31.10 สรุป Part 31

ในบทนี้คุณได้เรียนรู้:

✅ Android architecture  
✅ Android Studio installation  
✅ Project structure  
✅ AndroidManifest.xml  
✅ Activity พื้นฐาน (onCreate, Lifecycle)  
✅ Layout XML (LinearLayout, TextView, Button, EditText)  
✅ Event listeners  
✅ Toast messages  
✅ Build and run app  

---

*[← Part 30: Java Security](./part-30-security.md) | [Part 32: Android Views & Layouts →](./part-32-android-views.md)*
