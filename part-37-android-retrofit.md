# Part 37: Retrofit & REST API
## หลักสูตร Java & Android Development - ระดับ Intermediate Android

---

## 37.1 Retrofit คืออะไร

Retrofit คือ HTTP client library ที่แปลง REST API เป็น Java interface  
ทำให้เรียก API ง่ายมาก โดยไม่ต้องเขียน HTTP code เอง

---

## 37.2 Dependencies

```groovy
// build.gradle (app)
dependencies {
    implementation 'com.squareup.retrofit2:retrofit:2.9.0'
    implementation 'com.squareup.retrofit2:converter-gson:2.9.0'
    implementation 'com.squareup.okhttp3:okhttp:4.11.0'
    implementation 'com.squareup.okhttp3:logging-interceptor:4.11.0'
    implementation 'com.google.code.gson:gson:2.10.1'
}
```

```xml
<!-- AndroidManifest.xml - need internet permission -->
<uses-permission android:name="android.permission.INTERNET" />
```

---

## 37.3 Model Classes (POJO)

```java
// model/Post.java
package com.example.myapp.model;

import com.google.gson.annotations.SerializedName;

public class Post {
    @SerializedName("id")
    private int id;
    
    @SerializedName("userId")
    private int userId;
    
    @SerializedName("title")
    private String title;
    
    @SerializedName("body")
    private String body;
    
    // Constructors
    public Post() {}
    
    public Post(int userId, String title, String body) {
        this.userId = userId;
        this.title = title;
        this.body = body;
    }
    
    // Getters
    public int getId() { return id; }
    public int getUserId() { return userId; }
    public String getTitle() { return title; }
    public String getBody() { return body; }
    
    @Override
    public String toString() {
        return "Post{id=" + id + ", title='" + title + "'}";
    }
}
```

```java
// model/User.java
public class User {
    private int id;
    private String name;
    private String email;
    private String phone;
    private String website;
    private Address address;
    private Company company;
    
    public static class Address {
        private String street;
        private String city;
        private String zipcode;
        
        public String getCity() { return city; }
    }
    
    public static class Company {
        private String name;
        public String getName() { return name; }
    }
    
    // Getters
    public int getId() { return id; }
    public String getName() { return name; }
    public String getEmail() { return email; }
    public Address getAddress() { return address; }
    public Company getCompany() { return company; }
}
```

---

## 37.4 API Interface

```java
// api/ApiService.java
package com.example.myapp.api;

import com.example.myapp.model.*;
import retrofit2.Call;
import retrofit2.http.*;
import java.util.List;

public interface ApiService {
    
    // GET all posts
    @GET("posts")
    Call<List<Post>> getPosts();
    
    // GET post by ID
    @GET("posts/{id}")
    Call<Post> getPostById(@Path("id") int postId);
    
    // GET posts by user
    @GET("posts")
    Call<List<Post>> getPostsByUser(@Query("userId") int userId);
    
    // GET with multiple query params
    @GET("posts")
    Call<List<Post>> getPosts(
        @Query("_start") int start,
        @Query("_limit") int limit);
    
    // POST create
    @POST("posts")
    Call<Post> createPost(@Body Post post);
    
    // PUT update
    @PUT("posts/{id}")
    Call<Post> updatePost(@Path("id") int id, @Body Post post);
    
    // PATCH partial update
    @PATCH("posts/{id}")
    Call<Post> patchPost(@Path("id") int id, @Body Post post);
    
    // DELETE
    @DELETE("posts/{id}")
    Call<Void> deletePost(@Path("id") int id);
    
    // Headers
    @GET("posts")
    @Headers({
        "Accept: application/json",
        "Cache-Control: max-age=640000"
    })
    Call<List<Post>> getPostsWithHeaders();
    
    // Dynamic header
    @GET("posts")
    Call<List<Post>> getPostsAuth(@Header("Authorization") String authToken);
    
    // GET users
    @GET("users")
    Call<List<User>> getUsers();
    
    @GET("users/{id}")
    Call<User> getUserById(@Path("id") int userId);
    
    // Multipart form data (file upload)
    @Multipart
    @POST("upload")
    Call<Void> uploadFile(
        @Part("description") okhttp3.RequestBody description,
        @Part okhttp3.MultipartBody.Part file);
}
```

