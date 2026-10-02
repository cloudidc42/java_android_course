# Part 07: Strings
## หลักสูตร Java & Android Development - ระดับเริ่มต้น

---

## 7.1 String ใน Java

String เป็น class ที่สำคัญมากใน Java ไม่ใช่ primitive type แต่เป็น object ที่ immutable (ไม่เปลี่ยนแปลงได้)

```java
// String pool
String s1 = "Hello";        // String literal -> String pool
String s2 = "Hello";        // ชี้ไปที่ object เดิมใน pool
String s3 = new String("Hello"); // สร้าง object ใหม่ใน heap

System.out.println(s1 == s2);      // true (same reference in pool)
System.out.println(s1 == s3);      // false (different objects)
System.out.println(s1.equals(s3)); // true (same content)
```

---

## 7.2 String Methods ที่สำคัญ

```java
public class StringMethods {
    public static void main(String[] args) {
        String s = "  Hello, Java World!  ";
        
        // ความยาวและตำแหน่ง
        System.out.println("length: " + s.length());          // 22
        System.out.println("isEmpty: " + "".isEmpty());       // true
        System.out.println("isBlank: " + "   ".isBlank());    // true (Java 11+)
        
        // ค้นหา
        System.out.println("indexOf('o'): " + s.indexOf('o'));    // 6
        System.out.println("indexOf(\"Java\"): " + s.indexOf("Java")); // 9
        System.out.println("lastIndexOf('o'): " + s.lastIndexOf('o')); // 15
        System.out.println("contains(\"World\"): " + s.contains("World")); // true
        System.out.println("startsWith(\"  He\"): " + s.startsWith("  He")); // true
        System.out.println("endsWith(\"!  \"): " + s.endsWith("!  ")); // true
        
        // ดึงส่วนของ String
        System.out.println("charAt(7): " + s.charAt(7));         // H (after trim)
        System.out.println("substring(7): " + s.substring(7));   // Hello, Java World!  
        System.out.println("substring(9,13): " + s.substring(9, 13)); // Java
        
        // แปลง
        System.out.println("toUpperCase: " + s.toUpperCase());
        System.out.println("toLowerCase: " + s.toLowerCase());
        System.out.println("trim: '" + s.trim() + "'");              // ตัด spaces หัว/ท้าย
        System.out.println("strip: '" + s.strip() + "'");            // Java 11+ (รองรับ Unicode)
        System.out.println("stripLeading: '" + s.stripLeading() + "'"); // ตัดซ้าย
        System.out.println("stripTrailing: '" + s.stripTrailing() + "'"); // ตัดขวา
        
        // แทนที่
        System.out.println("replace: " + s.replace("Java", "Python"));
        System.out.println("replaceAll: " + s.replaceAll("\\s+", " ")); // regex
        System.out.println("replaceFirst: " + s.replaceFirst("o", "0"));
        
        // แบ่ง
        String csv = "one,two,three,four";
        String[] parts = csv.split(",");
        System.out.println("split count: " + parts.length);  // 4
        for (String part : parts) System.out.println("  " + part);
        
        // เปรียบเทียบ
        String a = "Apple", b = "Banana";
        System.out.println("compareTo: " + a.compareTo(b));     // negative (A < B)
        System.out.println("equalsIgnoreCase: " + a.equalsIgnoreCase("APPLE")); // true
        
        // ตรวจสอบ
        System.out.println("matches: " + "12345".matches("\\d+"));  // true
        System.out.println("matches: " + "12a45".matches("\\d+"));  // false
    }
}
```

---

## 7.3 String Formatting

