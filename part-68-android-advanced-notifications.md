# Part 68: Advanced Notifications & FCM
## หลักสูตร Java & Android Development - ระดับ World-Class Android

---

## 68.1 FCM Data vs Notification Messages

```
Notification message:          Data message:
- แสดง notification อัตโนมัติ  - ส่งถึง app code เสมอ
- เมื่อกด → เปิด app           - จัดการเองใน onMessageReceived
- ปรับแต่ง UI ได้จำกัด         - ปรับแต่ง notification เอง
- เหมาะสำหรับ simple alerts    - เหมาะสำหรับ complex logic
```

---

## 68.2 Advanced FCM Service

```java
// MyFirebaseMessagingService.java
public class MyFirebaseMessagingService extends FirebaseMessagingService {
    
    private static final String TAG = "FCM";
    
    @Override
    public void onNewToken(String token) {
        Log.d(TAG, "New FCM token: " + token);
        sendTokenToServer(token);
    }
    
    @Override
    public void onMessageReceived(RemoteMessage remoteMessage) {
        String from = remoteMessage.getFrom();
        Map<String, String> data = remoteMessage.getData();
        
        // Parse notification type from data
        String type = data.get("type");
        if (type == null) type = "general";
        
        switch (type) {
            case "chat":
                handleChatMessage(remoteMessage, data);
                break;
            case "order_update":
                handleOrderUpdate(remoteMessage, data);
                break;
            case "promotion":
                handlePromotion(remoteMessage, data);
                break;
            default:
                handleGeneralNotification(remoteMessage);
                break;
        }
    }
    
    private void handleChatMessage(RemoteMessage msg, Map<String, String> data) {
        String senderName = data.get("sender_name");
        String message = data.get("message");
        String chatId = data.get("chat_id");
        String avatarUrl = data.get("avatar_url");
        
        // Build messaging style notification
        NotificationCompat.MessagingStyle style = 
            new NotificationCompat.MessagingStyle(new Person.Builder()
                .setName("You").build());
        
        style.addMessage(message,
            System.currentTimeMillis(),
            new Person.Builder().setName(senderName).build());
        
        // Deep link to chat screen
        Intent intent = new Intent(Intent.ACTION_VIEW,
            Uri.parse("myapp://chat/" + chatId));
        PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, intent,
            PendingIntent.FLAG_IMMUTABLE);
        
        NotificationCompat.Builder builder = new NotificationCompat.Builder(this, "messages")
            .setSmallIcon(R.drawable.ic_chat)
            .setStyle(style)
            .setContentIntent(pendingIntent)
            .setAutoCancel(true)
            .setPriority(NotificationCompat.PRIORITY_HIGH)
            .setCategory(NotificationCompat.CATEGORY_MESSAGE);
        
        // Quick reply action
        RemoteInput remoteInput = new RemoteInput.Builder("reply_text")
            .setLabel("ตอบกลับ")
            .build();
        
        Intent replyIntent = new Intent(this, NotificationReplyReceiver.class);
        replyIntent.putExtra("chat_id", chatId);
        PendingIntent replyPendingIntent = PendingIntent.getBroadcast(this, 1, replyIntent,
            PendingIntent.FLAG_MUTABLE);
        
        NotificationCompat.Action replyAction = new NotificationCompat.Action.Builder(
            R.drawable.ic_reply, "ตอบ", replyPendingIntent)
            .addRemoteInput(remoteInput)
            .build();
        
        builder.addAction(replyAction);
        
        NotificationManagerCompat.from(this)
            .notify(chatId.hashCode(), builder.build());
    }
    
    private void handleOrderUpdate(RemoteMessage msg, Map<String, String> data) {
        String orderId     = data.get("order_id");
        String status      = data.get("status");
        String description = data.get("description");
        
        // Determine icon and color by status
        int iconRes;
        int color;
        switch (status != null ? status : "") {
            case "processing": iconRes = R.drawable.ic_processing; color = 0xFF2196F3; break;
            case "shipped":    iconRes = R.drawable.ic_shipping;   color = 0xFF4CAF50; break;
            case "delivered":  iconRes = R.drawable.ic_done;       color = 0xFF4CAF50; break;
            default:           iconRes = R.drawable.ic_info;       color = 0xFF9E9E9E; break;
        }
        
        Intent intent = new Intent(this, OrderDetailActivity.class);
        intent.putExtra("order_id", orderId);
        PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, intent,
            PendingIntent.FLAG_IMMUTABLE | PendingIntent.FLAG_UPDATE_CURRENT);
        
        Notification notification = new NotificationCompat.Builder(this, "orders")
            .setSmallIcon(iconRes)
            .setColor(color)
            .setContentTitle("อัปเดตคำสั่งซื้อ #" + orderId)
            .setContentText(description)
            .setContentIntent(pendingIntent)
            .setAutoCancel(true)
            .build();
        
        NotificationManagerCompat.from(this).notify(orderId.hashCode(), notification);
    }
    
    private void handleGeneralNotification(RemoteMessage msg) {
        RemoteMessage.Notification notification = msg.getNotification();
        if (notification == null) return;
        
        NotificationHelper.getInstance(this).showSimpleNotification(
            notification.getTitle(), notification.getBody());
    }
    
    private void sendTokenToServer(String token) {
        // Send to your backend
        SharedPreferences prefs = PreferenceManager.getDefaultSharedPreferences(this);
        prefs.edit().putString("fcm_token", token).apply();
        // Also send via API if user is logged in
    }
}
```

