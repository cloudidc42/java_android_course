# Part 03: Operators & Expressions
## หลักสูตร Java & Android Development - ระดับเริ่มต้น

---

## 3.1 Operators คืออะไร?

Operator (ตัวดำเนินการ) คือสัญลักษณ์พิเศษที่ใช้ทำการคำนวณหรือเปรียบเทียบค่า Java มี Operator หลายประเภท ดังนี้:

1. Arithmetic Operators (ตัวดำเนินการทางคณิตศาสตร์)
2. Assignment Operators (ตัวดำเนินการกำหนดค่า)
3. Comparison Operators (ตัวดำเนินการเปรียบเทียบ)
4. Logical Operators (ตัวดำเนินการตรรกะ)
5. Bitwise Operators (ตัวดำเนินการบิต)
6. Unary Operators (ตัวดำเนินการเดี่ยว)
7. Ternary Operator (ตัวดำเนินการสามส่วน)
8. instanceof Operator

---

## 3.2 Arithmetic Operators

| Operator | ชื่อ | ตัวอย่าง | ผลลัพธ์ |
|----------|------|---------|---------|
| `+` | Addition | `5 + 3` | `8` |
| `-` | Subtraction | `5 - 3` | `2` |
| `*` | Multiplication | `5 * 3` | `15` |
| `/` | Division | `5 / 3` | `1` (int division) |
| `%` | Modulus | `5 % 3` | `2` |

```java
public class ArithmeticOperators {
    public static void main(String[] args) {
        int a = 10, b = 3;
        
        System.out.println("a = " + a + ", b = " + b);
        System.out.println("a + b = " + (a + b));   // 13
        System.out.println("a - b = " + (a - b));   // 7
        System.out.println("a * b = " + (a * b));   // 30
        System.out.println("a / b = " + (a / b));   // 3 (integer division!)
        System.out.println("a % b = " + (a % b));   // 1
        
        // Integer Division vs Double Division
        System.out.println("\n--- Integer vs Double Division ---");
        System.out.println("10 / 3 = " + (10 / 3));           // 3 (int)
        System.out.println("10.0 / 3 = " + (10.0 / 3));       // 3.333...
        System.out.println("10 / 3.0 = " + (10 / 3.0));       // 3.333...
        System.out.println("(double)10/3 = " + ((double)10/3)); // 3.333...
        
        // Modulus ใช้กับ double ได้
        System.out.println("\n--- Modulus ---");
        System.out.println("10 % 3 = " + (10 % 3));     // 1
        System.out.println("10.5 % 3 = " + (10.5 % 3)); // 1.5
        System.out.println("-7 % 3 = " + (-7 % 3));     // -1 (เครื่องหมายตาม dividend)
        System.out.println("7 % -3 = " + (7 % -3));     // 1
        
        // String concatenation กับ +
        System.out.println("\n--- String Concatenation ---");
        System.out.println("Hello" + " " + "World"); // Hello World
        System.out.println("5 + 3 = " + (5 + 3));    // 5 + 3 = 8
        System.out.println("5 + 3 = " + 5 + 3);      // 5 + 3 = 53 (concat!)
    }
}
```

### Modulus Use Cases

```java
public class ModulusUseCases {
    public static void main(String[] args) {
        
        // ตรวจสอบเลขคู่/คี่
        for (int i = 1; i <= 10; i++) {
            if (i % 2 == 0) {
                System.out.println(i + " เป็นเลขคู่");
            } else {
                System.out.println(i + " เป็นเลขคี่");
            }
        }
        
        System.out.println();
        
        // แปลง seconds เป็น minutes:seconds
        int totalSeconds = 3661;
        int hours = totalSeconds / 3600;
        int minutes = (totalSeconds % 3600) / 60;
        int seconds = totalSeconds % 60;
        System.out.printf("%d วินาที = %d:%02d:%02d%n", totalSeconds, hours, minutes, seconds);
        
        // Circular array index
        int[] items = {1, 2, 3, 4, 5};
        for (int i = 0; i < 15; i++) {
            System.out.print(items[i % items.length] + " ");
        }
        System.out.println();
    }
}
```

