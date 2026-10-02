# Part 42: Android Permissions
## หลักสูตร Java & Android Development - ระดับ Intermediate Android

---

## 42.1 Permission ประเภทต่างๆ

```
Normal permissions (ไม่ต้อง request runtime):
- INTERNET
- ACCESS_NETWORK_STATE
- VIBRATE
- RECEIVE_BOOT_COMPLETED

Dangerous permissions (ต้อง request runtime, Android 6+):
- Camera      CAMERA
- Location    ACCESS_FINE_LOCATION, ACCESS_COARSE_LOCATION
- Contacts    READ_CONTACTS, WRITE_CONTACTS
- Storage     READ_EXTERNAL_STORAGE, WRITE_EXTERNAL_STORAGE
- Microphone  RECORD_AUDIO
- Phone       CALL_PHONE, READ_PHONE_STATE
- SMS         READ_SMS, SEND_SMS
- Calendar    READ_CALENDAR, WRITE_CALENDAR

Special permissions:
- MANAGE_EXTERNAL_STORAGE (Android 11+)
- SCHEDULE_EXACT_ALARM
- POST_NOTIFICATIONS (Android 13+)
```

---

## 42.2 Permission Helper

```java
// util/PermissionHelper.java
package com.example.myapp.util;

import android.Manifest;
import android.app.Activity;
import android.content.Context;
import android.content.pm.PackageManager;
import androidx.activity.result.ActivityResultLauncher;
import androidx.activity.result.contract.ActivityResultContracts;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;

public class PermissionHelper {
    
    // Permission groups
    public static final String[] CAMERA_PERMISSIONS = {
        Manifest.permission.CAMERA
    };
    
    public static final String[] LOCATION_PERMISSIONS = {
        Manifest.permission.ACCESS_FINE_LOCATION,
        Manifest.permission.ACCESS_COARSE_LOCATION
    };
    
    public static final String[] STORAGE_PERMISSIONS;
    static {
        if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.TIRAMISU) {
            STORAGE_PERMISSIONS = new String[]{
                Manifest.permission.READ_MEDIA_IMAGES,
                Manifest.permission.READ_MEDIA_VIDEO,
                Manifest.permission.READ_MEDIA_AUDIO
            };
        } else {
            STORAGE_PERMISSIONS = new String[]{
                Manifest.permission.READ_EXTERNAL_STORAGE,
                Manifest.permission.WRITE_EXTERNAL_STORAGE
            };
        }
    }
    
    // Check single permission
    public static boolean hasPermission(Context context, String permission) {
        return ContextCompat.checkSelfPermission(context, permission)
            == PackageManager.PERMISSION_GRANTED;
    }
    
    // Check multiple permissions
    public static boolean hasPermissions(Context context, String... permissions) {
        for (String permission : permissions) {
            if (!hasPermission(context, permission)) return false;
        }
        return true;
    }
    
    // Should show rationale
    public static boolean shouldShowRationale(Activity activity, String permission) {
        return ActivityCompat.shouldShowRequestPermissionRationale(activity, permission);
    }
    
    public interface PermissionCallback {
        void onGranted();
        void onDenied(boolean isPermanentlyDenied);
    }
}
```

---

## 42.3 Camera Permission Example

