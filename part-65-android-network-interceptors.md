# Part 65: Network Interceptors & Advanced OkHttp
## หลักสูตร Java & Android Development - ระดับ World-Class Android

---

## 65.1 OkHttp Interceptor คืออะไร

Interceptor ทำงานเหมือน middleware สำหรับ HTTP requests/responses:

```
Request → [App Interceptors] → [Network Interceptors] → Server
                                                            ↓
Response ← [App Interceptors] ← [Network Interceptors] ←──┘
```

---

## 65.2 Auth Token Interceptor

```java
// AuthInterceptor.java - แนบ token ทุก request อัตโนมัติ
public class AuthInterceptor implements Interceptor {
    
    private final TokenManager tokenManager;
    
    public AuthInterceptor(TokenManager tokenManager) {
        this.tokenManager = tokenManager;
    }
    
    @Override
    public Response intercept(Chain chain) throws IOException {
        Request original = chain.request();
        
        String token = tokenManager.getAccessToken();
        if (token == null) {
            return chain.proceed(original);  // no token, proceed without
        }
        
        Request modified = original.newBuilder()
            .header("Authorization", "Bearer " + token)
            .header("Accept", "application/json")
            .build();
        
        return chain.proceed(modified);
    }
}

// TokenRefreshInterceptor.java - auto refresh token on 401
public class TokenRefreshInterceptor implements Authenticator {
    
    private final TokenManager tokenManager;
    private final AuthApiService authService;
    
    public TokenRefreshInterceptor(TokenManager tokenManager, AuthApiService authService) {
        this.tokenManager = tokenManager;
        this.authService = authService;
    }
    
    @Nullable
    @Override
    public Request authenticate(Route route, Response response) throws IOException {
        // Already retried, give up
        if (responseCount(response) >= 3) return null;
        
        // Try to refresh token
        String refreshToken = tokenManager.getRefreshToken();
        if (refreshToken == null) {
            // No refresh token → force logout
            tokenManager.clearTokens();
            return null;
        }
        
        try {
            // Synchronous call to refresh token
            retrofit2.Response<TokenResponse> refreshResponse =
                authService.refreshToken(new RefreshRequest(refreshToken)).execute();
            
            if (refreshResponse.isSuccessful() && refreshResponse.body() != null) {
                TokenResponse newToken = refreshResponse.body();
                tokenManager.saveTokens(newToken.accessToken, newToken.refreshToken);
                
                // Retry original request with new token
                return response.request().newBuilder()
                    .header("Authorization", "Bearer " + newToken.accessToken)
                    .build();
            }
        } catch (Exception e) {
            Log.e("TokenRefresh", "Failed to refresh token", e);
        }
        
        tokenManager.clearTokens();
        return null;
    }
    
    private int responseCount(Response response) {
        int count = 1;
        while ((response = response.priorResponse()) != null) count++;
        return count;
    }
}
```

---

## 65.3 Logging Interceptor

```java
// CustomLoggingInterceptor.java
public class CustomLoggingInterceptor implements Interceptor {
    
    private static final String TAG = "HTTP";
    
    @Override
    public Response intercept(Chain chain) throws IOException {
        Request request = chain.request();
        long startTime = System.currentTimeMillis();
        
        // Log request
        Log.d(TAG, "→ " + request.method() + " " + request.url());
        if (request.body() != null) {
            Buffer buffer = new Buffer();
            request.body().writeTo(buffer);
            Log.d(TAG, "  Body: " + buffer.readUtf8());
        }
        
        Response response = chain.proceed(request);
        long duration = System.currentTimeMillis() - startTime;
        
        // Log response
        ResponseBody responseBody = response.body();
        String bodyString = responseBody != null ? responseBody.string() : "";
        
        Log.d(TAG, "← " + response.code() + " " + request.url() + " (" + duration + "ms)");
        Log.d(TAG, "  Response: " + bodyString.substring(0, Math.min(200, bodyString.length())));
        
        // IMPORTANT: must rebuild response since body can only be read once
        return response.newBuilder()
            .body(ResponseBody.create(bodyString, responseBody.contentType()))
            .build();
    }
}
```

