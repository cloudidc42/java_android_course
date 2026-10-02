# Part 81: Android Security
## หลักสูตร Java & Android Development - ระดับ World-Class

---

## 81.1 Encrypted SharedPreferences

```groovy
// build.gradle
implementation "androidx.security:security-crypto:1.1.0-alpha06"
```

```java
// SecurePrefsManager.java
public class SecurePrefsManager {
    
    private final SharedPreferences securePrefs;
    
    public SecurePrefsManager(Context context) throws Exception {
        // Generate/get master key
        MasterKey masterKey = new MasterKey.Builder(context)
            .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
            .build();
        
        // Create encrypted shared preferences
        securePrefs = EncryptedSharedPreferences.create(
            context,
            "secure_prefs",         // file name
            masterKey,
            EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,   // key encryption
            EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM  // value encryption
        );
    }
    
    public void saveToken(String token) {
        securePrefs.edit().putString("auth_token", token).apply();
    }
    
    public String getToken() {
        return securePrefs.getString("auth_token", null);
    }
    
    public void clearToken() {
        securePrefs.edit().remove("auth_token").apply();
    }
    
    public void saveUserCredentials(String email, String refreshToken) {
        securePrefs.edit()
            .putString("user_email", email)
            .putString("refresh_token", refreshToken)
            .apply();
    }
}
```

---

## 81.2 Android Keystore - Store Cryptographic Keys

```java
// KeystoreHelper.java - AES-GCM encryption with Keystore-backed key
public class KeystoreHelper {
    
    private static final String KEY_ALIAS = "MyAppEncryptionKey";
    private static final String ANDROID_KEYSTORE = "AndroidKeyStore";
    
    // Generate or retrieve key
    private SecretKey getOrCreateKey() throws Exception {
        KeyStore keyStore = KeyStore.getInstance(ANDROID_KEYSTORE);
        keyStore.load(null);
        
        if (!keyStore.containsAlias(KEY_ALIAS)) {
            KeyGenerator keyGenerator = KeyGenerator.getInstance(
                KeyProperties.KEY_ALGORITHM_AES, ANDROID_KEYSTORE);
            
            KeyGenParameterSpec spec = new KeyGenParameterSpec.Builder(
                KEY_ALIAS,
                KeyProperties.PURPOSE_ENCRYPT | KeyProperties.PURPOSE_DECRYPT)
                .setBlockModes(KeyProperties.BLOCK_MODE_GCM)
                .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_NONE)
                .setKeySize(256)
                .setUserAuthenticationRequired(false)  // no biometric required
                .build();
            
            keyGenerator.init(spec);
            keyGenerator.generateKey();
        }
        
        return ((KeyStore.SecretKeyEntry) keyStore.getEntry(KEY_ALIAS, null)).getSecretKey();
    }
    
    // Encrypt data
    public byte[] encrypt(String plaintext) throws Exception {
        Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
        cipher.init(Cipher.ENCRYPT_MODE, getOrCreateKey());
        
        byte[] iv         = cipher.getIV();
        byte[] ciphertext = cipher.doFinal(plaintext.getBytes(StandardCharsets.UTF_8));
        
        // Prepend IV to ciphertext
        byte[] combined = new byte[iv.length + ciphertext.length];
        System.arraycopy(iv, 0, combined, 0, iv.length);
        System.arraycopy(ciphertext, 0, combined, iv.length, ciphertext.length);
        return combined;
    }
    
    // Decrypt data
    public String decrypt(byte[] combined) throws Exception {
        // Extract IV (first 12 bytes for GCM)
        byte[] iv         = Arrays.copyOfRange(combined, 0, 12);
        byte[] ciphertext = Arrays.copyOfRange(combined, 12, combined.length);
        
        Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
        GCMParameterSpec spec = new GCMParameterSpec(128, iv);
        cipher.init(Cipher.DECRYPT_MODE, getOrCreateKey(), spec);
        
        byte[] plaintext = cipher.doFinal(ciphertext);
        return new String(plaintext, StandardCharsets.UTF_8);
    }
    
    // Store as Base64 string
    public String encryptToString(String plaintext) throws Exception {
        return Base64.encodeToString(encrypt(plaintext), Base64.NO_WRAP);
    }
    
    public String decryptFromString(String encoded) throws Exception {
        return decrypt(Base64.decode(encoded, Base64.NO_WRAP));
    }
}
```

---

## 81.3 Biometric Authentication

```groovy
implementation "androidx.biometric:biometric:1.2.0-alpha05"
```

