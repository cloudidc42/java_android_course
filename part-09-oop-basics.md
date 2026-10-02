# Part 09: OOP - Classes & Objects
## หลักสูตร Java & Android Development - ระดับ Intermediate

---

## 9.1 Object-Oriented Programming คืออะไร?

OOP (Object-Oriented Programming) เป็นแนวคิดการเขียนโปรแกรมที่จำลองสิ่งของจริงในโลกให้เป็น object ใน code

### หลักการ 4 ข้อของ OOP:
1. **Encapsulation** - ห่อหุ้มข้อมูลและฟังก์ชัน
2. **Inheritance** - สืบทอดคุณสมบัติ
3. **Polymorphism** - มีรูปร่างหลายแบบ
4. **Abstraction** - ซ่อนรายละเอียด

### ทำไมต้องใช้ OOP?

| Procedural (เดิม) | OOP |
|-------------------|-----|
| โค้ดยุ่งเหยิงเมื่อ project ใหญ่ | จัดระเบียบดี |
| Hard to maintain | ง่ายต่อการดูแลรักษา |
| ยากต่อการ reuse | Reusable |
| ยากต่อการ test | Testable |

---

## 9.2 Class และ Object

```java
// Class = blueprint (แบบแปลน)
// Object = instance (สิ่งของที่สร้างจาก blueprint)

public class Car {
    // Fields (คุณสมบัติ)
    String brand;
    String model;
    int year;
    String color;
    double price;
    boolean isRunning;
    
    // Constructor (ตัวสร้าง object)
    Car(String brand, String model, int year) {
        this.brand = brand;
        this.model = model;
        this.year = year;
        this.isRunning = false;
    }
    
    // Methods (พฤติกรรม)
    void start() {
        if (!isRunning) {
            isRunning = true;
            System.out.println(brand + " " + model + " สตาร์ทแล้ว!");
        } else {
            System.out.println("รถกำลังทำงานอยู่แล้ว");
        }
    }
    
    void stop() {
        if (isRunning) {
            isRunning = false;
            System.out.println(brand + " " + model + " ดับเครื่องแล้ว");
        }
    }
    
    void accelerate(int speed) {
        if (isRunning) {
            System.out.println("เร่งความเร็วเป็น " + speed + " km/h");
        } else {
            System.out.println("สตาร์ทรถก่อน!");
        }
    }
    
    String getInfo() {
        return String.format("%d %s %s (%.0f บาท)", year, brand, model, price);
    }
    
    public static void main(String[] args) {
        // สร้าง object จาก class
        Car myCar = new Car("Toyota", "Camry", 2023);
        myCar.color = "Silver";
        myCar.price = 1200000;
        
        Car friendCar = new Car("Honda", "Civic", 2022);
        friendCar.color = "Red";
        friendCar.price = 950000;
        
        // ใช้งาน object
        System.out.println(myCar.getInfo());
        myCar.start();
        myCar.accelerate(100);
        myCar.stop();
        
        System.out.println();
        System.out.println(friendCar.getInfo());
        friendCar.start();
        friendCar.accelerate(80);
    }
}
```

---

## 9.3 Constructors

```java
public class Person {
    String name;
    int age;
    String email;
    
    // Default constructor (no args)
    Person() {
        this.name = "Unknown";
        this.age = 0;
        this.email = "";
    }
    
    // Constructor with params
    Person(String name, int age) {
        this.name = name;
        this.age = age;
        this.email = "";
    }
    
    // Constructor with all params
    Person(String name, int age, String email) {
        this(name, age);  // เรียก constructor อื่น
        this.email = email;
    }
    
    // Copy constructor
    Person(Person other) {
        this.name = other.name;
        this.age = other.age;
        this.email = other.email;
    }
    
    @Override
    public String toString() {
        return String.format("Person{name='%s', age=%d, email='%s'}", name, age, email);
    }
    
    public static void main(String[] args) {
        Person p1 = new Person();                          // default
        Person p2 = new Person("Alice", 25);               // 2 params
        Person p3 = new Person("Bob", 30, "bob@test.com"); // 3 params
        Person p4 = new Person(p3);                        // copy
        
        System.out.println(p1);
        System.out.println(p2);
        System.out.println(p3);
        System.out.println(p4);
        
        // p4 เป็น copy ของ p3 (different object, same data)
        System.out.println("p3 == p4: " + (p3 == p4));            // false
        System.out.println("Same name: " + p3.name.equals(p4.name)); // true
    }
}
```