```java
public class StringFormatting {
    public static void main(String[] args) {
        
        // String.format
        String name = "Alice";
        int age = 25;
        double salary = 55000.75;
        
        String info = String.format("%-10s %3d ยอด%,10.2f บาท", name, age, salary);
        System.out.println(info);
        
        // Format specifiers
        System.out.printf("%-15s = left-align%n", "hello");
        System.out.printf("%15s = right-align%n", "hello");
        System.out.printf("%+d%n", 42);       // +42
        System.out.printf("%+d%n", -42);      // -42
        System.out.printf("%05d%n", 42);      // 00042 (zero-padding)
        System.out.printf("%,d%n", 1234567);  // 1,234,567 (thousands separator)
        System.out.printf("%.5f%n", Math.PI); // 3.14159
        System.out.printf("%e%n", 123456.789); // scientific notation
        System.out.printf("%X%n", 255);        // FF (hex uppercase)
        System.out.printf("%o%n", 8);          // 10 (octal)
        
        // Text blocks (Java 15+)
        String json = """
                {
                    "name": "Alice",
                    "age": 25,
                    "city": "Bangkok"
                }
                """;
        System.out.println(json);
        
        String html = """
                <html>
                    <body>
                        <h1>Hello, %s!</h1>
                    </body>
                </html>
                """.formatted(name);
        System.out.println(html);
    }
}
```

---

## 7.4 StringBuilder และ StringBuffer

```java
public class StringBuilderDemo {
    public static void main(String[] args) {
        
        // String เป็น immutable ทุกการ + สร้าง object ใหม่
        // ถ้าต้อง concat เยอะๆ ใช้ StringBuilder แทน
        
        // Performance test
        long start = System.currentTimeMillis();
        String s = "";
        for (int i = 0; i < 10000; i++) {
            s += i;  // สร้าง String ใหม่ทุกครั้ง!
        }
        long timeString = System.currentTimeMillis() - start;
        
        start = System.currentTimeMillis();
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < 10000; i++) {
            sb.append(i);  // แก้ไข object เดิม
        }
        long timeSB = System.currentTimeMillis() - start;
        
        System.out.println("String concat: " + timeString + " ms");
        System.out.println("StringBuilder: " + timeSB + " ms");
        
        // StringBuilder methods
        StringBuilder builder = new StringBuilder("Hello");
        System.out.println("\nInitial: " + builder);
        
        builder.append(", World");
        System.out.println("append: " + builder);
        
        builder.insert(5, " Beautiful");
        System.out.println("insert(5): " + builder);
        
        builder.delete(5, 15);
        System.out.println("delete(5,15): " + builder);
        
        builder.replace(0, 5, "Hi");
        System.out.println("replace: " + builder);
        
        builder.reverse();
        System.out.println("reverse: " + builder);
        
        builder.deleteCharAt(0);
        System.out.println("deleteCharAt(0): " + builder);
        
        System.out.println("length: " + builder.length());
        System.out.println("charAt(0): " + builder.charAt(0));
        System.out.println("indexOf(\"W\"): " + builder.indexOf("W"));
        System.out.println("toString: " + builder.toString());
        
        // Method chaining
        String result = new StringBuilder()
            .append("Hello")
            .append(", ")
            .append("World")
            .append("!")
            .toString();
        System.out.println("\nChaining: " + result);
        
        // StringBuffer (thread-safe แต่ช้ากว่า StringBuilder)
        StringBuffer buffer = new StringBuffer("Thread-safe");
        buffer.append(" text");
        System.out.println("StringBuffer: " + buffer);
    }
}
```

---

## 7.5 Regular Expressions

