# Part 02: Variables & Data Types
## หลักสูตร Java & Android Development - ระดับเริ่มต้น

---

## 2.1 Variable คืออะไร?

Variable (ตัวแปร) คือพื้นที่ในหน่วยความจำที่ใช้เก็บข้อมูล ในการเขียนโปรแกรมเราต้องการเก็บข้อมูลต่างๆ เช่น ชื่อผู้ใช้, อายุ, ราคาสินค้า เป็นต้น

```
หน่วยความจำ (RAM)
┌─────────────────────────────────┐
│  ตำแหน่ง  │  ชื่อตัวแปร  │  ค่า  │
├───────────┼─────────────┼───────┤
│ 0x001A    │    age      │  25   │
│ 0x001B    │   price     │ 99.5  │
│ 0x001C    │   name      │"John" │
└─────────────────────────────────┘
```

### การประกาศ Variable

```java
// syntax: type variableName;
// syntax: type variableName = value;

int age;           // ประกาศอย่างเดียว (ยังไม่มีค่า)
int score = 100;   // ประกาศพร้อมกำหนดค่า
String name = "Alice";  // String
double price = 49.99;   // Double
boolean active = true;  // Boolean
```

---

## 2.2 Primitive Data Types ทั้ง 8 ชนิด

Java มี Primitive Data Type (ชนิดข้อมูลพื้นฐาน) ทั้งหมด 8 ชนิด:

### 2.2.1 Integer Types (จำนวนเต็ม)

| Type | ขนาด | ช่วงค่า | ใช้สำหรับ |
|------|------|---------|---------|
| `byte` | 1 byte (8 bits) | -128 ถึง 127 | ข้อมูลขนาดเล็ก |
| `short` | 2 bytes (16 bits) | -32,768 ถึง 32,767 | ประหยัด memory |
| `int` | 4 bytes (32 bits) | -2,147,483,648 ถึง 2,147,483,647 | **ใช้บ่อยที่สุด** |
| `long` | 8 bytes (64 bits) | -9,223,372,036,854,775,808 ถึง 9,223,372,036,854,775,807 | ตัวเลขขนาดใหญ่ |

```java
public class IntegerTypes {
    public static void main(String[] args) {
        
        byte myByte = 127;
        System.out.println("byte: " + myByte);
        System.out.println("byte max: " + Byte.MAX_VALUE);  // 127
        System.out.println("byte min: " + Byte.MIN_VALUE);  // -128
        
        short myShort = 32767;
        System.out.println("short: " + myShort);
        System.out.println("short max: " + Short.MAX_VALUE);  // 32767
        
        int myInt = 2_147_483_647;  // underscore ใช้แบ่งหลักได้
        System.out.println("int: " + myInt);
        System.out.println("int max: " + Integer.MAX_VALUE);
        
        long myLong = 9_223_372_036_854_775_807L;  // ต้องใส่ L ท้าย
        System.out.println("long: " + myLong);
        System.out.println("long max: " + Long.MAX_VALUE);
        
        // Integer Overflow - ระวัง!
        int maxInt = Integer.MAX_VALUE;
        System.out.println("maxInt + 1 = " + (maxInt + 1));  // ได้ -2147483648 !
    }
}
```

### 2.2.2 Floating Point Types (เลขทศนิยม)

| Type | ขนาด | ความแม่นยำ | ใช้สำหรับ |
|------|------|-----------|---------|
| `float` | 4 bytes | ~7 ตำแหน่งทศนิยม | ประหยัด memory |
| `double` | 8 bytes | ~15-16 ตำแหน่งทศนิยม | **ใช้บ่อยที่สุด** |

```java
public class FloatingPoint {
    public static void main(String[] args) {
        
        float myFloat = 3.14f;   // ต้องใส่ f ท้าย
        double myDouble = 3.141592653589793;
        
        System.out.println("float:  " + myFloat);
        System.out.println("double: " + myDouble);
        
        // ความแม่นยำ
        float f = 0.1f + 0.2f;
        double d = 0.1 + 0.2;
        System.out.println("float 0.1 + 0.2 = " + f);   // 0.3 (อาจไม่แม่นยำ)
        System.out.println("double 0.1 + 0.2 = " + d);  // 0.30000000000000004 !
        
        // สำหรับการคำนวณเงิน ใช้ BigDecimal แทน
        java.math.BigDecimal bd1 = new java.math.BigDecimal("0.1");
        java.math.BigDecimal bd2 = new java.math.BigDecimal("0.2");
        System.out.println("BigDecimal: " + bd1.add(bd2));  // 0.3 แม่นยำ
        
        // Special values
        System.out.println("Positive infinity: " + Double.POSITIVE_INFINITY);
        System.out.println("Negative infinity: " + Double.NEGATIVE_INFINITY);
        System.out.println("NaN: " + Double.NaN);
        System.out.println("1.0/0 = " + (1.0/0));  // Infinity
        System.out.println("0.0/0 = " + (0.0/0));  // NaN
    }
}
```

