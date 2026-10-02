# Part 05: Loops - for, while, do-while
## หลักสูตร Java & Android Development - ระดับเริ่มต้น

---

## 5.1 Loop คืออะไร?

Loop (วนซ้ำ) คือโครงสร้างที่ทำให้โปรแกรมทำงานซ้ำๆ โดยไม่ต้องเขียนโค้ดซ้ำ Java มี Loop 3 ประเภทหลัก:

1. **for loop** - รู้จำนวนรอบที่แน่นอน
2. **while loop** - ไม่รู้จำนวนรอบ วนจนกว่าเงื่อนไขจะเป็น false
3. **do-while loop** - ทำอย่างน้อย 1 ครั้ง แล้วค่อยตรวจเงื่อนไข

---

## 5.2 for Loop

```java
// Syntax
for (initialization; condition; update) {
    // code ที่ทำซ้ำ
}
```

```java
public class ForLoop {
    public static void main(String[] args) {
        
        // Basic for loop
        System.out.println("--- นับ 1-5 ---");
        for (int i = 1; i <= 5; i++) {
            System.out.println("i = " + i);
        }
        
        // นับถอยหลัง
        System.out.println("\n--- นับถอยหลัง ---");
        for (int i = 10; i >= 1; i--) {
            System.out.print(i + " ");
        }
        System.out.println();
        
        // ข้ามทีละ 2
        System.out.println("\n--- เลขคู่ 2-20 ---");
        for (int i = 2; i <= 20; i += 2) {
            System.out.print(i + " ");
        }
        System.out.println();
        
        // สูตรคูณแม่ 3
        System.out.println("\n--- สูตรคูณแม่ 3 ---");
        for (int i = 1; i <= 12; i++) {
            System.out.printf("3 × %2d = %2d%n", i, 3 * i);
        }
        
        // ผลรวม 1-100
        int sum = 0;
        for (int i = 1; i <= 100; i++) {
            sum += i;
        }
        System.out.println("\nผลรวม 1 ถึง 100 = " + sum);  // 5050
        
        // Factorial
        int n = 10;
        long factorial = 1;
        for (int i = 1; i <= n; i++) {
            factorial *= i;
        }
        System.out.println(n + "! = " + factorial);
    }
}
```

### for Loop แบบต่างๆ

```java
public class ForLoopVariations {
    public static void main(String[] args) {
        
        // Multiple variables
        System.out.println("--- Multiple init/update ---");
        for (int i = 0, j = 10; i < j; i++, j--) {
            System.out.println("i=" + i + ", j=" + j);
        }
        
        // Infinite loop (ต้องมี break)
        System.out.println("\n--- Loop with break ---");
        int count = 0;
        for (;;) {  // หรือ for (;;)
            count++;
            if (count >= 5) break;
            System.out.println("count = " + count);
        }
        
        // ไม่มี body (empty body)
        int x = 1;
        for (; x < 100; x *= 2);  // หา 2^n ที่ >= 100
        System.out.println("\nFirst power of 2 >= 100: " + x);  // 128
        
        // for กับ String
        String text = "Hello";
        for (int i = 0; i < text.length(); i++) {
            System.out.print(text.charAt(i) + " ");
        }
        System.out.println();
    }
}
```

---

## 5.3 Nested for Loop

