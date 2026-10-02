# Part 04: Control Flow - if/else & switch
## หลักสูตร Java & Android Development - ระดับเริ่มต้น

---

## 4.1 Control Flow คืออะไร?

Control Flow (การควบคุมการทำงาน) คือความสามารถในการกำหนดว่าโปรแกรมจะทำงานส่วนไหน ในเงื่อนไขใด Java มีโครงสร้างควบคุมดังนี้:

```
โปรแกรมจะทำงานตามลำดับ (Sequential)
  ↓
  ↓ ← if/else (conditional branching)
  ↓
  ↓ ← switch (multiple choices)
  ↓
  ↓ ← loops (repetition) [Part 05]
```

---

## 4.2 if Statement

```java
// Syntax
if (condition) {
    // ทำงานถ้า condition เป็น true
}

// ตัวอย่าง
int age = 18;
if (age >= 18) {
    System.out.println("ผู้ใหญ่");
}
```

```java
public class IfStatement {
    public static void main(String[] args) {
        
        int score = 75;
        
        // if เดียว
        if (score >= 50) {
            System.out.println("สอบผ่าน!");
        }
        
        // if กับ single statement (ไม่ใส่ {} ได้ แต่ไม่แนะนำ)
        if (score >= 50)
            System.out.println("ผ่าน");  // ใช้ได้แต่อ่านยาก
        
        // if กับ complex condition
        if (score >= 60 && score < 80) {
            System.out.println("ได้เกรด C หรือ B");
        }
        
        // if กับ boolean variable
        boolean isValid = (score >= 0 && score <= 100);
        if (isValid) {
            System.out.println("คะแนนถูกต้อง");
        }
    }
}
```

---

## 4.3 if-else Statement

```java
public class IfElse {
    public static void main(String[] args) {
        int number = -5;
        
        // if-else
        if (number >= 0) {
            System.out.println(number + " เป็นเลขบวก (หรือศูนย์)");
        } else {
            System.out.println(number + " เป็นเลขลบ");
        }
        
        // if-else if-else chain
        int score = 85;
        String grade;
        
        if (score >= 90) {
            grade = "A";
        } else if (score >= 80) {
            grade = "B";
        } else if (score >= 70) {
            grade = "C";
        } else if (score >= 60) {
            grade = "D";
        } else {
            grade = "F";
        }
        
        System.out.println("คะแนน " + score + " → เกรด " + grade);
        
        // Nested if-else
        int x = 10, y = 20;
        
        if (x > 0) {
            if (y > 0) {
                System.out.println("ทั้ง x และ y เป็นบวก");
            } else {
                System.out.println("x เป็นบวก แต่ y ไม่บวก");
            }
        } else {
            System.out.println("x ไม่บวก");
        }
    }
}
```

---

## 4.4 switch Statement

switch ใช้เมื่อต้องเปรียบเทียบค่ากับหลายๆ case

```java
// Syntax
switch (expression) {
    case value1:
        // code
        break;
    case value2:
        // code
        break;
    default:
        // code ถ้าไม่ตรงกับ case ใดเลย
}
```

```java
public class SwitchStatement {
    public static void main(String[] args) {
        
        int day = 3;
        String dayName;
        
        switch (day) {
            case 1:
                dayName = "จันทร์";
                break;
            case 2:
                dayName = "อังคาร";
                break;
            case 3:
                dayName = "พุธ";
                break;
            case 4:
                dayName = "พฤหัสบดี";
                break;
            case 5:
                dayName = "ศุกร์";
                break;
            case 6:
                dayName = "เสาร์";
                break;
            case 7:
                dayName = "อาทิตย์";
                break;
            default:
                dayName = "ไม่ถูกต้อง";
        }
        
        System.out.println("วันที่ " + day + " คือวัน" + dayName);
        
        // switch กับ String
        String season = "summer";
        
        switch (season) {
            case "spring":
                System.out.println("ฤดูใบไม้ผลิ - อากาศอบอุ่น");
                break;
            case "summer":
                System.out.println("ฤดูร้อน - อากาศร้อน");
                break;
            case "autumn":
                System.out.println("ฤดูใบไม้ร่วง - อากาศเย็นขึ้น");
                break;
            case "winter":
                System.out.println("ฤดูหนาว - อากาศหนาว");
                break;
            default:
                System.out.println("ไม่รู้จักฤดูกาลนี้");
        }
    }
}
```