### 2.2.3 char Type (ตัวอักษร)

```java
public class CharType {
    public static void main(String[] args) {
        
        char letter = 'A';           // ใช้ single quotes
        char digit = '7';
        char thaiChar = 'ก';         // รองรับ Unicode
        char unicodeChar = 'A'; // Unicode สำหรับ 'A'
        char newline = '\n';         // escape character
        char tab = '\t';
        
        System.out.println("letter: " + letter);
        System.out.println("digit: " + digit);
        System.out.println("thai: " + thaiChar);
        
        // char เป็น numeric ใน Java
        char c = 'A';
        System.out.println("'A' as int: " + (int) c);  // 65
        System.out.println("'A' + 1 = " + (char)(c + 1));  // B
        
        // Escape Characters
        System.out.println("Tab:\there");
        System.out.println("Newline:\nhere");
        System.out.println("Quote: \"Hello\"");
        System.out.println("Backslash: \\");
    }
}
```

### Escape Characters ที่ใช้บ่อย

| Escape | ความหมาย |
|--------|---------|
| `\n` | New line |
| `\t` | Tab |
| `\\` | Backslash |
| `\"` | Double quote |
| `\'` | Single quote |
| `\r` | Carriage return |
| `\0` | Null character |

### 2.2.4 boolean Type (ค่าจริง/เท็จ)

```java
public class BooleanType {
    public static void main(String[] args) {
        
        boolean isTrue = true;
        boolean isFalse = false;
        
        // Boolean expressions
        int age = 18;
        boolean isAdult = age >= 18;
        boolean canVote = age >= 18 && age <= 100;
        
        System.out.println("isAdult: " + isAdult);   // true
        System.out.println("canVote: " + canVote);   // true
        
        // Boolean ใช้ใน conditions
        if (isAdult) {
            System.out.println("ผู้ใหญ่แล้ว");
        }
        
        // Boolean methods ใน String
        String text = "Hello";
        System.out.println(text.isEmpty());         // false
        System.out.println(text.contains("ell"));   // true
        System.out.println(text.startsWith("He"));  // true
    }
}
```

---

## 2.3 String Type

String ไม่ใช่ Primitive Type แต่เป็น Class ที่ใช้บ่อยมาก

```java
public class StringBasics {
    public static void main(String[] args) {
        
        // การสร้าง String
        String s1 = "Hello, World!";
        String s2 = new String("Hello, World!");  // วิธีนี้ไม่แนะนำ
        
        // String Methods พื้นฐาน
        System.out.println("Length: " + s1.length());           // 13
        System.out.println("Upper: " + s1.toUpperCase());       // HELLO, WORLD!
        System.out.println("Lower: " + s1.toLowerCase());       // hello, world!
        System.out.println("Trim: " + "  spaces  ".trim());     // spaces
        System.out.println("Contains: " + s1.contains("World")); // true
        System.out.println("Replace: " + s1.replace("World", "Java")); // Hello, Java!
        System.out.println("Substring: " + s1.substring(7));    // World!
        System.out.println("Substring: " + s1.substring(7, 12)); // World
        System.out.println("Index of: " + s1.indexOf("W"));    // 7
        System.out.println("Char at 0: " + s1.charAt(0));      // H
        
        // String Concatenation (การต่อ String)
        String firstName = "John";
        String lastName = "Doe";
        String fullName = firstName + " " + lastName;
        System.out.println("Full name: " + fullName);
        
        // String.format
        String formatted = String.format("ชื่อ: %s, อายุ: %d", "Alice", 25);
        System.out.println(formatted);
        
        // String comparison (ต้องใช้ .equals() ไม่ใช่ ==)
        String a = "hello";
        String b = "hello";
        String c = new String("hello");
        
        System.out.println(a == b);        // true (ชี้ไปที่ String pool เดียวกัน)
        System.out.println(a == c);        // false (object ต่างกัน)
        System.out.println(a.equals(c));   // true (เนื้อหาเหมือนกัน)
        System.out.println(a.equalsIgnoreCase("HELLO")); // true
        
        // String เป็น immutable (ไม่เปลี่ยนแปลงได้)
        String original = "Hello";
        String modified = original.concat(" World");
        System.out.println("original: " + original);  // Hello (ไม่เปลี่ยน)
        System.out.println("modified: " + modified);  // Hello World
    }
}
```

