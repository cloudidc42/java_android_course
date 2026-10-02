# Part 64: Jetpack DataStore (แทน SharedPreferences)
## หลักสูตร Java & Android Development - ระดับ World-Class Android

---

## 64.1 DataStore vs SharedPreferences

```
SharedPreferences (เก่า):          DataStore (ใหม่):
- Synchronous                       - Asynchronous (Coroutines/RxJava)
- No type safety                    - Type safe (Preferences / Proto)
- Not safe for complex data         - Error handling via Flow/exceptions
- UI thread blocking                - Never blocks main thread
- No migrations                     - Support migrations

ประเภท DataStore:
1. Preferences DataStore - key-value like SharedPreferences
2. Proto DataStore       - typed objects (Protocol Buffers)
```

---

## 64.2 Preferences DataStore Setup

```groovy
// build.gradle
implementation "androidx.datastore:datastore-preferences:1.0.0"
implementation "androidx.datastore:datastore-preferences-rxjava3:1.0.0"  // for RxJava
```

```java
// UserPreferencesDataStore.java
public class UserPreferencesDataStore {
    
    // Keys
    static final Preferences.Key<String>  KEY_USERNAME   = PreferencesKeys.stringKey("username");
    static final Preferences.Key<String>  KEY_EMAIL      = PreferencesKeys.stringKey("email");
    static final Preferences.Key<Boolean> KEY_DARK_MODE  = PreferencesKeys.booleanKey("dark_mode");
    static final Preferences.Key<Integer> KEY_FONT_SIZE  = PreferencesKeys.intKey("font_size");
    static final Preferences.Key<String>  KEY_LANGUAGE   = PreferencesKeys.stringKey("language");
    static final Preferences.Key<Long>    KEY_LAST_LOGIN = PreferencesKeys.longKey("last_login");
    
    // Create DataStore (singleton via Context extension)
    private final RxDataStore<Preferences> dataStore;
    
    public UserPreferencesDataStore(Context context) {
        // Build DataStore with RxJava support
        dataStore = new RxPreferenceDataStoreBuilder(context, "user_prefs").build();
    }
    
    // Read username
    public Single<String> getUsername() {
        return dataStore.data()
            .map(prefs -> prefs.get(KEY_USERNAME) != null ? prefs.get(KEY_USERNAME) : "")
            .firstOrError();
    }
    
    // Save username
    public Completable saveUsername(String username) {
        return dataStore.updateDataAsync(prefs -> {
            MutablePreferences mutablePrefs = prefs.toMutablePreferences();
            mutablePrefs.set(KEY_USERNAME, username);
            return Single.just(mutablePrefs);
        });
    }
    
    // Observe dark mode changes
    public Flowable<Boolean> observeDarkMode() {
        return dataStore.data()
            .map(prefs -> prefs.get(KEY_DARK_MODE) != null ? prefs.get(KEY_DARK_MODE) : false);
    }
    
    // Save dark mode
    public Completable setDarkMode(boolean isDarkMode) {
        return dataStore.updateDataAsync(prefs -> {
            MutablePreferences mutablePrefs = prefs.toMutablePreferences();
            mutablePrefs.set(KEY_DARK_MODE, isDarkMode);
            return Single.just(mutablePrefs);
        });
    }
    
    // Get all settings at once
    public Single<UserSettings> getAllSettings() {
        return dataStore.data()
            .map(prefs -> new UserSettings(
                prefs.get(KEY_USERNAME),
                prefs.get(KEY_EMAIL),
                Boolean.TRUE.equals(prefs.get(KEY_DARK_MODE)),
                prefs.get(KEY_FONT_SIZE) != null ? prefs.get(KEY_FONT_SIZE) : 14,
                prefs.get(KEY_LANGUAGE) != null ? prefs.get(KEY_LANGUAGE) : "en"
            ))
            .firstOrError();
    }
    
    // Clear all preferences
    public Completable clearAll() {
        return dataStore.updateDataAsync(prefs -> {
            MutablePreferences mutablePrefs = prefs.toMutablePreferences();
            mutablePrefs.clear();
            return Single.just(mutablePrefs);
        });
    }
}

// UserSettings.java - data class
public class UserSettings {
    public final String  username;
    public final String  email;
    public final boolean darkMode;
    public final int     fontSize;
    public final String  language;
    
    public UserSettings(String username, String email, boolean darkMode,
            int fontSize, String language) {
        this.username = username;
        this.email    = email;
        this.darkMode = darkMode;
        this.fontSize = fontSize;
        this.language = language;
    }
}
```

---

## 64.3 Inject DataStore via Hilt