### Fall-through ใน switch

```java
public class SwitchFallThrough {
    public static void main(String[] args) {
        
        // Fall-through: ถ้าไม่มี break จะทำงานต่อ case ถัดไป
        int month = 4;  // เมษายน
        
        System.out.print("เดือนที่ " + month + " มี ");
        
        switch (month) {
            case 2:
                System.out.println("28 หรือ 29 วัน");
                break;
            case 4:
            case 6:
            case 9:
            case 11:
                System.out.println("30 วัน");  // เดือน 4,6,9,11 ตกที่นี่
                break;
            default:
                System.out.println("31 วัน");
        }
        
        // Fall-through โดยตั้งใจ
        int level = 2;
        System.out.print("Level " + level + " สิทธิ์: ");
        
        switch (level) {
            case 3:
                System.out.print("Admin ");   // ไม่มี break -> fall-through
            case 2:
                System.out.print("Delete "); // ไม่มี break -> fall-through
            case 1:
                System.out.print("Write ");  // ไม่มี break -> fall-through
            case 0:
                System.out.print("Read");
                break;
            default:
                System.out.print("None");
        }
        System.out.println();
    }
}
```

---

## 4.5 switch Expression (Java 14+)

```java
public class SwitchExpression {
    public static void main(String[] args) {
        
        // Switch Expression - แบบใหม่ (Java 14+)
        int day = 3;
        
        // ไม่ต้องใช้ break อีกต่อไป
        String dayName = switch (day) {
            case 1 -> "จันทร์";
            case 2 -> "อังคาร";
            case 3 -> "พุธ";
            case 4 -> "พฤหัสบดี";
            case 5 -> "ศุกร์";
            case 6 -> "เสาร์";
            case 7 -> "อาทิตย์";
            default -> "ไม่ถูกต้อง";
        };
        
        System.out.println("วัน: " + dayName);
        
        // Multiple values per case
        boolean isWeekend = switch (day) {
            case 6, 7 -> true;
            default -> false;
        };
        
        System.out.println("วันหยุด: " + isWeekend);
        
        // Switch Expression กับ block
        String message = switch (day) {
            case 1, 2, 3, 4, 5 -> {
                int remaining = 5 - day;
                yield "วันทำงาน เหลืออีก " + remaining + " วัน";
            }
            case 6, 7 -> "วันหยุด สบายๆ";
            default -> "ไม่ถูกต้อง";
        };
        
        System.out.println(message);
        
        // Switch กับ enum
        String grade = "B";
        int points = switch (grade) {
            case "A" -> 4;
            case "B" -> 3;
            case "C" -> 2;
            case "D" -> 1;
            default -> 0;
        };
        System.out.println("เกรด " + grade + " = " + points + " หน่วยกิต");
    }
}
```

---

## 4.6 โปรแกรมตัวอย่าง: Menu System