---

## 2.4 Type Casting (การแปลงชนิดข้อมูล)

### Widening Casting (อัตโนมัติ - เล็กไปใหญ่)

```java
public class WideningCasting {
    public static void main(String[] args) {
        
        // byte → short → int → long → float → double
        byte b = 100;
        short s = b;    // byte to short (อัตโนมัติ)
        int i = s;      // short to int (อัตโนมัติ)
        long l = i;     // int to long (อัตโนมัติ)
        float f = l;    // long to float (อัตโนมัติ)
        double d = f;   // float to double (อัตโนมัติ)
        
        System.out.println("byte: " + b);
        System.out.println("short: " + s);
        System.out.println("int: " + i);
        System.out.println("long: " + l);
        System.out.println("float: " + f);
        System.out.println("double: " + d);
    }
}
```

### Narrowing Casting (ต้องทำเอง - ใหญ่ไปเล็ก)

```java
public class NarrowingCasting {
    public static void main(String[] args) {
        
        // double → float → long → int → short → byte
        double d = 9.99;
        float f = (float) d;    // ต้องบอก type ที่ต้องการ
        long l = (long) f;      // ทศนิยมถูกตัดทิ้ง
        int i = (int) l;
        short s = (short) i;
        byte by = (byte) s;
        
        System.out.println("double: " + d);    // 9.99
        System.out.println("float: " + f);     // 9.99
        System.out.println("long: " + l);      // 9 (ตัดทศนิยม)
        System.out.println("int: " + i);       // 9
        System.out.println("short: " + s);     // 9
        System.out.println("byte: " + by);     // 9
        
        // Data Loss ตัวอย่าง
        int bigNumber = 1000;
        byte small = (byte) bigNumber; // 1000 เกิน byte ได้ 232 (overflow)
        System.out.println("1000 as byte: " + small); // -24 (overflow!)
        
        // char to int
        char c = 'A';
        int ascii = c;
        System.out.println("'A' = " + ascii);  // 65
        
        // int to char
        int num = 66;
        char letter = (char) num;
        System.out.println("66 = '" + letter + "'");  // B
    }
}
```

### String Conversion

```java
public class StringConversion {
    public static void main(String[] args) {
        
        // String to number
        String numStr = "42";
        int num = Integer.parseInt(numStr);
        double d = Double.parseDouble("3.14");
        long l = Long.parseLong("1234567890");
        boolean b = Boolean.parseBoolean("true");
        
        System.out.println("String to int: " + num);
        System.out.println("String to double: " + d);
        System.out.println("String to long: " + l);
        System.out.println("String to boolean: " + b);
        
        // Number to String
        int age = 25;
        String ageStr1 = String.valueOf(age);     // วิธีที่ 1
        String ageStr2 = Integer.toString(age);   // วิธีที่ 2
        String ageStr3 = "" + age;                // วิธีที่ 3 (ไม่แนะนำ)
        
        System.out.println("int to String: " + ageStr1);
        
        // Parse Error - ถ้า String ไม่ใช่ตัวเลข
        try {
            int badParse = Integer.parseInt("abc");
        } catch (NumberFormatException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}
```

---

## 2.5 Constants (ค่าคงที่)