```java
// CameraActivity.java
package com.example.myapp;

import android.Manifest;
import android.content.Intent;
import android.content.pm.PackageManager;
import android.net.Uri;
import android.os.Bundle;
import android.provider.Settings;
import android.widget.*;
import androidx.activity.result.*;
import androidx.activity.result.contract.*;
import androidx.appcompat.app.AlertDialog;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.content.ContextCompat;

public class CameraActivity extends AppCompatActivity {

    private ImageView ivPreview;

    // Permission launcher
    private final ActivityResultLauncher<String> requestCameraPermission =
        registerForActivityResult(
            new ActivityResultContracts.RequestPermission(),
            granted -> {
                if (granted) {
                    openCamera();
                } else {
                    if (shouldShowRequestPermissionRationale(Manifest.permission.CAMERA)) {
                        // Show rationale dialog
                        showRationaleDialog();
                    } else {
                        // Permanently denied - go to settings
                        showPermissionDeniedDialog();
                    }
                }
            });

    // Camera launcher
    private final ActivityResultLauncher<Void> takePictureLauncher =
        registerForActivityResult(
            new ActivityResultContracts.TakePicturePreview(),
            bitmap -> {
                if (bitmap != null) {
                    ivPreview.setImageBitmap(bitmap);
                    Toast.makeText(this, "ถ่ายรูปสำเร็จ", Toast.LENGTH_SHORT).show();
                }
            });

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_camera);

        ivPreview = findViewById(R.id.ivPreview);
        Button btnCamera = findViewById(R.id.btnCamera);

        btnCamera.setOnClickListener(v -> checkAndRequestCameraPermission());
    }

    private void checkAndRequestCameraPermission() {
        if (ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA)
                == PackageManager.PERMISSION_GRANTED) {
            openCamera();
        } else {
            requestCameraPermission.launch(Manifest.permission.CAMERA);
        }
    }

    private void openCamera() {
        takePictureLauncher.launch(null);
    }

    private void showRationaleDialog() {
        new AlertDialog.Builder(this)
            .setTitle("ต้องการอนุญาตกล้อง")
            .setMessage("แอปต้องการอนุญาตกล้องเพื่อถ่ายรูป\nกรุณาอนุญาตเพื่อใช้งานฟีเจอร์นี้")
            .setPositiveButton("อนุญาต", (d, w) -> 
                requestCameraPermission.launch(Manifest.permission.CAMERA))
            .setNegativeButton("ยกเลิก", null)
            .show();
    }

    private void showPermissionDeniedDialog() {
        new AlertDialog.Builder(this)
            .setTitle("ถูกปฏิเสธถาวร")
            .setMessage("กรุณาเปิดอนุญาตกล้องในการตั้งค่า")
            .setPositiveButton("ไปที่การตั้งค่า", (d, w) -> openAppSettings())
            .setNegativeButton("ยกเลิก", null)
            .show();
    }

    private void openAppSettings() {
        Intent intent = new Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS);
        intent.setData(Uri.fromParts("package", getPackageName(), null));
        startActivity(intent);
    }
}
```

---

## 42.4 Multiple Permissions (Location + Storage)

```java
// MultiPermissionActivity.java
public class MultiPermissionActivity extends AppCompatActivity {

    private final ActivityResultLauncher<String[]> requestMultiplePermissions =
        registerForActivityResult(
            new ActivityResultContracts.RequestMultiplePermissions(),
            permissions -> {
                boolean locationGranted = Boolean.TRUE.equals(
                    permissions.get(Manifest.permission.ACCESS_FINE_LOCATION));
                boolean storageGranted = Boolean.TRUE.equals(
                    permissions.get(Manifest.permission.READ_EXTERNAL_STORAGE));
                
                if (locationGranted && storageGranted) {
                    Toast.makeText(this, "ได้รับอนุญาตทั้งหมด", Toast.LENGTH_SHORT).show();
                    proceed();
                } else {
                    StringBuilder denied = new StringBuilder("ไม่ได้รับอนุญาต: ");
                    if (!locationGranted) denied.append("Location ");
                    if (!storageGranted) denied.append("Storage ");
                    Toast.makeText(this, denied.toString(), Toast.LENGTH_LONG).show();
                }
            });

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_multi_permission);

        Button btnRequest = findViewById(R.id.btnRequest);
        btnRequest.setOnClickListener(v -> {
            String[] permissions = {
                Manifest.permission.ACCESS_FINE_LOCATION,
                Manifest.permission.READ_EXTERNAL_STORAGE
            };
            
            // Check which ones we need
            boolean needLocation = ContextCompat.checkSelfPermission(
                this, Manifest.permission.ACCESS_FINE_LOCATION) != PackageManager.PERMISSION_GRANTED;
            boolean needStorage = ContextCompat.checkSelfPermission(
                this, Manifest.permission.READ_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED;
            
            if (!needLocation && !needStorage) {
                proceed();
            } else {
                requestMultiplePermissions.launch(permissions);
            }
        });
    }

    private void proceed() {
        Toast.makeText(this, "เริ่มดำเนินการ...", Toast.LENGTH_SHORT).show();
    }
}
```

