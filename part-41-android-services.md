# Part 41: Android Services
## หลักสูตร Java & Android Development - ระดับ Intermediate Android

---

## 41.1 Service คืออะไร

Service คือ component ที่ทำงานใน background ไม่มี UI  
ใช้สำหรับ: เล่นเพลง, ดาวน์โหลดไฟล์, sync ข้อมูล

**ประเภท:**
- **Started Service**: เริ่มด้วย `startService()` - ทำงานจนเสร็จ
- **Bound Service**: bind กับ Activity - ส่งผลลัพธ์กลับได้
- **Foreground Service**: แสดง notification - ไม่ถูก kill

---

## 41.2 IntentService (Simple Background Task)

```java
// service/DownloadService.java
package com.example.myapp.service;

import android.app.IntentService;
import android.content.Intent;
import android.os.Bundle;
import android.os.ResultReceiver;
import android.util.Log;

public class DownloadService extends IntentService {
    
    private static final String TAG = "DownloadService";
    public static final String ACTION_DOWNLOAD = "ACTION_DOWNLOAD";
    public static final String EXTRA_URL = "url";
    public static final String EXTRA_RECEIVER = "receiver";
    public static final int STATUS_RUNNING = 1;
    public static final int STATUS_FINISHED = 2;
    public static final int STATUS_ERROR = -1;
    
    public DownloadService() {
        super("DownloadService");
    }
    
    @Override
    protected void onHandleIntent(Intent intent) {
        if (intent == null) return;
        
        String url = intent.getStringExtra(EXTRA_URL);
        ResultReceiver receiver = intent.getParcelableExtra(EXTRA_RECEIVER);
        
        if (url == null) return;
        
        // Report: started
        Bundle bundle = new Bundle();
        bundle.putString("message", "กำลังดาวน์โหลด...");
        if (receiver != null) receiver.send(STATUS_RUNNING, bundle);
        
        try {
            // Simulate download
            Log.d(TAG, "Downloading: " + url);
            Thread.sleep(3000);  // simulate 3 second download
            
            // Report: done
            bundle.putString("message", "ดาวน์โหลดเสร็จแล้ว");
            bundle.putString("filePath", "/sdcard/Download/file.jpg");
            if (receiver != null) receiver.send(STATUS_FINISHED, bundle);
            
        } catch (Exception e) {
            Log.e(TAG, "Error: " + e.getMessage());
            bundle.putString("error", e.getMessage());
            if (receiver != null) receiver.send(STATUS_ERROR, bundle);
        }
    }
}
```

---

## 41.3 Foreground Service (Music Player)

