# Part 48: Firebase (Realtime Database, FCM, Auth)
## หลักสูตร Java & Android Development - ระดับ Advanced Android

---

## 48.1 Firebase Setup

```groovy
// project-level build.gradle
buildscript {
    dependencies {
        classpath 'com.google.gms:google-services:4.4.0'
    }
}

// app-level build.gradle
plugins {
    id 'com.google.gms.google-services'
}

dependencies {
    // Firebase BoM (handles version compatibility)
    implementation platform('com.google.firebase:firebase-bom:32.7.0')
    
    implementation 'com.google.firebase:firebase-auth'
    implementation 'com.google.firebase:firebase-database'
    implementation 'com.google.firebase:firebase-firestore'
    implementation 'com.google.firebase:firebase-messaging'
    implementation 'com.google.firebase:firebase-storage'
    implementation 'com.google.firebase:firebase-analytics'
}
```

ดาวน์โหลด `google-services.json` จาก Firebase Console และวางไว้ใน `app/` directory

---

## 48.2 Firebase Authentication

```java
// AuthManager.java
import com.google.firebase.auth.*;

public class AuthManager {
    
    private final FirebaseAuth auth;
    
    public AuthManager() {
        auth = FirebaseAuth.getInstance();
    }
    
    // Get current user
    public FirebaseUser getCurrentUser() {
        return auth.getCurrentUser();
    }
    
    public boolean isLoggedIn() {
        return auth.getCurrentUser() != null;
    }
    
    // Sign up with email/password
    public void signUp(String email, String password, AuthCallback callback) {
        auth.createUserWithEmailAndPassword(email, password)
            .addOnCompleteListener(task -> {
                if (task.isSuccessful()) {
                    callback.onSuccess(auth.getCurrentUser());
                } else {
                    callback.onError(task.getException());
                }
            });
    }
    
    // Sign in
    public void signIn(String email, String password, AuthCallback callback) {
        auth.signInWithEmailAndPassword(email, password)
            .addOnCompleteListener(task -> {
                if (task.isSuccessful()) {
                    callback.onSuccess(auth.getCurrentUser());
                } else {
                    callback.onError(task.getException());
                }
            });
    }
    
    // Sign out
    public void signOut() {
        auth.signOut();
    }
    
    // Reset password
    public void sendPasswordReset(String email, AuthCallback callback) {
        auth.sendPasswordResetEmail(email)
            .addOnCompleteListener(task -> {
                if (task.isSuccessful()) callback.onSuccess(null);
                else callback.onError(task.getException());
            });
    }
    
    // Update display name
    public void updateDisplayName(String name, AuthCallback callback) {
        FirebaseUser user = auth.getCurrentUser();
        if (user == null) { callback.onError(new Exception("Not logged in")); return; }
        
        UserProfileChangeRequest profile = new UserProfileChangeRequest.Builder()
            .setDisplayName(name)
            .build();
        
        user.updateProfile(profile).addOnCompleteListener(task -> {
            if (task.isSuccessful()) callback.onSuccess(user);
            else callback.onError(task.getException());
        });
    }
    
    // Get ID token (for server verification)
    public void getIdToken(TokenCallback callback) {
        FirebaseUser user = auth.getCurrentUser();
        if (user == null) { callback.onToken(null); return; }
        
        user.getIdToken(true)
            .addOnSuccessListener(result -> callback.onToken(result.getToken()))
            .addOnFailureListener(e -> callback.onToken(null));
    }
    
    public interface AuthCallback {
        void onSuccess(FirebaseUser user);
        void onError(Exception e);
    }
    
    public interface TokenCallback {
        void onToken(String token);
    }
}
```

---

## 48.3 Firebase Realtime Database

