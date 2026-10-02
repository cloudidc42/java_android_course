# Part 30: Java Security
## หลักสูตร Java & Android Development - ระดับ Advanced

---

## 30.1 Cryptography พื้นฐาน

```java
import javax.crypto.*;
import javax.crypto.spec.*;
import java.security.*;
import java.util.Base64;

public class CryptoDemo {
    
    // --- Hashing ---
    static String sha256(String input) throws Exception {
        MessageDigest md = MessageDigest.getInstance("SHA-256");
        byte[] hash = md.digest(input.getBytes("UTF-8"));
        StringBuilder sb = new StringBuilder();
        for (byte b : hash) sb.append(String.format("%02x", b));
        return sb.toString();
    }
    
    // Password hashing with salt
    static String hashPassword(String password) throws Exception {
        // Generate random salt
        SecureRandom random = new SecureRandom();
        byte[] salt = new byte[16];
        random.nextBytes(salt);
        
        // PBKDF2 - Password-Based Key Derivation Function
        var spec = new javax.crypto.spec.PBEKeySpec(
            password.toCharArray(), salt, 310_000, 256);
        var factory = SecretKeyFactory.getInstance("PBKDF2WithHmacSHA256");
        byte[] hash = factory.generateSecret(spec).getEncoded();
        
        // Combine salt + hash
        String saltB64 = Base64.getEncoder().encodeToString(salt);
        String hashB64 = Base64.getEncoder().encodeToString(hash);
        return saltB64 + ":" + hashB64;
    }
    
    static boolean verifyPassword(String password, String stored) throws Exception {
        String[] parts = stored.split(":");
        byte[] salt = Base64.getDecoder().decode(parts[0]);
        byte[] expectedHash = Base64.getDecoder().decode(parts[1]);
        
        var spec = new javax.crypto.spec.PBEKeySpec(
            password.toCharArray(), salt, 310_000, 256);
        var factory = SecretKeyFactory.getInstance("PBKDF2WithHmacSHA256");
        byte[] hash = factory.generateSecret(spec).getEncoded();
        
        return MessageDigest.isEqual(hash, expectedHash);
    }
    
    // --- AES Symmetric Encryption ---
    static SecretKey generateAESKey() throws Exception {
        KeyGenerator kg = KeyGenerator.getInstance("AES");
        kg.init(256);  // 128, 192, or 256 bits
        return kg.generateKey();
    }
    
    static byte[] encrypt(String plaintext, SecretKey key) throws Exception {
        Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
        
        // Random IV (Initialization Vector)
        byte[] iv = new byte[12];
        new SecureRandom().nextBytes(iv);
        cipher.init(Cipher.ENCRYPT_MODE, key, new GCMParameterSpec(128, iv));
        
        byte[] ciphertext = cipher.doFinal(plaintext.getBytes("UTF-8"));
        
        // Prepend IV to ciphertext
        byte[] result = new byte[iv.length + ciphertext.length];
        System.arraycopy(iv, 0, result, 0, iv.length);
        System.arraycopy(ciphertext, 0, result, iv.length, ciphertext.length);
        return result;
    }
    
    static String decrypt(byte[] ivAndCiphertext, SecretKey key) throws Exception {
        // Extract IV and ciphertext
        byte[] iv = new byte[12];
        byte[] ciphertext = new byte[ivAndCiphertext.length - 12];
        System.arraycopy(ivAndCiphertext, 0, iv, 0, 12);
        System.arraycopy(ivAndCiphertext, 12, ciphertext, 0, ciphertext.length);
        
        Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
        cipher.init(Cipher.DECRYPT_MODE, key, new GCMParameterSpec(128, iv));
        return new String(cipher.doFinal(ciphertext), "UTF-8");
    }
    
    // --- RSA Asymmetric Encryption ---
    static KeyPair generateRSAKeyPair() throws Exception {
        KeyPairGenerator kpg = KeyPairGenerator.getInstance("RSA");
        kpg.initialize(2048);
        return kpg.generateKeyPair();
    }
    
    static byte[] rsaEncrypt(String message, PublicKey publicKey) throws Exception {
        Cipher cipher = Cipher.getInstance("RSA/ECB/OAEPWithSHA-256AndMGF1Padding");
        cipher.init(Cipher.ENCRYPT_MODE, publicKey);
        return cipher.doFinal(message.getBytes("UTF-8"));
    }
    
    static String rsaDecrypt(byte[] ciphertext, PrivateKey privateKey) throws Exception {
        Cipher cipher = Cipher.getInstance("RSA/ECB/OAEPWithSHA-256AndMGF1Padding");
        cipher.init(Cipher.DECRYPT_MODE, privateKey);
        return new String(cipher.doFinal(ciphertext), "UTF-8");
    }
    
    public static void main(String[] args) throws Exception {
        // Hashing
        System.out.println("=== Hashing ===");
        System.out.println("SHA-256: " + sha256("Hello World"));
        
        String stored = hashPassword("myPassword123!");
        System.out.println("Stored: " + stored.substring(0, 30) + "...");
        System.out.println("Valid:  " + verifyPassword("myPassword123!", stored));
        System.out.println("Wrong:  " + verifyPassword("wrongPassword", stored));
        
        // AES
        System.out.println("\n=== AES Encryption ===");
        SecretKey aesKey = generateAESKey();
        String original = "Sensitive data: บัตรเครดิต 1234-5678";
        byte[] encrypted = encrypt(original, aesKey);
        String decrypted = decrypt(encrypted, aesKey);
        System.out.println("Original:  " + original);
        System.out.println("Encrypted: " + Base64.getEncoder().encodeToString(encrypted).substring(0, 40) + "...");
        System.out.println("Decrypted: " + decrypted);
        
        // RSA
        System.out.println("\n=== RSA Encryption ===");
        KeyPair rsaKeys = generateRSAKeyPair();
        byte[] rsaEncrypted = rsaEncrypt("Secret message", rsaKeys.getPublic());
        String rsaDecrypted = rsaDecrypt(rsaEncrypted, rsaKeys.getPrivate());
        System.out.println("RSA Decrypted: " + rsaDecrypted);
    }
}
```

