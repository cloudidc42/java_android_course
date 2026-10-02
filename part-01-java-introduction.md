# Part 01: Introduction to Java & Setup Environment
## หลักสูตร Java & Android Development - ระดับเริ่มต้น

---

## 1.1 Java คืออะไร?

Java เป็นภาษาโปรแกรมที่พัฒนาโดย **Sun Microsystems** (ปัจจุบันเป็นของ Oracle) ในปี 1995 โดย James Gosling และทีมงาน Java มีหลักการสำคัญคือ **"Write Once, Run Anywhere"** (WORA) หมายความว่าโค้ดที่เขียนครั้งเดียวสามารถทำงานบนระบบปฏิบัติการใดก็ได้ที่มี JVM (Java Virtual Machine) ติดตั้งอยู่

### ทำไมต้องเรียน Java?

1. **ความนิยมสูง** - Java เป็นหนึ่งในภาษาโปรแกรมที่ได้รับความนิยมมากที่สุดในโลก
2. **Android Development** - Java เป็นภาษาหลักในการพัฒนา Android App
3. **Enterprise Applications** - บริษัทขนาดใหญ่ใช้ Java ในระบบ Backend
4. **Job Opportunities** - มีงานด้าน Java Developer มากมาย
5. **Community ขนาดใหญ่** - มีทรัพยากรการเรียนรู้มากมาย
6. **Object-Oriented** - เรียนรู้แนวคิด OOP ที่ใช้ได้กับทุกภาษา

### Java ถูกใช้ที่ไหนบ้าง?

- **Android Apps** - แอปบน Android เกือบทั้งหมด
- **Web Backend** - Spring Framework, Java EE
- **Big Data** - Hadoop, Spark
- **Financial Systems** - ระบบธนาคาร, ตลาดหุ้น
- **Scientific Computing** - การคำนวณทางวิทยาศาสตร์
- **IoT** - Internet of Things

---

## 1.2 Java Architecture

```
┌─────────────────────────────────────────────┐
│              Java Source Code (.java)        │
└─────────────────────┬───────────────────────┘
                      │ Java Compiler (javac)
                      ▼
┌─────────────────────────────────────────────┐
│              Bytecode (.class)               │
└─────────────────────┬───────────────────────┘
                      │
         ┌────────────┼────────────┐
         ▼            ▼            ▼
    ┌─────────┐  ┌─────────┐  ┌─────────┐
    │ JVM     │  │ JVM     │  │ JVM     │
    │ Windows │  │ Linux   │  │ macOS   │
    └─────────┘  └─────────┘  └─────────┘
```

### JDK, JRE, JVM คืออะไร?

| ชื่อ | ย่อมาจาก | หน้าที่ |
|------|----------|---------|
| **JDK** | Java Development Kit | ชุดเครื่องมือสำหรับพัฒนา Java (รวม JRE และ compiler) |
| **JRE** | Java Runtime Environment | สภาพแวดล้อมสำหรับรัน Java program (รวม JVM) |
| **JVM** | Java Virtual Machine | เครื่องเสมือนที่รัน bytecode |

---

## 1.3 การติดตั้ง JDK

### ขั้นตอนการติดตั้ง JDK บน Windows

1. **ดาวน์โหลด JDK** จาก Oracle หรือ OpenJDK
   - Oracle JDK: https://www.oracle.com/java/technologies/downloads/
   - OpenJDK (แนะนำ): https://adoptium.net/

2. **ติดตั้ง JDK**
   - เปิดไฟล์ที่ดาวน์โหลดมา
   - ทำตามขั้นตอนการติดตั้ง
   - จดจำ path ที่ติดตั้ง เช่น `C:\Program Files\Java\jdk-17`

3. **ตั้งค่า Environment Variables**
   - เปิด System Properties → Advanced → Environment Variables
   - เพิ่ม `JAVA_HOME` = `C:\Program Files\Java\jdk-17`
   - เพิ่ม `;%JAVA_HOME%\bin` ใน `PATH`

4. **ตรวจสอบการติดตั้ง**
   ```bash
   java -version
   javac -version
   ```

### ขั้นตอนการติดตั้ง JDK บน macOS