---

## 9.4 this Keyword

```java
public class Counter {
    int count;
    int step;
    String name;
    
    Counter(String name, int step) {
        this.name = name;  // this.field = parameter
        this.step = step;
        this.count = 0;
    }
    
    void increment() {
        this.count += this.step;
    }
    
    void reset() {
        this.count = 0;
    }
    
    Counter add(Counter other) {
        Counter result = new Counter(this.name + "+" + other.name, 1);
        result.count = this.count + other.count;
        return result;
    }
    
    // this() เรียก constructor อื่น
    Counter() {
        this("Default", 1);  // เรียก Counter(String, int)
    }
    
    void printInfo() {
        // this ใน method หมายถึง current object
        System.out.println(this.name + ": " + this.count);
    }
    
    // Return this สำหรับ method chaining
    Counter incrementBy(int n) {
        this.count += n;
        return this;  // ส่ง object ตัวเอง
    }
    
    public static void main(String[] args) {
        Counter c1 = new Counter("c1", 5);
        Counter c2 = new Counter("c2", 10);
        Counter c3 = new Counter();
        
        c1.increment(); c1.increment(); c1.increment();
        c2.increment(); c2.increment();
        
        c1.printInfo();  // c1: 15
        c2.printInfo();  // c2: 20
        
        Counter sum = c1.add(c2);
        sum.printInfo();  // c1+c2: 35
        
        // Method chaining
        c3.incrementBy(10).incrementBy(20).incrementBy(30);
        c3.printInfo();  // Default: 60
    }
}
```

---

## 9.5 static Members

```java
public class BankAccount {
    // Instance variables (แต่ละ object มีของตัวเอง)
    private String accountNo;
    private String owner;
    private double balance;
    
    // Static variables (ใช้ร่วมกันทุก object)
    private static int totalAccounts = 0;
    private static double totalDeposits = 0;
    static final double MIN_BALANCE = 100.0;
    static final double INTEREST_RATE = 0.025;
    
    BankAccount(String owner, double initialDeposit) {
        totalAccounts++;
        this.accountNo = String.format("ACC%05d", totalAccounts);
        this.owner = owner;
        this.balance = initialDeposit;
        totalDeposits += initialDeposit;
    }
    
    void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
            totalDeposits += amount;
            System.out.printf("ฝาก %.2f - ยอด: %.2f%n", amount, balance);
        }
    }
    
    boolean withdraw(double amount) {
        if (amount <= 0 || balance - amount < MIN_BALANCE) {
            System.out.println("ถอนไม่ได้! ยอดไม่พอ");
            return false;
        }
        balance -= amount;
        System.out.printf("ถอน %.2f - ยอด: %.2f%n", amount, balance);
        return true;
    }
    
    void applyInterest() {
        double interest = balance * INTEREST_RATE;
        balance += interest;
        System.out.printf("ดอกเบี้ย %.2f - ยอดใหม่: %.2f%n", interest, balance);
    }
    
    // Static methods
    static int getTotalAccounts() { return totalAccounts; }
    static double getTotalDeposits() { return totalDeposits; }
    
    // Static factory method
    static BankAccount createWithZero(String owner) {
        return new BankAccount(owner, MIN_BALANCE);
    }
    
    @Override
    public String toString() {
        return String.format("BankAccount{%s, owner=%s, balance=%.2f}",
            accountNo, owner, balance);
    }
    
    public static void main(String[] args) {
        BankAccount acc1 = new BankAccount("Alice", 5000);
        BankAccount acc2 = new BankAccount("Bob", 3000);
        BankAccount acc3 = BankAccount.createWithZero("Charlie");
        
        System.out.println(acc1);
        System.out.println(acc2);
        System.out.println(acc3);
        
        acc1.deposit(1000);
        acc1.withdraw(500);
        acc1.applyInterest();
        
        System.out.println("\n=== สถิติรวม ===");
        System.out.println("บัญชีทั้งหมด: " + BankAccount.getTotalAccounts());
        System.out.printf("ยอดฝากรวม: %.2f%n", BankAccount.getTotalDeposits());
    }
}
```