```java
// DataStoreModule.java
@Module
@InstallIn(SingletonComponent.class)
public class DataStoreModule {
    
    @Provides
    @Singleton
    public UserPreferencesDataStore provideUserPreferencesDataStore(
            @ApplicationContext Context context) {
        return new UserPreferencesDataStore(context);
    }
}

// SettingsViewModel.java
@HiltViewModel
public class SettingsViewModel extends ViewModel {
    
    private final UserPreferencesDataStore dataStore;
    private final CompositeDisposable disposable = new CompositeDisposable();
    
    private final MutableLiveData<UserSettings> settings = new MutableLiveData<>();
    private final MutableLiveData<Boolean> darkMode = new MutableLiveData<>();
    
    @Inject
    public SettingsViewModel(UserPreferencesDataStore dataStore) {
        this.dataStore = dataStore;
        loadSettings();
        observeDarkMode();
    }
    
    private void loadSettings() {
        disposable.add(
            dataStore.getAllSettings()
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(settings::setValue, Throwable::printStackTrace)
        );
    }
    
    private void observeDarkMode() {
        disposable.add(
            dataStore.observeDarkMode()
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(darkMode::setValue)
        );
    }
    
    public void setDarkMode(boolean enabled) {
        disposable.add(
            dataStore.setDarkMode(enabled)
                .subscribeOn(Schedulers.io())
                .subscribe()
        );
    }
    
    public void saveUsername(String name) {
        disposable.add(
            dataStore.saveUsername(name)
                .subscribeOn(Schedulers.io())
                .subscribe()
        );
    }
    
    public LiveData<UserSettings> getSettings() { return settings; }
    public LiveData<Boolean> getDarkMode()       { return darkMode; }
    
    @Override
    protected void onCleared() {
        disposable.clear();
    }
}
```

---

## 64.4 DataStore ใน Fragment

```java
@AndroidEntryPoint
public class SettingsFragment extends Fragment {
    
    @Inject UserPreferencesDataStore preferencesDataStore;
    
    private SettingsViewModel viewModel;
    private final CompositeDisposable disposable = new CompositeDisposable();
    
    @Override
    public void onViewCreated(View view, Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        
        viewModel = new ViewModelProvider(this).get(SettingsViewModel.class);
        
        SwitchCompat switchDarkMode = view.findViewById(R.id.switchDarkMode);
        EditText etUsername = view.findViewById(R.id.etUsername);
        
        // Observe settings
        viewModel.getSettings().observe(getViewLifecycleOwner(), settings -> {
            switchDarkMode.setChecked(settings.darkMode);
            etUsername.setText(settings.username);
        });
        
        // Toggle dark mode
        switchDarkMode.setOnCheckedChangeListener((btn, isChecked) -> {
            viewModel.setDarkMode(isChecked);
            // Apply dark mode immediately
            AppCompatDelegate.setDefaultNightMode(isChecked ?
                AppCompatDelegate.MODE_NIGHT_YES : AppCompatDelegate.MODE_NIGHT_NO);
        });
        
        // Save username on focus lost
        etUsername.setOnFocusChangeListener((v, hasFocus) -> {
            if (!hasFocus) {
                viewModel.saveUsername(etUsername.getText().toString().trim());
            }
        });
    }
    
    @Override
    public void onDestroyView() {
        super.onDestroyView();
        disposable.clear();
    }
}
```

---

## 64.5 Migration จาก SharedPreferences

```java
// DataStore with migration from SharedPreferences
public class MigratedDataStore {
    
    private final RxDataStore<Preferences> dataStore;
    
    public MigratedDataStore(Context context) {
        SharedPreferencesMigration migration = new SharedPreferencesMigration(
            context,
            "my_old_prefs",    // old SharedPreferences name
            new java.util.HashSet<>(java.util.Arrays.asList(
                "username", "dark_mode", "language"  // keys to migrate
            ))
        );
        
        dataStore = new RxPreferenceDataStoreBuilder(context, "user_prefs")
            .addMigration(migration)
            .build();
    }
}
```

---

## 64.6 สรุป Part 64

ในบทนี้คุณได้เรียนรู้:

✅ DataStore vs SharedPreferences  
✅ Preferences DataStore setup  
✅ Read/write preferences (RxJava)  
✅ Observe changes as Flowable  
✅ UserSettings data class  
✅ Hilt injection of DataStore  
✅ SettingsViewModel pattern  
✅ Apply dark mode dynamically  
✅ Migration from SharedPreferences  

---

*[← Part 63: MotionLayout](./part-63-android-motionlayout.md) | [Part 65: Network Interceptors →](./part-65-android-network-interceptors.md)*
