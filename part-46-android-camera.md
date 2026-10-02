# Part 46: Camera2 API และ Image Processing
## หลักสูตร Java & Android Development - ระดับ Advanced Android

---

## 46.1 วิธีถ่ายรูปด้วย Camera Intent (ง่ายที่สุด)

```java
// CameraIntentActivity.java
public class CameraIntentActivity extends AppCompatActivity {
    
    private ImageView ivPhoto;
    private Uri photoUri;
    
    // TakePicture เก็บเป็น file
    private final ActivityResultLauncher<Uri> takePictureLauncher =
        registerForActivityResult(
            new ActivityResultContracts.TakePicture(),
            success -> {
                if (success) {
                    ivPhoto.setImageURI(photoUri);
                }
            });
    
    private final ActivityResultLauncher<String> requestCameraPermission =
        registerForActivityResult(
            new ActivityResultContracts.RequestPermission(),
            granted -> {
                if (granted) launchCamera();
            });
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_camera);
        
        ivPhoto = findViewById(R.id.ivPhoto);
        
        findViewById(R.id.btnCapture).setOnClickListener(v -> {
            if (ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA)
                    == PackageManager.PERMISSION_GRANTED) {
                launchCamera();
            } else {
                requestCameraPermission.launch(Manifest.permission.CAMERA);
            }
        });
    }
    
    private void launchCamera() {
        // Create temp file for photo
        File photoFile = createImageFile();
        if (photoFile == null) return;
        
        // FileProvider URI (needed for Android 7+)
        photoUri = FileProvider.getUriForFile(this,
            getPackageName() + ".fileprovider",
            photoFile);
        
        takePictureLauncher.launch(photoUri);
    }
    
    private File createImageFile() {
        try {
            String timestamp = new SimpleDateFormat("yyyyMMdd_HHmmss", Locale.getDefault())
                .format(new Date());
            File storageDir = getExternalFilesDir(Environment.DIRECTORY_PICTURES);
            return File.createTempFile("IMG_" + timestamp, ".jpg", storageDir);
        } catch (IOException e) {
            return null;
        }
    }
}
```

**AndroidManifest.xml:**
```xml
<uses-permission android:name="android.permission.CAMERA"/>

<provider
    android:name="androidx.core.content.FileProvider"
    android:authorities="${applicationId}.fileprovider"
    android:exported="false"
    android:grantUriPermissions="true">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/file_paths"/>
</provider>
```

**res/xml/file_paths.xml:**
```xml
<?xml version="1.0" encoding="utf-8"?>
<paths>
    <external-files-path name="images" path="Pictures/"/>
    <cache-path name="cache" path="/"/>
</paths>
```

---

## 46.2 CameraX (แนะนำสำหรับ Camera ใหม่)

```groovy
// build.gradle
def camerax_version = "1.3.0"
implementation "androidx.camera:camera-core:$camerax_version"
implementation "androidx.camera:camera-camera2:$camerax_version"
implementation "androidx.camera:camera-lifecycle:$camerax_version"
implementation "androidx.camera:camera-view:$camerax_version"
```

```xml
<!-- activity_camerax.xml -->
<androidx.camera.view.PreviewView
    android:id="@+id/previewView"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />

<Button
    android:id="@+id/btnCapture"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_gravity="bottom|center"
    android:layout_marginBottom="32dp"
    android:text="ถ่ายรูป" />
```

