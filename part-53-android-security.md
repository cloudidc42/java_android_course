# Part 53: Android Security Best Practices
## หลักสูตร Java & Android Development - ระดับ Professional Android

---

## 53.1 OWASP Mobile Top 10

```
1.  Improper Platform Usage       - misuse of permissions, Intents
2.  Insecure Data Storage         - plaintext credentials, logs
3.  Insecure Communication        - no HTTPS, no cert pinning  
4.  Insecure Authentication       - weak PIN, no biometric
5.  Insufficient Cryptography     - MD5, ECB mode, hardcoded keys
6.  Insecure Authorization        - exposed components
7.  Client Code Quality           - buffer overflow, injection
8.  Code Tampering                - debug checks, root detection
9.  Reverse Engineering           - no obfuscation
10. Extraneous Functionality      - debug logs in production
```

---

## 53.2 Secure Data Storage

```java
// EncryptedPreferences.java
import androidx.security.crypto.*;

public class SecureStorage {
    
    private static SharedPreferences getEncryptedPrefs(Context context) throws Exception {
        MasterKey masterKey = new MasterKey.Builder(context)
            .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
            .build();
        
        return EncryptedSharedPreferences.create(
            context,
            "secure_prefs",
            masterKey,
            EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
            EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
        );
    }
    
    public static void saveCredentials(Context context, String token) {
        try {
            SharedPreferences prefs = getEncryptedPrefs(context);
            prefs.edit().putString("auth_token", token).apply();
        } catch (Exception e) {
            Log.e("SecureStorage", "Error saving credentials", e);
        }
    }
    
    public static String getCredentials(Context context) {
        try {
            SharedPreferences prefs = getEncryptedPrefs(context);
            return prefs.getString("auth_token", null);
        } catch (Exception e) {
            return null;
        }
    }
    
    // Save to Keystore (most secure)
    public static void saveToKeystore(String alias, byte[] data) throws Exception {
        KeyStore keyStore = KeyStore.getInstance("AndroidKeyStore");
        keyStore.load(null);
        
        if (!keyStore.containsAlias(alias)) {
            KeyGenerator keyGen = KeyGenerator.getInstance(
                KeyProperties.KEY_ALGORITHM_AES, "AndroidKeyStore");
            
            keyGen.init(new KeyGenParameterSpec.Builder(alias,
                KeyProperties.PURPOSE_ENCRYPT | KeyProperties.PURPOSE_DECRYPT)
                .setBlockModes(KeyProperties.BLOCK_MODE_GCM)
                .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_NONE)
                .build());
            
            keyGen.generateKey();
        }
        
        SecretKey key = (SecretKey) keyStore.getKey(alias, null);
        Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
        cipher.init(Cipher.ENCRYPT_MODE, key);
        
        byte[] iv         = cipher.getIV();
        byte[] encrypted  = cipher.doFinal(data);
        
        // Store iv + encrypted data together
        // ...
    }
}
```

---

## 53.3 Network Security - Certificate Pinning

```java
// RetrofitWithPinning.java
public class SecureRetrofitClient {
    
    private static Retrofit instance;
    
    public static Retrofit getInstance() {
        if (instance == null) {
            OkHttpClient client = new OkHttpClient.Builder()
                .certificatePinner(buildCertificatePinner())
                .hostnameVerifier((hostname, session) -> {
                    // Verify hostname strictly
                    return HttpsURLConnection.getDefaultHostnameVerifier()
                        .verify("api.example.com", session);
                })
                .build();
            
            instance = new Retrofit.Builder()
                .baseUrl("https://api.example.com/")
                .client(client)
                .addConverterFactory(GsonConverterFactory.create())
                .build();
        }
        return instance;
    }
    
    private static CertificatePinner buildCertificatePinner() {
        // Get hash: openssl s_client -connect api.example.com:443 | openssl x509 -pubkey -noout | openssl pkey -pubin -outform der | openssl dgst -sha256 -binary | base64
        return new CertificatePinner.Builder()
            .add("api.example.com", 
                "sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=")
            .add("api.example.com",
                "sha256/BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB=")  // backup pin
            .build();
    }
}
```