```java
// RealtimeDatabaseHelper.java
import com.google.firebase.database.*;

public class RealtimeDatabaseHelper {
    
    private final DatabaseReference db;
    
    public RealtimeDatabaseHelper() {
        db = FirebaseDatabase.getInstance().getReference();
    }
    
    // Write data
    public void writeUser(String userId, UserModel user) {
        db.child("users").child(userId).setValue(user)
            .addOnSuccessListener(unused -> Log.d("DB", "User saved"))
            .addOnFailureListener(e -> Log.e("DB", "Error: " + e.getMessage()));
    }
    
    // Read once
    public void readUser(String userId, UserCallback callback) {
        db.child("users").child(userId).get()
            .addOnCompleteListener(task -> {
                if (task.isSuccessful()) {
                    DataSnapshot snapshot = task.getResult();
                    UserModel user = snapshot.getValue(UserModel.class);
                    callback.onUser(user);
                } else {
                    callback.onUser(null);
                }
            });
    }
    
    // Listen for realtime changes
    public ValueEventListener listenToMessages(String chatId,
            DataCallback<List<Message>> callback) {
        
        ValueEventListener listener = new ValueEventListener() {
            @Override
            public void onDataChange(DataSnapshot snapshot) {
                List<Message> messages = new ArrayList<>();
                for (DataSnapshot child : snapshot.getChildren()) {
                    Message msg = child.getValue(Message.class);
                    if (msg != null) messages.add(msg);
                }
                callback.onData(messages);
            }
            
            @Override
            public void onCancelled(DatabaseError error) {
                Log.e("DB", "Error: " + error.getMessage());
            }
        };
        
        db.child("chats").child(chatId).child("messages")
            .addValueEventListener(listener);
        
        return listener;  // save reference to remove later
    }
    
    // Stop listening
    public void removeListener(String chatId, ValueEventListener listener) {
        db.child("chats").child(chatId).child("messages")
            .removeEventListener(listener);
    }
    
    // Send message (push = auto-generate key)
    public void sendMessage(String chatId, Message message) {
        String key = db.child("chats").child(chatId).child("messages").push().getKey();
        if (key == null) return;
        
        message.setId(key);
        message.setTimestamp(ServerValue.TIMESTAMP);  // server timestamp
        
        db.child("chats").child(chatId).child("messages").child(key)
            .setValue(message);
    }
    
    // Update specific field
    public void updateField(String path, String field, Object value) {
        db.child(path).child(field).setValue(value);
    }
    
    // Batch update
    public void batchUpdate(Map<String, Object> updates) {
        db.updateChildren(updates);
    }
    
    // Delete
    public void delete(String path) {
        db.child(path).removeValue();
    }
    
    // Query (order + filter)
    public void getTopUsers(int limit, DataCallback<List<UserModel>> callback) {
        db.child("users")
            .orderByChild("score")
            .limitToLast(limit)  // top N
            .addListenerForSingleValueEvent(new ValueEventListener() {
                @Override
                public void onDataChange(DataSnapshot snapshot) {
                    List<UserModel> users = new ArrayList<>();
                    for (DataSnapshot child : snapshot.getChildren()) {
                        UserModel u = child.getValue(UserModel.class);
                        if (u != null) users.add(0, u);  // reverse order
                    }
                    callback.onData(users);
                }
                
                @Override
                public void onCancelled(DatabaseError error) {}
            });
    }
    
    public interface UserCallback {
        void onUser(UserModel user);
    }
    
    public interface DataCallback<T> {
        void onData(T data);
    }
    
    // Models
    public static class UserModel {
        public String uid, name, email;
        public long score;
        public UserModel() {}
        public UserModel(String uid, String name, String email) {
            this.uid = uid;
            this.name = name;
            this.email = email;
        }
    }
    
    public static class Message {
        public String id, senderId, text;
        public Object timestamp;
        public Message() {}
        public Message(String senderId, String text) {
            this.senderId = senderId;
            this.text = text;
        }
        public void setId(String id) { this.id = id; }
        public void setTimestamp(Object ts) { this.timestamp = ts; }
    }
}
```

---

## 48.4 Firebase Cloud Messaging (FCM)