```java
import java.util.Scanner;

public class MenuSystem {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        
        System.out.println("╔══════════════════════╗");
        System.out.println("║   ร้านอาหาร Java Cafe  ║");
        System.out.println("╠══════════════════════╣");
        System.out.println("║ 1. ข้าวผัด      45 บาท ║");
        System.out.println("║ 2. ราดหน้า      50 บาท ║");
        System.out.println("║ 3. ก๋วยเตี๋ยว   35 บาท ║");
        System.out.println("║ 4. ข้าวมันไก่   55 บาท ║");
        System.out.println("║ 5. ออกจากร้าน          ║");
        System.out.println("╚══════════════════════╝");
        System.out.print("เลือกเมนู (1-5): ");
        
        int choice = sc.nextInt();
        
        String foodName;
        int price;
        
        switch (choice) {
            case 1:
                foodName = "ข้าวผัด";
                price = 45;
                break;
            case 2:
                foodName = "ราดหน้า";
                price = 50;
                break;
            case 3:
                foodName = "ก๋วยเตี๋ยว";
                price = 35;
                break;
            case 4:
                foodName = "ข้าวมันไก่";
                price = 55;
                break;
            case 5:
                System.out.println("ขอบคุณที่ใช้บริการ!");
                sc.close();
                return;
            default:
                System.out.println("ไม่มีเมนูนี้!");
                sc.close();
                return;
        }
        
        System.out.print("จำนวน: ");
        int qty = sc.nextInt();
        
        int total = price * qty;
        double vat = total * 0.07;
        double grandTotal = total + vat;
        
        System.out.println("\n=== ใบเสร็จ ===");
        System.out.printf("%s × %d = %d บาท%n", foodName, qty, total);
        System.out.printf("VAT 7%%           = %.2f บาท%n", vat);
        System.out.printf("รวมทั้งสิ้น       = %.2f บาท%n", grandTotal);
        
        sc.close();
    }
}
```

---

## 4.7 โปรแกรมตัวอย่าง: Leap Year Checker

```java
import java.util.Scanner;

public class LeapYearChecker {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        
        System.out.print("ใส่ปี (ค.ศ.): ");
        int year = sc.nextInt();
        
        // กฎปีอธิกสุรทิน:
        // หารด้วย 4 ลงตัว AND
        // (ไม่หารด้วย 100 ลงตัว OR หารด้วย 400 ลงตัว)
        boolean isLeap;
        
        if (year % 400 == 0) {
            isLeap = true;
        } else if (year % 100 == 0) {
            isLeap = false;
        } else if (year % 4 == 0) {
            isLeap = true;
        } else {
            isLeap = false;
        }
        
        // หรือเขียนสั้นๆ
        // boolean isLeap = (year % 4 == 0 && year % 100 != 0) || (year % 400 == 0);
        
        if (isLeap) {
            System.out.println("ปี " + year + " เป็นปีอธิกสุรทิน (มี 366 วัน)");
        } else {
            System.out.println("ปี " + year + " ไม่ใช่ปีอธิกสุรทิน (มี 365 วัน)");
        }
        
        // แสดงจำนวนวันใน February
        int febDays = isLeap ? 29 : 28;
        System.out.println("กุมภาพันธ์ปีนี้มี " + febDays + " วัน");
        
        sc.close();
    }
}
```

---

## 4.8 โปรแกรมตัวอย่าง: Season Detector

```java
import java.util.Scanner;

public class SeasonDetector {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        
        System.out.print("ใส่เดือน (1-12): ");
        int month = sc.nextInt();
        
        // ตรวจสอบว่าเดือนถูกต้อง
        if (month < 1 || month > 12) {
            System.out.println("เดือนไม่ถูกต้อง!");
            sc.close();
            return;
        }
        
        String monthName;
        String season;
        String weather;
        int daysInMonth;
        
        // ชื่อเดือน
        switch (month) {
            case 1:  monthName = "มกราคม";     daysInMonth = 31; break;
            case 2:  monthName = "กุมภาพันธ์"; daysInMonth = 28; break;
            case 3:  monthName = "มีนาคม";     daysInMonth = 31; break;
            case 4:  monthName = "เมษายน";     daysInMonth = 30; break;
            case 5:  monthName = "พฤษภาคม";   daysInMonth = 31; break;
            case 6:  monthName = "มิถุนายน";   daysInMonth = 30; break;
            case 7:  monthName = "กรกฎาคม";   daysInMonth = 31; break;
            case 8:  monthName = "สิงหาคม";   daysInMonth = 31; break;
            case 9:  monthName = "กันยายน";   daysInMonth = 30; break;
            case 10: monthName = "ตุลาคม";    daysInMonth = 31; break;
            case 11: monthName = "พฤศจิกายน"; daysInMonth = 30; break;
            case 12: monthName = "ธันวาคม";   daysInMonth = 31; break;
            default: monthName = "ไม่ถูกต้อง"; daysInMonth = 0;
        }
        
        // ฤดูกาล (สำหรับประเทศไทย)
        if (month >= 3 && month <= 5) {
            season = "ฤดูร้อน";
            weather = "ร้อน อากาศแห้ง";
        } else if (month >= 6 && month <= 10) {
            season = "ฤดูฝน";
            weather = "ฝนตก อากาศชื้น";
        } else {
            season = "ฤดูหนาว";
            weather = "เย็น อากาศแห้ง";
        }
        
        System.out.println("\n=== ข้อมูลเดือน ===");
        System.out.println("เดือน: " + monthName);
        System.out.println("จำนวนวัน: " + daysInMonth + " วัน");
        System.out.println("ฤดูกาล: " + season);
        System.out.println("สภาพอากาศ: " + weather);
        
        sc.close();
    }
}
```