```xml
<!-- res/xml/network_security_config.xml -->
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <!-- Allow cleartext only for debug -->
    <base-config cleartextTrafficPermitted="false">
        <trust-anchors>
            <certificates src="system"/>
        </trust-anchors>
    </base-config>
    
    <!-- Pin certificates for production -->
    <domain-config>
        <domain includeSubdomains="true">api.example.com</domain>
        <pin-set expiration="2025-01-01">
            <pin digest="SHA-256">AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=</pin>
            <pin digest="SHA-256">BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB=</pin>
        </pin-set>
    </domain-config>
    
    <!-- Allow cleartext for debug only -->
    <debug-overrides>
        <trust-anchors>
            <certificates src="user"/>
        </trust-anchors>
    </debug-overrides>
</network-security-config>
```

```xml
<!-- AndroidManifest.xml -->
<application
    android:networkSecurityConfig="@xml/network_security_config"
    ...>
```

---

## 53.4 Biometric Authentication

```java
// BiometricAuthHelper.java
import androidx.biometric.*;

public class BiometricAuthHelper {
    
    private final AppCompatActivity activity;
    
    public interface AuthCallback {
        void onSuccess();
        void onFailed();
        void onError(String message);
    }
    
    public BiometricAuthHelper(AppCompatActivity activity) {
        this.activity = activity;
    }
    
    public boolean isBiometricAvailable() {
        BiometricManager manager = BiometricManager.from(activity);
        int result = manager.canAuthenticate(
            BiometricManager.Authenticators.BIOMETRIC_STRONG |
            BiometricManager.Authenticators.DEVICE_CREDENTIAL);
        return result == BiometricManager.BIOMETRIC_SUCCESS;
    }
    
    public void authenticate(String title, String subtitle, AuthCallback callback) {
        BiometricPrompt.PromptInfo promptInfo = new BiometricPrompt.PromptInfo.Builder()
            .setTitle(title)
            .setSubtitle(subtitle)
            .setAllowedAuthenticators(
                BiometricManager.Authenticators.BIOMETRIC_STRONG |
                BiometricManager.Authenticators.DEVICE_CREDENTIAL)
            .build();
        
        BiometricPrompt biometricPrompt = new BiometricPrompt(activity,
            ContextCompat.getMainExecutor(activity),
            new BiometricPrompt.AuthenticationCallback() {
                
                @Override
                public void onAuthenticationSucceeded(BiometricPrompt.AuthenticationResult result) {
                    super.onAuthenticationSucceeded(result);
                    callback.onSuccess();
                }
                
                @Override
                public void onAuthenticationFailed() {
                    super.onAuthenticationFailed();
                    callback.onFailed();
                }
                
                @Override
                public void onAuthenticationError(int errorCode, CharSequence errString) {
                    super.onAuthenticationError(errorCode, errString);
                    callback.onError(errString.toString());
                }
            });
        
        biometricPrompt.authenticate(promptInfo);
    }
}

// Usage in Activity
biometricHelper.authenticate(
    "ยืนยันตัวตน",
    "ใช้ลายนิ้วมือหรือรหัสหน้าจอ",
    new BiometricAuthHelper.AuthCallback() {
        @Override
        public void onSuccess() {
            openSecureContent();
        }
        @Override
        public void onFailed() {
            Toast.makeText(context, "ยืนยันไม่สำเร็จ", Toast.LENGTH_SHORT).show();
        }
        @Override
        public void onError(String message) {
            Toast.makeText(context, message, Toast.LENGTH_SHORT).show();
        }
    });
```

---

## 53.5 Root Detection

```java
// RootDetector.java
public class RootDetector {
    
    public static boolean isRooted() {
        return checkRootFiles() || checkBuildTags() || checkSuperuserAPK();
    }
    
    private static boolean checkRootFiles() {
        String[] paths = {
            "/system/app/Superuser.apk",
            "/sbin/su", "/system/bin/su",
            "/system/xbin/su",
            "/data/local/xbin/su",
            "/data/local/bin/su",
            "/system/sd/xbin/su",
            "/system/bin/failsafe/su",
            "/data/local/su"
        };
        for (String path : paths) {
            if (new File(path).exists()) return true;
        }
        return false;
    }
    
    private static boolean checkBuildTags() {
        String buildTags = android.os.Build.TAGS;
        return buildTags != null && buildTags.contains("test-keys");
    }
    
    private static boolean checkSuperuserAPK() {
        try {
            Process process = Runtime.getRuntime().exec(new String[]{"which", "su"});
            BufferedReader reader = new BufferedReader(
                new InputStreamReader(process.getInputStream()));
            return reader.readLine() != null;
        } catch (Exception e) {
            return false;
        }
    }
    
    // In MainActivity
    public static void blockIfRooted(Activity activity) {
        if (isRooted()) {
            new AlertDialog.Builder(activity)
                .setTitle("ไม่รองรับอุปกรณ์ Root")
                .setMessage("แอปนี้ไม่รองรับอุปกรณ์ที่ถูก root เพื่อความปลอดภัย")
                .setCancelable(false)
                .setPositiveButton("ปิด", (d, w) -> activity.finish())
                .show();
        }
    }
}
```

