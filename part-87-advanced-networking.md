# Part 87: Advanced Networking - OkHttp, SSE, gRPC
## หลักสูตร Java & Android Development - ระดับ World-Class

---

## 87.1 Server-Sent Events (SSE)

```java
// SseClient.java - real-time server push (HTTP/1.1 streaming)
public class SseClient {
    
    private final OkHttpClient client;
    private Call activeCall;
    
    public interface SseListener {
        void onEvent(String eventType, String data);
        void onError(Exception e);
        void onConnected();
        void onDisconnected();
    }
    
    public SseClient() {
        client = new OkHttpClient.Builder()
            .connectTimeout(10, TimeUnit.SECONDS)
            .readTimeout(0, TimeUnit.SECONDS)     // no timeout for SSE stream
            .writeTimeout(10, TimeUnit.SECONDS)
            .retryOnConnectionFailure(true)
            .build();
    }
    
    public void connect(String url, String authToken, SseListener listener) {
        Request request = new Request.Builder()
            .url(url)
            .addHeader("Authorization", "Bearer " + authToken)
            .addHeader("Accept", "text/event-stream")
            .addHeader("Cache-Control", "no-cache")
            .build();
        
        activeCall = client.newCall(request);
        activeCall.enqueue(new Callback() {
            
            @Override
            public void onResponse(@NonNull Call call, @NonNull Response response) {
                if (!response.isSuccessful()) {
                    listener.onError(new IOException("HTTP " + response.code()));
                    return;
                }
                
                listener.onConnected();
                
                try (ResponseBody body = response.body()) {
                    if (body == null) return;
                    
                    BufferedSource source = body.source();
                    String eventType = "message";
                    StringBuilder dataBuffer = new StringBuilder();
                    
                    while (!source.exhausted()) {
                        String line = source.readUtf8Line();
                        if (line == null) break;
                        
                        if (line.startsWith("event:")) {
                            eventType = line.substring(6).trim();
                        } else if (line.startsWith("data:")) {
                            dataBuffer.append(line.substring(5).trim());
                        } else if (line.isEmpty() && dataBuffer.length() > 0) {
                            // Empty line = end of event
                            final String type = eventType;
                            final String data = dataBuffer.toString();
                            new Handler(Looper.getMainLooper()).post(
                                () -> listener.onEvent(type, data));
                            eventType = "message";
                            dataBuffer.setLength(0);
                        }
                    }
                } catch (IOException e) {
                    if (!call.isCanceled()) listener.onError(e);
                }
                
                listener.onDisconnected();
            }
            
            @Override
            public void onFailure(@NonNull Call call, @NonNull IOException e) {
                if (!call.isCanceled()) {
                    listener.onError(e);
                }
            }
        });
    }
    
    public void disconnect() {
        if (activeCall != null) activeCall.cancel();
    }
    
    public boolean isConnected() {
        return activeCall != null && !activeCall.isCanceled() && !activeCall.isExecuted();
    }
}

// Usage
SseClient sseClient = new SseClient();
sseClient.connect(
    "https://api.example.com/events/notifications",
    authToken,
    new SseClient.SseListener() {
        @Override
        public void onConnected() {
            Log.d("SSE", "Connected");
        }
        
        @Override
        public void onEvent(String eventType, String data) {
            switch (eventType) {
                case "order_update":
                    handleOrderUpdate(data);
                    break;
                case "new_message":
                    handleNewMessage(data);
                    break;
                default:
                    Log.d("SSE", eventType + ": " + data);
            }
        }
        
        @Override
        public void onError(Exception e) {
            Log.e("SSE", "Error: " + e.getMessage());
            // Reconnect after delay
        }
        
        @Override
        public void onDisconnected() {
            Log.d("SSE", "Disconnected");
        }
    });
```

---

## 87.2 Multipart Upload (File Upload)

