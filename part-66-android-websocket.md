# Part 66: WebSocket & Real-time Communication
## หลักสูตร Java & Android Development - ระดับ World-Class Android

---

## 66.1 WebSocket คืออะไร

```
HTTP (Request/Response):     WebSocket (Full-duplex):
Client → Request → Server    Client ⇌ Server (persistent connection)
Client ← Response ← Server  ข้อมูลส่งได้ทั้งสองทิศทาง
                             เหมาะสำหรับ: chat, live scores, trading
```

---

## 66.2 OkHttp WebSocket

```java
// ChatWebSocketManager.java
public class ChatWebSocketManager {
    
    private final OkHttpClient client;
    private WebSocket webSocket;
    private WebSocketListener currentListener;
    
    public interface ChatListener {
        void onConnected();
        void onMessageReceived(String message);
        void onError(String error);
        void onDisconnected(int code, String reason);
    }
    
    public ChatWebSocketManager(OkHttpClient client) {
        this.client = client;
    }
    
    public void connect(String url, ChatListener listener) {
        Request request = new Request.Builder().url(url).build();
        
        currentListener = new WebSocketListener() {
            @Override
            public void onOpen(WebSocket ws, Response response) {
                webSocket = ws;
                new Handler(Looper.getMainLooper()).post(listener::onConnected);
            }
            
            @Override
            public void onMessage(WebSocket ws, String text) {
                new Handler(Looper.getMainLooper()).post(() -> 
                    listener.onMessageReceived(text));
            }
            
            @Override
            public void onMessage(WebSocket ws, ByteString bytes) {
                // Binary message
                new Handler(Looper.getMainLooper()).post(() ->
                    listener.onMessageReceived(bytes.utf8()));
            }
            
            @Override
            public void onClosing(WebSocket ws, int code, String reason) {
                ws.close(1000, null);
                new Handler(Looper.getMainLooper()).post(() ->
                    listener.onDisconnected(code, reason));
            }
            
            @Override
            public void onFailure(WebSocket ws, Throwable t, Response response) {
                new Handler(Looper.getMainLooper()).post(() ->
                    listener.onError(t.getMessage()));
            }
        };
        
        webSocket = client.newWebSocket(request, currentListener);
    }
    
    public boolean sendMessage(String message) {
        if (webSocket != null) {
            return webSocket.send(message);
        }
        return false;
    }
    
    public boolean sendJson(Object data) {
        String json = new Gson().toJson(data);
        return sendMessage(json);
    }
    
    public void disconnect() {
        if (webSocket != null) {
            webSocket.close(1000, "User disconnected");
            webSocket = null;
        }
    }
    
    public boolean isConnected() {
        return webSocket != null;
    }
}
```

---

## 66.3 Chat Protocol with JSON

