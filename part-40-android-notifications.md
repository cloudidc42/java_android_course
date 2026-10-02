# Part 40: Android Notifications
## หลักสูตร Java & Android Development - ระดับ Intermediate Android

---

## 40.1 Notification พื้นฐาน

```java
// util/NotificationHelper.java
package com.example.myapp.util;

import android.app.*;
import android.content.*;
import android.content.pm.PackageManager;
import android.graphics.BitmapFactory;
import android.os.Build;
import androidx.core.app.*;
import androidx.core.content.ContextCompat;

public class NotificationHelper {
    
    // Channel IDs
    public static final String CHANNEL_GENERAL  = "channel_general";
    public static final String CHANNEL_MESSAGES = "channel_messages";
    public static final String CHANNEL_ALERTS   = "channel_alerts";
    
    // Notification IDs
    public static final int NOTIF_GENERAL = 1000;
    public static final int NOTIF_MESSAGE = 1001;
    public static final int NOTIF_ALERT   = 1002;
    
    private final Context context;
    private final NotificationManager manager;
    
    public NotificationHelper(Context context) {
        this.context = context;
        this.manager = (NotificationManager) context.getSystemService(Context.NOTIFICATION_SERVICE);
        createChannels();
    }
    
    // Create notification channels (Android 8+)
    private void createChannels() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            
            // General channel
            NotificationChannel general = new NotificationChannel(
                CHANNEL_GENERAL,
                "การแจ้งเตือนทั่วไป",
                NotificationManager.IMPORTANCE_DEFAULT);
            general.setDescription("การแจ้งเตือนทั่วไปของแอป");
            
            // Messages channel (high importance)
            NotificationChannel messages = new NotificationChannel(
                CHANNEL_MESSAGES,
                "ข้อความ",
                NotificationManager.IMPORTANCE_HIGH);
            messages.setDescription("การแจ้งเตือนข้อความใหม่");
            messages.enableLights(true);
            messages.enableVibration(true);
            messages.setLockscreenVisibility(Notification.VISIBILITY_PRIVATE);
            
            // Alerts channel
            NotificationChannel alerts = new NotificationChannel(
                CHANNEL_ALERTS,
                "การแจ้งเตือนสำคัญ",
                NotificationManager.IMPORTANCE_MAX);
            alerts.setDescription("การแจ้งเตือนสำคัญและฉุกเฉิน");
            
            manager.createNotificationChannel(general);
            manager.createNotificationChannel(messages);
            manager.createNotificationChannel(alerts);
        }
    }
    
    // Simple notification
    public void showSimpleNotification(String title, String content) {
        NotificationCompat.Builder builder = new NotificationCompat.Builder(context, CHANNEL_GENERAL)
            .setSmallIcon(R.drawable.ic_notification)
            .setContentTitle(title)
            .setContentText(content)
            .setPriority(NotificationCompat.PRIORITY_DEFAULT)
            .setAutoCancel(true);  // dismiss on click
        
        manager.notify(NOTIF_GENERAL, builder.build());
    }
    
    // Notification with tap action
    public void showClickableNotification(String title, String content, Class<?> targetActivity) {
        Intent intent = new Intent(context, targetActivity);
        intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_CLEAR_TASK);
        
        PendingIntent pendingIntent = PendingIntent.getActivity(
            context, 0, intent,
            PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        
        NotificationCompat.Builder builder = new NotificationCompat.Builder(context, CHANNEL_GENERAL)
            .setSmallIcon(R.drawable.ic_notification)
            .setContentTitle(title)
            .setContentText(content)
            .setPriority(NotificationCompat.PRIORITY_DEFAULT)
            .setContentIntent(pendingIntent)
            .setAutoCancel(true);
        
        manager.notify(NOTIF_GENERAL, builder.build());
    }
    
    // Big text notification
    public void showBigTextNotification(String title, String content, String bigText) {
        NotificationCompat.Builder builder = new NotificationCompat.Builder(context, CHANNEL_GENERAL)
            .setSmallIcon(R.drawable.ic_notification)
            .setContentTitle(title)
            .setContentText(content)
            .setPriority(NotificationCompat.PRIORITY_DEFAULT)
            .setStyle(new NotificationCompat.BigTextStyle()
                .bigText(bigText)
                .setBigContentTitle(title)
                .setSummaryText("อ่านเพิ่มเติม"))
            .setAutoCancel(true);
        
        manager.notify(NOTIF_GENERAL, builder.build());
    }
    
    // Notification with action buttons
    public void showActionNotification(String title, String content) {
        // Action intents
        Intent acceptIntent = new Intent(context, NotificationReceiver.class);
        acceptIntent.setAction("ACTION_ACCEPT");
        PendingIntent acceptPending = PendingIntent.getBroadcast(
            context, 0, acceptIntent,
            PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        
        Intent declineIntent = new Intent(context, NotificationReceiver.class);
        declineIntent.setAction("ACTION_DECLINE");
        PendingIntent declinePending = PendingIntent.getBroadcast(
            context, 1, declineIntent,
            PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        
        NotificationCompat.Builder builder = new NotificationCompat.Builder(context, CHANNEL_MESSAGES)
            .setSmallIcon(R.drawable.ic_notification)
            .setContentTitle(title)
            .setContentText(content)
            .setPriority(NotificationCompat.PRIORITY_HIGH)
            .addAction(R.drawable.ic_check, "ยอมรับ", acceptPending)
            .addAction(R.drawable.ic_close, "ปฏิเสธ", declinePending)
            .setAutoCancel(true);
        
        manager.notify(NOTIF_MESSAGE, builder.build());
    }
    
    // Progress notification
    public void showProgressNotification(String title, int progress, boolean indeterminate) {
        NotificationCompat.Builder builder = new NotificationCompat.Builder(context, CHANNEL_GENERAL)
            .setSmallIcon(R.drawable.ic_download)
            .setContentTitle(title)
            .setContentText(indeterminate ? "กำลังดาวน์โหลด..." : progress + "%")
            .setPriority(NotificationCompat.PRIORITY_LOW)
            .setOngoing(true)  // can't be dismissed
            .setProgress(100, progress, indeterminate);
        
        manager.notify(NOTIF_GENERAL, builder.build());
    }
    
    // Cancel notification
    public void cancelNotification(int notifId) {
        manager.cancel(notifId);
    }
    
    public void cancelAll() {
        manager.cancelAll();
    }
    
    // Check permission (Android 13+)
    public boolean hasNotificationPermission() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
            return ContextCompat.checkSelfPermission(context,
                android.Manifest.permission.POST_NOTIFICATIONS) ==
                PackageManager.PERMISSION_GRANTED;
        }
        return true;
    }
}
```