---

## 4.9 โปรแกรมตัวอย่าง: Simple ATM

```java
import java.util.Scanner;

public class SimpleATM {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        
        // ข้อมูลบัญชี
        String accountNo = "1234567890";
        String pin = "1234";
        double balance = 10000.00;
        
        System.out.println("╔═══════════════════╗");
        System.out.println("║   ยินดีต้อนรับ ATM  ║");
        System.out.println("╚═══════════════════╝");
        
        // ตรวจสอบ PIN
        System.out.print("เลขบัตร: ");
        String inputAccount = sc.next();
        
        System.out.print("PIN: ");
        String inputPin = sc.next();
        
        if (!inputAccount.equals(accountNo) || !inputPin.equals(pin)) {
            System.out.println("เลขบัตรหรือ PIN ไม่ถูกต้อง!");
            sc.close();
            return;
        }
        
        System.out.println("เข้าสู่ระบบสำเร็จ!");
        System.out.println();
        
        // เมนูหลัก
        System.out.println("1. เช็คยอดเงิน");
        System.out.println("2. ฝากเงิน");
        System.out.println("3. ถอนเงิน");
        System.out.println("4. ออกจากระบบ");
        System.out.print("เลือก: ");
        
        int choice = sc.nextInt();
        
        switch (choice) {
            case 1:
                System.out.printf("ยอดเงินคงเหลือ: %,.2f บาท%n", balance);
                break;
                
            case 2:
                System.out.print("จำนวนเงินที่ต้องการฝาก: ");
                double deposit = sc.nextDouble();
                
                if (deposit <= 0) {
                    System.out.println("จำนวนเงินไม่ถูกต้อง");
                } else {
                    balance += deposit;
                    System.out.printf("ฝากเงินสำเร็จ!%n");
                    System.out.printf("ยอดเงินใหม่: %,.2f บาท%n", balance);
                }
                break;
                
            case 3:
                System.out.print("จำนวนเงินที่ต้องการถอน: ");
                double withdraw = sc.nextDouble();
                
                if (withdraw <= 0) {
                    System.out.println("จำนวนเงินไม่ถูกต้อง");
                } else if (withdraw > balance) {
                    System.out.println("เงินในบัญชีไม่พอ!");
                    System.out.printf("ยอดเงินปัจจุบัน: %,.2f บาท%n", balance);
                } else {
                    balance -= withdraw;
                    System.out.printf("ถอนเงินสำเร็จ!%n");
                    System.out.printf("ยอดเงินคงเหลือ: %,.2f บาท%n", balance);
                }
                break;
                
            case 4:
                System.out.println("ขอบคุณที่ใช้บริการ!");
                break;
                
            default:
                System.out.println("ตัวเลือกไม่ถูกต้อง");
        }
        
        sc.close();
    }
}
```

---

## 4.10 Pattern Matching (Java 16+)

```java
public class PatternMatching {
    public static void main(String[] args) {
        
        // instanceof Pattern Matching
        Object obj = "Hello, World!";
        
        // แบบเก่า
        if (obj instanceof String) {
            String s = (String) obj;
            System.out.println("Length: " + s.length());
        }
        
        // แบบใหม่ (Java 16+)
        if (obj instanceof String s) {
            System.out.println("Length: " + s.length());
            System.out.println("Upper: " + s.toUpperCase());
        }
        
        // switch Pattern Matching (Java 21+)
        Object[] objects = {42, "Hello", 3.14, true, null};
        
        for (Object o : objects) {
            String description = switch (o) {
                case Integer i -> "Integer: " + i;
                case String s -> "String: " + s;
                case Double d -> String.format("Double: %.2f", d);
                case Boolean b -> "Boolean: " + b;
                case null -> "null value";
                default -> "Unknown: " + o;
            };
            System.out.println(description);
        }
    }
}
```

