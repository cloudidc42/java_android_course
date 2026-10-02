# Part 08: Methods & Functions
## หลักสูตร Java & Android Development - ระดับเริ่มต้น

---

## 8.1 Method คืออะไร?

Method คือกลุ่มของโค้ดที่ทำงานเฉพาะอย่าง สามารถเรียกใช้ซ้ำได้ ช่วยจัดระเบียบโค้ดและลดการเขียนซ้ำ

```java
// Syntax
returnType methodName(parameter1Type param1, ...) {
    // method body
    return value; // ถ้า returnType ไม่ใช่ void
}
```

---

## 8.2 Method พื้นฐาน

```java
public class MethodBasics {
    
    // Method ที่ไม่รับ parameter และไม่ return ค่า
    static void greet() {
        System.out.println("สวัสดี!");
    }
    
    // Method ที่รับ parameter
    static void greetUser(String name) {
        System.out.println("สวัสดี, " + name + "!");
    }
    
    // Method ที่ return ค่า
    static int add(int a, int b) {
        return a + b;
    }
    
    // Method ที่รับหลาย parameter
    static double calculateBMI(double weight, double heightCm) {
        double heightM = heightCm / 100;
        return weight / (heightM * heightM);
    }
    
    // Method ที่ return String
    static String getGrade(double score) {
        if (score >= 80) return "A";
        if (score >= 70) return "B";
        if (score >= 60) return "C";
        if (score >= 50) return "D";
        return "F";
    }
    
    public static void main(String[] args) {
        greet();
        greetUser("Alice");
        greetUser("Bob");
        
        int result = add(5, 3);
        System.out.println("5 + 3 = " + result);
        
        double bmi = calculateBMI(70, 175);
        System.out.printf("BMI: %.2f%n", bmi);
        
        System.out.println("Grade 85: " + getGrade(85));
        System.out.println("Grade 55: " + getGrade(55));
    }
}
```

---

## 8.3 Method Overloading

```java
public class MethodOverloading {
    
    // ชื่อเหมือนกัน parameter ต่างกัน
    static int add(int a, int b) {
        System.out.println("add(int, int)");
        return a + b;
    }
    
    static double add(double a, double b) {
        System.out.println("add(double, double)");
        return a + b;
    }
    
    static int add(int a, int b, int c) {
        System.out.println("add(int, int, int)");
        return a + b + c;
    }
    
    static String add(String a, String b) {
        System.out.println("add(String, String)");
        return a + b;
    }
    
    // print overloaded
    static void print(int value) {
        System.out.println("int: " + value);
    }
    
    static void print(double value) {
        System.out.println("double: " + value);
    }
    
    static void print(String value) {
        System.out.println("String: " + value);
    }
    
    static void print(int[] arr) {
        System.out.print("int[]: ");
        for (int n : arr) System.out.print(n + " ");
        System.out.println();
    }
    
    public static void main(String[] args) {
        System.out.println(add(1, 2));         // add(int, int)
        System.out.println(add(1.5, 2.5));     // add(double, double)
        System.out.println(add(1, 2, 3));      // add(int, int, int)
        System.out.println(add("Hello", " World")); // add(String, String)
        
        System.out.println();
        print(42);
        print(3.14);
        print("Hello");
        print(new int[]{1, 2, 3});
    }
}
```

---

## 8.4 Varargs (Variable Arguments)