---

## 3.3 Assignment Operators

| Operator | ตัวอย่าง | เทียบกับ |
|----------|---------|---------|
| `=` | `a = 5` | `a = 5` |
| `+=` | `a += 3` | `a = a + 3` |
| `-=` | `a -= 3` | `a = a - 3` |
| `*=` | `a *= 3` | `a = a * 3` |
| `/=` | `a /= 3` | `a = a / 3` |
| `%=` | `a %= 3` | `a = a % 3` |

```java
public class AssignmentOperators {
    public static void main(String[] args) {
        int x = 10;
        System.out.println("x = " + x);  // 10
        
        x += 5;
        System.out.println("x += 5 → " + x);  // 15
        
        x -= 3;
        System.out.println("x -= 3 → " + x);  // 12
        
        x *= 2;
        System.out.println("x *= 2 → " + x);  // 24
        
        x /= 4;
        System.out.println("x /= 4 → " + x);  // 6
        
        x %= 4;
        System.out.println("x %= 4 → " + x);  // 2
        
        // Multiple assignment
        int a, b, c;
        a = b = c = 10;  // a=10, b=10, c=10
        System.out.println("a=" + a + ", b=" + b + ", c=" + c);
        
        // String +=
        String s = "Hello";
        s += " World";
        s += "!";
        System.out.println(s);  // Hello World!
    }
}
```

---

## 3.4 Comparison Operators (Relational)

| Operator | ชื่อ | ตัวอย่าง | ผลลัพธ์ |
|----------|------|---------|---------|
| `==` | Equal to | `5 == 5` | `true` |
| `!=` | Not equal to | `5 != 3` | `true` |
| `>` | Greater than | `5 > 3` | `true` |
| `<` | Less than | `5 < 3` | `false` |
| `>=` | Greater than or equal | `5 >= 5` | `true` |
| `<=` | Less than or equal | `5 <= 6` | `true` |

```java
public class ComparisonOperators {
    public static void main(String[] args) {
        int x = 10, y = 20;
        
        System.out.println("x = " + x + ", y = " + y);
        System.out.println("x == y: " + (x == y));   // false
        System.out.println("x != y: " + (x != y));   // true
        System.out.println("x > y:  " + (x > y));    // false
        System.out.println("x < y:  " + (x < y));    // true
        System.out.println("x >= y: " + (x >= y));   // false
        System.out.println("x <= y: " + (x <= y));   // true
        System.out.println("x >= x: " + (x >= x));   // true (equal)
        
        // Comparison กับ char
        char c1 = 'A', c2 = 'B';
        System.out.println("\n'A' < 'B': " + (c1 < c2));  // true (65 < 66)
        
        // ระวัง == กับ String
        String s1 = "hello";
        String s2 = "hello";
        String s3 = new String("hello");
        
        System.out.println("\nString comparison:");
        System.out.println("s1 == s2: " + (s1 == s2));           // true (String pool)
        System.out.println("s1 == s3: " + (s1 == s3));           // false (new object)
        System.out.println("s1.equals(s3): " + s1.equals(s3));   // true ✓
        
        // Compare double
        double d1 = 0.1 + 0.2;
        double d2 = 0.3;
        System.out.println("\n0.1 + 0.2 == 0.3: " + (d1 == d2));  // false!
        System.out.println("diff = " + Math.abs(d1 - d2));
        // วิธีที่ถูกต้องสำหรับ double comparison
        System.out.println("correct compare: " + (Math.abs(d1 - d2) < 1e-10));
    }
}
```

---

## 3.5 Logical Operators

| Operator | ชื่อ | คำอธิบาย |
|----------|------|---------|
| `&&` | AND | true ถ้าทั้งสองเป็น true |
| `\|\|` | OR | true ถ้าอย่างน้อยหนึ่งเป็น true |
| `!` | NOT | กลับค่า true/false |