```java
public class NestedForLoop {
    public static void main(String[] args) {
        
        // สร้างตาราง multiplication table
        System.out.println("=== ตารางสูตรคูณ ===");
        System.out.print("   ");
        for (int i = 1; i <= 9; i++) {
            System.out.printf("%4d", i);
        }
        System.out.println();
        System.out.println("  " + "-".repeat(37));
        
        for (int i = 1; i <= 9; i++) {
            System.out.printf("%2d |", i);
            for (int j = 1; j <= 9; j++) {
                System.out.printf("%4d", i * j);
            }
            System.out.println();
        }
        
        // รูปสามเหลี่ยม
        System.out.println("\n=== รูปสามเหลี่ยม ===");
        int rows = 5;
        
        // รูปที่ 1: ขวาบนลงล่าง
        for (int i = 1; i <= rows; i++) {
            for (int j = 1; j <= i; j++) {
                System.out.print("* ");
            }
            System.out.println();
        }
        
        System.out.println();
        
        // รูปที่ 2: กลับหัว
        for (int i = rows; i >= 1; i--) {
            for (int j = 1; j <= i; j++) {
                System.out.print("* ");
            }
            System.out.println();
        }
        
        System.out.println();
        
        // รูปที่ 3: ปิรามิด
        for (int i = 1; i <= rows; i++) {
            // spaces
            for (int j = i; j < rows; j++) {
                System.out.print("  ");
            }
            // stars
            for (int j = 1; j <= (2*i - 1); j++) {
                System.out.print("* ");
            }
            System.out.println();
        }
        
        System.out.println();
        
        // Pattern กับตัวเลข
        System.out.println("=== Number Pattern ===");
        for (int i = 1; i <= 5; i++) {
            for (int j = 1; j <= i; j++) {
                System.out.print(j + " ");
            }
            System.out.println();
        }
    }
}
```

---

## 5.4 while Loop

```java
// Syntax
while (condition) {
    // code ที่ทำซ้ำ
}
```

```java
public class WhileLoop {
    public static void main(String[] args) {
        
        // Basic while
        System.out.println("--- Basic while ---");
        int i = 1;
        while (i <= 5) {
            System.out.println("i = " + i);
            i++;
        }
        
        // หา sum ของ digits
        System.out.println("\n--- Sum of digits ---");
        int number = 12345;
        int original = number;
        int sumDigits = 0;
        
        while (number > 0) {
            sumDigits += number % 10;  // เอาหลักสุดท้าย
            number /= 10;              // ตัดหลักสุดท้ายออก
        }
        System.out.println("ผลรวมหลักของ " + original + " = " + sumDigits);
        
        // แปลงเลขฐาน 10 เป็น binary
        System.out.println("\n--- Decimal to Binary ---");
        int decimal = 42;
        StringBuilder binary = new StringBuilder();
        int temp = decimal;
        
        while (temp > 0) {
            binary.insert(0, temp % 2);
            temp /= 2;
        }
        System.out.println(decimal + " = " + binary + " (binary)");
        
        // Collatz sequence
        System.out.println("\n--- Collatz sequence from 27 ---");
        int n = 27;
        int steps = 0;
        System.out.print(n);
        
        while (n != 1) {
            if (n % 2 == 0) {
                n /= 2;
            } else {
                n = 3 * n + 1;
            }
            System.out.print(" → " + n);
            steps++;
        }
        System.out.println("\nใช้ " + steps + " steps");
    }
}
```

---

## 5.5 do-while Loop

```java
// Syntax
do {
    // ทำงานอย่างน้อย 1 ครั้ง
} while (condition);
```

```java
public class DoWhileLoop {
    public static void main(String[] args) {
        
        // Basic do-while
        System.out.println("--- Basic do-while ---");
        int i = 1;
        do {
            System.out.println("i = " + i);
            i++;
        } while (i <= 5);
        
        // ทำงานแม้ condition เป็น false ตั้งแต่แรก
        System.out.println("\n--- do-while vs while ---");
        int x = 10;
        
        System.out.print("while (x < 5): ");
        while (x < 5) {
            System.out.print(x + " ");
            x++;
        }
        System.out.println("(ไม่ทำงานเลย)");
        
        x = 10;
        System.out.print("do-while (x < 5): ");
        do {
            System.out.print(x + " ");
            x++;
        } while (x < 5);
        System.out.println("(ทำงาน 1 ครั้ง)");
        
        // Menu loop (ใช้ do-while เหมาะมาก)
        System.out.println("\n--- Menu with do-while ---");
        java.util.Scanner sc = new java.util.Scanner(System.in);
        int choice;
        
        do {
            System.out.println("\n1. ทางเลือก A");
            System.out.println("2. ทางเลือก B");
            System.out.println("0. ออก");
            System.out.print("เลือก: ");
            choice = sc.nextInt();
            
            switch (choice) {
                case 1: System.out.println("คุณเลือก A"); break;
                case 2: System.out.println("คุณเลือก B"); break;
                case 0: System.out.println("ออกจากโปรแกรม"); break;
                default: System.out.println("ไม่มีตัวเลือกนี้!");
            }
        } while (choice != 0);
        
        sc.close();
    }
}
```