---

## 9.6 Inner Classes

```java
public class OuterClass {
    private int x = 10;
    
    // Inner class (non-static)
    class Inner {
        int y = 20;
        void display() {
            System.out.println("x=" + x + ", y=" + y);  // เข้าถึง outer x ได้
        }
    }
    
    // Static nested class
    static class StaticNested {
        int z = 30;
        void display() {
            // ไม่สามารถเข้าถึง x โดยตรง (ต้องสร้าง object ก่อน)
            System.out.println("z=" + z);
        }
    }
    
    void method() {
        // Local class
        class Local {
            void hello() { System.out.println("Local class!"); }
        }
        new Local().hello();
        
        // Anonymous class
        Runnable r = new Runnable() {
            @Override
            public void run() {
                System.out.println("Anonymous class running!");
            }
        };
        r.run();
    }
    
    public static void main(String[] args) {
        OuterClass outer = new OuterClass();
        
        // สร้าง inner class object
        OuterClass.Inner inner = outer.new Inner();
        inner.display();
        
        // สร้าง static nested class
        OuterClass.StaticNested nested = new OuterClass.StaticNested();
        nested.display();
        
        outer.method();
    }
}
```

---

## 9.7 โปรแกรมตัวอย่าง: Library System

```java
import java.util.*;

public class LibrarySystem {
    
    // Book class
    static class Book {
        private static int nextId = 1;
        
        int id;
        String title;
        String author;
        String isbn;
        int year;
        boolean available;
        
        Book(String title, String author, String isbn, int year) {
            this.id = nextId++;
            this.title = title;
            this.author = author;
            this.isbn = isbn;
            this.year = year;
            this.available = true;
        }
        
        @Override
        public String toString() {
            return String.format("[%d] %-30s | %-20s | %d | %s",
                id, title, author, year, available ? "✓ Available" : "✗ Borrowed");
        }
    }
    
    // Member class
    static class Member {
        private static int nextId = 1001;
        
        int id;
        String name;
        String email;
        List<Book> borrowedBooks;
        
        Member(String name, String email) {
            this.id = nextId++;
            this.name = name;
            this.email = email;
            this.borrowedBooks = new ArrayList<>();
        }
        
        boolean borrow(Book book) {
            if (!book.available) {
                System.out.println("หนังสือ '" + book.title + "' ถูกยืมไปแล้ว!");
                return false;
            }
            if (borrowedBooks.size() >= 3) {
                System.out.println("ยืมได้ไม่เกิน 3 เล่ม!");
                return false;
            }
            book.available = false;
            borrowedBooks.add(book);
            System.out.println(name + " ยืม '" + book.title + "' สำเร็จ");
            return true;
        }
        
        boolean returnBook(Book book) {
            if (borrowedBooks.remove(book)) {
                book.available = true;
                System.out.println(name + " คืน '" + book.title + "' สำเร็จ");
                return true;
            }
            System.out.println("ไม่พบหนังสือในรายการยืม");
            return false;
        }
        
        void showBorrowedBooks() {
            if (borrowedBooks.isEmpty()) {
                System.out.println(name + ": ไม่มีหนังสือที่ยืม");
            } else {
                System.out.println(name + " กำลังยืม:");
                borrowedBooks.forEach(b -> System.out.println("  - " + b.title));
            }
        }
    }
    
    // Library class
    static class Library {
        String name;
        List<Book> books;
        List<Member> members;
        
        Library(String name) {
            this.name = name;
            this.books = new ArrayList<>();
            this.members = new ArrayList<>();
        }
        
        void addBook(Book book) {
            books.add(book);
            System.out.println("เพิ่มหนังสือ: " + book.title);
        }
        
        void registerMember(Member member) {
            members.add(member);
            System.out.println("ลงทะเบียนสมาชิก: " + member.name);
        }
        
        void showAllBooks() {
            System.out.println("\n=== รายการหนังสือใน " + name + " ===");
            System.out.printf("%-4s %-30s %-20s %-6s %-12s%n",
                "ID", "ชื่อเรื่อง", "ผู้แต่ง", "ปี", "สถานะ");
            System.out.println("-".repeat(75));
            books.forEach(System.out::println);
        }
        
        List<Book> searchByTitle(String keyword) {
            List<Book> result = new ArrayList<>();
            for (Book b : books) {
                if (b.title.toLowerCase().contains(keyword.toLowerCase())) {
                    result.add(b);
                }
            }
            return result;
        }
        
        void showStats() {
            long available = books.stream().filter(b -> b.available).count();
            System.out.printf("\nหนังสือทั้งหมด: %d, ว่าง: %d, ถูกยืม: %d%n",
                books.size(), available, books.size() - available);
            System.out.println("สมาชิก: " + members.size() + " คน");
        }
    }
    
    public static void main(String[] args) {
        Library lib = new Library("ห้องสมุดกลาง");
        
        // เพิ่มหนังสือ
        Book b1 = new Book("Clean Code", "Robert C. Martin", "978-0132350884", 2008);
        Book b2 = new Book("Design Patterns", "Gang of Four", "978-0201633610", 1994);
        Book b3 = new Book("Java: The Complete Reference", "Herbert Schildt", "978-1260440232", 2020);
        Book b4 = new Book("Effective Java", "Joshua Bloch", "978-0134685991", 2018);
        Book b5 = new Book("Head First Java", "Kathy Sierra", "978-0596009205", 2005);
        
        lib.addBook(b1); lib.addBook(b2); lib.addBook(b3);
        lib.addBook(b4); lib.addBook(b5);
        
        // ลงทะเบียนสมาชิก
        Member m1 = new Member("Alice", "alice@example.com");
        Member m2 = new Member("Bob", "bob@example.com");
        
        lib.registerMember(m1);
        lib.registerMember(m2);
        
        lib.showAllBooks();
        
        // ยืมหนังสือ
        System.out.println("\n=== การยืม-คืน ===");
        m1.borrow(b1);
        m1.borrow(b3);
        m2.borrow(b2);
        
        m1.showBorrowedBooks();
        m2.showBorrowedBooks();
        
        // คืนหนังสือ
        m1.returnBook(b1);
        
        lib.showAllBooks();
        lib.showStats();
        
        // ค้นหา
        System.out.println("\n=== ค้นหา 'java' ===");
        lib.searchByTitle("java").forEach(System.out::println);
    }
}
```