```java
public class Varargs {
    
    // รับ parameter ไม่จำกัดจำนวน
    static int sum(int... numbers) {
        int total = 0;
        for (int n : numbers) total += n;
        return total;
    }
    
    static double average(double... values) {
        if (values.length == 0) return 0;
        double sum = 0;
        for (double v : values) sum += v;
        return sum / values.length;
    }
    
    static String concat(String separator, String... parts) {
        return String.join(separator, parts);
    }
    
    static void log(String level, String format, Object... args) {
        String message = String.format(format, args);
        System.out.printf("[%s] %s%n", level, message);
    }
    
    public static void main(String[] args) {
        System.out.println("sum(): " + sum());          // 0
        System.out.println("sum(1): " + sum(1));        // 1
        System.out.println("sum(1,2,3): " + sum(1,2,3)); // 6
        System.out.println("sum(1-5): " + sum(1,2,3,4,5)); // 15
        
        // ส่ง array ได้เลย
        int[] arr = {10, 20, 30};
        System.out.println("sum(array): " + sum(arr));  // 60
        
        System.out.printf("avg: %.2f%n", average(10, 20, 30, 40, 50));
        
        System.out.println(concat("-", "A", "B", "C"));   // A-B-C
        System.out.println(concat(", ", "one", "two", "three")); // one, two, three
        
        log("INFO", "User %s logged in at %s", "Alice", "10:30 AM");
        log("ERROR", "Failed to connect: %s (code %d)", "timeout", 503);
    }
}
```

---

## 8.5 Recursive Methods

```java
public class RecursiveMethods {
    
    // Factorial
    static long factorial(int n) {
        if (n <= 1) return 1;  // base case
        return n * factorial(n - 1);  // recursive case
    }
    
    // Fibonacci
    static long fibonacci(int n) {
        if (n <= 1) return n;
        return fibonacci(n-1) + fibonacci(n-2);
    }
    
    // Sum 1 to n
    static int sumTo(int n) {
        if (n <= 0) return 0;
        return n + sumTo(n - 1);
    }
    
    // Power
    static double power(double base, int exp) {
        if (exp == 0) return 1;
        if (exp < 0) return 1 / power(base, -exp);
        return base * power(base, exp - 1);
    }
    
    // Binary Search (recursive)
    static int binarySearch(int[] arr, int target, int left, int right) {
        if (left > right) return -1;
        
        int mid = (left + right) / 2;
        if (arr[mid] == target) return mid;
        if (arr[mid] < target) return binarySearch(arr, target, mid+1, right);
        return binarySearch(arr, target, left, mid-1);
    }
    
    // GCD (Greatest Common Divisor) - Euclidean Algorithm
    static int gcd(int a, int b) {
        if (b == 0) return a;
        return gcd(b, a % b);
    }
    
    // Reverse String
    static String reverse(String s) {
        if (s.length() <= 1) return s;
        return reverse(s.substring(1)) + s.charAt(0);
    }
    
    // Tower of Hanoi
    static void hanoi(int n, char from, char to, char aux) {
        if (n == 1) {
            System.out.println("ย้ายจาก " + from + " ไป " + to);
            return;
        }
        hanoi(n-1, from, aux, to);
        System.out.println("ย้ายจาก " + from + " ไป " + to);
        hanoi(n-1, aux, to, from);
    }
    
    public static void main(String[] args) {
        // Factorial
        for (int i = 0; i <= 10; i++) {
            System.out.printf("%2d! = %d%n", i, factorial(i));
        }
        
        // Fibonacci
        System.out.println("\nFibonacci:");
        for (int i = 0; i <= 10; i++) {
            System.out.printf("fib(%d) = %d%n", i, fibonacci(i));
        }
        
        // Sum
        System.out.println("\nSum 1-10: " + sumTo(10));   // 55
        System.out.println("Sum 1-100: " + sumTo(100));   // 5050
        
        // Power
        System.out.println("\n2^10 = " + (long)power(2, 10));  // 1024
        
        // GCD
        System.out.println("\nGCD(48,18) = " + gcd(48, 18));  // 6
        System.out.println("GCD(100,75) = " + gcd(100, 75));  // 25
        
        // Reverse
        System.out.println("\nreverse: " + reverse("Hello"));
        
        // Hanoi
        System.out.println("\nTower of Hanoi (3 discs):");
        hanoi(3, 'A', 'C', 'B');
    }
}
```

---