```java
public class Constants {
    // Class-level constants (ใช้ static final)
    static final double PI = 3.14159265358979;
    static final int MAX_SIZE = 100;
    static final String APP_NAME = "MyApp";
    static final String VERSION = "1.0.0";
    
    public static void main(String[] args) {
        
        // Local constant
        final int MAX_RETRY = 3;
        final String SEPARATOR = "=".repeat(20);
        
        System.out.println(SEPARATOR);
        System.out.println("App: " + APP_NAME + " v" + VERSION);
        System.out.println(SEPARATOR);
        
        // คำนวณพื้นที่วงกลม
        double radius = 5.0;
        double area = PI * radius * radius;
        System.out.printf("พื้นที่วงกลม r=%.1f = %.2f%n", radius, area);
        
        // Constants จาก Math class
        System.out.println("Math.PI = " + Math.PI);
        System.out.println("Math.E = " + Math.E);
        
        // Integer constants
        System.out.println("Integer.MAX_VALUE = " + Integer.MAX_VALUE);
        System.out.println("Integer.MIN_VALUE = " + Integer.MIN_VALUE);
        System.out.println("Double.MAX_VALUE = " + Double.MAX_VALUE);
    }
}
```

---

## 2.6 Variable Scope (ขอบเขตของตัวแปร)

```java
public class VariableScope {
    
    // Instance variable (ระดับ class)
    int instanceVar = 10;
    
    // Class variable (static)
    static int classVar = 20;
    
    public void method1() {
        // Local variable (ระดับ method)
        int localVar = 30;
        
        System.out.println("Instance: " + instanceVar);  // ✓
        System.out.println("Class: " + classVar);         // ✓
        System.out.println("Local: " + localVar);         // ✓
    }
    
    public void method2() {
        System.out.println("Instance: " + instanceVar);  // ✓
        System.out.println("Class: " + classVar);         // ✓
        // System.out.println(localVar); // ✗ Error! ไม่เห็น localVar
    }
    
    public static void main(String[] args) {
        
        VariableScope obj = new VariableScope();
        obj.method1();
        
        // Block scope
        {
            int blockVar = 40;
            System.out.println("Block var: " + blockVar); // ✓
        }
        // System.out.println(blockVar); // ✗ Error! ออก scope แล้ว
        
        // Loop scope
        for (int i = 0; i < 3; i++) {
            System.out.println("i = " + i); // ✓
        }
        // System.out.println(i); // ✗ Error! ออก scope แล้ว
    }
}
```

---

## 2.7 var Keyword (Java 10+)

```java
public class VarKeyword {
    public static void main(String[] args) {
        
        // var ให้ compiler infer type อัตโนมัติ
        var age = 25;              // int
        var name = "Alice";        // String
        var price = 99.99;         // double
        var active = true;         // boolean
        
        // ดูชนิดข้อมูล
        System.out.println(((Object)age).getClass().getSimpleName());    // Integer
        System.out.println(name.getClass().getSimpleName());             // String
        System.out.println(((Object)price).getClass().getSimpleName());  // Double
        
        // var กับ collections
        var list = new java.util.ArrayList<String>();
        list.add("Apple");
        list.add("Banana");
        
        for (var item : list) {
            System.out.println(item);
        }
        
        // ข้อจำกัดของ var
        // var x;           // ✗ ต้องกำหนดค่าเริ่มต้น
        // var y = null;    // ✗ ไม่รู้ type
        // var[] arr = ...; // ✗ ไม่ใช้กับ array type
    }
}
```

---

## 2.8 การทำงานกับ Math Class

```java
public class MathOperations {
    public static void main(String[] args) {
        
        // Basic Math
        System.out.println("abs(-5) = " + Math.abs(-5));        // 5
        System.out.println("max(10,20) = " + Math.max(10, 20));  // 20
        System.out.println("min(10,20) = " + Math.min(10, 20));  // 10
        
        // Power and Root
        System.out.println("pow(2,10) = " + Math.pow(2, 10));   // 1024.0
        System.out.println("sqrt(16) = " + Math.sqrt(16));       // 4.0
        System.out.println("cbrt(27) = " + Math.cbrt(27));       // 3.0
        
        // Rounding
        System.out.println("ceil(4.1) = " + Math.ceil(4.1));    // 5.0
        System.out.println("floor(4.9) = " + Math.floor(4.9));  // 4.0
        System.out.println("round(4.5) = " + Math.round(4.5));  // 5
        System.out.println("round(4.4) = " + Math.round(4.4));  // 4
        
        // Log and Exp
        System.out.println("log(Math.E) = " + Math.log(Math.E));  // 1.0
        System.out.println("log10(100) = " + Math.log10(100));     // 2.0
        System.out.println("exp(1) = " + Math.exp(1));             // 2.718...
        
        // Trigonometry
        System.out.println("sin(90°) = " + Math.sin(Math.toRadians(90)));  // 1.0
        System.out.println("cos(0°) = " + Math.cos(Math.toRadians(0)));    // 1.0
        
        // Random
        double rand = Math.random();  // 0.0 ถึง 1.0
        System.out.println("random: " + rand);
        
        // Random int ระหว่าง 1-10
        int randInt = (int)(Math.random() * 10) + 1;
        System.out.println("random 1-10: " + randInt);
    }
}
```

