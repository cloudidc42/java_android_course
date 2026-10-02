# Part 59: Internationalization (i18n) & Localization
## หลักสูตร Java & Android Development - ระดับ World-Class Android

---

## 59.1 แนวคิด i18n

**i18n** = internationalization = การเตรียมแอปรองรับหลายภาษา  
**l10n** = localization = การแปลเนื้อหาสำหรับแต่ละภาษา

```
res/
├── values/               ← ภาษา default (English หรือ Thai)
│   ├── strings.xml
│   └── plurals.xml
├── values-th/            ← Thai overrides
│   └── strings.xml
├── values-zh-rCN/        ← Chinese Simplified
│   └── strings.xml
├── values-ja/            ← Japanese
│   └── strings.xml
└── values-night/         ← Dark mode colors
    └── colors.xml
```

---

## 59.2 Strings Resources

```xml
<!-- res/values/strings.xml (English default) -->
<resources>
    <string name="app_name">MyApp</string>
    <string name="btn_login">Login</string>
    <string name="btn_logout">Logout</string>
    <string name="msg_welcome">Welcome, %1$s!</string>
    <string name="msg_error">Error: %1$s</string>
    <string name="label_items">Items</string>
    
    <!-- Plurals -->
    <plurals name="items_count">
        <item quantity="one">%d item</item>
        <item quantity="other">%d items</item>
    </plurals>
    
    <!-- String array -->
    <string-array name="days_of_week">
        <item>Monday</item>
        <item>Tuesday</item>
        <item>Wednesday</item>
        <item>Thursday</item>
        <item>Friday</item>
        <item>Saturday</item>
        <item>Sunday</item>
    </string-array>
</resources>

<!-- res/values-th/strings.xml (Thai) -->
<resources>
    <string name="btn_login">เข้าสู่ระบบ</string>
    <string name="btn_logout">ออกจากระบบ</string>
    <string name="msg_welcome">ยินดีต้อนรับ, %1$s!</string>
    <string name="msg_error">เกิดข้อผิดพลาด: %1$s</string>
    <string name="label_items">รายการ</string>
    
    <plurals name="items_count">
        <item quantity="other">%d รายการ</item>
    </plurals>
    
    <string-array name="days_of_week">
        <item>จันทร์</item>
        <item>อังคาร</item>
        <item>พุธ</item>
        <item>พฤหัสบดี</item>
        <item>ศุกร์</item>
        <item>เสาร์</item>
        <item>อาทิตย์</item>
    </string-array>
</resources>
```

```java
// Using strings in code
String welcome = getString(R.string.msg_welcome, userName);
String items = getResources().getQuantityString(R.plurals.items_count, count, count);
String[] days = getResources().getStringArray(R.array.days_of_week);

// Formatted string
String error = getString(R.string.msg_error, "Network unavailable");
```

---

## 59.3 Language-Specific Resources

```
# Layout direction (RTL - Right To Left languages like Arabic, Hebrew)
res/
├── layout/
│   └── activity_main.xml       ← LTR default
└── layout-ldrtl/
    └── activity_main.xml       ← RTL layout (mirrored)

# Use start/end instead of left/right for RTL support
```

```xml
<!-- Use start/end, not left/right -->
<TextView
    android:layout_marginStart="16dp"   <!-- NOT layout_marginLeft -->
    android:layout_marginEnd="16dp"     <!-- NOT layout_marginRight -->
    android:textAlignment="viewStart"   <!-- NOT textAlignment="left" -->
    android:drawableStart="@drawable/ic_icon" /> <!-- NOT drawableLeft -->

<!-- Manifest: RTL support -->
<application
    android:supportsRtl="true">
```

---

## 59.4 Locale and Language Management

```java
// LocaleHelper.java
public class LocaleHelper {
    
    private static final String PREF_LANGUAGE = "language";
    
    public static Context applyLocale(Context context) {
        String language = getLanguage(context);
        return setLocale(context, language);
    }
    
    public static Context setLocale(Context context, String language) {
        saveLanguage(context, language);
        
        Locale locale = new Locale(language);
        Locale.setDefault(locale);
        
        Resources res = context.getResources();
        Configuration config = new Configuration(res.getConfiguration());
        config.setLocale(locale);
        
        return context.createConfigurationContext(config);
    }
    
    public static String getLanguage(Context context) {
        SharedPreferences prefs = PreferenceManager.getDefaultSharedPreferences(context);
        return prefs.getString(PREF_LANGUAGE, getSystemLanguage());
    }
    
    private static void saveLanguage(Context context, String language) {
        PreferenceManager.getDefaultSharedPreferences(context)
            .edit().putString(PREF_LANGUAGE, language).apply();
    }
    
    private static String getSystemLanguage() {
        return Locale.getDefault().getLanguage();
    }
}

// BaseActivity.java - apply to all activities
public abstract class BaseActivity extends AppCompatActivity {
    
    @Override
    protected void attachBaseContext(Context base) {
        super.attachBaseContext(LocaleHelper.applyLocale(base));
    }
}

// In Application class
public class MyApplication extends Application {
    
    @Override
    protected void attachBaseContext(Context base) {
        super.attachBaseContext(LocaleHelper.applyLocale(base));
    }
}
```