### Truth Table

```
AND (&&):
true  && true  = true
true  && false = false
false && true  = false
false && false = false

OR (||):
true  || true  = true
true  || false = true
false || true  = true
false || false = false

NOT (!):
!true  = false
!false = true
```

```java
public class LogicalOperators {
    public static void main(String[] args) {
        int age = 25;
        boolean hasID = true;
        boolean hasMoney = true;
        
        // AND - ต้องทั้งสองเป็น true
        boolean canBuyAlcohol = (age >= 20) && hasID;
        System.out.println("ซื้อเหล้าได้: " + canBuyAlcohol);  // true
        
        // OR - อย่างน้อยหนึ่งเป็น true
        boolean canPay = hasMoney || hasID;  // มีอย่างน้อยหนึ่งอย่าง
        System.out.println("จ่ายเงินได้: " + canPay);           // true
        
        // NOT - กลับค่า
        boolean isDoorLocked = true;
        boolean canEnter = !isDoorLocked;
        System.out.println("เข้าได้: " + canEnter);             // false
        
        // Complex conditions
        int score = 75;
        boolean passed = score >= 50;
        boolean excellent = score >= 90;
        boolean good = score >= 75 && !excellent;
        
        System.out.println("\nคะแนน: " + score);
        System.out.println("ผ่าน: " + passed);        // true
        System.out.println("ยอดเยี่ยม: " + excellent); // false
        System.out.println("ดี: " + good);             // true
        
        // Short-circuit evaluation
        System.out.println("\n--- Short-circuit ---");
        int x = 0;
        // false && ... ไม่ evaluate ส่วนขวา
        boolean result1 = (x != 0) && (10 / x > 1);  // ไม่ error เพราะ short-circuit
        System.out.println("Short-circuit AND: " + result1);
        
        // true || ... ไม่ evaluate ส่วนขวา
        boolean result2 = (x == 0) || (10 / x > 1);  // ไม่ error
        System.out.println("Short-circuit OR: " + result2);
    }
}
```

---

## 3.6 Unary Operators

| Operator | ชื่อ | ตัวอย่าง |
|----------|------|---------|
| `+` | Unary plus | `+5` |
| `-` | Unary minus | `-5` |
| `++` | Increment | `++x` หรือ `x++` |
| `--` | Decrement | `--x` หรือ `x--` |
| `!` | Logical NOT | `!true` |

```java
public class UnaryOperators {
    public static void main(String[] args) {
        
        // Increment
        int a = 5;
        System.out.println("a = " + a);      // 5
        
        // Pre-increment: เพิ่มก่อนแล้วค่อยใช้
        System.out.println("++a = " + (++a)); // 6 (a เป็น 6 แล้ว)
        System.out.println("a = " + a);       // 6
        
        // Post-increment: ใช้ก่อนแล้วค่อยเพิ่ม
        System.out.println("a++ = " + (a++)); // 6 (ส่งค่าเก่า 6 ไปก่อน)
        System.out.println("a = " + a);       // 7 (หลังใช้แล้วค่อยเพิ่ม)
        
        // Decrement
        int b = 5;
        System.out.println("\nb = " + b);      // 5
        System.out.println("--b = " + (--b)); // 4
        System.out.println("b-- = " + (b--)); // 4
        System.out.println("b = " + b);       // 3
        
        // Practical example
        System.out.println("\n--- Loop with increment ---");
        int counter = 0;
        counter++;  // คำนวณโดยไม่สนใจ return value
        counter++;
        counter++;
        System.out.println("counter = " + counter); // 3
        
        // Unary minus
        int x = 10;
        int y = -x;
        System.out.println("\nx = " + x + ", -x = " + y);
        
        // Logical NOT
        boolean flag = true;
        System.out.println("flag = " + flag);    // true
        System.out.println("!flag = " + (!flag)); // false
        flag = !flag;
        System.out.println("flag = " + flag);    // false (toggle)
    }
}
```

---