---

## 9.8 แบบฝึกหัด Part 09

### แบบฝึกหัดที่ 1: Student Class

```java
public class Student {
    private String id;
    private String name;
    private int grade; // year 1-4
    private double[] scores; // midterm, final, assignment
    
    Student(String name, int grade) {
        this.id = "STD" + System.currentTimeMillis() % 10000;
        this.name = name;
        this.grade = grade;
        this.scores = new double[3];
    }
    
    void setScores(double midterm, double finalExam, double assignment) {
        scores[0] = midterm;
        scores[1] = finalExam;
        scores[2] = assignment;
    }
    
    double calculateTotal() {
        return scores[0] * 0.4 + scores[1] * 0.4 + scores[2] * 0.2;
    }
    
    String getLetterGrade() {
        double total = calculateTotal();
        if (total >= 80) return "A";
        if (total >= 70) return "B";
        if (total >= 60) return "C";
        if (total >= 50) return "D";
        return "F";
    }
    
    @Override
    public String toString() {
        return String.format("[%s] %s (ชั้นปีที่ %d) | คะแนน: %.2f | เกรด: %s",
            id, name, grade, calculateTotal(), getLetterGrade());
    }
    
    public static void main(String[] args) {
        Student[] students = {
            new Student("Alice", 1),
            new Student("Bob", 2),
            new Student("Charlie", 1)
        };
        
        students[0].setScores(85, 90, 88);
        students[1].setScores(70, 65, 75);
        students[2].setScores(45, 50, 60);
        
        System.out.println("=== รายชื่อนักเรียน ===");
        for (Student s : students) {
            System.out.println(s);
        }
        
        // หาค่าเฉลี่ยของชั้น
        double classAvg = 0;
        for (Student s : students) classAvg += s.calculateTotal();
        System.out.printf("\nค่าเฉลี่ยชั้น: %.2f%n", classAvg / students.length);
    }
}
```