---

## 42.5 Location Permission + GPS

```java
// LocationActivity.java
import android.location.*;
import com.google.android.gms.location.*;

public class LocationActivity extends AppCompatActivity {

    private FusedLocationProviderClient fusedLocationClient;
    private TextView tvLocation;

    private final ActivityResultLauncher<String[]> locationPermissionRequest =
        registerForActivityResult(
            new ActivityResultContracts.RequestMultiplePermissions(),
            permissions -> {
                boolean fine = Boolean.TRUE.equals(
                    permissions.get(Manifest.permission.ACCESS_FINE_LOCATION));
                boolean coarse = Boolean.TRUE.equals(
                    permissions.get(Manifest.permission.ACCESS_COARSE_LOCATION));
                
                if (fine || coarse) {
                    getLocation();
                } else {
                    Toast.makeText(this, "ไม่ได้รับอนุญาต GPS", Toast.LENGTH_SHORT).show();
                }
            });

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_location);

        fusedLocationClient = LocationServices.getFusedLocationProviderClient(this);
        tvLocation = findViewById(R.id.tvLocation);

        Button btnGetLocation = findViewById(R.id.btnGetLocation);
        btnGetLocation.setOnClickListener(v -> checkLocationPermission());
    }

    private void checkLocationPermission() {
        if (ContextCompat.checkSelfPermission(this, Manifest.permission.ACCESS_FINE_LOCATION)
                == PackageManager.PERMISSION_GRANTED) {
            getLocation();
        } else {
            locationPermissionRequest.launch(new String[]{
                Manifest.permission.ACCESS_FINE_LOCATION,
                Manifest.permission.ACCESS_COARSE_LOCATION
            });
        }
    }

    @SuppressLint("MissingPermission")
    private void getLocation() {
        fusedLocationClient.getLastLocation()
            .addOnSuccessListener(this, location -> {
                if (location != null) {
                    String text = String.format("ละติจูด: %.6f\nลองจิจูด: %.6f",
                        location.getLatitude(), location.getLongitude());
                    tvLocation.setText(text);
                } else {
                    requestNewLocation();
                }
            });
    }

    @SuppressLint("MissingPermission")
    private void requestNewLocation() {
        LocationRequest request = new LocationRequest.Builder(Priority.PRIORITY_HIGH_ACCURACY, 1000)
            .setWaitForAccurateLocation(false)
            .setMinUpdateIntervalMillis(500)
            .setMaxUpdates(1)
            .build();
        
        fusedLocationClient.requestLocationUpdates(request,
            new LocationCallback() {
                @Override
                public void onLocationResult(LocationResult result) {
                    Location location = result.getLastLocation();
                    if (location != null) {
                        tvLocation.setText(String.format(
                            "ละติจูด: %.6f\nลองจิจูด: %.6f",
                            location.getLatitude(), location.getLongitude()));
                    }
                }
            },
            getMainLooper());
    }
}
```

---

## 42.6 สรุป Part 42

ในบทนี้คุณได้เรียนรู้:

✅ Normal vs Dangerous permissions  
✅ Request permission at runtime  
✅ Handle permission result  
✅ Show rationale dialog  
✅ Permanently denied → app settings  
✅ Multiple permissions request  
✅ Camera permission + TakePicture  
✅ Location permission + FusedLocationProvider  

---

*[← Part 41: Android Services](./part-41-android-services.md) | [Part 43: Custom Views →](./part-43-android-custom-views.md)*
