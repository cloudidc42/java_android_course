# Part 82: Advanced Camera & Media
## หลักสูตร Java & Android Development - ระดับ World-Class

---

## 82.1 CameraX ขั้นสูง

```groovy
// build.gradle
def camerax_version = "1.3.4"
implementation "androidx.camera:camera-core:$camerax_version"
implementation "androidx.camera:camera-camera2:$camerax_version"
implementation "androidx.camera:camera-lifecycle:$camerax_version"
implementation "androidx.camera:camera-view:$camerax_version"
implementation "androidx.camera:camera-extensions:$camerax_version"
```

```java
// CameraFragment.java
@AndroidEntryPoint
public class CameraFragment extends Fragment {
    
    private PreviewView previewView;
    private ImageCapture imageCapture;
    private VideoCapture<Recorder> videoCapture;
    private Recording currentRecording;
    
    private static final int REQUIRED_PERMISSIONS_REQUEST = 10;
    private static final String[] REQUIRED_PERMISSIONS = {
        Manifest.permission.CAMERA,
        Manifest.permission.RECORD_AUDIO
    };
    
    @Override
    public void onViewCreated(View view, Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        previewView = view.findViewById(R.id.previewView);
        
        if (allPermissionsGranted()) {
            startCamera();
        } else {
            requestPermissions(REQUIRED_PERMISSIONS, REQUIRED_PERMISSIONS_REQUEST);
        }
        
        view.findViewById(R.id.btnCapture).setOnClickListener(v -> takePhoto());
        view.findViewById(R.id.btnRecord).setOnClickListener(v -> toggleRecording());
    }
    
    private void startCamera() {
        ListenableFuture<ProcessCameraProvider> future =
            ProcessCameraProvider.getInstance(requireContext());
        
        future.addListener(() -> {
            try {
                ProcessCameraProvider cameraProvider = future.get();
                bindCameraUseCases(cameraProvider);
            } catch (Exception e) {
                Log.e("Camera", "Failed to bind", e);
            }
        }, ContextCompat.getMainExecutor(requireContext()));
    }
    
    private void bindCameraUseCases(ProcessCameraProvider cameraProvider) {
        // Preview
        Preview preview = new Preview.Builder()
            .setTargetAspectRatio(AspectRatio.RATIO_16_9)
            .build();
        preview.setSurfaceProvider(previewView.getSurfaceProvider());
        
        // Image capture
        imageCapture = new ImageCapture.Builder()
            .setCaptureMode(ImageCapture.CAPTURE_MODE_MAXIMIZE_QUALITY)
            .setTargetRotation(requireView().getDisplay().getRotation())
            .setFlashMode(ImageCapture.FLASH_MODE_AUTO)
            .build();
        
        // Video capture
        Recorder recorder = new Recorder.Builder()
            .setQualitySelector(QualitySelector.from(Quality.HIGHEST))
            .build();
        videoCapture = VideoCapture.withOutput(recorder);
        
        // Image analysis (real-time processing)
        ImageAnalysis imageAnalysis = new ImageAnalysis.Builder()
            .setTargetResolution(new Size(1280, 720))
            .setBackpressureStrategy(ImageAnalysis.STRATEGY_KEEP_ONLY_LATEST)
            .build();
        imageAnalysis.setAnalyzer(
            ContextCompat.getMainExecutor(requireContext()),
            image -> {
                analyzeImage(image);
                image.close();  // MUST close!
            });
        
        CameraSelector cameraSelector = CameraSelector.DEFAULT_BACK_CAMERA;
        
        cameraProvider.unbindAll();
        Camera camera = cameraProvider.bindToLifecycle(
            getViewLifecycleOwner(),
            cameraSelector,
            preview, imageCapture, videoCapture, imageAnalysis);
        
        // Pinch-to-zoom
        ScaleGestureDetector scaleDetector = new ScaleGestureDetector(requireContext(),
            new ScaleGestureDetector.SimpleOnScaleGestureListener() {
                @Override
                public boolean onScale(@NonNull ScaleGestureDetector detector) {
                    float currentZoom = camera.getCameraInfo()
                        .getZoomState().getValue().getZoomRatio();
                    float delta = detector.getScaleFactor();
                    camera.getCameraControl().setZoomRatio(currentZoom * delta);
                    return true;
                }
            });
        previewView.setOnTouchListener((v, event) -> {
            scaleDetector.onTouchEvent(event);
            return true;
        });
    }
    
    private void takePhoto() {
        if (imageCapture == null) return;
        
        // Create output file
        File outputDir = requireContext().getCacheDir();
        File outputFile;
        try {
            outputFile = File.createTempFile("IMG_", ".jpg", outputDir);
        } catch (IOException e) {
            return;
        }
        
        ImageCapture.OutputFileOptions options = new ImageCapture.OutputFileOptions
            .Builder(outputFile).build();
        
        imageCapture.takePicture(
            options,
            ContextCompat.getMainExecutor(requireContext()),
            new ImageCapture.OnImageSavedCallback() {
                @Override
                public void onImageSaved(@NonNull ImageCapture.OutputFileResults results) {
                    Uri savedUri = results.getSavedUri() != null
                        ? results.getSavedUri()
                        : Uri.fromFile(outputFile);
                    
                    Log.d("Camera", "Photo saved: " + savedUri);
                    // Process/display the photo
                    showPhoto(savedUri);
                }
                
                @Override
                public void onError(@NonNull ImageCaptureException error) {
                    Log.e("Camera", "Photo failed", error);
                }
            });
    }
    
    @SuppressLint("MissingPermission")
    private void toggleRecording() {
        if (currentRecording != null) {
            currentRecording.stop();
            currentRecording = null;
            return;
        }
        
        File videoFile = new File(requireContext().getExternalFilesDir(
            Environment.DIRECTORY_MOVIES), "VID_" + System.currentTimeMillis() + ".mp4");
        
        FileOutputOptions options = new FileOutputOptions.Builder(videoFile).build();
        
        currentRecording = videoCapture.getOutput()
            .prepareRecording(requireContext(), options)
            .withAudioEnabled()
            .start(ContextCompat.getMainExecutor(requireContext()), event -> {
                if (event instanceof VideoRecordEvent.Start) {
                    Log.d("Camera", "Recording started");
                } else if (event instanceof VideoRecordEvent.Finalize) {
                    VideoRecordEvent.Finalize finalEvent = (VideoRecordEvent.Finalize) event;
                    if (!finalEvent.hasError()) {
                        Log.d("Camera", "Video saved: " + videoFile.getAbsolutePath());
                    }
                }
            });
    }
    
    private void analyzeImage(ImageProxy image) {
        // Example: calculate average brightness
        ImageProxy.PlaneProxy plane = image.getPlanes()[0];
        ByteBuffer buffer = plane.getBuffer();
        byte[] bytes = new byte[buffer.remaining()];
        buffer.get(bytes);
        
        int total = 0;
        for (byte b : bytes) total += (b & 0xFF);
        float avgBrightness = (float) total / bytes.length;
        // Use avgBrightness to auto-adjust exposure, etc.
    }
    
    private void showPhoto(Uri uri) {
        // Display photo in an ImageView
        ImageView preview = requireView().findViewById(R.id.photoPreview);
        Glide.with(this).load(uri).into(preview);
    }
    
    private boolean allPermissionsGranted() {
        for (String permission : REQUIRED_PERMISSIONS) {
            if (ContextCompat.checkSelfPermission(requireContext(), permission)
                    != PackageManager.PERMISSION_GRANTED) return false;
        }
        return true;
    }
}
```