```java
// service/MusicService.java
package com.example.myapp.service;

import android.app.*;
import android.content.*;
import android.media.MediaPlayer;
import android.os.*;
import androidx.core.app.NotificationCompat;

public class MusicService extends Service {
    
    private static final int NOTIF_ID = 1;
    private static final String CHANNEL_ID = "music_channel";
    
    private MediaPlayer mediaPlayer;
    private boolean isPaused = false;
    
    // Binder for bound clients
    private final IBinder binder = new MusicBinder();
    
    public class MusicBinder extends Binder {
        public MusicService getService() { return MusicService.this; }
    }
    
    @Override
    public void onCreate() {
        super.onCreate();
        createNotificationChannel();
    }
    
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        String action = intent != null ? intent.getAction() : null;
        
        if ("ACTION_PLAY".equals(action)) {
            String songUrl = intent.getStringExtra("songUrl");
            play(songUrl);
        } else if ("ACTION_PAUSE".equals(action)) {
            pause();
        } else if ("ACTION_STOP".equals(action)) {
            stopSelf();
        }
        
        // Restart if killed by system
        return START_STICKY;
    }
    
    public void play(String url) {
        if (mediaPlayer != null) {
            mediaPlayer.release();
        }
        
        mediaPlayer = new MediaPlayer();
        try {
            mediaPlayer.setDataSource(url);
            mediaPlayer.prepareAsync();
            mediaPlayer.setOnPreparedListener(mp -> {
                mp.start();
                showNotification("กำลังเล่น", "ชื่อเพลง");
            });
            mediaPlayer.setOnCompletionListener(mp -> stopSelf());
        } catch (Exception e) {
            stopSelf();
        }
    }
    
    public void pause() {
        if (mediaPlayer != null && mediaPlayer.isPlaying()) {
            mediaPlayer.pause();
            isPaused = true;
            showNotification("หยุดชั่วคราว", "ชื่อเพลง");
        }
    }
    
    public void resume() {
        if (mediaPlayer != null && isPaused) {
            mediaPlayer.start();
            isPaused = false;
            showNotification("กำลังเล่น", "ชื่อเพลง");
        }
    }
    
    public boolean isPlaying() {
        return mediaPlayer != null && mediaPlayer.isPlaying();
    }
    
    private void showNotification(String title, String song) {
        // Intent for tap to open app
        Intent openIntent = new Intent(this, MainActivity.class);
        PendingIntent pendingOpen = PendingIntent.getActivity(this, 0, openIntent,
            PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        
        // Pause/Play action
        Intent pauseIntent = new Intent(this, MusicService.class);
        pauseIntent.setAction(isPlaying() ? "ACTION_PAUSE" : "ACTION_PLAY");
        PendingIntent pendingPause = PendingIntent.getService(this, 0, pauseIntent,
            PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        
        // Stop action
        Intent stopIntent = new Intent(this, MusicService.class);
        stopIntent.setAction("ACTION_STOP");
        PendingIntent pendingStop = PendingIntent.getService(this, 1, stopIntent,
            PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        
        Notification notification = new NotificationCompat.Builder(this, CHANNEL_ID)
            .setContentTitle(title)
            .setContentText(song)
            .setSmallIcon(R.drawable.ic_music)
            .setContentIntent(pendingOpen)
            .addAction(R.drawable.ic_pause, isPlaying() ? "หยุด" : "เล่น", pendingPause)
            .addAction(R.drawable.ic_stop, "หยุด", pendingStop)
            .setOngoing(true)
            .build();
        
        // Must call startForeground within 5 seconds of starting service
        startForeground(NOTIF_ID, notification);
    }
    
    private void createNotificationChannel() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            NotificationChannel channel = new NotificationChannel(
                CHANNEL_ID, "เพลง", NotificationManager.IMPORTANCE_LOW);
            channel.setDescription("การเล่นเพลง");
            NotificationManager manager = getSystemService(NotificationManager.class);
            if (manager != null) manager.createNotificationChannel(channel);
        }
    }
    
    @Override
    public IBinder onBind(Intent intent) { return binder; }
    
    @Override
    public void onDestroy() {
        super.onDestroy();
        if (mediaPlayer != null) {
            mediaPlayer.stop();
            mediaPlayer.release();
            mediaPlayer = null;
        }
    }
}
```

---

## 41.4 Using the Service in Activity

```java
// MusicActivity.java
public class MusicActivity extends AppCompatActivity implements ServiceConnection {

    private MusicService musicService;
    private boolean isBound = false;
    private Button btnPlay, btnPause, btnStop;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_music);

        btnPlay  = findViewById(R.id.btnPlay);
        btnPause = findViewById(R.id.btnPause);
        btnStop  = findViewById(R.id.btnStop);

        // Start foreground service
        btnPlay.setOnClickListener(v -> {
            Intent intent = new Intent(this, MusicService.class);
            intent.setAction("ACTION_PLAY");
            intent.putExtra("songUrl", "https://example.com/song.mp3");
            
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
                startForegroundService(intent);  // required for Android 8+
            } else {
                startService(intent);
            }
            bindToService();
        });

        btnPause.setOnClickListener(v -> {
            if (isBound && musicService != null) {
                if (musicService.isPlaying()) musicService.pause();
                else musicService.resume();
            }
        });

        btnStop.setOnClickListener(v -> {
            Intent intent = new Intent(this, MusicService.class);
            stopService(intent);
        });
    }

    private void bindToService() {
        Intent intent = new Intent(this, MusicService.class);
        bindService(intent, this, Context.BIND_AUTO_CREATE);
    }

    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {
        MusicService.MusicBinder binder = (MusicService.MusicBinder) service;
        musicService = binder.getService();
        isBound = true;
    }

    @Override
    public void onServiceDisconnected(ComponentName name) {
        isBound = false;
        musicService = null;
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        if (isBound) {
            unbindService(this);
            isBound = false;
        }
    }
}
```