---

## 4.11 Common Mistakes กับ if-else

```java
public class CommonMistakes {
    public static void main(String[] args) {
        
        // Mistake 1: ใช้ = แทน ==
        int x = 5;
        // if (x = 5) { ... }  // Error: ไม่สามารถใช้ = ใน condition
        if (x == 5) {
            System.out.println("x equals 5");  // ✓
        }
        
        // Mistake 2: Dangling else
        int a = 1, b = 2;
        if (a == 1)
            if (b == 2)
                System.out.println("a=1 and b=2");
        else
            System.out.println("This else belongs to inner if!");  // ระวัง!
        
        // ใส่ {} ให้ชัดเจน
        if (a == 1) {
            if (b == 2) {
                System.out.println("a=1 and b=2");
            }
        } else {
            System.out.println("a is not 1");  // ชัดเจน
        }
        
        // Mistake 3: Float comparison
        double d = 0.1 + 0.2;
        if (d == 0.3) {
            System.out.println("equal");  // ไม่ execute!
        }
        // แก้ไข:
        if (Math.abs(d - 0.3) < 1e-10) {
            System.out.println("approximately equal");  // ✓
        }
        
        // Mistake 4: Boolean unnecessary comparison
        boolean flag = true;
        if (flag == true) { ... }  // ไม่จำเป็น
        if (flag) { ... }          // ✓ เขียนแบบนี้ดีกว่า
        
        if (flag == false) { ... } // ไม่จำเป็น
        if (!flag) { ... }         // ✓ เขียนแบบนี้ดีกว่า
        
        // Mistake 5: String comparison ด้วย ==
        String s1 = new String("hello");
        String s2 = new String("hello");
        if (s1 == s2) {
            System.out.println("same");  // ไม่ execute!
        }
        if (s1.equals(s2)) {
            System.out.println("same content");  // ✓
        }
    }
}
```

---

## 4.12 แบบฝึกหัด Part 04

### แบบฝึกหัดที่ 1: Triangle Type Checker

```java
import java.util.Scanner;

// TODO: สร้างโปรแกรมรับด้าน 3 ด้านของสามเหลี่ยม
// แล้วบอกว่าเป็นสามเหลี่ยมชนิดใด:
// - ด้านเท่า (Equilateral): ทั้ง 3 ด้านเท่ากัน
// - หน้าจั่ว (Isosceles): 2 ด้านเท่ากัน
// - ด้านไม่เท่า (Scalene): ทุกด้านไม่เท่ากัน
// - ไม่ใช่สามเหลี่ยม: ตรวจสอบ triangle inequality
```

**เฉลย:**
```java
import java.util.Scanner;

public class TriangleChecker {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        
        System.out.println("=== ตรวจสอบรูปสามเหลี่ยม ===");
        System.out.print("ด้านที่ 1: ");
        double a = sc.nextDouble();
        System.out.print("ด้านที่ 2: ");
        double b = sc.nextDouble();
        System.out.print("ด้านที่ 3: ");
        double c = sc.nextDouble();
        
        // ตรวจสอบว่าเป็นสามเหลี่ยมได้
        if (a + b > c && b + c > a && a + c > b) {
            System.out.print("ชนิด: ");
            
            if (a == b && b == c) {
                System.out.println("สามเหลี่ยมด้านเท่า (Equilateral)");
            } else if (a == b || b == c || a == c) {
                System.out.println("สามเหลี่ยมหน้าจั่ว (Isosceles)");
            } else {
                System.out.println("สามเหลี่ยมด้านไม่เท่า (Scalene)");
            }
            
            // คำนวณพื้นที่ (Heron's formula)
            double s = (a + b + c) / 2;
            double area = Math.sqrt(s * (s-a) * (s-b) * (s-c));
            System.out.printf("พื้นที่: %.2f%n", area);
            
        } else {
            System.out.println("ไม่ใช่รูปสามเหลี่ยม!");
        }
        
        sc.close();
    }
}
```