## 3.7 Ternary Operator

Ternary Operator เป็น shorthand ของ if-else:

```java
// syntax: condition ? valueIfTrue : valueIfFalse
int max = (a > b) ? a : b;
```

```java
public class TernaryOperator {
    public static void main(String[] args) {
        
        // Basic ternary
        int a = 10, b = 20;
        int max = (a > b) ? a : b;
        System.out.println("max(" + a + "," + b + ") = " + max);  // 20
        
        // Ternary กับ String
        int score = 75;
        String grade = (score >= 50) ? "ผ่าน" : "ไม่ผ่าน";
        System.out.println("คะแนน " + score + ": " + grade);
        
        // Nested ternary (ไม่แนะนำถ้าซับซ้อนเกินไป)
        String letterGrade = (score >= 80) ? "A" : 
                             (score >= 70) ? "B" : 
                             (score >= 60) ? "C" : 
                             (score >= 50) ? "D" : "F";
        System.out.println("เกรด: " + letterGrade);  // B
        
        // Ternary กับ method call
        int num = -5;
        int absValue = (num >= 0) ? num : -num;
        System.out.println("abs(" + num + ") = " + absValue);
        
        // ใช้ใน println ได้เลย
        boolean isEven = (num % 2 == 0);
        System.out.println(num + " เป็นเลข" + (isEven ? "คู่" : "คี่"));
        
        // ระวัง: ternary ต้อง return ชนิดเดียวกัน
        // int x = (true) ? 1 : "hello";  // Error!
    }
}
```

---

## 3.8 Bitwise Operators

```java
public class BitwiseOperators {
    public static void main(String[] args) {
        
        int a = 60;  // binary: 0011 1100
        int b = 13;  // binary: 0000 1101
        
        System.out.println("a = " + a + " (" + Integer.toBinaryString(a) + ")");
        System.out.println("b = " + b + " (" + Integer.toBinaryString(b) + ")");
        
        // AND (&) - 1 ถ้าทั้งคู่เป็น 1
        int andResult = a & b;   // 0000 1100 = 12
        System.out.println("a & b = " + andResult);
        
        // OR (|) - 1 ถ้าอย่างน้อยหนึ่งเป็น 1
        int orResult = a | b;    // 0011 1101 = 61
        System.out.println("a | b = " + orResult);
        
        // XOR (^) - 1 ถ้าต่างกัน
        int xorResult = a ^ b;   // 0011 0001 = 49
        System.out.println("a ^ b = " + xorResult);
        
        // NOT (~) - กลับบิต
        int notResult = ~a;      // 1100 0011 = -61
        System.out.println("~a = " + notResult);
        
        // Left shift (<<) - เลื่อนบิตซ้าย = คูณด้วย 2^n
        int leftShift = a << 2;  // 0011 1100 << 2 = 1111 0000 = 240
        System.out.println("a << 2 = " + leftShift);  // 240
        
        // Right shift (>>) - เลื่อนบิตขวา = หารด้วย 2^n
        int rightShift = a >> 2; // 0011 1100 >> 2 = 0000 1111 = 15
        System.out.println("a >> 2 = " + rightShift);  // 15
        
        // Practical: เช็คเลขคู่/คี่ด้วย bitwise
        System.out.println("\n--- Bitwise odd/even check ---");
        for (int i = 0; i < 8; i++) {
            System.out.println(i + " & 1 = " + (i & 1) + " → " + ((i & 1) == 0 ? "คู่" : "คี่"));
        }
        
        // Powers of 2 ด้วย left shift
        System.out.println("\n--- Powers of 2 ---");
        for (int i = 0; i < 10; i++) {
            System.out.println("2^" + i + " = " + (1 << i));
        }
    }
}
```

---

## 3.9 instanceof Operator

