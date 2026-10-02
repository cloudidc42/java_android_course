# Part 39: SharedPreferences และ Settings
## หลักสูตร Java & Android Development - ระดับ Intermediate Android

---

## 39.1 SharedPreferences คืออะไร

SharedPreferences คือ simple key-value storage สำหรับเก็บข้อมูลเล็กๆ  
เหมาะสำหรับ: user settings, login state, app preferences

---

## 39.2 SharedPreferences พื้นฐาน

```java
// SharedPreferencesDemo.java
import android.content.Context;
import android.content.SharedPreferences;

public class PrefsDemo {
    
    private static final String PREFS_NAME = "MyAppPrefs";
    
    // --- WRITE ---
    static void saveUserPrefs(Context context, String username, boolean isDarkMode, int fontSize) {
        SharedPreferences prefs = context.getSharedPreferences(PREFS_NAME, Context.MODE_PRIVATE);
        SharedPreferences.Editor editor = prefs.edit();
        
        editor.putString("username", username);
        editor.putBoolean("dark_mode", isDarkMode);
        editor.putInt("font_size", fontSize);
        editor.putLong("last_login", System.currentTimeMillis());
        
        editor.apply();  // async (recommended)
        // editor.commit();  // synchronous (blocks UI)
    }
    
    // --- READ ---
    static String getUsername(Context context) {
        SharedPreferences prefs = context.getSharedPreferences(PREFS_NAME, Context.MODE_PRIVATE);
        return prefs.getString("username", "ผู้ใช้");  // default value
    }
    
    static boolean isDarkMode(Context context) {
        SharedPreferences prefs = context.getSharedPreferences(PREFS_NAME, Context.MODE_PRIVATE);
        return prefs.getBoolean("dark_mode", false);
    }
    
    static int getFontSize(Context context) {
        SharedPreferences prefs = context.getSharedPreferences(PREFS_NAME, Context.MODE_PRIVATE);
        return prefs.getInt("font_size", 14);
    }
    
    // --- DELETE ---
    static void removeKey(Context context, String key) {
        SharedPreferences prefs = context.getSharedPreferences(PREFS_NAME, Context.MODE_PRIVATE);
        prefs.edit().remove(key).apply();
    }
    
    static void clearAll(Context context) {
        SharedPreferences prefs = context.getSharedPreferences(PREFS_NAME, Context.MODE_PRIVATE);
        prefs.edit().clear().apply();
    }
    
    // --- CHECK ---
    static boolean contains(Context context, String key) {
        SharedPreferences prefs = context.getSharedPreferences(PREFS_NAME, Context.MODE_PRIVATE);
        return prefs.contains(key);
    }
    
    // --- LISTEN ---
    static void registerChangeListener(Context context) {
        SharedPreferences prefs = context.getSharedPreferences(PREFS_NAME, Context.MODE_PRIVATE);
        prefs.registerOnSharedPreferenceChangeListener((p, key) -> {
            // Called when any value changes
            System.out.println("Changed: " + key + " = " + p.getAll().get(key));
        });
    }
}
```

---

## 39.3 Preference Manager (Helper class)