```java
// ChatMessage.java
public class ChatMessage {
    
    public enum Type { JOIN, LEAVE, MESSAGE, TYPING, READ_RECEIPT }
    
    @SerializedName("type")      public Type   type;
    @SerializedName("from")      public String from;
    @SerializedName("to")        public String to;
    @SerializedName("content")   public String content;
    @SerializedName("timestamp") public long   timestamp;
    @SerializedName("id")        public String id;
    
    public static ChatMessage textMessage(String from, String to, String content) {
        ChatMessage msg = new ChatMessage();
        msg.type = Type.MESSAGE;
        msg.from = from;
        msg.to = to;
        msg.content = content;
        msg.timestamp = System.currentTimeMillis();
        msg.id = UUID.randomUUID().toString();
        return msg;
    }
    
    public static ChatMessage joinMessage(String userId) {
        ChatMessage msg = new ChatMessage();
        msg.type = Type.JOIN;
        msg.from = userId;
        msg.timestamp = System.currentTimeMillis();
        return msg;
    }
}

// ChatViewModel.java
@HiltViewModel
public class ChatViewModel extends ViewModel {
    
    private final ChatWebSocketManager wsManager;
    private final Gson gson = new Gson();
    
    private final MutableLiveData<List<ChatMessage>> messages = 
        new MutableLiveData<>(new ArrayList<>());
    private final MutableLiveData<Boolean> connected = new MutableLiveData<>(false);
    private final MutableLiveData<String> error = new MutableLiveData<>();
    
    @Inject
    public ChatViewModel(ChatWebSocketManager wsManager) {
        this.wsManager = wsManager;
    }
    
    public void connectToRoom(String roomId, String userId) {
        String wsUrl = "wss://chat.example.com/ws/room/" + roomId;
        
        wsManager.connect(wsUrl, new ChatWebSocketManager.ChatListener() {
            @Override
            public void onConnected() {
                connected.setValue(true);
                // Send join message
                wsManager.sendJson(ChatMessage.joinMessage(userId));
            }
            
            @Override
            public void onMessageReceived(String message) {
                try {
                    ChatMessage chatMsg = gson.fromJson(message, ChatMessage.class);
                    List<ChatMessage> current = messages.getValue();
                    if (current != null) {
                        List<ChatMessage> updated = new ArrayList<>(current);
                        updated.add(chatMsg);
                        messages.setValue(updated);
                    }
                } catch (Exception e) {
                    Log.e("Chat", "Failed to parse message", e);
                }
            }
            
            @Override
            public void onError(String errorMsg) {
                error.setValue(errorMsg);
                connected.setValue(false);
            }
            
            @Override
            public void onDisconnected(int code, String reason) {
                connected.setValue(false);
            }
        });
    }
    
    public void sendMessage(String userId, String roomId, String content) {
        ChatMessage msg = ChatMessage.textMessage(userId, roomId, content);
        wsManager.sendJson(msg);
        
        // Optimistically add to UI
        List<ChatMessage> current = messages.getValue();
        if (current != null) {
            List<ChatMessage> updated = new ArrayList<>(current);
            updated.add(msg);
            messages.setValue(updated);
        }
    }
    
    public LiveData<List<ChatMessage>> getMessages() { return messages; }
    public LiveData<Boolean>          isConnected()  { return connected; }
    public LiveData<String>           getError()     { return error; }
    
    @Override
    protected void onCleared() {
        wsManager.disconnect();
    }
}
```

---

## 66.4 Reconnection Logic

```java
// ReconnectingWebSocketManager.java
public class ReconnectingWebSocketManager {
    
    private static final int MAX_RECONNECT_ATTEMPTS = 5;
    private static final long INITIAL_DELAY_MS = 1000;
    
    private final ChatWebSocketManager wsManager;
    private final Handler handler = new Handler(Looper.getMainLooper());
    
    private int reconnectAttempts = 0;
    private String currentUrl;
    private ChatWebSocketManager.ChatListener currentListener;
    
    public ReconnectingWebSocketManager(ChatWebSocketManager wsManager) {
        this.wsManager = wsManager;
    }
    
    public void connect(String url, ChatWebSocketManager.ChatListener listener) {
        this.currentUrl = url;
        this.currentListener = listener;
        reconnectAttempts = 0;
        
        wsManager.connect(url, new ChatWebSocketManager.ChatListener() {
            @Override public void onConnected() {
                reconnectAttempts = 0;
                listener.onConnected();
            }
            
            @Override public void onMessageReceived(String message) {
                listener.onMessageReceived(message);
            }
            
            @Override public void onError(String error) {
                listener.onError(error);
                scheduleReconnect();
            }
            
            @Override public void onDisconnected(int code, String reason) {
                listener.onDisconnected(code, reason);
                if (code != 1000) scheduleReconnect(); // abnormal close
            }
        });
    }
    
    private void scheduleReconnect() {
        if (reconnectAttempts >= MAX_RECONNECT_ATTEMPTS) {
            Log.w("WS", "Max reconnect attempts reached");
            return;
        }
        
        long delay = INITIAL_DELAY_MS * (long) Math.pow(2, reconnectAttempts);
        reconnectAttempts++;
        
        Log.d("WS", "Reconnecting in " + delay + "ms (attempt " + reconnectAttempts + ")");
        handler.postDelayed(() -> connect(currentUrl, currentListener), delay);
    }
    
    public boolean sendMessage(String msg) { return wsManager.sendMessage(msg); }
    public void disconnect() {
        handler.removeCallbacksAndMessages(null);
        wsManager.disconnect();
    }
}
```

---

## 66.5 สรุป Part 66

ในบทนี้คุณได้เรียนรู้:

✅ WebSocket vs HTTP  
✅ OkHttp WebSocket API  
✅ WebSocketListener callbacks  
✅ JSON chat protocol  
✅ ChatViewModel with LiveData  
✅ Optimistic UI updates  
✅ Auto-reconnect with exponential backoff  

---

*[← Part 65: Network Interceptors](./part-65-android-network-interceptors.md) | [Part 67: Firebase Analytics →](./part-67-android-analytics.md)*