```java
import java.util.regex.*;

public class RegexDemo {
    public static void main(String[] args) {
        
        // การใช้ matches()
        System.out.println("--- matches() ---");
        System.out.println("digits: " + "12345".matches("\\d+"));           // true
        System.out.println("email: " + "test@example.com".matches("[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}"));
        System.out.println("phone: " + "0812345678".matches("0[0-9]{9}"));  // true
        
        // Pattern และ Matcher
        System.out.println("\n--- Pattern & Matcher ---");
        String text = "Contact: john@email.com or jane@work.org";
        Pattern emailPattern = Pattern.compile("[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}");
        Matcher matcher = emailPattern.matcher(text);
        
        System.out.println("Emails found:");
        while (matcher.find()) {
            System.out.println("  " + matcher.group());
        }
        
        // Groups
        System.out.println("\n--- Groups ---");
        String dateStr = "Today is 2024-01-15 and tomorrow is 2024-01-16";
        Pattern datePattern = Pattern.compile("(\\d{4})-(\\d{2})-(\\d{2})");
        Matcher dateMatcher = datePattern.matcher(dateStr);
        
        while (dateMatcher.find()) {
            System.out.printf("Date: %s, Year: %s, Month: %s, Day: %s%n",
                dateMatcher.group(0),
                dateMatcher.group(1),
                dateMatcher.group(2),
                dateMatcher.group(3));
        }
        
        // replaceAll กับ regex
        System.out.println("\n--- replaceAll ---");
        String dirty = "Hello   World\t\t  Java  ";
        String clean = dirty.replaceAll("\\s+", " ").trim();
        System.out.println("Cleaned: '" + clean + "'");
        
        // Validation methods
        System.out.println("\n--- Validation ---");
        String[] inputs = {"test@gmail.com", "invalid-email", "0812345678", "abc"};
        for (String input : inputs) {
            System.out.printf("%-20s -> email:%b, phone:%b%n",
                input,
                input.matches("[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}"),
                input.matches("0[0-9]{9}"));
        }
    }
}
```

---

## 7.6 String.join และ String.valueOf

```java
import java.util.List;

public class StringJoinDemo {
    public static void main(String[] args) {
        
        // String.join
        String joined = String.join(", ", "Apple", "Banana", "Cherry");
        System.out.println("join: " + joined);  // Apple, Banana, Cherry
        
        // join กับ array
        String[] fruits = {"Mango", "Papaya", "Guava"};
        String joinedArr = String.join(" | ", fruits);
        System.out.println("joinArr: " + joinedArr);
        
        // join กับ List
        List<String> names = List.of("Alice", "Bob", "Carol");
        String joinedList = String.join(", ", names);
        System.out.println("joinList: " + joinedList);
        
        // String.valueOf
        System.out.println("\n--- String.valueOf ---");
        System.out.println(String.valueOf(42));        // 42
        System.out.println(String.valueOf(3.14));      // 3.14
        System.out.println(String.valueOf(true));      // true
        System.out.println(String.valueOf('A'));        // A
        System.out.println(String.valueOf((Object)null)); // null (ไม่ throw NPE)
        
        // char array to String
        char[] chars = {'H', 'e', 'l', 'l', 'o'};
        String fromChars = String.valueOf(chars);
        System.out.println("from chars: " + fromChars);
        
        // String to char array
        String str = "Hello";
        char[] charArr = str.toCharArray();
        System.out.print("to chars: ");
        for (char c : charArr) System.out.print(c + " ");
        System.out.println();
        
        // repeat (Java 11+)
        System.out.println("\n--- repeat ---");
        System.out.println("=".repeat(20));
        System.out.println("Ha".repeat(5));  // HaHaHaHaHa
        
        // strip (Java 11+) - Unicode-aware
        String withUnicode = " Hello ";  // Unicode spaces
        System.out.println("trim: '" + withUnicode.trim() + "'");    // ไม่ตัด Unicode space
        System.out.println("strip: '" + withUnicode.strip() + "'");  // ตัด Unicode space
        
        // lines (Java 11+)
        System.out.println("\n--- lines ---");
        String multiline = "Line 1\nLine 2\nLine 3";
        multiline.lines().forEach(System.out::println);
        System.out.println("Count: " + multiline.lines().count());
    }
}
```

---

## 7.7 โปรแกรมตัวอย่าง: Text Processor