```bash
# ใช้ Homebrew (แนะนำ)
brew install openjdk@17

# เพิ่มใน ~/.zshrc หรือ ~/.bash_profile
export JAVA_HOME=/usr/local/opt/openjdk@17
export PATH="$JAVA_HOME/bin:$PATH"

# ตรวจสอบ
java -version
```

### ขั้นตอนการติดตั้ง JDK บน Ubuntu/Linux

```bash
# อัปเดต package list
sudo apt update

# ติดตั้ง OpenJDK 17
sudo apt install openjdk-17-jdk

# ตรวจสอบ
java -version
javac -version

# ถ้ามีหลาย version ให้เลือก
sudo update-alternatives --config java
```

---

## 1.4 การติดตั้ง IDE

### IntelliJ IDEA (แนะนำสำหรับ Java)

1. ดาวน์โหลดจาก https://www.jetbrains.com/idea/
2. เลือก **Community Edition** (ฟรี) สำหรับเริ่มต้น
3. ติดตั้งและเปิดโปรแกรม
4. สร้าง Project ใหม่ → เลือก Java → ตั้งชื่อ Project

### VS Code (ทางเลือก)

1. ดาวน์โหลดจาก https://code.visualstudio.com/
2. ติดตั้ง Extension Pack for Java
3. เปิดโฟลเดอร์ project

### Eclipse (ทางเลือก)

1. ดาวน์โหลดจาก https://www.eclipse.org/downloads/
2. เลือก Eclipse IDE for Java Developers
3. ติดตั้งและ configure workspace

---

## 1.5 โปรแกรมแรก: Hello, World!

ตามธรรมเนียมการเรียนภาษาโปรแกรมใหม่ เราจะเริ่มด้วย "Hello, World!"

### สร้างไฟล์ HelloWorld.java

```java
// นี่คือโปรแกรม Java แรกของเรา
// ชื่อไฟล์ต้องตรงกับชื่อ class เสมอ: HelloWorld.java

public class HelloWorld {
    // main method คือจุดเริ่มต้นของทุก Java program
    public static void main(String[] args) {
        // แสดงข้อความบนหน้าจอ
        System.out.println("Hello, World!");
    }
}
```

### อธิบายโค้ด

```java
public class HelloWorld {
```
- `public` = access modifier บอกว่า class นี้เข้าถึงได้จากทุกที่
- `class` = คำสงวน (keyword) สำหรับสร้าง class
- `HelloWorld` = ชื่อ class (ต้องตรงกับชื่อไฟล์)
- `{` = เริ่มต้น body ของ class

```java
    public static void main(String[] args) {
```
- `public` = เข้าถึงได้จากทุกที่ (JVM ต้องเรียกใช้ได้)
- `static` = ไม่ต้องสร้าง object เพื่อเรียกใช้
- `void` = method นี้ไม่ return ค่าใดๆ
- `main` = ชื่อ method ที่ JVM เรียกก่อนเสมอ
- `String[] args` = parameter รับค่าจาก command line

```java
        System.out.println("Hello, World!");
```
- `System` = class ใน Java standard library
- `out` = object สำหรับ output
- `println` = method พิมพ์ข้อความแล้วขึ้นบรรทัดใหม่

### Compile และ Run

```bash
# Compile
javac HelloWorld.java

# จะได้ไฟล์ HelloWorld.class

# Run
java HelloWorld

# Output:
# Hello, World!
```

---

## 1.6 โครงสร้างพื้นฐานของโปรแกรม Java

```java
// 1. Package declaration (ไม่จำเป็นสำหรับโปรแกรมเล็กๆ)
package com.example.myapp;

// 2. Import statements
import java.util.Scanner;
import java.util.ArrayList;

// 3. Class declaration
public class MyFirstProgram {
    
    // 4. Class variables (fields)
    private String name = "Java";
    private int version = 17;
    
    // 5. Constructor
    public MyFirstProgram() {
        // เรียกเมื่อสร้าง object
    }
    
    // 6. Methods
    public void greet() {
        System.out.println("Welcome to " + name + " " + version);
    }
    
    // 7. Main method - จุดเริ่มต้นของโปรแกรม
    public static void main(String[] args) {
        MyFirstProgram program = new MyFirstProgram();
        program.greet();
    }
}
```

---

## 1.7 การรับ Input จากผู้ใช้