```java
// util/PreferenceManager.java
package com.example.myapp.util;

import android.content.Context;
import android.content.SharedPreferences;

public class PreferenceManager {
    
    private static final String PREF_NAME = "AppPreferences";
    
    // Keys
    private static final String KEY_IS_LOGGED_IN = "is_logged_in";
    private static final String KEY_USER_ID      = "user_id";
    private static final String KEY_USER_NAME    = "user_name";
    private static final String KEY_USER_EMAIL   = "user_email";
    private static final String KEY_DARK_MODE    = "dark_mode";
    private static final String KEY_LANGUAGE     = "language";
    private static final String KEY_FONT_SIZE    = "font_size";
    private static final String KEY_NOTIFICATIONS = "notifications_enabled";
    private static final String KEY_FIRST_RUN    = "first_run";
    
    private final SharedPreferences prefs;
    private static PreferenceManager instance;
    
    private PreferenceManager(Context context) {
        prefs = context.getApplicationContext()
            .getSharedPreferences(PREF_NAME, Context.MODE_PRIVATE);
    }
    
    public static PreferenceManager getInstance(Context context) {
        if (instance == null) {
            synchronized (PreferenceManager.class) {
                if (instance == null) instance = new PreferenceManager(context);
            }
        }
        return instance;
    }
    
    // User session
    public void setLoggedIn(boolean loggedIn) { put(KEY_IS_LOGGED_IN, loggedIn); }
    public boolean isLoggedIn() { return prefs.getBoolean(KEY_IS_LOGGED_IN, false); }
    
    public void setUserId(int id) { put(KEY_USER_ID, id); }
    public int getUserId() { return prefs.getInt(KEY_USER_ID, -1); }
    
    public void setUserName(String name) { put(KEY_USER_NAME, name); }
    public String getUserName() { return prefs.getString(KEY_USER_NAME, ""); }
    
    public void setUserEmail(String email) { put(KEY_USER_EMAIL, email); }
    public String getUserEmail() { return prefs.getString(KEY_USER_EMAIL, ""); }
    
    public void saveUser(int id, String name, String email) {
        prefs.edit()
            .putBoolean(KEY_IS_LOGGED_IN, true)
            .putInt(KEY_USER_ID, id)
            .putString(KEY_USER_NAME, name)
            .putString(KEY_USER_EMAIL, email)
            .apply();
    }
    
    public void logout() {
        prefs.edit()
            .remove(KEY_IS_LOGGED_IN)
            .remove(KEY_USER_ID)
            .remove(KEY_USER_NAME)
            .remove(KEY_USER_EMAIL)
            .apply();
    }
    
    // App settings
    public void setDarkMode(boolean dark) { put(KEY_DARK_MODE, dark); }
    public boolean isDarkMode() { return prefs.getBoolean(KEY_DARK_MODE, false); }
    
    public void setLanguage(String lang) { put(KEY_LANGUAGE, lang); }
    public String getLanguage() { return prefs.getString(KEY_LANGUAGE, "th"); }
    
    public void setFontSize(int size) { put(KEY_FONT_SIZE, size); }
    public int getFontSize() { return prefs.getInt(KEY_FONT_SIZE, 14); }
    
    public void setNotificationsEnabled(boolean enabled) { put(KEY_NOTIFICATIONS, enabled); }
    public boolean isNotificationsEnabled() { return prefs.getBoolean(KEY_NOTIFICATIONS, true); }
    
    public boolean isFirstRun() { return prefs.getBoolean(KEY_FIRST_RUN, true); }
    public void setFirstRunDone() { put(KEY_FIRST_RUN, false); }
    
    // Generic helper
    private void put(String key, Object value) {
        SharedPreferences.Editor editor = prefs.edit();
        if (value instanceof String)  editor.putString(key, (String) value);
        else if (value instanceof Integer) editor.putInt(key, (Integer) value);
        else if (value instanceof Boolean) editor.putBoolean(key, (Boolean) value);
        else if (value instanceof Long)    editor.putLong(key, (Long) value);
        else if (value instanceof Float)   editor.putFloat(key, (Float) value);
        editor.apply();
    }
    
    public void clearAll() { prefs.edit().clear().apply(); }
}
```

---

## 39.4 Settings Screen (PreferenceFragment)