```java
public class InstanceofOperator {
    public static void main(String[] args) {
        
        Object obj1 = "Hello";  // String
        Object obj2 = 42;       // Integer
        Object obj3 = 3.14;     // Double
        
        // instanceof ตรวจสอบว่า object เป็นชนิดนั้นหรือไม่
        System.out.println("obj1 instanceof String: " + (obj1 instanceof String));   // true
        System.out.println("obj1 instanceof Integer: " + (obj1 instanceof Integer)); // false
        System.out.println("obj2 instanceof Integer: " + (obj2 instanceof Integer)); // true
        System.out.println("null instanceof String: " + (null instanceof String));   // false
        
        // ใช้งานจริง
        Object[] objects = {"Text", 42, 3.14, true};
        
        for (Object obj : objects) {
            if (obj instanceof String str) {  // Pattern matching (Java 16+)
                System.out.println("String: " + str.toUpperCase());
            } else if (obj instanceof Integer num) {
                System.out.println("Integer: " + (num * 2));
            } else if (obj instanceof Double d) {
                System.out.printf("Double: %.3f%n", d);
            } else if (obj instanceof Boolean b) {
                System.out.println("Boolean: " + !b);
            }
        }
    }
}
```

---

## 3.10 Operator Precedence (ลำดับความสำคัญ)

```
ลำดับความสำคัญ (สูงไปต่ำ):
1. ()                    parentheses
2. ++, --, !, ~, +, -    unary
3. *, /, %               multiplicative
4. +, -                  additive
5. <<, >>, >>>           shift
6. <, >, <=, >=          relational
7. ==, !=                equality
8. &                     bitwise AND
9. ^                     bitwise XOR
10. |                    bitwise OR
11. &&                   logical AND
12. ||                   logical OR
13. ?:                   ternary
14. =, +=, -=, etc.      assignment
```

```java
public class OperatorPrecedence {
    public static void main(String[] args) {
        
        // คำนวณตามลำดับความสำคัญ
        int result1 = 2 + 3 * 4;       // 2 + 12 = 14 (ไม่ใช่ 20)
        int result2 = (2 + 3) * 4;     // 5 * 4 = 20
        
        System.out.println("2 + 3 * 4 = " + result1);    // 14
        System.out.println("(2 + 3) * 4 = " + result2);  // 20
        
        // Complex expression
        int a = 5, b = 3, c = 2;
        int result3 = a + b * c - a / b + a % c;
        // = 5 + (3*2) - (5/3) + (5%2)
        // = 5 + 6 - 1 + 1
        // = 11
        System.out.println("5 + 3*2 - 5/3 + 5%2 = " + result3);  // 11
        
        // Logical precedence
        boolean x = true, y = false, z = true;
        boolean result4 = x || y && z;    // x || (y && z) = true || false = true
        boolean result5 = (x || y) && z;  // (true || false) && true = true
        
        System.out.println("\ntrue || false && true = " + result4);    // true
        System.out.println("(true || false) && true = " + result5);   // true
        
        // ตัวอย่างที่สับสน
        int n = 5;
        int result6 = n++ + ++n;
        // n++ ส่ง 5 แล้ว n=6
        // ++n เพิ่ม n=7 แล้วส่ง 7
        // result = 5 + 7 = 12
        System.out.println("\nn = 5; n++ + ++n = " + result6 + " (n is now " + n + ")");
        
        // คำแนะนำ: ใช้วงเล็บเสมอถ้าไม่แน่ใจ
        int safe = (2 + 3) * (4 - 1);  // ชัดเจน
        System.out.println("(2+3)*(4-1) = " + safe);  // 15
    }
}
```

---

## 3.11 โปรแกรมตัวอย่าง: Grade Calculator