---

## 5.6 break และ continue

```java
public class BreakContinue {
    public static void main(String[] args) {
        
        // break - หยุด loop ทันที
        System.out.println("--- break ---");
        for (int i = 1; i <= 10; i++) {
            if (i == 6) break;  // หยุดเมื่อ i = 6
            System.out.print(i + " ");
        }
        System.out.println("(stopped at 6)");
        
        // continue - ข้ามรอบนี้แล้วไป loop ต่อ
        System.out.println("\n--- continue ---");
        for (int i = 1; i <= 10; i++) {
            if (i % 2 == 0) continue;  // ข้ามเลขคู่
            System.out.print(i + " ");
        }
        System.out.println("(only odd)");
        
        // break ใน nested loop (ออกแค่ loop ในสุด)
        System.out.println("\n--- break in nested loop ---");
        outer:
        for (int i = 1; i <= 3; i++) {
            for (int j = 1; j <= 3; j++) {
                if (j == 2) break;  // ออกแค่ inner loop
                System.out.println("i=" + i + ", j=" + j);
            }
        }
        
        // Labeled break - ออก loop ที่ระบุ
        System.out.println("\n--- labeled break ---");
        outerLoop:
        for (int i = 1; i <= 3; i++) {
            for (int j = 1; j <= 3; j++) {
                if (i == 2 && j == 2) break outerLoop;  // ออก outer loop
                System.out.println("i=" + i + ", j=" + j);
            }
        }
        System.out.println("After labeled break");
        
        // Labeled continue
        System.out.println("\n--- labeled continue ---");
        outer2:
        for (int i = 1; i <= 3; i++) {
            for (int j = 1; j <= 3; j++) {
                if (j == 2) continue outer2;  // ข้ามไป outer loop
                System.out.println("i=" + i + ", j=" + j);
            }
        }
        
        // หาตัวเลข prime ด้วย break
        System.out.println("\n--- Prime numbers 1-50 ---");
        for (int num = 2; num <= 50; num++) {
            boolean isPrime = true;
            for (int divisor = 2; divisor <= Math.sqrt(num); divisor++) {
                if (num % divisor == 0) {
                    isPrime = false;
                    break;  // ไม่ต้องตรวจต่อแล้ว
                }
            }
            if (isPrime) System.out.print(num + " ");
        }
        System.out.println();
    }
}
```

---

## 5.7 Enhanced for Loop (for-each)

```java
import java.util.List;
import java.util.ArrayList;

public class EnhancedForLoop {
    public static void main(String[] args) {
        
        // for-each กับ array
        int[] numbers = {1, 2, 3, 4, 5};
        System.out.println("--- Array ---");
        for (int num : numbers) {
            System.out.print(num + " ");
        }
        System.out.println();
        
        // for-each กับ String array
        String[] fruits = {"Apple", "Banana", "Cherry", "Date"};
        System.out.println("\n--- String Array ---");
        for (String fruit : fruits) {
            System.out.println("🍎 " + fruit);
        }
        
        // for-each กับ List
        List<Integer> primes = List.of(2, 3, 5, 7, 11, 13);
        System.out.println("\n--- List ---");
        for (int prime : primes) {
            System.out.print(prime + " ");
        }
        System.out.println();
        
        // for-each กับ char array ของ String
        String text = "Hello";
        System.out.println("\n--- Chars of String ---");
        for (char c : text.toCharArray()) {
            System.out.print(c + " ");
        }
        System.out.println();
        
        // หาผลรวมด้วย for-each
        double[] prices = {9.99, 24.50, 5.99, 15.00, 49.99};
        double total = 0;
        for (double price : prices) {
            total += price;
        }
        System.out.printf("\nรวม: %.2f บาท%n", total);
    }
}
```