```xml
<!-- res/xml/preferences.xml -->
<?xml version="1.0" encoding="utf-8"?>
<PreferenceScreen xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <!-- Appearance -->
    <PreferenceCategory app:title="การแสดงผล">
        
        <SwitchPreferenceCompat
            app:key="dark_mode"
            app:title="โหมดมืด"
            app:summary="เปิดใช้งาน Dark Theme"
            app:defaultValue="false" />
        
        <ListPreference
            app:key="font_size"
            app:title="ขนาดตัวอักษร"
            app:entries="@array/font_size_labels"
            app:entryValues="@array/font_size_values"
            app:defaultValue="14" />
            
        <ListPreference
            app:key="language"
            app:title="ภาษา"
            app:entries="@array/language_labels"
            app:entryValues="@array/language_codes"
            app:defaultValue="th" />
    </PreferenceCategory>

    <!-- Notifications -->
    <PreferenceCategory app:title="การแจ้งเตือน">
        
        <SwitchPreferenceCompat
            app:key="notifications_enabled"
            app:title="รับการแจ้งเตือน"
            app:defaultValue="true" />
        
        <SwitchPreferenceCompat
            app:key="sound_enabled"
            app:title="เสียงแจ้งเตือน"
            app:dependency="notifications_enabled"
            app:defaultValue="true" />
    </PreferenceCategory>

    <!-- Account -->
    <PreferenceCategory app:title="บัญชีผู้ใช้">
        
        <Preference
            app:key="profile"
            app:title="แก้ไขโปรไฟล์"
            app:summary="เปลี่ยนชื่อ อีเมล รูปภาพ" />
        
        <Preference
            app:key="change_password"
            app:title="เปลี่ยนรหัสผ่าน" />
        
        <Preference
            app:key="logout"
            app:title="ออกจากระบบ"
            app:icon="@drawable/ic_logout" />
    </PreferenceCategory>

    <!-- About -->
    <PreferenceCategory app:title="เกี่ยวกับ">
        
        <Preference
            app:key="version"
            app:title="เวอร์ชัน"
            app:summary="1.0.0" />
        
        <Preference
            app:key="privacy_policy"
            app:title="นโยบายความเป็นส่วนตัว" />
    </PreferenceCategory>

</PreferenceScreen>
```

```java
// SettingsFragment.java
import androidx.preference.*;

public class SettingsFragment extends PreferenceFragmentCompat {

    @Override
    public void onCreatePreferences(Bundle savedInstanceState, String rootKey) {
        setPreferencesFromResource(R.xml.preferences, rootKey);
        
        // Dark mode
        SwitchPreferenceCompat darkMode = findPreference("dark_mode");
        if (darkMode != null) {
            darkMode.setOnPreferenceChangeListener((pref, newValue) -> {
                boolean isDark = (Boolean) newValue;
                // Apply theme
                AppCompatDelegate.setDefaultNightMode(
                    isDark ? AppCompatDelegate.MODE_NIGHT_YES
                           : AppCompatDelegate.MODE_NIGHT_NO);
                return true;
            });
        }
        
        // Logout
        Preference logout = findPreference("logout");
        if (logout != null) {
            logout.setOnPreferenceClickListener(pref -> {
                showLogoutDialog();
                return true;
            });
        }
        
        // Version
        Preference version = findPreference("version");
        if (version != null) {
            try {
                String versionName = requireContext().getPackageManager()
                    .getPackageInfo(requireContext().getPackageName(), 0).versionName;
                version.setSummary(versionName);
            } catch (Exception e) {}
        }
    }
    
    private void showLogoutDialog() {
        new androidx.appcompat.app.AlertDialog.Builder(requireContext())
            .setTitle("ออกจากระบบ")
            .setMessage("ต้องการออกจากระบบหรือไม่?")
            .setPositiveButton("ออกจากระบบ", (dialog, which) -> {
                PreferenceManager.getInstance(requireContext()).logout();
                // Navigate to login
                startActivity(new Intent(requireContext(), LoginActivity.class)
                    .addFlags(Intent.FLAG_ACTIVITY_CLEAR_TASK | Intent.FLAG_ACTIVITY_NEW_TASK));
            })
            .setNegativeButton("ยกเลิก", null)
            .show();
    }
}
```

```java
// SettingsActivity.java
public class SettingsActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_settings);
        
        if (getSupportActionBar() != null) {
            getSupportActionBar().setDisplayHomeAsUpEnabled(true);
            getSupportActionBar().setTitle("ตั้งค่า");
        }
        
        getSupportFragmentManager()
            .beginTransaction()
            .replace(R.id.settingsContainer, new SettingsFragment())
            .commit();
    }
}
```

---

## 39.5 สรุป Part 39

ในบทนี้คุณได้เรียนรู้:

✅ SharedPreferences (read, write, delete)  
✅ PreferenceManager helper class  
✅ User session management (login/logout)  
✅ App settings (dark mode, language, font)  
✅ PreferenceFragment  
✅ Preference XML (SwitchPreference, ListPreference)  
✅ Preference change listeners  

---

*[← Part 38: MVVM Architecture](./part-38-android-mvvm.md) | [Part 40: Android Notifications →](./part-40-android-notifications.md)*