```java
import java.util.Scanner;

public class GradeCalculator {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        
        System.out.println("=== ระบบคำนวณเกรด ===");
        
        System.out.print("คะแนน Midterm (40%): ");
        double midterm = sc.nextDouble();
        
        System.out.print("คะแนน Final (40%): ");
        double finalExam = sc.nextDouble();
        
        System.out.print("คะแนน Assignment (20%): ");
        double assignment = sc.nextDouble();
        
        // คำนวณคะแนนรวม
        double total = (midterm * 0.40) + (finalExam * 0.40) + (assignment * 0.20);
        
        // กำหนดเกรด
        String grade;
        String status;
        
        if (total >= 80) {
            grade = "A";
        } else if (total >= 70) {
            grade = "B";
        } else if (total >= 60) {
            grade = "C";
        } else if (total >= 50) {
            grade = "D";
        } else {
            grade = "F";
        }
        
        status = (total >= 50) ? "ผ่าน ✓" : "ไม่ผ่าน ✗";
        
        // แสดงผล
        System.out.println("\n" + "=".repeat(35));
        System.out.println("         ใบรายงานผล");
        System.out.println("=".repeat(35));
        System.out.printf("Midterm   (40%%): %5.1f × 0.40 = %5.2f%n", midterm, midterm * 0.40);
        System.out.printf("Final     (40%%): %5.1f × 0.40 = %5.2f%n", finalExam, finalExam * 0.40);
        System.out.printf("Assignment(20%%): %5.1f × 0.20 = %5.2f%n", assignment, assignment * 0.20);
        System.out.println("-".repeat(35));
        System.out.printf("คะแนนรวม        : %5.2f / 100%n", total);
        System.out.printf("เกรด            : %s%n", grade);
        System.out.printf("สถานะ           : %s%n", status);
        System.out.println("=".repeat(35));
        
        sc.close();
    }
}
```

---

## 3.12 โปรแกรมตัวอย่าง: Number Analyzer

```java
import java.util.Scanner;

public class NumberAnalyzer {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        
        System.out.print("ใส่จำนวนเต็ม: ");
        int n = sc.nextInt();
        
        System.out.println("\n=== วิเคราะห์เลข " + n + " ===");
        
        // เลขบวก/ลบ/ศูนย์
        System.out.println("ประเภท: " + (n > 0 ? "บวก" : n < 0 ? "ลบ" : "ศูนย์"));
        
        // เลขคู่/คี่
        System.out.println("คู่/คี่: " + (n % 2 == 0 ? "เลขคู่" : "เลขคี่"));
        
        // หารด้วย 5 ลงตัว?
        System.out.println("หาร 5 ลงตัว: " + (n % 5 == 0 ? "ใช่" : "ไม่ใช่"));
        
        // หารด้วย 3 ลงตัว?
        System.out.println("หาร 3 ลงตัว: " + (n % 3 == 0 ? "ใช่" : "ไม่ใช่"));
        
        // FizzBuzz
        String fizzBuzz;
        if (n % 15 == 0) fizzBuzz = "FizzBuzz";
        else if (n % 3 == 0) fizzBuzz = "Fizz";
        else if (n % 5 == 0) fizzBuzz = "Buzz";
        else fizzBuzz = String.valueOf(n);
        System.out.println("FizzBuzz: " + fizzBuzz);
        
        // จำนวนหลัก
        int absN = Math.abs(n);
        int digits = (absN == 0) ? 1 : (int) Math.log10(absN) + 1;
        System.out.println("จำนวนหลัก: " + digits);
        
        // Binary representation
        System.out.println("Binary: " + Integer.toBinaryString(n));
        System.out.println("Hex: " + Integer.toHexString(n));
        System.out.println("Octal: " + Integer.toOctalString(n));
        
        sc.close();
    }
}
```

---

## 3.13 แบบฝึกหัด Part 03

### แบบฝึกหัดที่ 1: Operator Practice

```java
public class OperatorPractice {
    public static void main(String[] args) {
        int a = 15, b = 4;
        
        // TODO: คำนวณและแสดงผล
        // 1. ผลรวม ผลต่าง ผลคูณ ผลหาร เศษหาร
        // 2. ผลของ a++ และ ++a
        // 3. ผลของ (a > b) && (a % 2 == 1)
        // 4. a ternary ที่แสดงว่า a หรือ b ใหญ่กว่า
    }
}
```