```java
import java.util.Scanner;

public class UserInput {
    public static void main(String[] args) {
        // สร้าง Scanner object เพื่ออ่าน input
        Scanner scanner = new Scanner(System.in);
        
        // แสดงข้อความขอ input
        System.out.print("กรุณาใส่ชื่อของคุณ: ");
        
        // อ่าน input
        String name = scanner.nextLine();
        
        // แสดงผล
        System.out.println("สวัสดี, " + name + "!");
        
        // อย่าลืมปิด Scanner
        scanner.close();
    }
}
```

### Scanner Methods ที่ใช้บ่อย

| Method | ใช้สำหรับ | ตัวอย่าง |
|--------|----------|---------|
| `nextLine()` | อ่านทั้งบรรทัด (String) | `String text = sc.nextLine();` |
| `next()` | อ่านคำเดียว (String) | `String word = sc.next();` |
| `nextInt()` | อ่านเลขจำนวนเต็ม | `int num = sc.nextInt();` |
| `nextDouble()` | อ่านเลขทศนิยม | `double d = sc.nextDouble();` |
| `nextBoolean()` | อ่าน true/false | `boolean b = sc.nextBoolean();` |

---

## 1.8 Comments ใน Java

Comments คือข้อความที่เขียนไว้สำหรับผู้อ่านโค้ด (คน) ไม่ใช่สำหรับ compiler

```java
public class CommentExample {
    public static void main(String[] args) {
        
        // นี่คือ single-line comment (บรรทัดเดียว)
        // ใช้เครื่องหมาย // นำหน้า
        
        /*
         * นี่คือ multi-line comment (หลายบรรทัด)
         * ใช้สำหรับอธิบายโค้ดที่ซับซ้อน
         * หรืออธิบาย algorithm
         */
        
        /**
         * นี่คือ Javadoc comment
         * ใช้สร้าง documentation ของ class หรือ method
         * @param args command line arguments
         * @return ไม่ return อะไร (void)
         */
        
        System.out.println("Hello!"); // comment ท้ายบรรทัด
    }
}
```

### Best Practices สำหรับ Comments

1. **เขียน comment ที่อธิบาย "ทำไม" ไม่ใช่ "อะไร"**
   ```java
   // ✗ ไม่ดี - อธิบายสิ่งที่เห็นอยู่แล้ว
   // เพิ่ม 1 เข้าไปใน counter
   counter++;
   
   // ✓ ดี - อธิบายเหตุผล
   // นับจำนวน retry หากเกิน MAX_RETRY จะหยุด
   counter++;
   ```

2. **อัปเดต comment เมื่อแก้ไขโค้ด**
3. **ไม่ comment โค้ดที่ชัดเจนอยู่แล้ว**

---

## 1.9 การพิมพ์ Output

```java
public class OutputExample {
    public static void main(String[] args) {
        
        // println - พิมพ์แล้วขึ้นบรรทัดใหม่
        System.out.println("บรรทัดที่ 1");
        System.out.println("บรรทัดที่ 2");
        
        // print - พิมพ์โดยไม่ขึ้นบรรทัดใหม่
        System.out.print("Hello ");
        System.out.print("World");
        System.out.println(); // ขึ้นบรรทัดใหม่
        
        // printf - พิมพ์แบบกำหนดรูปแบบ (format)
        String name = "Java";
        int year = 1995;
        System.out.printf("ภาษา %s ถูกสร้างในปี %d%n", name, year);
        
        // String.format - สร้าง String ที่มีรูปแบบ
        String message = String.format("ราคา: %.2f บาท", 99.5);
        System.out.println(message);
        
        // แสดงตัวเลข
        System.out.println(42);        // int
        System.out.println(3.14);      // double
        System.out.println(true);      // boolean
        System.out.println('A');       // char
    }
}
```

### Format Specifiers ที่ใช้บ่อย

| Specifier | ใช้สำหรับ | ตัวอย่าง |
|-----------|----------|---------|
| `%s` | String | `"Hello %s", "World"` → `Hello World` |
| `%d` | Integer | `"Age: %d", 25` → `Age: 25` |
| `%f` | Float/Double | `"Price: %f", 9.99` → `Price: 9.990000` |
| `%.2f` | Float 2 ตำแหน่ง | `"Price: %.2f", 9.99` → `Price: 9.99` |
| `%n` | Newline | ขึ้นบรรทัดใหม่ |
| `%b` | Boolean | `"Valid: %b", true` → `Valid: true` |
| `%c` | Character | `"Char: %c", 'A'` → `Char: A` |