---

## 40.2 Request Notification Permission (Android 13+)

```java
// In Activity
public class MainActivity extends AppCompatActivity {

    private NotificationHelper notifHelper;
    
    private final ActivityResultLauncher<String> requestPermissionLauncher =
        registerForActivityResult(
            new ActivityResultContracts.RequestPermission(),
            granted -> {
                if (granted) {
                    Toast.makeText(this, "รับการแจ้งเตือนได้แล้ว", Toast.LENGTH_SHORT).show();
                } else {
                    Toast.makeText(this, "ไม่ได้รับอนุญาตส่งการแจ้งเตือน",
                        Toast.LENGTH_SHORT).show();
                }
            });

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        notifHelper = new NotificationHelper(this);
        
        // Request permission on Android 13+
        if (!notifHelper.hasNotificationPermission()) {
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
                requestPermissionLauncher.launch(
                    android.Manifest.permission.POST_NOTIFICATIONS);
            }
        }
        
        // Test notifications
        findViewById(R.id.btnSimple).setOnClickListener(v -> {
            notifHelper.showSimpleNotification(
                "สวัสดี!", "นี่คือการแจ้งเตือนทดสอบ");
        });
        
        findViewById(R.id.btnAction).setOnClickListener(v -> {
            notifHelper.showActionNotification(
                "คำขอเป็นเพื่อน", "สมชาย ขอเป็นเพื่อนกับคุณ");
        });
        
        // Download progress demo
        findViewById(R.id.btnProgress).setOnClickListener(v -> {
            simulateDownload();
        });
    }
    
    private void simulateDownload() {
        new Thread(() -> {
            for (int i = 0; i <= 100; i += 10) {
                final int progress = i;
                notifHelper.showProgressNotification("กำลังดาวน์โหลด...", progress, false);
                try { Thread.sleep(500); } catch (InterruptedException e) {}
            }
            // Done
            runOnUiThread(() -> {
                notifHelper.cancelNotification(NotificationHelper.NOTIF_GENERAL);
                notifHelper.showSimpleNotification("ดาวน์โหลดเสร็จแล้ว", "ไฟล์ถูกบันทึกแล้ว");
            });
        }).start();
    }
}
```

---

## 40.3 BroadcastReceiver