## 8.6 Pass by Value vs Pass by Reference

```java
public class PassByValue {
    
    // Java เป็น pass by value เสมอ
    
    static void modifyPrimitive(int x) {
        x = 100;  // แก้ไข copy ไม่กระทบ original
    }
    
    static void modifyArray(int[] arr) {
        arr[0] = 100;  // แก้ไข content ของ array ได้
    }
    
    static void replaceArray(int[] arr) {
        arr = new int[]{1, 2, 3};  // แค่เปลี่ยน reference local copy
    }
    
    static void modifyStringBuilder(StringBuilder sb) {
        sb.append(" World");  // แก้ไข object ได้
    }
    
    static void replaceString(String s) {
        s = "New String";  // String เป็น immutable, แค่เปลี่ยน reference
    }
    
    public static void main(String[] args) {
        // Primitive: ไม่เปลี่ยน
        int num = 5;
        modifyPrimitive(num);
        System.out.println("num: " + num);  // 5 (ไม่เปลี่ยน)
        
        // Array: เปลี่ยน content ได้
        int[] arr = {1, 2, 3};
        modifyArray(arr);
        System.out.println("arr[0]: " + arr[0]);  // 100 (เปลี่ยน!)
        
        // Array: replace ไม่กระทบ original
        replaceArray(arr);
        System.out.println("arr after replace: " + java.util.Arrays.toString(arr));  // [100, 2, 3]
        
        // StringBuilder: เปลี่ยนได้
        StringBuilder sb = new StringBuilder("Hello");
        modifyStringBuilder(sb);
        System.out.println("sb: " + sb);  // Hello World
        
        // String: ไม่เปลี่ยน
        String str = "Original";
        replaceString(str);
        System.out.println("str: " + str);  // Original (ไม่เปลี่ยน)
    }
}
```

---

## 8.7 Method Chaining

```java
public class MethodChaining {
    
    // Builder pattern
    static class Person {
        String name;
        int age;
        String email;
        String phone;
        
        Person setName(String name) {
            this.name = name;
            return this;  // return this เพื่อ chain
        }
        
        Person setAge(int age) {
            this.age = age;
            return this;
        }
        
        Person setEmail(String email) {
            this.email = email;
            return this;
        }
        
        Person setPhone(String phone) {
            this.phone = phone;
            return this;
        }
        
        void print() {
            System.out.println("Name: " + name);
            System.out.println("Age: " + age);
            System.out.println("Email: " + email);
            System.out.println("Phone: " + phone);
        }
    }
    
    public static void main(String[] args) {
        // Method chaining
        new Person()
            .setName("Alice")
            .setAge(25)
            .setEmail("alice@example.com")
            .setPhone("0812345678")
            .print();
        
        // StringBuilder chaining
        String result = new StringBuilder()
            .append("Hello")
            .append(", ")
            .append("World")
            .append("!")
            .reverse()
            .toString();
        System.out.println(result);
        
        // String method chaining
        String text = "  Hello, World!  ";
        String processed = text.trim().toLowerCase().replace(",", "").replace("!", "");
        System.out.println(processed);
    }
}
```

---

## 8.8 Higher-Order Functions (Java 8+)