```java
// MyFirebaseMessagingService.java
import com.google.firebase.messaging.*;

public class MyFirebaseMessagingService extends FirebaseMessagingService {
    
    @Override
    public void onNewToken(String token) {
        super.onNewToken(token);
        // Save token to server
        Log.d("FCM", "New token: " + token);
        sendTokenToServer(token);
    }
    
    @Override
    public void onMessageReceived(RemoteMessage message) {
        super.onMessageReceived(message);
        
        String title = "";
        String body  = "";
        
        // Notification payload (from Firebase Console)
        if (message.getNotification() != null) {
            title = message.getNotification().getTitle();
            body  = message.getNotification().getBody();
        }
        
        // Data payload
        Map<String, String> data = message.getData();
        if (!data.isEmpty()) {
            String type     = data.get("type");
            String targetId = data.get("targetId");
            // handle based on type
        }
        
        // Show notification
        showNotification(title, body, data);
    }
    
    private void showNotification(String title, String body, Map<String, String> data) {
        NotificationHelper helper = new NotificationHelper(this);
        helper.showSimpleNotification(title, body);
    }
    
    private void sendTokenToServer(String token) {
        // Send to your backend API
        // (use Retrofit or HttpURLConnection)
    }
    
    // Get current FCM token
    public static void getCurrentToken(TokenCallback callback) {
        FirebaseMessaging.getInstance().getToken()
            .addOnCompleteListener(task -> {
                if (task.isSuccessful()) {
                    callback.onToken(task.getResult());
                } else {
                    callback.onToken(null);
                }
            });
    }
    
    // Subscribe to topic
    public static void subscribeToTopic(String topic) {
        FirebaseMessaging.getInstance().subscribeToTopic(topic)
            .addOnCompleteListener(task -> {
                Log.d("FCM", "Subscribed to " + topic + ": " + task.isSuccessful());
            });
    }
    
    public interface TokenCallback {
        void onToken(String token);
    }
}
```

```xml
<!-- AndroidManifest.xml -->
<service
    android:name=".MyFirebaseMessagingService"
    android:exported="false">
    <intent-filter>
        <action android:name="com.google.firebase.MESSAGING_EVENT"/>
    </intent-filter>
</service>
```

---

## 48.5 Firebase Storage (Upload/Download Files)

```java
// StorageHelper.java
import com.google.firebase.storage.*;

public class StorageHelper {
    
    private final StorageReference storageRef;
    
    public StorageHelper() {
        storageRef = FirebaseStorage.getInstance().getReference();
    }
    
    // Upload image
    public void uploadImage(Uri imageUri, String userId, UploadCallback callback) {
        String filename = "users/" + userId + "/profile.jpg";
        StorageReference fileRef = storageRef.child(filename);
        
        UploadTask uploadTask = fileRef.putFile(imageUri);
        
        uploadTask.addOnProgressListener(snapshot -> {
            double progress = 100.0 * snapshot.getBytesTransferred() / snapshot.getTotalByteCount();
            callback.onProgress((int) progress);
        });
        
        uploadTask.addOnSuccessListener(snapshot -> {
            // Get download URL
            fileRef.getDownloadUrl().addOnSuccessListener(uri -> {
                callback.onSuccess(uri.toString());
            });
        });
        
        uploadTask.addOnFailureListener(e -> {
            callback.onFailure(e.getMessage());
        });
    }
    
    // Download as bitmap
    public void downloadImage(String storagePath, ImageCallback callback) {
        final long MAX_BYTES = 5 * 1024 * 1024;  // 5MB
        
        storageRef.child(storagePath).getBytes(MAX_BYTES)
            .addOnSuccessListener(bytes -> {
                Bitmap bitmap = BitmapFactory.decodeByteArray(bytes, 0, bytes.length);
                callback.onImage(bitmap);
            })
            .addOnFailureListener(e -> callback.onImage(null));
    }
    
    // Delete file
    public void deleteFile(String storagePath) {
        storageRef.child(storagePath).delete()
            .addOnSuccessListener(unused -> Log.d("Storage", "Deleted"))
            .addOnFailureListener(e -> Log.e("Storage", "Delete failed: " + e.getMessage()));
    }
    
    public interface UploadCallback {
        void onProgress(int percent);
        void onSuccess(String downloadUrl);
        void onFailure(String error);
    }
    
    public interface ImageCallback {
        void onImage(Bitmap bitmap);
    }
}
```

---

## 48.6 สรุป Part 48

ในบทนี้คุณได้เรียนรู้:

✅ Firebase project setup (google-services.json)  
✅ Firebase Authentication (sign up, sign in, sign out)  
✅ Reset password, update profile  
✅ Firebase Realtime Database (read, write, listen)  
✅ Push messages (auto-generated key)  
✅ Database queries (orderByChild, limitToLast)  
✅ FCM (receive notifications, token management, topics)  
✅ Firebase Storage (upload, download, delete)  

---

*[← Part 47: Maps](./part-47-android-maps.md) | [Part 49: Jetpack Navigation →](./part-49-android-navigation.md)*