---

## 53.6 Input Validation

```java
// SecurityValidator.java
public class SecurityValidator {
    
    // Validate email
    public static boolean isValidEmail(String email) {
        return email != null && 
            android.util.Patterns.EMAIL_ADDRESS.matcher(email).matches();
    }
    
    // Validate phone (Thai format)
    public static boolean isValidThaiPhone(String phone) {
        String cleaned = phone.replaceAll("[\\s-]", "");
        return cleaned.matches("^(0[689]\\d{8}|\\+66[689]\\d{8})$");
    }
    
    // Sanitize input (remove dangerous characters)
    public static String sanitizeInput(String input) {
        if (input == null) return "";
        return input
            .replaceAll("[<>\"'%;()&+]", "")
            .trim();
    }
    
    // Check password strength
    public static PasswordStrength checkPasswordStrength(String password) {
        if (password == null || password.length() < 6) return PasswordStrength.WEAK;
        
        boolean hasUpper   = password.matches(".*[A-Z].*");
        boolean hasLower   = password.matches(".*[a-z].*");
        boolean hasDigit   = password.matches(".*\\d.*");
        boolean hasSpecial = password.matches(".*[!@#$%^&*()_+].*");
        boolean hasLength  = password.length() >= 10;
        
        int score = 0;
        if (hasUpper)   score++;
        if (hasLower)   score++;
        if (hasDigit)   score++;
        if (hasSpecial) score++;
        if (hasLength)  score++;
        
        if (score <= 2) return PasswordStrength.WEAK;
        if (score <= 3) return PasswordStrength.MEDIUM;
        return PasswordStrength.STRONG;
    }
    
    public enum PasswordStrength { WEAK, MEDIUM, STRONG }
    
    // Prevent log leaks
    public static void secureLog(String tag, String message) {
        if (BuildConfig.DEBUG) {
            Log.d(tag, message);
        }
        // Never log in production
    }
}
```

---

## 53.7 Manifest Security

```xml
<!-- AndroidManifest.xml - security best practices -->
<application
    android:allowBackup="false"         <!-- prevent backup of sensitive data -->
    android:debuggable="false"          <!-- never true in production -->
    android:usesCleartextTraffic="false" <!-- force HTTPS -->
    ...>
    
    <!-- Export only what you need to -->
    <activity android:name=".MainActivity"
        android:exported="true">         <!-- Only entry points -->
        <intent-filter>
            <action android:name="android.intent.action.MAIN"/>
            <category android:name="android.intent.category.LAUNCHER"/>
        </intent-filter>
    </activity>
    
    <activity android:name=".DetailActivity"
        android:exported="false"/>       <!-- Internal only -->
    
    <!-- Protect services and receivers -->
    <receiver android:name=".MyReceiver"
        android:exported="false"/>
    
    <!-- FileProvider - never exported=true -->
    <provider
        android:name="androidx.core.content.FileProvider"
        android:exported="false"
        android:grantUriPermissions="true"
        .../>
</application>
```

---

## 53.8 สรุป Part 53

ในบทนี้คุณได้เรียนรู้:

✅ OWASP Mobile Top 10  
✅ EncryptedSharedPreferences  
✅ Android Keystore  
✅ Certificate pinning (OkHttp + network_security_config.xml)  
✅ Biometric authentication  
✅ Root detection  
✅ Input validation and sanitization  
✅ Manifest security (allowBackup, debuggable, exported)  

---

*[← Part 52: Performance](./part-52-android-performance.md) | [Part 54: CI/CD Android →](./part-54-android-cicd.md)*