```java
import java.util.function.*;
import java.util.*;

public class HigherOrderFunctions {
    
    // รับ function เป็น parameter
    static void applyToAll(int[] arr, IntUnaryOperator operation) {
        for (int i = 0; i < arr.length; i++) {
            arr[i] = operation.applyAsInt(arr[i]);
        }
    }
    
    static int[] filter(int[] arr, IntPredicate condition) {
        return java.util.Arrays.stream(arr)
            .filter(condition)
            .toArray();
    }
    
    static int reduce(int[] arr, int identity, IntBinaryOperator accumulator) {
        int result = identity;
        for (int n : arr) {
            result = accumulator.applyAsInt(result, n);
        }
        return result;
    }
    
    public static void main(String[] args) {
        int[] numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
        
        // Apply double to all
        int[] doubled = numbers.clone();
        applyToAll(doubled, n -> n * 2);
        System.out.println("Doubled: " + java.util.Arrays.toString(doubled));
        
        // Filter even
        int[] evens = filter(numbers, n -> n % 2 == 0);
        System.out.println("Evens: " + java.util.Arrays.toString(evens));
        
        // Reduce sum
        int sum = reduce(numbers, 0, (a, b) -> a + b);
        System.out.println("Sum: " + sum);
        
        // Product
        int product = reduce(new int[]{1,2,3,4,5}, 1, (a, b) -> a * b);
        System.out.println("Product: " + product);
        
        // Functional interfaces
        Function<String, Integer> strLen = String::length;
        Function<Integer, Boolean> isEven = n -> n % 2 == 0;
        Function<String, Boolean> strLenIsEven = strLen.andThen(isEven);
        
        System.out.println("'Hello' len is even: " + strLenIsEven.apply("Hello"));  // false
        System.out.println("'Java' len is even: " + strLenIsEven.apply("Java"));    // true
        
        // Predicate
        Predicate<String> notEmpty = s -> !s.isEmpty();
        Predicate<String> longEnough = s -> s.length() >= 3;
        Predicate<String> valid = notEmpty.and(longEnough);
        
        List<String> names = Arrays.asList("", "Al", "Alice", "Bob", "Charlie");
        names.stream()
            .filter(valid)
            .forEach(System.out::println);
    }
}
```

---

## 8.9 โปรแกรมตัวอย่าง: Math Library

```java
public class MathLibrary {
    
    static final double PI = Math.PI;
    
    // Circle
    static double circleArea(double r) { return PI * r * r; }
    static double circlePerimeter(double r) { return 2 * PI * r; }
    
    // Rectangle
    static double rectArea(double w, double h) { return w * h; }
    static double rectPerimeter(double w, double h) { return 2 * (w + h); }
    
    // Triangle
    static double triArea(double base, double height) { return 0.5 * base * height; }
    static double triHeron(double a, double b, double c) {
        double s = (a + b + c) / 2;
        return Math.sqrt(s * (s-a) * (s-b) * (s-c));
    }
    
    // Number theory
    static boolean isPrime(int n) {
        if (n < 2) return false;
        for (int i = 2; i <= Math.sqrt(n); i++)
            if (n % i == 0) return false;
        return true;
    }
    
    static int gcd(int a, int b) { return b == 0 ? a : gcd(b, a % b); }
    static int lcm(int a, int b) { return a / gcd(a, b) * b; }
    
    static long factorial(int n) {
        if (n <= 1) return 1;
        return n * factorial(n - 1);
    }
    
    static long fibonacci(int n) {
        if (n <= 1) return n;
        long a = 0, b = 1;
        for (int i = 2; i <= n; i++) {
            long c = a + b;
            a = b;
            b = c;
        }
        return b;
    }
    
    // Statistics
    static double mean(double[] arr) {
        double sum = 0;
        for (double x : arr) sum += x;
        return sum / arr.length;
    }
    
    static double standardDeviation(double[] arr) {
        double avg = mean(arr);
        double sumSqDiff = 0;
        for (double x : arr) sumSqDiff += (x - avg) * (x - avg);
        return Math.sqrt(sumSqDiff / arr.length);
    }
    
    public static void main(String[] args) {
        System.out.println("=== Geometry ===");
        System.out.printf("Circle r=5: Area=%.2f, Perimeter=%.2f%n",
            circleArea(5), circlePerimeter(5));
        System.out.printf("Rect 4x6: Area=%.2f, Perimeter=%.2f%n",
            rectArea(4, 6), rectPerimeter(4, 6));
        System.out.printf("Triangle 3-4-5: Area=%.2f%n", triHeron(3, 4, 5));
        
        System.out.println("\n=== Number Theory ===");
        System.out.print("Primes up to 30: ");
        for (int i = 2; i <= 30; i++) if (isPrime(i)) System.out.print(i + " ");
        System.out.println();
        
        System.out.println("GCD(48,18)=" + gcd(48,18) + ", LCM(4,6)=" + lcm(4,6));
        System.out.println("10! = " + factorial(10));
        System.out.println("fib(10) = " + fibonacci(10));
        
        System.out.println("\n=== Statistics ===");
        double[] data = {2, 4, 4, 4, 5, 5, 7, 9};
        System.out.printf("Mean: %.2f, StdDev: %.2f%n", mean(data), standardDeviation(data));
    }
}
```