---

## 41.5 WorkManager (Background Tasks)

```groovy
// build.gradle
implementation 'androidx.work:work-runtime:2.8.1'
```

```java
// worker/SyncWorker.java
import androidx.work.*;

public class SyncWorker extends Worker {
    
    public SyncWorker(Context context, WorkerParameters params) {
        super(context, params);
    }
    
    @Override
    public Result doWork() {
        // Get input data
        String userId = getInputData().getString("userId");
        
        try {
            // Simulate sync operation
            Thread.sleep(2000);
            
            // Return success with output
            Data output = new Data.Builder()
                .putString("syncResult", "Synced " + userId)
                .putLong("syncTime", System.currentTimeMillis())
                .build();
            return Result.success(output);
            
        } catch (Exception e) {
            // Return retry if temporary error
            return Result.retry();
            // Return failure if permanent error
            // return Result.failure();
        }
    }
}

// Schedule work
class WorkScheduler {
    
    static void scheduleOneTimeSync(Context context, String userId) {
        Data inputData = new Data.Builder()
            .putString("userId", userId)
            .build();
        
        OneTimeWorkRequest syncWork = new OneTimeWorkRequest.Builder(SyncWorker.class)
            .setInputData(inputData)
            .setConstraints(new Constraints.Builder()
                .setRequiredNetworkType(NetworkType.CONNECTED)
                .build())
            .setBackoffCriteria(BackoffPolicy.EXPONENTIAL, 15, TimeUnit.MINUTES)
            .build();
        
        WorkManager.getInstance(context)
            .getWorkInfoByIdLiveData(syncWork.getId())
            .observe((LifecycleOwner) context, workInfo -> {
                if (workInfo != null && workInfo.getState().isFinished()) {
                    if (workInfo.getState() == WorkInfo.State.SUCCEEDED) {
                        String result = workInfo.getOutputData().getString("syncResult");
                        Log.d("Work", "Done: " + result);
                    }
                }
            });
        
        WorkManager.getInstance(context).enqueue(syncWork);
    }
    
    // Periodic work (every 1 hour, at least)
    static void schedulePeriodicSync(Context context) {
        PeriodicWorkRequest periodicWork = new PeriodicWorkRequest.Builder(
            SyncWorker.class, 1, TimeUnit.HOURS)
            .setConstraints(new Constraints.Builder()
                .setRequiredNetworkType(NetworkType.CONNECTED)
                .setRequiresBatteryNotLow(true)
                .build())
            .build();
        
        WorkManager.getInstance(context).enqueueUniquePeriodicWork(
            "sync_work",
            ExistingPeriodicWorkPolicy.KEEP,  // don't replace if already scheduled
            periodicWork);
    }
}
```

---

## 41.6 Manifest

```xml
<uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>

<application ...>
    <!-- Services must be declared -->
    <service android:name=".service.MusicService"
        android:exported="false"
        android:foregroundServiceType="mediaPlayback"/>
    
    <service android:name=".service.DownloadService"
        android:exported="false"/>
</application>
```

---

## 41.7 สรุป Part 41

ในบทนี้คุณได้เรียนรู้:

✅ IntentService (simple background task)  
✅ Foreground Service (music player example)  
✅ Bound Service (communicate with Activity)  
✅ startForeground() with notification  
✅ bindService / unbindService  
✅ WorkManager (reliable background work)  
✅ OneTimeWorkRequest vs PeriodicWorkRequest  
✅ Work constraints  

---

*[← Part 40: Notifications](./part-40-android-notifications.md) | [Part 42: Android Permissions →](./part-42-android-permissions.md)*