### แบบฝึกหัดที่ 2: Calculator with Switch

```java
import java.util.Scanner;

public class SwitchCalculator {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        
        System.out.print("ตัวเลขที่ 1: ");
        double a = sc.nextDouble();
        
        System.out.print("ตัวดำเนินการ (+, -, *, /): ");
        String op = sc.next();
        
        System.out.print("ตัวเลขที่ 2: ");
        double b = sc.nextDouble();
        
        double result;
        boolean valid = true;
        
        switch (op) {
            case "+": result = a + b; break;
            case "-": result = a - b; break;
            case "*": result = a * b; break;
            case "/":
                if (b == 0) {
                    System.out.println("หารด้วย 0 ไม่ได้!");
                    valid = false;
                    result = 0;
                } else {
                    result = a / b;
                }
                break;
            default:
                System.out.println("ตัวดำเนินการไม่ถูกต้อง!");
                valid = false;
                result = 0;
        }
        
        if (valid) {
            System.out.printf("%.2f %s %.2f = %.4f%n", a, op, b, result);
        }
        
        sc.close();
    }
}
```

### แบบฝึกหัดที่ 3: Student Grade System

```java
import java.util.Scanner;

public class StudentGradeSystem {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        
        System.out.println("=== ระบบตัดเกรดนักเรียน ===");
        
        System.out.print("ชื่อนักเรียน: ");
        String name = sc.nextLine();
        
        System.out.print("คะแนนสอบ (0-100): ");
        double score = sc.nextDouble();
        
        // Validate
        if (score < 0 || score > 100) {
            System.out.println("คะแนนไม่ถูกต้อง (ต้องอยู่ระหว่าง 0-100)");
            sc.close();
            return;
        }
        
        String grade;
        String comment;
        double gpa;
        
        if (score >= 80) {
            grade = "A";
            gpa = 4.0;
            comment = "ยอดเยี่ยม!";
        } else if (score >= 75) {
            grade = "B+";
            gpa = 3.5;
            comment = "ดีมาก!";
        } else if (score >= 70) {
            grade = "B";
            gpa = 3.0;
            comment = "ดี";
        } else if (score >= 65) {
            grade = "C+";
            gpa = 2.5;
            comment = "ค่อนข้างดี";
        } else if (score >= 60) {
            grade = "C";
            gpa = 2.0;
            comment = "พอใช้";
        } else if (score >= 55) {
            grade = "D+";
            gpa = 1.5;
            comment = "ต้องปรับปรุง";
        } else if (score >= 50) {
            grade = "D";
            gpa = 1.0;
            comment = "ต้องพยายามมากขึ้น";
        } else {
            grade = "F";
            gpa = 0.0;
            comment = "สอบตก";
        }
        
        System.out.println("\n=== ผลการสอบ ===");
        System.out.println("ชื่อ     : " + name);
        System.out.printf("คะแนน   : %.1f/100%n", score);
        System.out.println("เกรด    : " + grade);
        System.out.printf("GPA     : %.1f%n", gpa);
        System.out.println("ความเห็น : " + comment);
        System.out.println("สถานะ   : " + (gpa >= 1.0 ? "✓ ผ่าน" : "✗ ไม่ผ่าน"));
        
        sc.close();
    }
}
```

---

## 4.13 สรุป Part 04

ในบทนี้คุณได้เรียนรู้:

✅ if statement  
✅ if-else statement  
✅ if-else if-else chain  
✅ Nested if-else  
✅ switch statement (แบบเก่า)  
✅ switch Expression (Java 14+, แบบใหม่)  
✅ Fall-through ใน switch  
✅ Pattern Matching (Java 16+)  
✅ Common Mistakes และวิธีแก้  

---

*[← Part 03: Operators](./part-03-operators.md) | [Part 05: Loops →](./part-05-loops.md)*