---

## 1.10 โปรแกรมตัวอย่าง: Calculator อย่างง่าย

```java
import java.util.Scanner;

public class SimpleCalculator {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        
        System.out.println("=== เครื่องคิดเลขอย่างง่าย ===");
        
        // รับตัวเลขจากผู้ใช้
        System.out.print("ใส่ตัวเลขที่ 1: ");
        double num1 = scanner.nextDouble();
        
        System.out.print("ใส่ตัวเลขที่ 2: ");
        double num2 = scanner.nextDouble();
        
        // คำนวณ
        double sum = num1 + num2;
        double diff = num1 - num2;
        double product = num1 * num2;
        
        // แสดงผล
        System.out.println("\n=== ผลลัพธ์ ===");
        System.out.printf("%.2f + %.2f = %.2f%n", num1, num2, sum);
        System.out.printf("%.2f - %.2f = %.2f%n", num1, num2, diff);
        System.out.printf("%.2f × %.2f = %.2f%n", num1, num2, product);
        
        if (num2 != 0) {
            double quotient = num1 / num2;
            System.out.printf("%.2f ÷ %.2f = %.2f%n", num1, num2, quotient);
        } else {
            System.out.println("ไม่สามารถหารด้วย 0 ได้!");
        }
        
        scanner.close();
    }
}
```

**ผลลัพธ์ตัวอย่าง:**
```
=== เครื่องคิดเลขอย่างง่าย ===
ใส่ตัวเลขที่ 1: 10
ใส่ตัวเลขที่ 2: 3

=== ผลลัพธ์ ===
10.00 + 3.00 = 13.00
10.00 - 3.00 = 7.00
10.00 × 3.00 = 30.00
10.00 ÷ 3.00 = 3.33
```

---

## 1.11 Java Conventions และ Best Practices

### การตั้งชื่อ (Naming Conventions)

```java
// Class names - ใช้ PascalCase (ตัวใหญ่ขึ้นต้น)
public class MyFirstClass { }
public class BankAccount { }
public class StudentManagement { }

// Variable names - ใช้ camelCase (ตัวเล็กขึ้นต้น)
int studentAge = 20;
String firstName = "John";
boolean isActive = true;

// Constant names - ใช้ UPPER_SNAKE_CASE
final int MAX_SIZE = 100;
final double PI = 3.14159;
final String APP_NAME = "MyApp";

// Method names - ใช้ camelCase
public void calculateTotal() { }
public int getAge() { }
public boolean isValid() { }

// Package names - ใช้ lowercase
package com.example.myapp;
package org.university.project;
```

### Code Style

```java
// ✓ ดี - indent ด้วย 4 spaces
public class GoodStyle {
    public static void main(String[] args) {
        if (true) {
            System.out.println("Correct indentation");
        }
    }
}

// ✗ ไม่ดี - ไม่มี indent
public class BadStyle {
public static void main(String[] args) {
if (true) {
System.out.println("No indentation");
}
}
}
```

---

## 1.12 แบบฝึกหัด Part 01

### แบบฝึกหัดที่ 1: Hello World Variations

สร้างโปรแกรมที่แสดงผลดังนี้:
```
**********************
*                    *
*   Hello, Java!     *
*                    *
**********************
```

**เฉลย:**
```java
public class HelloBox {
    public static void main(String[] args) {
        System.out.println("**********************");
        System.out.println("*                    *");
        System.out.println("*   Hello, Java!     *");
        System.out.println("*                    *");
        System.out.println("**********************");
    }
}
```

### แบบฝึกหัดที่ 2: Personal Info Program

สร้างโปรแกรมที่รับข้อมูลและแสดงผล:

```java
import java.util.Scanner;

public class PersonalInfo {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        
        System.out.print("ชื่อ: ");
        String name = sc.nextLine();
        
        System.out.print("อายุ: ");
        int age = sc.nextInt();
        sc.nextLine(); // clear buffer
        
        System.out.print("อาชีพ: ");
        String job = sc.nextLine();
        
        System.out.println("\n=== ข้อมูลส่วนตัว ===");
        System.out.println("ชื่อ    : " + name);
        System.out.println("อายุ    : " + age + " ปี");
        System.out.println("อาชีพ   : " + job);
        
        sc.close();
    }
}
```