---

## 68.3 Notification Reply Receiver

```java
// NotificationReplyReceiver.java
public class NotificationReplyReceiver extends BroadcastReceiver {
    
    @Override
    public void onReceive(Context context, Intent intent) {
        CharSequence replyText = RemoteInput.getResultsFromIntent(intent)
            .getCharSequence("reply_text");
        
        String chatId = intent.getStringExtra("chat_id");
        
        if (replyText != null && !replyText.toString().isEmpty()) {
            // Send the reply via your chat service
            sendReply(context, chatId, replyText.toString());
            
            // Update notification to show message was sent
            NotificationCompat.Builder builder = new NotificationCompat.Builder(context, "messages")
                .setSmallIcon(R.drawable.ic_chat)
                .setContentText("ส่งข้อความแล้ว")
                .setAutoCancel(true);
            
            NotificationManagerCompat.from(context)
                .notify(chatId.hashCode(), builder.build());
        }
    }
    
    private void sendReply(Context context, String chatId, String message) {
        // Post to API (use WorkManager for reliability)
        Data inputData = new Data.Builder()
            .putString("chat_id", chatId)
            .putString("message", message)
            .build();
        
        OneTimeWorkRequest work = new OneTimeWorkRequest.Builder(SendMessageWorker.class)
            .setInputData(inputData)
            .setConstraints(new Constraints.Builder()
                .setRequiredNetworkType(NetworkType.CONNECTED)
                .build())
            .build();
        
        WorkManager.getInstance(context).enqueue(work);
    }
}
```

---

## 68.4 FCM Topic Subscription

```java
// NotificationTopicManager.java
public class NotificationTopicManager {
    
    // Subscribe to topic (all users receive)
    public static void subscribeToTopic(String topic) {
        FirebaseMessaging.getInstance().subscribeToTopic(topic)
            .addOnCompleteListener(task -> {
                if (!task.isSuccessful()) {
                    Log.w("FCM", "Subscribe to " + topic + " failed");
                }
            });
    }
    
    // Unsubscribe from topic
    public static void unsubscribeFromTopic(String topic) {
        FirebaseMessaging.getInstance().unsubscribeFromTopic(topic);
    }
    
    // Common topics
    public static void subscribeToUserTopics(User user) {
        subscribeToTopic("all_users");
        subscribeToTopic("lang_" + user.getLanguage());  // lang_th, lang_en
        if (user.isPremium()) subscribeToTopic("premium_users");
        subscribeToTopic("region_" + user.getRegion());
    }
    
    public static void unsubscribeAll() {
        unsubscribeFromTopic("all_users");
        unsubscribeFromTopic("premium_users");
    }
}
```

---

## 68.5 สรุป Part 68

ในบทนี้คุณได้เรียนรู้:

✅ FCM Data vs Notification messages  
✅ Handle multiple notification types (chat, order, promo)  
✅ MessagingStyle notification (chat bubbles)  
✅ Direct Reply action (ตอบกลับจาก notification)  
✅ NotificationReplyReceiver  
✅ WorkManager สำหรับ send reply reliably  
✅ FCM Topic subscription  

---

*[← Part 67: Analytics](./part-67-android-analytics.md) | [Part 69: Offline-First Architecture →](./part-69-android-offline-first.md)*