```java
import java.util.Scanner;
import java.util.*;

public class TextProcessor {
    
    static int countWords(String text) {
        if (text.isBlank()) return 0;
        return text.trim().split("\\s+").length;
    }
    
    static int countSentences(String text) {
        return text.split("[.!?]+").length;
    }
    
    static Map<Character, Integer> charFrequency(String text) {
        Map<Character, Integer> freq = new TreeMap<>();
        for (char c : text.toLowerCase().toCharArray()) {
            if (Character.isLetter(c)) {
                freq.put(c, freq.getOrDefault(c, 0) + 1);
            }
        }
        return freq;
    }
    
    static String reverseWords(String text) {
        String[] words = text.split("\\s+");
        StringBuilder sb = new StringBuilder();
        for (int i = words.length - 1; i >= 0; i--) {
            sb.append(words[i]);
            if (i > 0) sb.append(" ");
        }
        return sb.toString();
    }
    
    static boolean isPalindrome(String s) {
        String cleaned = s.toLowerCase().replaceAll("[^a-zA-Z0-9]", "");
        String reversed = new StringBuilder(cleaned).reverse().toString();
        return cleaned.equals(reversed);
    }
    
    static String titleCase(String s) {
        String[] words = s.toLowerCase().split("\\s+");
        StringBuilder sb = new StringBuilder();
        for (String word : words) {
            if (!word.isEmpty()) {
                sb.append(Character.toUpperCase(word.charAt(0)));
                sb.append(word.substring(1)).append(" ");
            }
        }
        return sb.toString().trim();
    }
    
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        
        System.out.println("=== Text Processor ===");
        System.out.print("ใส่ข้อความ: ");
        String text = sc.nextLine();
        
        System.out.println("\n--- ผลการวิเคราะห์ ---");
        System.out.println("ข้อความต้นฉบับ : " + text);
        System.out.println("ตัวอักษร       : " + text.length());
        System.out.println("คำ             : " + countWords(text));
        System.out.println("ประโยค         : " + countSentences(text));
        System.out.println("UPPERCASE      : " + text.toUpperCase());
        System.out.println("lowercase      : " + text.toLowerCase());
        System.out.println("Title Case     : " + titleCase(text));
        System.out.println("Reverse Words  : " + reverseWords(text));
        System.out.println("Palindrome?    : " + isPalindrome(text));
        
        System.out.println("\n--- ความถี่ตัวอักษร ---");
        Map<Character, Integer> freq = charFrequency(text);
        freq.entrySet().stream()
            .sorted(Map.Entry.<Character, Integer>comparingByValue().reversed())
            .limit(5)
            .forEach(e -> System.out.printf("'%c': %d ครั้ง%n", e.getKey(), e.getValue()));
        
        sc.close();
    }
}
```

---

## 7.8 โปรแกรมตัวอย่าง: Password Validator