---

## 65.4 Caching Interceptor

```java
// CacheInterceptor.java - offline-first caching
public class CacheInterceptor implements Interceptor {
    
    private final Context context;
    
    public CacheInterceptor(Context context) {
        this.context = context;
    }
    
    @Override
    public Response intercept(Chain chain) throws IOException {
        Request request = chain.request();
        
        if (!NetworkUtils.isConnected(context)) {
            // Offline: use cached data (up to 7 days)
            request = request.newBuilder()
                .cacheControl(new CacheControl.Builder()
                    .onlyIfCached()
                    .maxStale(7, TimeUnit.DAYS)
                    .build())
                .build();
        }
        
        Response response = chain.proceed(request);
        
        if (NetworkUtils.isConnected(context)) {
            // Online: cache response for 5 minutes
            return response.newBuilder()
                .header("Cache-Control", "public, max-age=300")
                .build();
        } else {
            // Offline: serve stale for 7 days
            return response.newBuilder()
                .header("Cache-Control", "public, only-if-cached, max-stale=604800")
                .build();
        }
    }
}

// NetworkModule with cache
@Provides
@Singleton
public OkHttpClient provideOkHttpClient(
        @ApplicationContext Context context,
        AuthInterceptor authInterceptor,
        CacheInterceptor cacheInterceptor) {
    
    File cacheDir = new File(context.getCacheDir(), "http_cache");
    Cache cache = new Cache(cacheDir, 50 * 1024 * 1024);  // 50MB
    
    return new OkHttpClient.Builder()
        .cache(cache)
        .addInterceptor(authInterceptor)        // app-level
        .addInterceptor(cacheInterceptor)       // app-level
        .addNetworkInterceptor(new HttpLoggingInterceptor())  // network-level
        .connectTimeout(30, TimeUnit.SECONDS)
        .readTimeout(30, TimeUnit.SECONDS)
        .writeTimeout(30, TimeUnit.SECONDS)
        .retryOnConnectionFailure(true)
        .build();
}
```

---

## 65.5 Request Retry Interceptor

```java
// RetryInterceptor.java
public class RetryInterceptor implements Interceptor {
    
    private final int maxRetries;
    private final long retryDelayMs;
    
    public RetryInterceptor(int maxRetries, long retryDelayMs) {
        this.maxRetries = maxRetries;
        this.retryDelayMs = retryDelayMs;
    }
    
    @Override
    public Response intercept(Chain chain) throws IOException {
        Request request = chain.request();
        int attempt = 0;
        
        while (true) {
            try {
                Response response = chain.proceed(request);
                
                if (response.isSuccessful() || attempt >= maxRetries) {
                    return response;
                }
                
                // Retry on server errors (5xx)
                if (response.code() >= 500) {
                    response.close();
                    attempt++;
                    sleep(retryDelayMs * (long) Math.pow(2, attempt));  // exponential backoff
                    continue;
                }
                
                return response;
                
            } catch (SocketTimeoutException | ConnectException e) {
                if (attempt >= maxRetries) throw e;
                attempt++;
                sleep(retryDelayMs * (long) Math.pow(2, attempt));
            }
        }
    }
    
    private void sleep(long ms) {
        try { Thread.sleep(ms); } catch (InterruptedException ignored) {}
    }
}
```

---

## 65.6 สรุป Part 65

ในบทนี้คุณได้เรียนรู้:

✅ OkHttp Interceptor (App vs Network level)  
✅ AuthInterceptor (auto attach Bearer token)  
✅ TokenRefreshInterceptor (Authenticator - auto refresh 401)  
✅ Custom logging interceptor  
✅ Offline caching interceptor  
✅ OkHttp Cache setup (50MB)  
✅ Retry interceptor with exponential backoff  

---

*[← Part 64: DataStore](./part-64-android-datastore.md) | [Part 66: WebSocket & Real-time →](./part-66-android-websocket.md)*