### แบบฝึกหัดที่ 3: Temperature Converter

สร้างโปรแกรมแปลงอุณหภูมิ Celsius เป็น Fahrenheit:

**สูตร:** F = (C × 9/5) + 32

```java
import java.util.Scanner;

public class TempConverter {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        
        System.out.print("ใส่อุณหภูมิ (Celsius): ");
        double celsius = sc.nextDouble();
        
        // แปลง
        double fahrenheit = (celsius * 9.0 / 5.0) + 32;
        double kelvin = celsius + 273.15;
        
        // แสดงผล
        System.out.printf("%.2f°C = %.2f°F%n", celsius, fahrenheit);
        System.out.printf("%.2f°C = %.2f K%n", celsius, kelvin);
        
        sc.close();
    }
}
```

**ผลลัพธ์:**
```
ใส่อุณหภูมิ (Celsius): 100
100.00°C = 212.00°F
100.00°C = 373.15 K
```

### แบบฝึกหัดที่ 4: Area Calculator

สร้างโปรแกรมคำนวณพื้นที่:

```java
import java.util.Scanner;

public class AreaCalculator {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        
        System.out.println("=== คำนวณพื้นที่ ===");
        System.out.println("1. สี่เหลี่ยมผืนผ้า");
        System.out.println("2. วงกลม");
        System.out.println("3. สามเหลี่ยม");
        System.out.print("เลือก (1-3): ");
        
        int choice = sc.nextInt();
        
        switch (choice) {
            case 1:
                System.out.print("กว้าง (เมตร): ");
                double width = sc.nextDouble();
                System.out.print("ยาว (เมตร): ");
                double length = sc.nextDouble();
                double rectArea = width * length;
                System.out.printf("พื้นที่ = %.2f ตารางเมตร%n", rectArea);
                break;
                
            case 2:
                System.out.print("รัศมี (เมตร): ");
                double radius = sc.nextDouble();
                double circleArea = Math.PI * radius * radius;
                System.out.printf("พื้นที่ = %.2f ตารางเมตร%n", circleArea);
                break;
                
            case 3:
                System.out.print("ฐาน (เมตร): ");
                double base = sc.nextDouble();
                System.out.print("สูง (เมตร): ");
                double height = sc.nextDouble();
                double triArea = 0.5 * base * height;
                System.out.printf("พื้นที่ = %.2f ตารางเมตร%n", triArea);
                break;
                
            default:
                System.out.println("ตัวเลือกไม่ถูกต้อง!");
        }
        
        sc.close();
    }
}
```

---

## 1.13 สรุป Part 01

ในบทนี้คุณได้เรียนรู้:

✅ Java คืออะไรและใช้ทำอะไร  
✅ Architecture ของ Java (JDK, JRE, JVM)  
✅ การติดตั้ง JDK และ IDE  
✅ การเขียนโปรแกรม Hello World  
✅ โครงสร้างพื้นฐานของโปรแกรม Java  
✅ การรับ Input และแสดง Output  
✅ Comments และ Best Practices  
✅ Naming Conventions  

---

## 1.14 Checklist ก่อนไป Part 02

- [ ] ติดตั้ง JDK สำเร็จ (ตรวจสอบด้วย `java -version`)
- [ ] ติดตั้ง IDE สำเร็จ
- [ ] เขียนและรัน Hello World ได้
- [ ] รับ input จากผู้ใช้ได้
- [ ] ทำแบบฝึกหัดทั้ง 4 ข้อครบ

---

## 1.15 Preview Part 02

ใน Part ถัดไปเราจะเรียนเรื่อง **Variables และ Data Types** ซึ่งเป็นหัวใจสำคัญของการเขียนโปรแกรม คุณจะได้เรียนรู้:
- ชนิดข้อมูลพื้นฐานทั้ง 8 ชนิดของ Java
- การประกาศและใช้งาน Variable
- การแปลงชนิดข้อมูล (Type Casting)
- Constants

---

*[ไปยัง Part 02: Variables & Data Types →](./part-02-variables-datatypes.md)*