```java
// CameraXActivity.java
import androidx.camera.core.*;
import androidx.camera.lifecycle.*;
import androidx.camera.view.*;
import java.util.concurrent.*;

public class CameraXActivity extends AppCompatActivity {
    
    private PreviewView previewView;
    private ImageCapture imageCapture;
    private ExecutorService cameraExecutor;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_camerax);
        
        previewView = findViewById(R.id.previewView);
        cameraExecutor = Executors.newSingleThreadExecutor();
        
        // Check permission and start camera
        if (ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA)
                == PackageManager.PERMISSION_GRANTED) {
            startCamera();
        } else {
            // Request permission (as in Part 42)
        }
        
        findViewById(R.id.btnCapture).setOnClickListener(v -> takePhoto());
    }
    
    private void startCamera() {
        ListenableFuture<ProcessCameraProvider> cameraProviderFuture =
            ProcessCameraProvider.getInstance(this);
        
        cameraProviderFuture.addListener(() -> {
            try {
                ProcessCameraProvider cameraProvider = cameraProviderFuture.get();
                
                // Preview
                Preview preview = new Preview.Builder().build();
                preview.setSurfaceProvider(previewView.getSurfaceProvider());
                
                // Image Capture
                imageCapture = new ImageCapture.Builder()
                    .setCaptureMode(ImageCapture.CAPTURE_MODE_MINIMIZE_LATENCY)
                    .build();
                
                // Image Analysis (realtime processing)
                ImageAnalysis imageAnalysis = new ImageAnalysis.Builder()
                    .setBackpressureStrategy(ImageAnalysis.STRATEGY_KEEP_ONLY_LATEST)
                    .build();
                imageAnalysis.setAnalyzer(cameraExecutor, image -> {
                    // Process each frame
                    int rotDeg = image.getImageInfo().getRotationDegrees();
                    // ... your analysis
                    image.close();
                });
                
                // Select back camera
                CameraSelector cameraSelector = CameraSelector.DEFAULT_BACK_CAMERA;
                
                // Unbind previous use cases
                cameraProvider.unbindAll();
                
                // Bind to lifecycle
                cameraProvider.bindToLifecycle(
                    this, cameraSelector, preview, imageCapture, imageAnalysis);
                
            } catch (Exception e) {
                Log.e("CameraX", "Use case binding failed", e);
            }
        }, ContextCompat.getMainExecutor(this));
    }
    
    private void takePhoto() {
        if (imageCapture == null) return;
        
        // Create output file
        File photoFile = new File(getExternalFilesDir(Environment.DIRECTORY_PICTURES),
            "photo_" + System.currentTimeMillis() + ".jpg");
        
        ImageCapture.OutputFileOptions outputOptions =
            new ImageCapture.OutputFileOptions.Builder(photoFile).build();
        
        imageCapture.takePicture(outputOptions, cameraExecutor,
            new ImageCapture.OnImageSavedCallback() {
                @Override
                public void onImageSaved(ImageCapture.OutputFileResults results) {
                    runOnUiThread(() -> Toast.makeText(CameraXActivity.this,
                        "บันทึกรูปที่: " + photoFile.getAbsolutePath(),
                        Toast.LENGTH_SHORT).show());
                }
                
                @Override
                public void onError(ImageCaptureException e) {
                    Log.e("CameraX", "Photo capture failed: " + e.getMessage());
                }
            });
    }
    
    @Override
    protected void onDestroy() {
        super.onDestroy();
        cameraExecutor.shutdown();
    }
}
```

---

## 46.3 Image Processing (Bitmap)