---

## 2.9 โปรแกรมตัวอย่าง: BMI Calculator

```java
import java.util.Scanner;

public class BMICalculator {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        
        System.out.println("=== คำนวณ BMI (Body Mass Index) ===");
        
        System.out.print("น้ำหนัก (กิโลกรัม): ");
        double weight = sc.nextDouble();
        
        System.out.print("ส่วนสูง (เซนติเมตร): ");
        double heightCm = sc.nextDouble();
        
        // แปลงเป็นเมตร
        double heightM = heightCm / 100.0;
        
        // คำนวณ BMI
        double bmi = weight / (heightM * heightM);
        
        // แสดงผล
        System.out.println("\n=== ผลลัพธ์ ===");
        System.out.printf("น้ำหนัก: %.1f กก.%n", weight);
        System.out.printf("ส่วนสูง: %.1f ซม. (%.2f ม.)%n", heightCm, heightM);
        System.out.printf("BMI: %.2f%n", bmi);
        
        // แสดงสถานะ
        String status;
        if (bmi < 18.5) {
            status = "น้ำหนักน้อย (Underweight)";
        } else if (bmi < 25.0) {
            status = "น้ำหนักปกติ (Normal)";
        } else if (bmi < 30.0) {
            status = "น้ำหนักเกิน (Overweight)";
        } else {
            status = "อ้วน (Obese)";
        }
        
        System.out.println("สถานะ: " + status);
        
        sc.close();
    }
}
```

---

## 2.10 โปรแกรมตัวอย่าง: Currency Converter

```java
import java.util.Scanner;

public class CurrencyConverter {
    
    // Exchange rates (เทียบกับ USD)
    static final double USD_TO_THB = 35.50;
    static final double USD_TO_EUR = 0.92;
    static final double USD_TO_JPY = 149.50;
    static final double USD_TO_GBP = 0.79;
    static final double USD_TO_CNY = 7.24;
    
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        
        System.out.println("=== Currency Converter ===");
        System.out.println("แปลงสกุลเงิน USD เป็นสกุลอื่น");
        System.out.println();
        
        System.out.print("ใส่จำนวนเงิน USD: $");
        double usd = sc.nextDouble();
        
        System.out.println("\nผลการแปลง:");
        System.out.printf("$%.2f USD = %.2f THB (บาท)%n", usd, usd * USD_TO_THB);
        System.out.printf("$%.2f USD = %.2f EUR (ยูโร)%n", usd, usd * USD_TO_EUR);
        System.out.printf("$%.2f USD = %.2f JPY (เยน)%n", usd, usd * USD_TO_JPY);
        System.out.printf("$%.2f USD = %.2f GBP (ปอนด์)%n", usd, usd * USD_TO_GBP);
        System.out.printf("$%.2f USD = %.2f CNY (หยวน)%n", usd, usd * USD_TO_CNY);
        
        sc.close();
    }
}
```

---

## 2.11 แบบฝึกหัด Part 02

### แบบฝึกหัดที่ 1: Data Types Practice

```java
public class DataTypePractice {
    public static void main(String[] args) {
        // TODO: ประกาศตัวแปรและแสดงผลต่อไปนี้
        // 1. เก็บอายุของคุณใน byte
        // 2. เก็บรหัสไปรษณีย์ใน int
        // 3. เก็บน้ำหนักของคุณใน double
        // 4. เก็บชื่อของคุณใน String
        // 5. เก็บว่าคุณเป็นนักศึกษาหรือไม่ใน boolean
        // แสดงผลข้อมูลทั้งหมดในรูปแบบสวยงาม
    }
}
```

**เฉลย:**
```java
public class DataTypeSolution {
    public static void main(String[] args) {
        byte age = 25;
        int zipCode = 10110;
        double weight = 65.5;
        String name = "สมชาย ใจดี";
        boolean isStudent = true;
        
        System.out.println("=== ข้อมูลส่วนตัว ===");
        System.out.printf("ชื่อ       : %s%n", name);
        System.out.printf("อายุ       : %d ปี%n", age);
        System.out.printf("น้ำหนัก    : %.1f กก.%n", weight);
        System.out.printf("รหัสไปรษณีย์: %d%n", zipCode);
        System.out.printf("เป็นนักศึกษา: %b%n", isStudent);
    }
}
```