---

## 59.5 Language Switcher UI

```java
// LanguageSettingsActivity.java
public class LanguageSettingsActivity extends AppCompatActivity {
    
    private String selectedLanguage;
    
    private static final String[] LANGUAGE_CODES = {"en", "th", "zh", "ja"};
    private static final String[] LANGUAGE_NAMES = {"English", "ภาษาไทย", "中文", "日本語"};
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_language_settings);
        
        selectedLanguage = LocaleHelper.getLanguage(this);
        
        RadioGroup rgLanguage = findViewById(R.id.rgLanguage);
        
        for (int i = 0; i < LANGUAGE_CODES.length; i++) {
            RadioButton rb = new RadioButton(this);
            rb.setText(LANGUAGE_NAMES[i]);
            rb.setTag(LANGUAGE_CODES[i]);
            if (LANGUAGE_CODES[i].equals(selectedLanguage)) rb.setChecked(true);
            rgLanguage.addView(rb);
        }
        
        rgLanguage.setOnCheckedChangeListener((group, checkedId) -> {
            RadioButton selected = group.findViewById(checkedId);
            selectedLanguage = (String) selected.getTag();
        });
        
        findViewById(R.id.btnApply).setOnClickListener(v -> applyLanguage());
    }
    
    private void applyLanguage() {
        LocaleHelper.setLocale(this, selectedLanguage);
        // Restart app to apply
        Intent intent = new Intent(this, MainActivity.class);
        intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TASK | Intent.FLAG_ACTIVITY_NEW_TASK);
        startActivity(intent);
    }
}
```

---

## 59.6 Date and Number Formatting

```java
// DateFormatHelper.java
public class DateFormatHelper {
    
    // Format date according to current locale
    public static String formatDate(long timestamp) {
        return DateFormat.getDateFormat(MyApp.getContext())
            .format(new Date(timestamp));
    }
    
    public static String formatDateTime(long timestamp) {
        DateFormat dateFmt = DateFormat.getDateFormat(MyApp.getContext());
        DateFormat timeFmt = DateFormat.getTimeFormat(MyApp.getContext());
        Date date = new Date(timestamp);
        return dateFmt.format(date) + " " + timeFmt.format(date);
    }
    
    // Thai calendar (Buddhist Era = CE + 543)
    public static String formatThaiBuddhistDate(long timestamp) {
        Calendar cal = Calendar.getInstance();
        cal.setTimeInMillis(timestamp);
        
        SimpleDateFormat sdf = new SimpleDateFormat("dd/MM/yyyy", new Locale("th"));
        sdf.setCalendar(new java.util.Calendar.Builder()
            .setCalendarType("buddhist").build());
        return sdf.format(cal.getTime());
    }
    
    // Relative time ("2 hours ago")
    public static String formatRelativeTime(long timestamp) {
        long now = System.currentTimeMillis();
        long diff = now - timestamp;
        
        if (diff < 60_000) return "เมื่อกี้";
        if (diff < 3_600_000) return (diff / 60_000) + " นาทีที่แล้ว";
        if (diff < 86_400_000) return (diff / 3_600_000) + " ชั่วโมงที่แล้ว";
        return (diff / 86_400_000) + " วันที่แล้ว";
    }
}

// NumberFormatHelper.java
public class NumberFormatHelper {
    
    public static String formatCurrency(double amount, String currencyCode) {
        NumberFormat format = NumberFormat.getCurrencyInstance();
        format.setCurrency(Currency.getInstance(currencyCode));
        return format.format(amount);
    }
    
    public static String formatThaiCurrency(double amount) {
        return formatCurrency(amount, "THB");  // ฿
    }
    
    public static String formatNumber(double number) {
        return NumberFormat.getNumberInstance().format(number);
    }
    
    public static String formatPercent(double fraction) {
        return NumberFormat.getPercentInstance().format(fraction);
        // 0.1234 → "12.34%"
    }
}
```

---

## 59.7 สรุป Part 59

ในบทนี้คุณได้เรียนรู้:

✅ i18n vs l10n concept  
✅ strings.xml แยกตาม locale  
✅ Plurals และ String arrays  
✅ RTL support (start/end attributes)  
✅ LocaleHelper (applyLocale, setLocale)  
✅ Language switcher UI  
✅ Date/number formatting per locale  
✅ Thai Buddhist calendar  

---

*[← Part 58: Accessibility](./part-58-android-accessibility.md) | [Part 60: Advanced Room Database →](./part-60-android-advanced-room.md)*