---

## 30.2 Secure Coding Practices

```java
import java.sql.*;
import java.util.regex.*;

public class SecureCoding {
    
    // 1. SQL Injection Prevention - ALWAYS use PreparedStatement
    static void safeQuery(Connection conn, String userInput) throws SQLException {
        // UNSAFE - SQL Injection vulnerable:
        // Statement stmt = conn.createStatement();
        // ResultSet rs = stmt.executeQuery("SELECT * FROM users WHERE name = '" + userInput + "'");
        
        // SAFE - PreparedStatement:
        PreparedStatement ps = conn.prepareStatement(
            "SELECT * FROM users WHERE name = ?");
        ps.setString(1, userInput);  // input is parameterized, not interpolated
        ResultSet rs = ps.executeQuery();
    }
    
    // 2. Input Validation
    static class InputValidator {
        private static final Pattern EMAIL = Pattern.compile(
            "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$");
        private static final Pattern PHONE = Pattern.compile("^[0-9]{9,10}$");
        private static final Pattern USERNAME = Pattern.compile("^[a-zA-Z0-9_]{3,20}$");
        
        static String sanitizeHtml(String input) {
            if (input == null) return null;
            return input
                .replace("&", "&amp;")
                .replace("<", "&lt;")
                .replace(">", "&gt;")
                .replace("\"", "&quot;")
                .replace("'", "&#x27;");
        }
        
        static boolean isValidEmail(String email) {
            return email != null && EMAIL.matcher(email).matches();
        }
        
        static boolean isValidPhone(String phone) {
            return phone != null && PHONE.matcher(phone).matches();
        }
        
        static boolean isValidUsername(String username) {
            return username != null && USERNAME.matcher(username).matches();
        }
        
        // Limit string length to prevent DoS
        static String truncate(String s, int maxLen) {
            if (s == null) return null;
            return s.length() <= maxLen ? s : s.substring(0, maxLen);
        }
    }
    
    // 3. Secrets Management - NEVER hardcode secrets
    static class Config {
        // BAD:
        // static final String DB_PASSWORD = "mypassword123";
        
        // GOOD: Read from environment variables
        static String getDbPassword() {
            String pwd = System.getenv("DB_PASSWORD");
            if (pwd == null || pwd.isBlank()) {
                throw new IllegalStateException("DB_PASSWORD not set");
            }
            return pwd;
        }
        
        // Or from properties file (outside the JAR)
        static String getApiKey() {
            return System.getProperty("api.key",
                System.getenv("API_KEY"));
        }
    }
    
    // 4. Secure Random
    static String generateToken() {
        byte[] bytes = new byte[32];  // 256 bits
        new java.security.SecureRandom().nextBytes(bytes);  // cryptographically secure
        return java.util.Base64.getUrlEncoder().withoutPadding().encodeToString(bytes);
    }
    
    // 5. Path Traversal Prevention
    static java.io.File safeFile(String baseDir, String fileName) {
        java.io.File base = new java.io.File(baseDir).getAbsoluteFile();
        java.io.File file = new java.io.File(base, fileName).getAbsoluteFile();
        
        // Verify the file is within the base directory
        if (!file.getPath().startsWith(base.getPath() + java.io.File.separator)) {
            throw new SecurityException("Path traversal attempt: " + fileName);
        }
        return file;
    }
    
    public static void main(String[] args) {
        System.out.println("=== Input Validation ===");
        System.out.println("Email valid: " + InputValidator.isValidEmail("test@example.com"));
        System.out.println("Email invalid: " + InputValidator.isValidEmail("not-an-email"));
        System.out.println("Username valid: " + InputValidator.isValidUsername("alice123"));
        System.out.println("Username invalid: " + InputValidator.isValidUsername("a<script>"));
        
        System.out.println("\n=== HTML Sanitization ===");
        String unsafe = "<script>alert('XSS')</script>";
        System.out.println("Sanitized: " + InputValidator.sanitizeHtml(unsafe));
        
        System.out.println("\n=== Secure Token ===");
        System.out.println("Token: " + generateToken());
        
        System.out.println("\n=== Path Traversal ===");
        try {
            safeFile("/var/www", "../etc/passwd");  // should fail
        } catch (SecurityException e) {
            System.out.println("Blocked: " + e.getMessage());
        }
        java.io.File safe = safeFile("/var/www", "index.html");
        System.out.println("Safe file: " + safe.getPath());
    }
}
```