---

## 5.8 โปรแกรมตัวอย่าง: Number Guessing Game

```java
import java.util.Scanner;
import java.util.Random;

public class NumberGuessingGame {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        Random random = new Random();
        
        int secretNumber = random.nextInt(100) + 1;  // 1-100
        int attempts = 0;
        int maxAttempts = 7;
        boolean won = false;
        
        System.out.println("╔══════════════════════════╗");
        System.out.println("║   ทายตัวเลข 1-100        ║");
        System.out.println("║   มี " + maxAttempts + " ครั้ง!                ║");
        System.out.println("╚══════════════════════════╝");
        
        while (attempts < maxAttempts && !won) {
            attempts++;
            System.out.printf("\nครั้งที่ %d/%d: ", attempts, maxAttempts);
            int guess = sc.nextInt();
            
            if (guess < 1 || guess > 100) {
                System.out.println("ใส่เลข 1-100 เท่านั้น!");
                attempts--;  // ไม่นับครั้งนี้
                continue;
            }
            
            if (guess < secretNumber) {
                System.out.println("น้อยเกินไป! ▲");
            } else if (guess > secretNumber) {
                System.out.println("มากเกินไป! ▼");
            } else {
                won = true;
                System.out.println("ถูกต้อง! 🎉");
            }
            
            // Hint เมื่อเหลือ 2 ครั้ง
            if (!won && attempts == maxAttempts - 2) {
                int range = 10;
                System.out.printf("Hint: เลขอยู่ระหว่าง %d-%d%n",
                    Math.max(1, secretNumber - range),
                    Math.min(100, secretNumber + range));
            }
        }
        
        if (won) {
            System.out.printf("\nยินดีด้วย! ทายถูกใน %d ครั้ง!%n", attempts);
            // Rating
            if (attempts <= 3) System.out.println("ระดับ: ⭐⭐⭐⭐⭐ Master!");
            else if (attempts <= 5) System.out.println("ระดับ: ⭐⭐⭐⭐ Expert!");
            else System.out.println("ระดับ: ⭐⭐⭐ Good!");
        } else {
            System.out.println("\nหมดครั้งแล้ว! เลขที่ถูกคือ " + secretNumber);
            System.out.println("เล่นอีกครั้งนะ!");
        }
        
        sc.close();
    }
}
```

---

## 5.9 โปรแกรมตัวอย่าง: Fibonacci Sequence

```java
public class FibonacciSequence {
    public static void main(String[] args) {
        int n = 20;  // แสดง 20 ตัวแรก
        
        System.out.println("=== Fibonacci Sequence (20 terms) ===");
        
        // วิธีที่ 1: Basic loop
        long a = 0, b = 1;
        System.out.printf("F(%2d) = %d%n", 0, a);
        System.out.printf("F(%2d) = %d%n", 1, b);
        
        for (int i = 2; i < n; i++) {
            long c = a + b;
            System.out.printf("F(%2d) = %d%n", i, c);
            a = b;
            b = c;
        }
        
        // วิธีที่ 2: Array
        System.out.println("\n=== Fibonacci Array ===");
        long[] fib = new long[20];
        fib[0] = 0;
        fib[1] = 1;
        
        for (int i = 2; i < fib.length; i++) {
            fib[i] = fib[i-1] + fib[i-2];
        }
        
        for (int i = 0; i < fib.length; i++) {
            System.out.printf("F[%2d] = %d%n", i, fib[i]);
        }
        
        // ตรวจสอบ: เลขใดอยู่ใน Fibonacci
        System.out.println("\n=== Is Fibonacci? ===");
        int[] testNums = {0, 1, 4, 8, 13, 20, 21};
        
        for (int num : testNums) {
            boolean found = false;
            for (long f : fib) {
                if (f == num) {
                    found = true;
                    break;
                }
            }
            System.out.println(num + ": " + (found ? "Yes ✓" : "No ✗"));
        }
    }
}
```