---

## 37.5 Retrofit Client

```java
// api/RetrofitClient.java
package com.example.myapp.api;

import okhttp3.OkHttpClient;
import okhttp3.logging.HttpLoggingInterceptor;
import retrofit2.Retrofit;
import retrofit2.converter.gson.GsonConverterFactory;
import java.util.concurrent.TimeUnit;

public class RetrofitClient {
    
    private static final String BASE_URL = "https://jsonplaceholder.typicode.com/";
    private static RetrofitClient instance;
    private ApiService apiService;
    
    private RetrofitClient() {
        // Logging interceptor (only in DEBUG builds)
        HttpLoggingInterceptor logging = new HttpLoggingInterceptor();
        logging.setLevel(HttpLoggingInterceptor.Level.BODY);
        
        // OkHttp client configuration
        OkHttpClient httpClient = new OkHttpClient.Builder()
            .addInterceptor(logging)
            .addInterceptor(chain -> {
                // Add auth header to every request
                okhttp3.Request original = chain.request();
                okhttp3.Request request = original.newBuilder()
                    .header("Accept", "application/json")
                    // .header("Authorization", "Bearer " + getToken())
                    .method(original.method(), original.body())
                    .build();
                return chain.proceed(request);
            })
            .connectTimeout(30, TimeUnit.SECONDS)
            .readTimeout(30, TimeUnit.SECONDS)
            .writeTimeout(30, TimeUnit.SECONDS)
            .build();
        
        Retrofit retrofit = new Retrofit.Builder()
            .baseUrl(BASE_URL)
            .addConverterFactory(GsonConverterFactory.create())
            .client(httpClient)
            .build();
        
        apiService = retrofit.create(ApiService.class);
    }
    
    public static synchronized RetrofitClient getInstance() {
        if (instance == null) {
            instance = new RetrofitClient();
        }
        return instance;
    }
    
    public ApiService getApi() { return apiService; }
}
```

---

## 37.6 Making API Calls

```java
// repository/PostRepository.java
package com.example.myapp.repository;

import androidx.lifecycle.MutableLiveData;
import com.example.myapp.api.RetrofitClient;
import com.example.myapp.model.Post;
import retrofit2.Call;
import retrofit2.Callback;
import retrofit2.Response;
import java.util.List;

public class PostRepository {
    
    private ApiService api = RetrofitClient.getInstance().getApi();
    
    public MutableLiveData<List<Post>> getPosts() {
        MutableLiveData<List<Post>> data = new MutableLiveData<>();
        
        api.getPosts().enqueue(new Callback<List<Post>>() {
            @Override
            public void onResponse(Call<List<Post>> call, Response<List<Post>> response) {
                if (response.isSuccessful() && response.body() != null) {
                    data.postValue(response.body());
                } else {
                    // Handle error response
                    data.postValue(null);
                }
            }
            
            @Override
            public void onFailure(Call<List<Post>> call, Throwable t) {
                // Network error
                data.postValue(null);
            }
        });
        
        return data;
    }
    
    public MutableLiveData<Post> createPost(Post post) {
        MutableLiveData<Post> result = new MutableLiveData<>();
        
        api.createPost(post).enqueue(new Callback<Post>() {
            @Override
            public void onResponse(Call<Post> call, Response<Post> response) {
                if (response.isSuccessful()) {
                    result.postValue(response.body());
                }
            }
            
            @Override
            public void onFailure(Call<Post> call, Throwable t) {
                result.postValue(null);
            }
        });
        
        return result;
    }
    
    public MutableLiveData<Boolean> deletePost(int id) {
        MutableLiveData<Boolean> result = new MutableLiveData<>();
        
        api.deletePost(id).enqueue(new Callback<Void>() {
            @Override
            public void onResponse(Call<Void> call, Response<Void> response) {
                result.postValue(response.isSuccessful());
            }
            
            @Override
            public void onFailure(Call<Void> call, Throwable t) {
                result.postValue(false);
            }
        });
        
        return result;
    }
}
```

---

## 37.7 Activity with API Calls