```java
import java.util.Scanner;

public class PasswordValidator {
    
    static String validate(String password) {
        StringBuilder issues = new StringBuilder();
        
        if (password.length() < 8)
            issues.append("- ต้องมีอย่างน้อย 8 ตัวอักษร\n");
        
        if (!password.matches(".*[A-Z].*"))
            issues.append("- ต้องมีตัวใหญ่อย่างน้อย 1 ตัว\n");
        
        if (!password.matches(".*[a-z].*"))
            issues.append("- ต้องมีตัวเล็กอย่างน้อย 1 ตัว\n");
        
        if (!password.matches(".*\\d.*"))
            issues.append("- ต้องมีตัวเลขอย่างน้อย 1 ตัว\n");
        
        if (!password.matches(".*[!@#$%^&*()_+].*"))
            issues.append("- ต้องมีอักขระพิเศษ (!@#$%^&*) อย่างน้อย 1 ตัว\n");
        
        if (password.contains(" "))
            issues.append("- ไม่ควรมีช่องว่าง\n");
        
        return issues.toString();
    }
    
    static String getStrength(String password) {
        int score = 0;
        if (password.length() >= 8) score++;
        if (password.length() >= 12) score++;
        if (password.matches(".*[A-Z].*")) score++;
        if (password.matches(".*[a-z].*")) score++;
        if (password.matches(".*\\d.*")) score++;
        if (password.matches(".*[!@#$%^&*].*")) score++;
        if (password.length() >= 16) score++;
        
        if (score <= 2) return "อ่อนมาก 🔴";
        if (score <= 4) return "อ่อน 🟡";
        if (score <= 5) return "ปานกลาง 🟠";
        if (score <= 6) return "แข็งแกร่ง 🟢";
        return "แข็งแกร่งมาก 💚";
    }
    
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        
        System.out.println("=== Password Validator ===");
        System.out.print("ใส่รหัสผ่าน: ");
        String password = sc.nextLine();
        
        String issues = validate(password);
        
        System.out.println("\n--- ผลการตรวจสอบ ---");
        if (issues.isEmpty()) {
            System.out.println("✓ รหัสผ่านถูกต้องตามเงื่อนไขทั้งหมด!");
        } else {
            System.out.println("✗ พบปัญหาดังนี้:");
            System.out.print(issues);
        }
        
        System.out.println("\nความแข็งแกร่ง: " + getStrength(password));
        
        sc.close();
    }
}
```

---

## 7.9 แบบฝึกหัด Part 07

### แบบฝึกหัดที่ 1: String Manipulation

```java
public class StringManipulation {
    public static void main(String[] args) {
        String text = "The quick brown fox jumps over the lazy dog";
        
        System.out.println("Original: " + text);
        System.out.println("Length: " + text.length());
        System.out.println("Words: " + text.split(" ").length);
        System.out.println("Upper: " + text.toUpperCase());
        System.out.println("Replace: " + text.replace("fox", "cat"));
        
        // นับตัวอักษร 'o'
        long countO = text.chars().filter(c -> c == 'o').count();
        System.out.println("Count 'o': " + countO);
        
        // Reverse
        System.out.println("Reversed: " + new StringBuilder(text).reverse());
        
        // Palindrome check
        String[] words = {"racecar", "hello", "level", "java", "madam"};
        for (String word : words) {
            String rev = new StringBuilder(word).reverse().toString();
            System.out.println(word + " palindrome: " + word.equals(rev));
        }
    }
}
```

### แบบฝึกหัดที่ 2: Email Validator

```java
import java.util.Scanner;

public class EmailValidator {
    
    static boolean isValidEmail(String email) {
        if (email == null || email.isEmpty()) return false;
        
        // Simple validation
        return email.matches("[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}") &&
               !email.contains("..") &&
               !email.startsWith(".") &&
               !email.endsWith(".");
    }
    
    static String extractDomain(String email) {
        int atIndex = email.indexOf('@');
        return email.substring(atIndex + 1);
    }
    
    public static void main(String[] args) {
        String[] emails = {
            "user@example.com",
            "invalid-email",
            "test@.com",
            "test@gmail.com",
            "name.surname@company.org",
            "bad@",
            "@domain.com",
            "test..test@gmail.com"
        };
        
        System.out.println("=== Email Validation ===");
        for (String email : emails) {
            boolean valid = isValidEmail(email);
            System.out.printf("%-30s → %s%n",
                email,
                valid ? "✓ Valid (domain: " + extractDomain(email) + ")" : "✗ Invalid");
        }
    }
}
```

---

## 7.10 สรุป Part 07

ในบทนี้คุณได้เรียนรู้:

✅ String immutability และ String pool  
✅ String methods ทั้งหมด  
✅ String formatting  
✅ StringBuilder และ StringBuffer  
✅ Regular Expressions  
✅ String.join, String.valueOf  
✅ Text Blocks (Java 15+)  

---

*[← Part 06: Arrays](./part-06-arrays.md) | [Part 08: Methods →](./part-08-methods.md)*