---

## 5.10 โปรแกรมตัวอย่าง: Pascal's Triangle

```java
public class PascalsTriangle {
    public static void main(String[] args) {
        int rows = 8;
        int[][] triangle = new int[rows][];
        
        // สร้าง Pascal's Triangle
        for (int i = 0; i < rows; i++) {
            triangle[i] = new int[i + 1];
            triangle[i][0] = 1;          // ตัวแรกเสมอ 1
            triangle[i][i] = 1;          // ตัวสุดท้ายเสมอ 1
            
            for (int j = 1; j < i; j++) {
                triangle[i][j] = triangle[i-1][j-1] + triangle[i-1][j];
            }
        }
        
        // แสดงผล
        System.out.println("=== Pascal's Triangle ===");
        for (int i = 0; i < rows; i++) {
            // indent
            for (int space = 0; space < rows - i; space++) {
                System.out.print("   ");
            }
            // numbers
            for (int j = 0; j <= i; j++) {
                System.out.printf("%6d", triangle[i][j]);
            }
            System.out.println();
        }
        
        // Binomial coefficients
        System.out.println("\n=== C(n,k) values ---");
        int n = 5;
        for (int k = 0; k <= n; k++) {
            System.out.printf("C(%d,%d) = %d%n", n, k, triangle[n][k]);
        }
    }
}
```

---

## 5.11 โปรแกรมตัวอย่าง: Pattern Printer

```java
import java.util.Scanner;

public class PatternPrinter {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        
        System.out.print("ขนาด (rows): ");
        int n = sc.nextInt();
        
        System.out.println("\n=== Pattern 1: Square ===");
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                System.out.print("* ");
            }
            System.out.println();
        }
        
        System.out.println("\n=== Pattern 2: Hollow Square ===");
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                if (i == 0 || i == n-1 || j == 0 || j == n-1) {
                    System.out.print("* ");
                } else {
                    System.out.print("  ");
                }
            }
            System.out.println();
        }
        
        System.out.println("\n=== Pattern 3: Diamond ===");
        // Upper half
        for (int i = 1; i <= n; i++) {
            for (int j = n - i; j > 0; j--) System.out.print(" ");
            for (int j = 0; j < 2*i - 1; j++) System.out.print("*");
            System.out.println();
        }
        // Lower half
        for (int i = n-1; i >= 1; i--) {
            for (int j = n - i; j > 0; j--) System.out.print(" ");
            for (int j = 0; j < 2*i - 1; j++) System.out.print("*");
            System.out.println();
        }
        
        System.out.println("\n=== Pattern 4: Checkerboard ===");
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                if ((i + j) % 2 == 0) {
                    System.out.print("# ");
                } else {
                    System.out.print(". ");
                }
            }
            System.out.println();
        }
        
        sc.close();
    }
}
```

---

## 5.12 แบบฝึกหัด Part 05

### แบบฝึกหัดที่ 1: Sum and Statistics

