# Part 50: Hilt - Dependency Injection
## หลักสูตร Java & Android Development - ระดับ Professional Android

---

## 50.1 Dependency Injection คืออะไร

DI = แทนที่จะสร้าง dependency เองใน class ให้ inject จากภายนอกแทน

**ปัญหาที่ DI แก้:**
```java
// ปัญหา: สร้าง dependency เองทุกที่ - hard to test, tight coupling
class OrderRepository {
    Database db = new Database("jdbc:...");  // hard-coded!
    ApiClient api = new ApiClient("https://...");
}

// ดีกว่า: inject via constructor
class OrderRepository {
    OrderRepository(Database db, ApiClient api) {
        this.db = db;
        this.api = api;
    }
}
```

**Hilt** คือ DI framework สำหรับ Android โดย Google (based on Dagger)

---

## 50.2 Setup Hilt

```groovy
// project-level build.gradle
buildscript {
    dependencies {
        classpath 'com.google.dagger:hilt-android-gradle-plugin:2.48'
    }
}

// app-level build.gradle
plugins {
    id 'com.google.dagger.hilt.android'
}

dependencies {
    implementation 'com.google.dagger:hilt-android:2.48'
    annotationProcessor 'com.google.dagger:hilt-compiler:2.48'
    
    // ViewModel integration
    implementation 'androidx.hilt:hilt-navigation-fragment:1.1.0'
}
```

---

## 50.3 @HiltAndroidApp - Application

```java
// MyApplication.java
@HiltAndroidApp  // Required - triggers Hilt code generation
public class MyApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        // Hilt automatically initializes
    }
}
```

```xml
<!-- AndroidManifest.xml -->
<application
    android:name=".MyApplication"
    ...>
```

---

## 50.4 Modules - ให้ Hilt รู้วิธีสร้าง dependencies

```java
// di/DatabaseModule.java
@Module
@InstallIn(SingletonComponent.class)  // App-wide singleton
public class DatabaseModule {
    
    @Provides
    @Singleton
    public AppDatabase provideDatabase(@ApplicationContext Context context) {
        return Room.databaseBuilder(context, AppDatabase.class, "app_db")
            .fallbackToDestructiveMigration()
            .build();
    }
    
    @Provides
    @Singleton
    public UserDao provideUserDao(AppDatabase db) {
        return db.userDao();
    }
    
    @Provides
    @Singleton
    public NoteDao provideNoteDao(AppDatabase db) {
        return db.noteDao();
    }
}

// di/NetworkModule.java
@Module
@InstallIn(SingletonComponent.class)
public class NetworkModule {
    
    private static final String BASE_URL = "https://api.example.com/";
    
    @Provides
    @Singleton
    public OkHttpClient provideOkHttpClient() {
        return new OkHttpClient.Builder()
            .connectTimeout(30, TimeUnit.SECONDS)
            .readTimeout(30, TimeUnit.SECONDS)
            .addInterceptor(new HttpLoggingInterceptor()
                .setLevel(HttpLoggingInterceptor.Level.BODY))
            .build();
    }
    
    @Provides
    @Singleton
    public Retrofit provideRetrofit(OkHttpClient okHttpClient) {
        return new Retrofit.Builder()
            .baseUrl(BASE_URL)
            .client(okHttpClient)
            .addConverterFactory(GsonConverterFactory.create())
            .build();
    }
    
    @Provides
    @Singleton
    public ApiService provideApiService(Retrofit retrofit) {
        return retrofit.create(ApiService.class);
    }
}

// di/RepositoryModule.java
@Module
@InstallIn(SingletonComponent.class)
public class RepositoryModule {
    
    @Provides
    @Singleton
    public UserRepository provideUserRepository(UserDao userDao, ApiService apiService) {
        return new UserRepositoryImpl(userDao, apiService);
    }
    
    @Provides
    @Singleton
    public NoteRepository provideNoteRepository(NoteDao noteDao) {
        return new NoteRepositoryImpl(noteDao);
    }
}
```