### แบบฝึกหัดที่ 2: Shopping Cart

```java
import java.util.*;

public class ShoppingCart {
    
    static class Product {
        String id, name;
        double price;
        int stock;
        
        Product(String id, String name, double price, int stock) {
            this.id = id; this.name = name;
            this.price = price; this.stock = stock;
        }
        
        @Override
        public String toString() {
            return String.format("%-10s %-20s %8.2f บาท (stock: %d)", id, name, price, stock);
        }
    }
    
    static class CartItem {
        Product product;
        int quantity;
        
        CartItem(Product product, int quantity) {
            this.product = product;
            this.quantity = quantity;
        }
        
        double getTotal() { return product.price * quantity; }
    }
    
    List<CartItem> items = new ArrayList<>();
    
    boolean addItem(Product product, int qty) {
        if (product.stock < qty) {
            System.out.println("Stock ไม่พอ! เหลือ " + product.stock);
            return false;
        }
        for (CartItem item : items) {
            if (item.product == product) {
                item.quantity += qty;
                product.stock -= qty;
                return true;
            }
        }
        items.add(new CartItem(product, qty));
        product.stock -= qty;
        return true;
    }
    
    double getTotal() {
        return items.stream().mapToDouble(CartItem::getTotal).sum();
    }
    
    void checkout() {
        System.out.println("\n=== ใบเสร็จ ===");
        for (CartItem item : items) {
            System.out.printf("%-20s × %d = %.2f%n",
                item.product.name, item.quantity, item.getTotal());
        }
        System.out.printf("รวม: %.2f บาท%n", getTotal());
    }
    
    public static void main(String[] args) {
        Product p1 = new Product("P001", "iPhone 15", 32900, 10);
        Product p2 = new Product("P002", "AirPods Pro", 9900, 5);
        Product p3 = new Product("P003", "MagSafe Case", 1500, 20);
        
        ShoppingCart cart = new ShoppingCart();
        cart.addItem(p1, 1);
        cart.addItem(p2, 2);
        cart.addItem(p3, 3);
        
        cart.checkout();
    }
}
```

---

## 9.9 สรุป Part 09

ในบทนี้คุณได้เรียนรู้:

✅ แนวคิด OOP (4 หลักการ)  
✅ Class และ Object  
✅ Fields และ Methods  
✅ Constructors  
✅ this keyword  
✅ static members  
✅ Inner Classes  

---

*[← Part 08: Methods](./part-08-methods.md) | [Part 10: Encapsulation →](./part-10-oop-encapsulation.md)*