```java
// FileUploadService.java
public interface FileUploadService {
    @Multipart
    @POST("upload/image")
    Call<UploadResponse> uploadImage(
        @Part("description") RequestBody description,
        @Part MultipartBody.Part image
    );
    
    @Multipart
    @POST("upload/multiple")
    Call<UploadResponse> uploadMultiple(
        @PartMap Map<String, RequestBody> fields,
        @Part List<MultipartBody.Part> files
    );
}

// Upload with progress
public class FileUploader {
    
    private final FileUploadService service;
    
    public interface UploadProgressListener {
        void onProgress(int percent);
        void onSuccess(UploadResponse response);
        void onError(Throwable t);
    }
    
    public void uploadImage(File imageFile, String description, 
            UploadProgressListener listener) {
        
        // Track upload progress
        CountingRequestBody countingBody = new CountingRequestBody(
            RequestBody.create(imageFile, MediaType.parse("image/jpeg")),
            (bytesWritten, contentLength) -> {
                int percent = (int) (bytesWritten * 100 / contentLength);
                new Handler(Looper.getMainLooper()).post(() -> listener.onProgress(percent));
            });
        
        MultipartBody.Part imagePart = MultipartBody.Part.createFormData(
            "image", imageFile.getName(), countingBody);
        
        RequestBody descBody = RequestBody.create(
            description, MediaType.parse("text/plain"));
        
        service.uploadImage(descBody, imagePart).enqueue(new Callback<UploadResponse>() {
            @Override
            public void onResponse(Call<UploadResponse> call, Response<UploadResponse> response) {
                if (response.isSuccessful() && response.body() != null) {
                    listener.onSuccess(response.body());
                } else {
                    listener.onError(new IOException("Upload failed: " + response.code()));
                }
            }
            
            @Override
            public void onFailure(Call<UploadResponse> call, Throwable t) {
                listener.onError(t);
            }
        });
    }
    
    // CountingRequestBody for progress tracking
    static class CountingRequestBody extends RequestBody {
        interface Listener {
            void onRequestProgress(long bytesWritten, long contentLength);
        }
        
        private final RequestBody delegate;
        private final Listener listener;
        
        CountingRequestBody(RequestBody delegate, Listener listener) {
            this.delegate = delegate;
            this.listener = listener;
        }
        
        @Override
        public MediaType contentType() { return delegate.contentType(); }
        
        @Override
        public long contentLength() throws IOException { return delegate.contentLength(); }
        
        @Override
        public void writeTo(@NonNull BufferedSink sink) throws IOException {
            CountingSink countingSink = new CountingSink(sink);
            BufferedSink bufferedSink = Okio.buffer(countingSink);
            delegate.writeTo(bufferedSink);
            bufferedSink.flush();
        }
        
        class CountingSink extends ForwardingSink {
            long bytesWritten = 0;
            
            CountingSink(Sink delegate) { super(delegate); }
            
            @Override
            public void write(@NonNull Buffer source, long byteCount) throws IOException {
                super.write(source, byteCount);
                bytesWritten += byteCount;
                listener.onRequestProgress(bytesWritten, contentLength());
            }
        }
    }
}
```

---

## 87.3 GraphQL with Retrofit

```java
// GraphQL request model
public class GraphQlRequest {
    private final String query;
    private final Map<String, Object> variables;
    
    public GraphQlRequest(String query, Map<String, Object> variables) {
        this.query = query;
        this.variables = variables;
    }
    
    public String getQuery()                  { return query; }
    public Map<String, Object> getVariables() { return variables; }
}

// GraphQL service
public interface GraphQlService {
    @POST("graphql")
    @Headers("Content-Type: application/json")
    Call<GraphQlResponse<ProductsData>> getProducts(
        @Body GraphQlRequest request);
}

// Usage
String query = "query GetProducts($categoryId: String) { " +
    "products(categoryId: $categoryId) { " +
    "  id name price imageUrl inStock " +
    "} }";

Map<String, Object> variables = new HashMap<>();
variables.put("categoryId", "electronics");

GraphQlRequest request = new GraphQlRequest(query, variables);
graphQlService.getProducts(request).enqueue(callback);
```

---

## 87.4 สรุป Part 87

ในบทนี้คุณได้เรียนรู้:

✅ Server-Sent Events (SSE) client  
✅ Read SSE event stream line by line  
✅ Multipart file upload with Retrofit  
✅ Upload progress tracking (CountingRequestBody)  
✅ GraphQL request via Retrofit  
✅ ForwardingSink for byte counting  

---

*[← Part 86: Compose Interop](./part-86-compose-interop.md) | [Part 88: Java Memory Management →](./part-88-java-memory.md)*