---

## 30.3 JWT (JSON Web Token)

```java
// Manual JWT implementation (in practice use a library like java-jwt or jjwt)
import java.util.Base64;
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.nio.charset.StandardCharsets;

public class JWTDemo {
    
    static String createJWT(String subject, long expiresInMs, String secret) throws Exception {
        // Header
        String header = Base64.getUrlEncoder().withoutPadding().encodeToString(
            "{\"alg\":\"HS256\",\"typ\":\"JWT\"}".getBytes(StandardCharsets.UTF_8));
        
        // Payload
        long now = System.currentTimeMillis() / 1000;
        long exp = now + expiresInMs / 1000;
        String payloadJson = String.format(
            "{\"sub\":\"%s\",\"iat\":%d,\"exp\":%d}", subject, now, exp);
        String payload = Base64.getUrlEncoder().withoutPadding().encodeToString(
            payloadJson.getBytes(StandardCharsets.UTF_8));
        
        // Signature
        String data = header + "." + payload;
        Mac mac = Mac.getInstance("HmacSHA256");
        mac.init(new SecretKeySpec(secret.getBytes(StandardCharsets.UTF_8), "HmacSHA256"));
        String signature = Base64.getUrlEncoder().withoutPadding().encodeToString(
            mac.doFinal(data.getBytes(StandardCharsets.UTF_8)));
        
        return header + "." + payload + "." + signature;
    }
    
    static boolean verifyJWT(String token, String secret) throws Exception {
        String[] parts = token.split("\\.");
        if (parts.length != 3) return false;
        
        String data = parts[0] + "." + parts[1];
        Mac mac = Mac.getInstance("HmacSHA256");
        mac.init(new SecretKeySpec(secret.getBytes(StandardCharsets.UTF_8), "HmacSHA256"));
        String expected = Base64.getUrlEncoder().withoutPadding().encodeToString(
            mac.doFinal(data.getBytes(StandardCharsets.UTF_8)));
        
        // Constant-time comparison to prevent timing attacks
        return MessageDigest.isEqual(expected.getBytes(), parts[2].getBytes());
    }
    
    static String getSubject(String token) {
        String[] parts = token.split("\\.");
        String payload = new String(Base64.getUrlDecoder().decode(parts[1]));
        // Simple extraction - use a proper JSON library in production
        int start = payload.indexOf("\"sub\":\"") + 7;
        int end = payload.indexOf("\"", start);
        return payload.substring(start, end);
    }
    
    public static void main(String[] args) throws Exception {
        String secret = "my-super-secret-key-32-chars-min!";
        String token = createJWT("user-123", 3600_000, secret);  // 1 hour
        
        System.out.println("JWT: " + token);
        System.out.println("Valid: " + verifyJWT(token, secret));
        System.out.println("Subject: " + getSubject(token));
        System.out.println("Invalid: " + verifyJWT(token + "tampered", secret));
    }
}
```

---

## 30.4 สรุป Part 30

ในบทนี้คุณได้เรียนรู้:

✅ SHA-256 hashing  
✅ Password hashing (PBKDF2 with salt)  
✅ AES symmetric encryption (GCM mode)  
✅ RSA asymmetric encryption  
✅ SQL injection prevention (PreparedStatement)  
✅ Input validation and sanitization  
✅ Secure secrets management  
✅ Path traversal prevention  
✅ JWT implementation  

---

*[← Part 29: Logging](./part-29-logging.md) | [Part 31: Android Fundamentals →](./part-31-android-intro.md)*