---

## 8.10 แบบฝึกหัด Part 08

### แบบฝึกหัดที่ 1: String Utilities

```java
public class StringUtils {
    
    static String reverse(String s) {
        return new StringBuilder(s).reverse().toString();
    }
    
    static boolean isPalindrome(String s) {
        String clean = s.toLowerCase().replaceAll("[^a-z0-9]", "");
        return clean.equals(reverse(clean));
    }
    
    static int countVowels(String s) {
        return (int) s.toLowerCase().chars().filter("aeiou"::indexOf).count();
    }
    
    static String capitalize(String s) {
        if (s == null || s.isEmpty()) return s;
        return Character.toUpperCase(s.charAt(0)) + s.substring(1).toLowerCase();
    }
    
    static String camelToSnake(String camel) {
        return camel.replaceAll("([A-Z])", "_$1").toLowerCase().replaceFirst("^_", "");
    }
    
    public static void main(String[] args) {
        String[] tests = {"Hello", "racecar", "A man a plan a canal Panama", "camelCaseName"};
        
        for (String t : tests) {
            System.out.println("Input: " + t);
            System.out.println("  Reverse: " + reverse(t));
            System.out.println("  Palindrome: " + isPalindrome(t));
            System.out.println("  Vowels: " + countVowels(t));
            System.out.println("  Capitalize: " + capitalize(t));
        }
        
        System.out.println("\ncamelToSnake:");
        System.out.println(camelToSnake("myVariableName"));
        System.out.println(camelToSnake("getUserById"));
    }
}
```

### แบบฝึกหัดที่ 2: Recursive Utilities

```java
public class RecursiveUtils {
    
    // Count digits
    static int countDigits(int n) {
        n = Math.abs(n);
        if (n < 10) return 1;
        return 1 + countDigits(n / 10);
    }
    
    // Sum of digits
    static int sumDigits(int n) {
        n = Math.abs(n);
        if (n < 10) return n;
        return n % 10 + sumDigits(n / 10);
    }
    
    // Is Armstrong number (e.g., 153 = 1^3 + 5^3 + 3^3)
    static boolean isArmstrong(int n) {
        int digits = countDigits(n);
        int sum = 0, temp = n;
        while (temp > 0) {
            sum += (int) Math.pow(temp % 10, digits);
            temp /= 10;
        }
        return sum == n;
    }
    
    public static void main(String[] args) {
        int[] numbers = {0, 5, 123, 1000};
        for (int n : numbers) {
            System.out.printf("%5d: digits=%d, sumDigits=%d%n",
                n, countDigits(n), sumDigits(n));
        }
        
        System.out.println("\nArmstrong numbers 1-999:");
        for (int i = 1; i <= 999; i++) {
            if (isArmstrong(i)) System.out.print(i + " ");
        }
        System.out.println();
    }
}
```

---

## 8.11 สรุป Part 08

ในบทนี้คุณได้เรียนรู้:

✅ Method declaration และ return types  
✅ Method overloading  
✅ Varargs  
✅ Recursive methods  
✅ Pass by value  
✅ Method chaining  
✅ Higher-order functions  

---

*[← Part 07: Strings](./part-07-strings.md) | [Part 09: OOP Basics →](./part-09-oop-basics.md)*