```java
// BiometricHelper.java
public class BiometricHelper {
    
    public static boolean isAvailable(Context context) {
        BiometricManager bm = BiometricManager.from(context);
        return bm.canAuthenticate(BiometricManager.Authenticators.BIOMETRIC_STRONG)
            == BiometricManager.BIOMETRIC_SUCCESS;
    }
    
    public static void authenticate(FragmentActivity activity,
            String title, String subtitle,
            Runnable onSuccess, Consumer<String> onError) {
        
        BiometricPrompt.PromptInfo promptInfo = new BiometricPrompt.PromptInfo.Builder()
            .setTitle(title)
            .setSubtitle(subtitle)
            .setNegativeButtonText("ยกเลิก")
            .setAllowedAuthenticators(BiometricManager.Authenticators.BIOMETRIC_STRONG)
            .build();
        
        BiometricPrompt biometricPrompt = new BiometricPrompt(activity,
            ContextCompat.getMainExecutor(activity),
            new BiometricPrompt.AuthenticationCallback() {
                @Override
                public void onAuthenticationSucceeded(@NonNull BiometricPrompt.AuthenticationResult result) {
                    onSuccess.run();
                }
                
                @Override
                public void onAuthenticationError(int errorCode, @NonNull CharSequence errString) {
                    onError.accept(errString.toString());
                }
                
                @Override
                public void onAuthenticationFailed() {
                    onError.accept("การยืนยันตัวตนล้มเหลว");
                }
            });
        
        biometricPrompt.authenticate(promptInfo);
    }
    
    // Combine biometric with Keystore (CryptoObject)
    public static void authenticateWithCrypto(FragmentActivity activity,
            Cipher cipher, Runnable onSuccess, Consumer<String> onError) {
        
        BiometricPrompt.PromptInfo promptInfo = new BiometricPrompt.PromptInfo.Builder()
            .setTitle("ยืนยันตัวตน")
            .setSubtitle("ใช้ลายนิ้วมือเพื่อเข้าถึงข้อมูล")
            .setNegativeButtonText("ยกเลิก")
            .setAllowedAuthenticators(BiometricManager.Authenticators.BIOMETRIC_STRONG)
            .build();
        
        BiometricPrompt prompt = new BiometricPrompt(activity,
            ContextCompat.getMainExecutor(activity),
            new BiometricPrompt.AuthenticationCallback() {
                @Override
                public void onAuthenticationSucceeded(@NonNull BiometricPrompt.AuthenticationResult result) {
                    // cipher is now authorized by biometric
                    onSuccess.run();
                }
                
                @Override
                public void onAuthenticationError(int code, @NonNull CharSequence msg) {
                    onError.accept(msg.toString());
                }
            });
        
        prompt.authenticate(promptInfo, new BiometricPrompt.CryptoObject(cipher));
    }
}

// Usage in Fragment
if (BiometricHelper.isAvailable(requireContext())) {
    BiometricHelper.authenticate(
        requireActivity(),
        "เข้าสู่ระบบ",
        "ใช้ลายนิ้วมือเพื่อเข้าสู่ระบบ",
        () -> {
            // Success - navigate to home
            Navigation.findNavController(requireView())
                .navigate(R.id.action_login_to_home);
        },
        errorMsg -> {
            Toast.makeText(requireContext(), errorMsg, Toast.LENGTH_SHORT).show();
        });
}
```

---

## 81.4 Network Security Config

```xml
<!-- res/xml/network_security_config.xml -->
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    
    <!-- Only allow HTTPS, disable cleartext for all domains -->
    <base-config cleartextTrafficPermitted="false">
        <trust-anchors>
            <certificates src="system" />
        </trust-anchors>
    </base-config>
    
    <!-- Certificate pinning for production API -->
    <domain-config cleartextTrafficPermitted="false">
        <domain includeSubdomains="true">api.myapp.com</domain>
        <pin-set expiration="2026-01-01">
            <!-- SHA-256 of SubjectPublicKeyInfo -->
            <pin digest="SHA-256">AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=</pin>
            <!-- Backup pin -->
            <pin digest="SHA-256">BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB=</pin>
        </pin-set>
    </domain-config>
    
    <!-- Allow debug builds to connect to localhost -->
    <debug-overrides>
        <trust-anchors>
            <certificates src="user" />
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

## 81.5 สรุป Part 81

ในบทนี้คุณได้เรียนรู้:

✅ EncryptedSharedPreferences (AES256-GCM)  
✅ Android Keystore (hardware-backed keys)  
✅ AES-GCM encrypt/decrypt with Keystore  
✅ Biometric authentication (fingerprint/face)  
✅ BiometricPrompt + CryptoObject  
✅ Network Security Config (Certificate Pinning)  
✅ Disable cleartext traffic  

---

*[← Part 80: Monetization](./part-80-android-monetization.md) | [Part 82: Advanced Camera & Media →](./part-82-android-camera-media.md)*