```java
// ImageProcessor.java
public class ImageProcessor {
    
    // Resize bitmap
    public static Bitmap resize(Bitmap original, int maxWidth, int maxHeight) {
        int width  = original.getWidth();
        int height = original.getHeight();
        
        float ratio = Math.min(
            (float) maxWidth / width,
            (float) maxHeight / height);
        
        int newWidth  = (int) (width * ratio);
        int newHeight = (int) (height * ratio);
        
        return Bitmap.createScaledBitmap(original, newWidth, newHeight, true);
    }
    
    // Rotate bitmap
    public static Bitmap rotate(Bitmap bitmap, float degrees) {
        Matrix matrix = new Matrix();
        matrix.postRotate(degrees);
        return Bitmap.createBitmap(bitmap, 0, 0,
            bitmap.getWidth(), bitmap.getHeight(), matrix, true);
    }
    
    // Fix EXIF rotation (photos from camera are often rotated)
    public static Bitmap fixRotation(String imagePath) throws IOException {
        ExifInterface exif = new ExifInterface(imagePath);
        int orientation = exif.getAttributeInt(
            ExifInterface.TAG_ORIENTATION,
            ExifInterface.ORIENTATION_NORMAL);
        
        Bitmap bitmap = BitmapFactory.decodeFile(imagePath);
        
        float degrees = 0;
        switch (orientation) {
            case ExifInterface.ORIENTATION_ROTATE_90:  degrees = 90;  break;
            case ExifInterface.ORIENTATION_ROTATE_180: degrees = 180; break;
            case ExifInterface.ORIENTATION_ROTATE_270: degrees = 270; break;
        }
        
        return degrees != 0 ? rotate(bitmap, degrees) : bitmap;
    }
    
    // Convert to grayscale
    public static Bitmap toGrayscale(Bitmap original) {
        Bitmap grayscale = Bitmap.createBitmap(
            original.getWidth(), original.getHeight(), Bitmap.Config.ARGB_8888);
        
        Canvas canvas = new Canvas(grayscale);
        Paint paint = new Paint();
        ColorMatrix cm = new ColorMatrix();
        cm.setSaturation(0);
        paint.setColorFilter(new ColorMatrixColorFilter(cm));
        canvas.drawBitmap(original, 0, 0, paint);
        
        return grayscale;
    }
    
    // Compress to byte array for upload
    public static byte[] toJpegBytes(Bitmap bitmap, int quality) {
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        bitmap.compress(Bitmap.CompressFormat.JPEG, quality, baos);
        return baos.toByteArray();
    }
    
    // Load bitmap from URI with downsampling (avoid OOM)
    public static Bitmap decodeSampledBitmapFromUri(Context context, Uri uri,
            int reqWidth, int reqHeight) throws IOException {
        
        InputStream is = context.getContentResolver().openInputStream(uri);
        if (is == null) return null;
        
        // First: measure dimensions only
        BitmapFactory.Options options = new BitmapFactory.Options();
        options.inJustDecodeBounds = true;
        BitmapFactory.decodeStream(is, null, options);
        is.close();
        
        // Calculate sample size
        options.inSampleSize = calculateInSampleSize(options, reqWidth, reqHeight);
        options.inJustDecodeBounds = false;
        
        // Decode with sample size
        is = context.getContentResolver().openInputStream(uri);
        Bitmap result = BitmapFactory.decodeStream(is, null, options);
        if (is != null) is.close();
        return result;
    }
    
    private static int calculateInSampleSize(BitmapFactory.Options options,
            int reqWidth, int reqHeight) {
        int height = options.outHeight;
        int width  = options.outWidth;
        int inSampleSize = 1;
        
        if (height > reqHeight || width > reqWidth) {
            int halfHeight = height / 2;
            int halfWidth  = width / 2;
            while ((halfHeight / inSampleSize) >= reqHeight
                    && (halfWidth / inSampleSize) >= reqWidth) {
                inSampleSize *= 2;
            }
        }
        return inSampleSize;
    }
}
```

---

## 46.4 Glide - Loading Images

```groovy
// build.gradle
implementation 'com.github.bumptech.glide:glide:4.16.0'
annotationProcessor 'com.github.bumptech.glide:compiler:4.16.0'
```

```java
// GlideExamples.java

// Basic load
Glide.with(context)
    .load(imageUrl)
    .into(imageView);

// With placeholder and error
Glide.with(context)
    .load(imageUrl)
    .placeholder(R.drawable.img_placeholder)
    .error(R.drawable.img_error)
    .into(imageView);

// Circular crop
Glide.with(context)
    .load(userPhotoUrl)
    .circleCrop()
    .into(avatarImageView);

// Rounded corners
Glide.with(context)
    .load(imageUrl)
    .transform(new RoundedCorners(16))
    .into(imageView);

// Resize + center crop
Glide.with(context)
    .load(imageUrl)
    .override(300, 300)
    .centerCrop()
    .into(imageView);

// Load from File/URI
Glide.with(context)
    .load(new File("/sdcard/photo.jpg"))
    .into(imageView);

Glide.with(context)
    .load(uri)
    .into(imageView);

// Preload
Glide.with(context)
    .load(imageUrl)
    .preload();

// Custom cache key (force refresh)
Glide.with(context)
    .load(imageUrl)
    .diskCacheStrategy(DiskCacheStrategy.NONE)
    .skipMemoryCache(true)
    .into(imageView);

// Get bitmap
Glide.with(context)
    .asBitmap()
    .load(imageUrl)
    .into(new CustomTarget<Bitmap>() {
        @Override
        public void onResourceReady(Bitmap resource, Transition<? super Bitmap> transition) {
            // use bitmap
        }
        @Override
        public void onLoadCleared(Drawable placeholder) {}
    });
```

---

## 46.5 สรุป Part 46

ในบทนี้คุณได้เรียนรู้:

✅ Camera Intent (TakePicture)  
✅ FileProvider for file URI  
✅ CameraX (Preview, ImageCapture, ImageAnalysis)  
✅ Bitmap resize, rotate, grayscale  
✅ EXIF rotation fix  
✅ Bitmap downsampling (avoid OOM)  
✅ Glide image loading library  

---

*[← Part 45: Content Providers](./part-45-android-content-providers.md) | [Part 47: Maps and Location →](./part-47-android-maps.md)*