---

## 82.2 MediaPlayer & ExoPlayer

```groovy
// ExoPlayer
implementation "androidx.media3:media3-exoplayer:1.3.1"
implementation "androidx.media3:media3-ui:1.3.1"
```

```java
// VideoPlayerFragment.java
public class VideoPlayerFragment extends Fragment {
    
    private ExoPlayer player;
    private PlayerView playerView;
    
    @Override
    public void onViewCreated(View view, Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        playerView = view.findViewById(R.id.playerView);
    }
    
    @Override
    public void onStart() {
        super.onStart();
        initPlayer();
    }
    
    private void initPlayer() {
        player = new ExoPlayer.Builder(requireContext()).build();
        playerView.setPlayer(player);
        
        // Play from URL
        String videoUrl = "https://example.com/video.mp4";
        MediaItem mediaItem = MediaItem.fromUri(videoUrl);
        
        // Playlist
        MediaItem item1 = MediaItem.fromUri("https://example.com/video1.mp4");
        MediaItem item2 = MediaItem.fromUri("https://example.com/video2.mp4");
        player.setMediaItems(Arrays.asList(item1, item2));
        
        // HLS stream
        MediaItem hlsItem = MediaItem.Builder()
            .setUri("https://example.com/stream.m3u8")
            .setMimeType(MimeTypes.APPLICATION_M3U8)
            .build();
        player.setMediaItem(hlsItem);
        
        player.prepare();
        player.setPlayWhenReady(true);
        
        // Listen to events
        player.addListener(new Player.Listener() {
            @Override
            public void onPlaybackStateChanged(int state) {
                if (state == Player.STATE_ENDED) {
                    // Video finished
                }
            }
            
            @Override
            public void onPlayerError(@NonNull PlaybackException error) {
                Log.e("Player", "Error: " + error.getMessage());
            }
        });
    }
    
    @Override
    public void onStop() {
        super.onStop();
        releasePlayer();
    }
    
    private void releasePlayer() {
        if (player != null) {
            player.release();
            player = null;
        }
    }
}
```

---

## 82.3 สรุป Part 82

ในบทนี้คุณได้เรียนรู้:

✅ CameraX Preview, ImageCapture, VideoCapture  
✅ ImageAnalysis (real-time processing)  
✅ Pinch-to-zoom gesture  
✅ takePicture with OutputFileOptions  
✅ Video recording with AudioEnabled  
✅ ExoPlayer setup (URL, HLS, Playlist)  
✅ Player state listener  

---

*[← Part 81: Android Security](./part-81-android-security.md) | [Part 83: Bluetooth & NFC →](./part-83-android-bluetooth-nfc.md)*