### แบบฝึกหัดที่ 2: Type Casting

จงเขียนโปรแกรมที่แสดงให้เห็น type casting ทั้ง widening และ narrowing

**เฉลย:**
```java
public class TypeCastingSolution {
    public static void main(String[] args) {
        System.out.println("=== Widening Casting ===");
        int i = 100;
        long l = i;         // อัตโนมัติ
        double d = l;       // อัตโนมัติ
        System.out.println("int " + i + " → long " + l + " → double " + d);
        
        System.out.println("\n=== Narrowing Casting ===");
        double pi = 3.14159;
        int piInt = (int) pi;   // ต้องทำเอง
        byte piByte = (byte) piInt;
        System.out.println("double " + pi + " → int " + piInt + " → byte " + piByte);
        
        System.out.println("\n=== String Conversion ===");
        int num = 42;
        String numStr = String.valueOf(num);
        int backToInt = Integer.parseInt(numStr);
        System.out.println("int → String → int: " + backToInt);
    }
}
```

### แบบฝึกหัดที่ 3: โปรแกรมคิดค่าจ้าง

```java
import java.util.Scanner;

public class SalaryCalculator {
    // Constants
    static final double OVERTIME_RATE = 1.5;
    static final int REGULAR_HOURS = 8;
    static final double TAX_RATE = 0.07;  // 7%
    
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        
        System.out.println("=== คำนวณค่าจ้าง ===");
        System.out.print("ค่าจ้างต่อชั่วโมง (บาท): ");
        double hourlyRate = sc.nextDouble();
        
        System.out.print("จำนวนชั่วโมงทำงาน: ");
        double hoursWorked = sc.nextDouble();
        
        // คำนวณ
        double regularPay, overtimePay, grossPay, tax, netPay;
        
        if (hoursWorked <= REGULAR_HOURS) {
            regularPay = hoursWorked * hourlyRate;
            overtimePay = 0;
        } else {
            regularPay = REGULAR_HOURS * hourlyRate;
            double overtimeHours = hoursWorked - REGULAR_HOURS;
            overtimePay = overtimeHours * hourlyRate * OVERTIME_RATE;
        }
        
        grossPay = regularPay + overtimePay;
        tax = grossPay * TAX_RATE;
        netPay = grossPay - tax;
        
        // แสดงผล
        System.out.println("\n=== ใบแจ้งเงินเดือน ===");
        System.out.printf("ค่าจ้างปกติ    : %,.2f บาท%n", regularPay);
        System.out.printf("ค่าล่วงเวลา    : %,.2f บาท%n", overtimePay);
        System.out.printf("รายได้รวม      : %,.2f บาท%n", grossPay);
        System.out.printf("หักภาษี (%.0f%%): %,.2f บาท%n", TAX_RATE * 100, tax);
        System.out.printf("รายได้สุทธิ    : %,.2f บาท%n", netPay);
        
        sc.close();
    }
}
```

---

## 2.12 สรุป Part 02

ในบทนี้คุณได้เรียนรู้:

✅ ความหมายและการประกาศ Variable  
✅ Primitive Data Types ทั้ง 8 ชนิด (byte, short, int, long, float, double, char, boolean)  
✅ String และ String methods พื้นฐาน  
✅ Widening และ Narrowing Casting  
✅ String Conversion  
✅ Constants (final)  
✅ Variable Scope  
✅ var keyword (Java 10+)  
✅ Math class  

---

## 2.13 Checklist ก่อนไป Part 03

- [ ] เข้าใจ Primitive Data Types ทั้ง 8 ชนิด
- [ ] ทราบช่วงค่าของแต่ละ type
- [ ] เข้าใจความแตกต่างระหว่าง int และ double
- [ ] ทำ Type Casting ได้
- [ ] แปลง String เป็น number และกลับได้
- [ ] ทำแบบฝึกหัดทั้ง 3 ข้อครบ

---

*[← Part 01: Introduction](./part-01-java-introduction.md) | [Part 03: Operators →](./part-03-operators.md)*