```java
// NotificationReceiver.java
package com.example.myapp;

import android.content.*;
import android.widget.Toast;
import androidx.core.app.NotificationManagerCompat;

public class NotificationReceiver extends BroadcastReceiver {
    
    @Override
    public void onReceive(Context context, Intent intent) {
        String action = intent.getAction();
        
        if ("ACTION_ACCEPT".equals(action)) {
            Toast.makeText(context, "ยอมรับแล้ว", Toast.LENGTH_SHORT).show();
            // Handle accept logic
        } else if ("ACTION_DECLINE".equals(action)) {
            Toast.makeText(context, "ปฏิเสธแล้ว", Toast.LENGTH_SHORT).show();
            // Handle decline logic
        }
        
        // Cancel the notification
        NotificationManagerCompat.from(context).cancel(NotificationHelper.NOTIF_MESSAGE);
    }
}
```

```xml
<!-- AndroidManifest.xml -->
<uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>

<receiver android:name=".NotificationReceiver"
    android:exported="false">
    <intent-filter>
        <action android:name="ACTION_ACCEPT"/>
        <action android:name="ACTION_DECLINE"/>
    </intent-filter>
</receiver>
```

---

## 40.4 Scheduled Notification (AlarmManager)

```java
// util/NotificationScheduler.java
import android.app.*;
import android.content.*;
import android.os.Build;
import java.util.Calendar;

public class NotificationScheduler {
    
    private Context context;
    private AlarmManager alarmManager;
    
    public NotificationScheduler(Context context) {
        this.context = context;
        this.alarmManager = (AlarmManager) context.getSystemService(Context.ALARM_SERVICE);
    }
    
    // Schedule one-time notification
    public void scheduleNotification(long triggerTime, String title, String message, int requestCode) {
        Intent intent = new Intent(context, ScheduledNotificationReceiver.class);
        intent.putExtra("title", title);
        intent.putExtra("message", message);
        
        PendingIntent pendingIntent = PendingIntent.getBroadcast(
            context, requestCode, intent,
            PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            alarmManager.setExactAndAllowWhileIdle(
                AlarmManager.RTC_WAKEUP, triggerTime, pendingIntent);
        } else {
            alarmManager.setExact(AlarmManager.RTC_WAKEUP, triggerTime, pendingIntent);
        }
    }
    
    // Schedule daily reminder at specific time
    public void scheduleDailyReminder(int hour, int minute, String title, String message) {
        Calendar calendar = Calendar.getInstance();
        calendar.set(Calendar.HOUR_OF_DAY, hour);
        calendar.set(Calendar.MINUTE, minute);
        calendar.set(Calendar.SECOND, 0);
        
        // If time already passed today, schedule for tomorrow
        if (calendar.getTimeInMillis() < System.currentTimeMillis()) {
            calendar.add(Calendar.DAY_OF_YEAR, 1);
        }
        
        Intent intent = new Intent(context, ScheduledNotificationReceiver.class);
        intent.putExtra("title", title);
        intent.putExtra("message", message);
        
        PendingIntent pendingIntent = PendingIntent.getBroadcast(
            context, 999, intent,
            PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        
        // Repeating alarm
        alarmManager.setRepeating(
            AlarmManager.RTC_WAKEUP,
            calendar.getTimeInMillis(),
            AlarmManager.INTERVAL_DAY,
            pendingIntent);
    }
    
    // Cancel scheduled notification
    public void cancelNotification(int requestCode) {
        Intent intent = new Intent(context, ScheduledNotificationReceiver.class);
        PendingIntent pendingIntent = PendingIntent.getBroadcast(
            context, requestCode, intent,
            PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE);
        alarmManager.cancel(pendingIntent);
    }
}

// ScheduledNotificationReceiver.java
public class ScheduledNotificationReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        String title = intent.getStringExtra("title");
        String message = intent.getStringExtra("message");
        
        NotificationHelper helper = new NotificationHelper(context);
        helper.showSimpleNotification(title, message);
    }
}
```

---

## 40.5 สรุป Part 40

ในบทนี้คุณได้เรียนรู้:

✅ Notification channels (Android 8+)  
✅ Simple, big text notifications  
✅ Action buttons in notifications  
✅ Progress notifications  
✅ Permission request (Android 13+)  
✅ BroadcastReceiver for actions  
✅ AlarmManager for scheduled notifications  

---

*[← Part 39: SharedPreferences](./part-39-android-preferences.md) | [Part 41: Android Services →](./part-41-android-services.md)*