```java
// PostListActivity.java
public class PostListActivity extends AppCompatActivity {

    private PostRepository repository;
    private PostAdapter adapter;
    private ProgressBar progressBar;
    private TextView tvError;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_post_list);

        RecyclerView recyclerView = findViewById(R.id.recyclerView);
        progressBar = findViewById(R.id.progressBar);
        tvError = findViewById(R.id.tvError);

        adapter = new PostAdapter();
        recyclerView.setAdapter(adapter);
        recyclerView.setLayoutManager(new LinearLayoutManager(this));

        repository = new PostRepository();

        loadPosts();
    }

    private void loadPosts() {
        progressBar.setVisibility(View.VISIBLE);
        tvError.setVisibility(View.GONE);

        repository.getPosts().observe(this, posts -> {
            progressBar.setVisibility(View.GONE);

            if (posts != null) {
                adapter.submitList(posts);
            } else {
                tvError.setVisibility(View.VISIBLE);
                tvError.setText("ไม่สามารถโหลดข้อมูลได้");
            }
        });
    }

    private void createNewPost() {
        Post newPost = new Post(1, "หัวข้อใหม่", "เนื้อหาโพสต์");

        repository.createPost(newPost).observe(this, created -> {
            if (created != null) {
                Toast.makeText(this,
                    "สร้างโพสต์ ID: " + created.getId() + " แล้ว",
                    Toast.LENGTH_SHORT).show();
            }
        });
    }
}
```

---

## 37.8 Error Handling

```java
// Generic API result wrapper
public class ApiResult<T> {
    public enum Status { SUCCESS, ERROR, LOADING }
    
    private Status status;
    private T data;
    private String message;
    
    private ApiResult(Status status, T data, String message) {
        this.status = status;
        this.data = data;
        this.message = message;
    }
    
    public static <T> ApiResult<T> success(T data) {
        return new ApiResult<>(Status.SUCCESS, data, null);
    }
    
    public static <T> ApiResult<T> error(String message) {
        return new ApiResult<>(Status.ERROR, null, message);
    }
    
    public static <T> ApiResult<T> loading() {
        return new ApiResult<>(Status.LOADING, null, null);
    }
    
    public Status getStatus() { return status; }
    public T getData() { return data; }
    public String getMessage() { return message; }
}

// Usage in repository:
public MutableLiveData<ApiResult<List<Post>>> getPostsWithResult() {
    MutableLiveData<ApiResult<List<Post>>> result = new MutableLiveData<>();
    result.postValue(ApiResult.loading());
    
    api.getPosts().enqueue(new Callback<List<Post>>() {
        @Override
        public void onResponse(Call<List<Post>> call, Response<List<Post>> response) {
            if (response.isSuccessful() && response.body() != null) {
                result.postValue(ApiResult.success(response.body()));
            } else {
                String error = "Error " + response.code() + ": " + response.message();
                result.postValue(ApiResult.error(error));
            }
        }
        @Override
        public void onFailure(Call<List<Post>> call, Throwable t) {
            result.postValue(ApiResult.error("Network error: " + t.getMessage()));
        }
    });
    
    return result;
}

// Usage in Activity:
viewModel.getPosts().observe(this, result -> {
    switch (result.getStatus()) {
        case LOADING:
            progressBar.setVisibility(View.VISIBLE);
            break;
        case SUCCESS:
            progressBar.setVisibility(View.GONE);
            adapter.submitList(result.getData());
            break;
        case ERROR:
            progressBar.setVisibility(View.GONE);
            Toast.makeText(this, result.getMessage(), Toast.LENGTH_LONG).show();
            break;
    }
});
```

---

## 37.9 สรุป Part 37

ในบทนี้คุณได้เรียนรู้:

✅ Retrofit setup  
✅ API interface annotations (@GET, @POST, @PUT, @DELETE)  
✅ @Path, @Query, @Body, @Header  
✅ OkHttp client configuration (logging, timeout)  
✅ Callback (enqueue)  
✅ Repository pattern with LiveData  
✅ Error handling (ApiResult wrapper)  
✅ Full POST list example  

---

*[← Part 36: Room Database](./part-36-android-room.md) | [Part 38: Android MVVM Architecture →](./part-38-android-mvvm.md)*