**เฉลย:**
```java
public class OperatorSolution {
    public static void main(String[] args) {
        int a = 15, b = 4;
        
        System.out.println("--- Arithmetic ---");
        System.out.printf("a + b = %d%n", a + b);   // 19
        System.out.printf("a - b = %d%n", a - b);   // 11
        System.out.printf("a * b = %d%n", a * b);   // 60
        System.out.printf("a / b = %d%n", a / b);   // 3
        System.out.printf("a %% b = %d%n", a % b);  // 3
        
        System.out.println("\n--- Unary ---");
        int x = a;
        System.out.println("a = " + a);
        System.out.println("a++ = " + x++);  // 15 (ใช้แล้วค่อยเพิ่ม)
        System.out.println("a is now: " + x); // 16
        System.out.println("++a = " + (++x)); // 17 (เพิ่มก่อนแล้วค่อยใช้)
        
        System.out.println("\n--- Logical ---");
        boolean result = (a > b) && (a % 2 == 1);
        System.out.println("(a > b) && (a % 2 == 1) = " + result);  // true
        
        System.out.println("\n--- Ternary ---");
        String larger = (a > b) ? "a (" + a + ")" : "b (" + b + ")";
        System.out.println("ใหญ่กว่าคือ: " + larger);
    }
}
```

### แบบฝึกหัดที่ 2: เครื่องคิดเลขขั้นสูง

```java
import java.util.Scanner;

public class AdvancedCalculator {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        
        System.out.print("ใส่ตัวเลข a: ");
        double a = sc.nextDouble();
        
        System.out.print("ใส่ตัวเลข b: ");
        double b = sc.nextDouble();
        
        System.out.println("\n=== ผลการคำนวณ ===");
        System.out.printf("a + b = %.4f%n", a + b);
        System.out.printf("a - b = %.4f%n", a - b);
        System.out.printf("a * b = %.4f%n", a * b);
        
        if (b != 0) {
            System.out.printf("a / b = %.4f%n", a / b);
            System.out.printf("a %% b = %.4f%n", a % b);
        } else {
            System.out.println("หารด้วย 0 ไม่ได้");
        }
        
        System.out.printf("a^b = %.4f%n", Math.pow(a, b));
        System.out.printf("|a| = %.4f%n", Math.abs(a));
        System.out.printf("√|a| = %.4f%n", Math.sqrt(Math.abs(a)));
        
        sc.close();
    }
}
```

### แบบฝึกหัดที่ 3: Swap Variables

```java
public class SwapVariables {
    public static void main(String[] args) {
        int a = 10, b = 20;
        System.out.println("ก่อน swap: a=" + a + ", b=" + b);
        
        // วิธีที่ 1: ใช้ตัวแปร temp
        int temp = a;
        a = b;
        b = temp;
        System.out.println("หลัง swap (temp): a=" + a + ", b=" + b);
        
        // วิธีที่ 2: ใช้ arithmetic
        a = a + b;
        b = a - b;
        a = a - b;
        System.out.println("หลัง swap (arithmetic): a=" + a + ", b=" + b);
        
        // วิธีที่ 3: ใช้ XOR
        a = a ^ b;
        b = a ^ b;
        a = a ^ b;
        System.out.println("หลัง swap (XOR): a=" + a + ", b=" + b);
    }
}
```

---

## 3.14 สรุป Part 03

ในบทนี้คุณได้เรียนรู้:

✅ Arithmetic Operators (+, -, *, /, %)  
✅ Assignment Operators (=, +=, -=, *=, /=, %=)  
✅ Comparison Operators (==, !=, >, <, >=, <=)  
✅ Logical Operators (&&, ||, !)  
✅ Unary Operators (++, --, !, +, -)  
✅ Ternary Operator (?:)  
✅ Bitwise Operators (&, |, ^, ~, <<, >>)  
✅ instanceof Operator  
✅ Operator Precedence  

---

*[← Part 02: Variables & Data Types](./part-02-variables-datatypes.md) | [Part 04: Control Flow →](./part-04-control-flow.md)*