```java
import java.util.Scanner;

public class SumAndStats {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        
        System.out.print("จำนวนตัวเลขที่จะใส่: ");
        int count = sc.nextInt();
        
        double sum = 0, min = Double.MAX_VALUE, max = Double.MIN_VALUE;
        
        for (int i = 1; i <= count; i++) {
            System.out.print("ตัวเลขที่ " + i + ": ");
            double num = sc.nextDouble();
            sum += num;
            if (num < min) min = num;
            if (num > max) max = num;
        }
        
        System.out.println("\n=== สถิติ ===");
        System.out.printf("ผลรวม   : %.2f%n", sum);
        System.out.printf("ค่าเฉลี่ย: %.2f%n", sum / count);
        System.out.printf("น้อยสุด : %.2f%n", min);
        System.out.printf("มากสุด  : %.2f%n", max);
        
        sc.close();
    }
}
```

### แบบฝึกหัดที่ 2: Prime Checker

```java
import java.util.Scanner;

public class PrimeChecker {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        
        System.out.print("ใส่ตัวเลข: ");
        int n = sc.nextInt();
        
        boolean isPrime = n > 1;
        
        if (n > 1) {
            for (int i = 2; i <= Math.sqrt(n); i++) {
                if (n % i == 0) {
                    isPrime = false;
                    break;
                }
            }
        }
        
        System.out.println(n + (isPrime ? " เป็นจำนวนเฉพาะ" : " ไม่ใช่จำนวนเฉพาะ"));
        
        // แสดง prime factors ถ้าไม่ใช่ prime
        if (!isPrime && n > 1) {
            System.out.print("ตัวประกอบ: ");
            int temp = n;
            for (int i = 2; i <= temp; i++) {
                while (temp % i == 0) {
                    System.out.print(i + " ");
                    temp /= i;
                }
            }
            System.out.println();
        }
        
        sc.close();
    }
}
```

### แบบฝึกหัดที่ 3: ร้านขายสินค้า

```java
import java.util.Scanner;

public class ShoppingCart {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        
        double total = 0;
        int itemCount = 0;
        String[] items = new String[50];
        double[] prices = new double[50];
        
        System.out.println("=== ตะกร้าสินค้า ===");
        System.out.println("(พิมพ์ 'done' เพื่อจบ)");
        
        while (true) {
            System.out.print("\nชื่อสินค้า: ");
            String name = sc.next();
            
            if (name.equalsIgnoreCase("done")) break;
            
            System.out.print("ราคา: ");
            double price = sc.nextDouble();
            
            if (price < 0) {
                System.out.println("ราคาต้องไม่ติดลบ!");
                continue;
            }
            
            items[itemCount] = name;
            prices[itemCount] = price;
            total += price;
            itemCount++;
            
            System.out.printf("เพิ่ม %s (%.2f บาท) แล้ว%n", name, price);
        }
        
        // แสดงใบเสร็จ
        System.out.println("\n" + "=".repeat(30));
        System.out.println("        ใบเสร็จ");
        System.out.println("=".repeat(30));
        
        for (int i = 0; i < itemCount; i++) {
            System.out.printf("%-15s %8.2f บาท%n", items[i], prices[i]);
        }
        
        System.out.println("-".repeat(30));
        System.out.printf("รวม (%d รายการ)  %8.2f บาท%n", itemCount, total);
        
        double discount = 0;
        if (total >= 1000) {
            discount = total * 0.10;
            System.out.printf("ส่วนลด 10%%        %8.2f บาท%n", discount);
        }
        
        System.out.printf("สุทธิ             %8.2f บาท%n", total - discount);
        System.out.println("=".repeat(30));
        
        sc.close();
    }
}
```

---

## 5.13 สรุป Part 05

ในบทนี้คุณได้เรียนรู้:

✅ for loop และ variations  
✅ Nested loops  
✅ while loop  
✅ do-while loop  
✅ break และ continue  
✅ Labeled break/continue  
✅ Enhanced for loop (for-each)  
✅ การประยุกต์ใช้: patterns, เกม, คณิตศาสตร์  

---

*[← Part 04: Control Flow](./part-04-control-flow.md) | [Part 06: Arrays →](./part-06-arrays.md)*