---

## 50.5 Inject ใน Activity, Fragment, ViewModel

```java
// MainActivity.java
@AndroidEntryPoint  // Required for Activities/Fragments
public class MainActivity extends AppCompatActivity {
    
    @Inject
    UserRepository userRepository;  // auto-injected!
    
    @Inject
    AnalyticsService analyticsService;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        // userRepository is ready to use
        userRepository.getAllUsers().observe(this, users -> { ... });
    }
}

// HomeFragment.java
@AndroidEntryPoint
public class HomeFragment extends Fragment {
    
    // Inject ViewModel with Hilt
    private HomeViewModel viewModel;
    
    @Override
    public void onViewCreated(View view, Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        
        // Use hiltNavGraphViewModels for shared ViewModel
        viewModel = new ViewModelProvider(this).get(HomeViewModel.class);
    }
}

// HomeViewModel.java
@HiltViewModel
public class HomeViewModel extends ViewModel {
    
    private final UserRepository userRepository;
    private final NoteRepository noteRepository;
    
    @Inject  // Hilt injects dependencies automatically
    public HomeViewModel(UserRepository userRepository, NoteRepository noteRepository) {
        this.userRepository = userRepository;
        this.noteRepository = noteRepository;
    }
    
    public LiveData<List<User>> getUsers() {
        return userRepository.getAllUsers();
    }
    
    public LiveData<List<Note>> getNotes() {
        return noteRepository.getAllNotes();
    }
}
```

---

## 50.6 Scopes

```java
// Scope annotations
@Singleton          // 1 instance for entire app lifetime
@ActivityScoped     // 1 instance per Activity
@FragmentScoped     // 1 instance per Fragment
@ViewModelScoped    // 1 instance per ViewModel

// Example: Activity-scoped
@Module
@InstallIn(ActivityComponent.class)
public class ActivityModule {
    
    @Provides
    @ActivityScoped
    public AnalyticsTracker provideAnalyticsTracker(@ActivityContext Context context) {
        return new AnalyticsTracker(context);
    }
}

// Inject in Activity
@AndroidEntryPoint
public class ProfileActivity extends AppCompatActivity {
    
    @Inject
    AnalyticsTracker analyticsTracker;  // new instance for each Activity
}
```

---

## 50.7 Interface Binding

```java
// Define interface
public interface ImageLoader {
    void load(String url, ImageView imageView);
}

// Implementation with Glide
public class GlideImageLoader implements ImageLoader {
    @Inject
    public GlideImageLoader() {}
    
    @Override
    public void load(String url, ImageView imageView) {
        Glide.with(imageView).load(url).into(imageView);
    }
}

// Bind interface → implementation
@Module
@InstallIn(SingletonComponent.class)
public abstract class ImageModule {
    
    @Binds
    @Singleton
    public abstract ImageLoader bindImageLoader(GlideImageLoader impl);
}

// Use
@AndroidEntryPoint
public class ProfileFragment extends Fragment {
    
    @Inject
    ImageLoader imageLoader;  // gets GlideImageLoader
    
    void loadAvatar(String url) {
        imageLoader.load(url, avatarImageView);
    }
}
```

---

## 50.8 สรุป Part 50

ในบทนี้คุณได้เรียนรู้:

✅ DI concept - inject vs new  
✅ Hilt setup (@HiltAndroidApp)  
✅ @Module + @Provides (Database, Network, Repository)  
✅ @InstallIn components (SingletonComponent, ActivityComponent)  
✅ @AndroidEntryPoint (Activity, Fragment)  
✅ @HiltViewModel + @Inject constructor  
✅ Scopes (@Singleton, @ActivityScoped)  
✅ @Binds interface to implementation  

---

*[← Part 49: Navigation](./part-49-android-navigation.md) | [Part 51: Testing Android Apps →](./part-51-android-testing.md)*
