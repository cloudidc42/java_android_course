# Part 12: OOP - Polymorphism
## หลักสูตร Java & Android Development - ระดับ Intermediate

---

## 12.1 Polymorphism คืออะไร?

Polymorphism (พหุสัณฐาน) คือความสามารถที่ object หลายรูปแบบสามารถใช้งานผ่าน interface เดียวกันได้

**2 ประเภทหลัก:**
1. **Compile-time Polymorphism** (Method Overloading)
2. **Runtime Polymorphism** (Method Overriding + Dynamic Dispatch)

---

## 12.2 Runtime Polymorphism

```java
public class Shape {
    String color;
    
    Shape(String color) { this.color = color; }
    
    double area() { return 0; }
    void draw() { System.out.println("Drawing a " + color + " shape"); }
    
    @Override
    public String toString() {
        return String.format("%s(area=%.2f)", getClass().getSimpleName(), area());
    }
}

public class Circle extends Shape {
    double radius;
    Circle(String color, double radius) { super(color); this.radius = radius; }
    
    @Override
    double area() { return Math.PI * radius * radius; }
    
    @Override
    void draw() {
        System.out.printf("Drawing %s circle r=%.1f%n", color, radius);
    }
}

public class Rectangle extends Shape {
    double w, h;
    Rectangle(String color, double w, double h) { super(color); this.w=w; this.h=h; }
    
    @Override
    double area() { return w * h; }
    
    @Override
    void draw() {
        System.out.printf("Drawing %s rectangle %sx%s%n", color, w, h);
    }
}

public class Triangle extends Shape {
    double base, height;
    Triangle(String color, double base, double height) {
        super(color); this.base=base; this.height=height;
    }
    
    @Override
    double area() { return 0.5 * base * height; }
    
    @Override
    void draw() {
        System.out.printf("Drawing %s triangle base=%.1f h=%.1f%n", color, base, height);
    }
}

public class PolymorphismDemo {
    static void printArea(Shape s) {
        // Runtime decides which area() to call
        System.out.printf("Area of %s: %.2f%n", s.getClass().getSimpleName(), s.area());
    }
    
    static double totalArea(Shape[] shapes) {
        double total = 0;
        for (Shape s : shapes) total += s.area();  // polymorphic call
        return total;
    }
    
    public static void main(String[] args) {
        Shape[] shapes = {
            new Circle("red", 5),
            new Rectangle("blue", 4, 6),
            new Triangle("green", 3, 8),
            new Circle("yellow", 2),
            new Rectangle("purple", 10, 3)
        };
        
        // Polymorphism: same method call, different behavior
        for (Shape s : shapes) {
            s.draw();
            printArea(s);
        }
        
        System.out.printf("\nTotal area: %.2f%n", totalArea(shapes));
    }
}
```

---

## 12.3 Upcasting และ Downcasting

```java
public class CastingDemo {
    
    static class Animal {
        String name;
        Animal(String name) { this.name = name; }
        void speak() { System.out.println(name + " makes a sound"); }
        String getInfo() { return "Animal: " + name; }
    }
    
    static class Dog extends Animal {
        String breed;
        Dog(String name, String breed) { super(name); this.breed = breed; }
        
        @Override
        void speak() { System.out.println(name + " barks: Woof!"); }
        
        void fetch() { System.out.println(name + " fetches!"); }
        String getInfo() { return "Dog: " + name + " (" + breed + ")"; }
    }
    
    static class Cat extends Animal {
        boolean isIndoor;
        Cat(String name, boolean isIndoor) { super(name); this.isIndoor = isIndoor; }
        
        @Override
        void speak() { System.out.println(name + " meows: Meow!"); }
        
        void purr() { System.out.println(name + " purrs~"); }
    }
    
    public static void main(String[] args) {
        // --- Upcasting: Subclass -> Superclass (Implicit, always safe) ---
        Dog dog = new Dog("Rex", "German Shepherd");
        Animal animal = dog;  // upcast: implicit
        
        animal.speak();       // "Rex barks: Woof!" - runtime uses Dog's speak()
        animal.getInfo();     // calls Dog's getInfo() - polymorphism
        // animal.fetch();    // ERROR: Animal doesn't know about fetch()
        
        // --- Downcasting: Superclass -> Subclass (Explicit, may fail) ---
        Animal a2 = new Dog("Buddy", "Labrador");
        
        if (a2 instanceof Dog) {  // ALWAYS check before casting
            Dog d = (Dog) a2;  // downcast: explicit
            d.fetch();         // now we can call Dog-specific method
            System.out.println(d.breed);
        }
        
        // --- instanceof Pattern Matching (Java 16+) ---
        Animal[] animals = {
            new Dog("Max", "Poodle"),
            new Cat("Kitty", true),
            new Dog("Rocky", "Bulldog"),
            new Cat("Whiskers", false)
        };
        
        for (Animal a : animals) {
            // Old way:
            // if (a instanceof Dog) { Dog d = (Dog) a; d.fetch(); }
            
            // New way (Java 16+):
            if (a instanceof Dog d) {
                System.out.println("Dog found: " + d.breed);
                d.fetch();
            } else if (a instanceof Cat c) {
                System.out.println("Cat found, indoor: " + c.isIndoor);
                c.purr();
            }
        }
        
        // --- ClassCastException demo ---
        try {
            Animal wrongAnimal = new Cat("Tom", false);
            Dog wrongCast = (Dog) wrongAnimal;  // ClassCastException!
        } catch (ClassCastException e) {
            System.out.println("Cannot cast Cat to Dog: " + e.getMessage());
        }
    }
}
```

---

## 12.4 Abstract Class + Polymorphism

```java
import java.util.*;

public class PaymentSystem {
    
    abstract static class Payment {
        protected String id;
        protected double amount;
        protected String currency;
        
        Payment(String id, double amount, String currency) {
            this.id = id;
            this.amount = amount;
            this.currency = currency;
        }
        
        abstract boolean process();
        abstract String getPaymentType();
        
        void printReceipt() {
            System.out.printf("[Receipt #%s]%n", id);
            System.out.printf("Type: %s%n", getPaymentType());
            System.out.printf("Amount: %.2f %s%n", amount, currency);
            System.out.printf("Status: %s%n", process() ? "SUCCESS" : "FAILED");
            System.out.println("-".repeat(30));
        }
    }
    
    static class CreditCardPayment extends Payment {
        String cardNumber;
        String cardHolder;
        int expiryYear;
        
        CreditCardPayment(String id, double amount, String cardNumber, String holder, int year) {
            super(id, amount, "THB");
            this.cardNumber = cardNumber;
            this.cardHolder = holder;
            this.expiryYear = year;
        }
        
        @Override
        boolean process() {
            if (expiryYear < 2024) { System.out.println("Card expired!"); return false; }
            System.out.printf("Processing CC: **** **** **** %s%n", cardNumber.substring(12));
            return true;
        }
        
        @Override
        String getPaymentType() { return "Credit Card"; }
    }
    
    static class PromptPayPayment extends Payment {
        String phoneNumber;
        
        PromptPayPayment(String id, double amount, String phone) {
            super(id, amount, "THB");
            this.phoneNumber = phone;
        }
        
        @Override
        boolean process() {
            System.out.printf("Sending PromptPay to %s%n", phoneNumber);
            return amount <= 100000;  // limit 100k
        }
        
        @Override
        String getPaymentType() { return "PromptPay"; }
    }
    
    static class CryptoPayment extends Payment {
        String walletAddress;
        String coin;
        double exchangeRate;
        
        CryptoPayment(String id, double amount, String wallet, String coin, double rate) {
            super(id, amount, "THB");
            this.walletAddress = wallet;
            this.coin = coin;
            this.exchangeRate = rate;
        }
        
        @Override
        boolean process() {
            double cryptoAmount = amount / exchangeRate;
            System.out.printf("Sending %.6f %s to %s...%n", cryptoAmount, coin, walletAddress.substring(0, 8) + "...");
            return true;
        }
        
        @Override
        String getPaymentType() { return "Crypto (" + coin + ")"; }
    }
    
    public static void main(String[] args) {
        List<Payment> payments = new ArrayList<>();
        payments.add(new CreditCardPayment("P001", 1500.00, "1234567890123456", "John Doe", 2026));
        payments.add(new PromptPayPayment("P002", 850.50, "0812345678"));
        payments.add(new CryptoPayment("P003", 5000.00, "0xABCDEF1234567890", "ETH", 130000));
        payments.add(new CreditCardPayment("P004", 200.00, "9876543210987654", "Jane Smith", 2022)); // expired
        
        System.out.println("=== Processing Payments ===\n");
        for (Payment p : payments) {
            p.printReceipt();
        }
        
        double total = payments.stream()
            .mapToDouble(p -> p.amount)
            .sum();
        System.out.printf("Total transactions: %.2f THB%n", total);
    }
}
```

---

## 12.5 Interface Polymorphism

```java
import java.util.*;

public class InterfacePolymorphism {
    
    interface Drawable {
        void draw();
        default void drawWithBorder() {
            System.out.println("+" + "-".repeat(20) + "+");
            draw();
            System.out.println("+" + "-".repeat(20) + "+");
        }
    }
    
    interface Resizable {
        void resize(double factor);
        double getSize();
    }
    
    interface Colorable {
        void setColor(String color);
        String getColor();
    }
    
    static class Square implements Drawable, Resizable, Colorable {
        private double side;
        private String color;
        
        Square(double side, String color) {
            this.side = side;
            this.color = color;
        }
        
        @Override public void draw() {
            System.out.printf("Drawing %s Square (side=%.1f, area=%.1f)%n",
                color, side, side*side);
        }
        @Override public void resize(double f) { side *= f; }
        @Override public double getSize() { return side; }
        @Override public void setColor(String c) { color = c; }
        @Override public String getColor() { return color; }
    }
    
    static class Circle implements Drawable, Resizable, Colorable {
        private double radius;
        private String color;
        
        Circle(double radius, String color) {
            this.radius = radius;
            this.color = color;
        }
        
        @Override public void draw() {
            System.out.printf("Drawing %s Circle (r=%.1f, area=%.1f)%n",
                color, radius, Math.PI*radius*radius);
        }
        @Override public void resize(double f) { radius *= f; }
        @Override public double getSize() { return radius; }
        @Override public void setColor(String c) { color = c; }
        @Override public String getColor() { return color; }
    }
    
    public static void main(String[] args) {
        List<Drawable> drawables = new ArrayList<>();
        drawables.add(new Square(5, "red"));
        drawables.add(new Circle(3, "blue"));
        drawables.add(new Square(8, "green"));
        
        System.out.println("=== Drawing all shapes ===");
        for (Drawable d : drawables) {
            d.draw();
        }
        
        // Use as Resizable
        List<Resizable> resizables = new ArrayList<>();
        resizables.add(new Square(4, "purple"));
        resizables.add(new Circle(2, "orange"));
        
        System.out.println("\n=== Before resize ===");
        resizables.forEach(r -> System.out.printf("Size: %.1f%n", r.getSize()));
        
        resizables.forEach(r -> r.resize(2.0));
        
        System.out.println("=== After resize x2 ===");
        resizables.forEach(r -> System.out.printf("Size: %.1f%n", r.getSize()));
    }
}
```

---

## 12.6 Polymorphism กับ Collections

```java
import java.util.*;
import java.util.stream.*;

public class PolymorphicCollections {
    
    interface Taxable {
        double calculateTax();
    }
    
    abstract static class Product implements Taxable {
        protected String name;
        protected double price;
        
        Product(String name, double price) {
            this.name = name;
            this.price = price;
        }
        
        abstract String getCategory();
        
        double getFinalPrice() { return price + calculateTax(); }
        
        @Override
        public String toString() {
            return String.format("%-20s %-15s %8.2f + tax %6.2f = %8.2f",
                name, getCategory(), price, calculateTax(), getFinalPrice());
        }
    }
    
    static class Food extends Product {
        Food(String name, double price) { super(name, price); }
        
        @Override
        public double calculateTax() { return 0; }  // VAT exempt
        
        @Override
        public String getCategory() { return "Food"; }
    }
    
    static class Electronics extends Product {
        Electronics(String name, double price) { super(name, price); }
        
        @Override
        public double calculateTax() { return price * 0.07; }  // 7% VAT
        
        @Override
        public String getCategory() { return "Electronics"; }
    }
    
    static class Luxury extends Product {
        Luxury(String name, double price) { super(name, price); }
        
        @Override
        public double calculateTax() { return price * 0.17; }  // 7% VAT + 10% luxury tax
        
        @Override
        public String getCategory() { return "Luxury"; }
    }
    
    public static void main(String[] args) {
        List<Product> cart = Arrays.asList(
            new Food("Apple", 50),
            new Food("Bread", 30),
            new Electronics("Phone", 15000),
            new Electronics("Headphone", 2500),
            new Luxury("Watch", 80000),
            new Luxury("Bag", 45000)
        );
        
        System.out.println("=== Shopping Cart ===");
        System.out.printf("%-20s %-15s %8s   %6s   %8s%n",
            "Name", "Category", "Price", "Tax", "Total");
        System.out.println("-".repeat(65));
        cart.forEach(System.out::println);
        System.out.println("-".repeat(65));
        
        double subtotal = cart.stream().mapToDouble(p -> p.price).sum();
        double totalTax = cart.stream().mapToDouble(Product::calculateTax).sum();
        double grandTotal = cart.stream().mapToDouble(Product::getFinalPrice).sum();
        
        System.out.printf("Subtotal: %,8.2f%n", subtotal);
        System.out.printf("Tax:      %,8.2f%n", totalTax);
        System.out.printf("Total:    %,8.2f%n", grandTotal);
        
        // Group by category
        System.out.println("\n=== By Category ===");
        Map<String, Double> byCategory = cart.stream()
            .collect(Collectors.groupingBy(Product::getCategory,
                Collectors.summingDouble(Product::getFinalPrice)));
        
        byCategory.entrySet().stream()
            .sorted(Map.Entry.<String,Double>comparingByValue().reversed())
            .forEach(e -> System.out.printf("%-15s: %,8.2f%n", e.getKey(), e.getValue()));
    }
}
```

---

## 12.7 แบบฝึกหัด Part 12

```java
// แบบฝึกหัด: สร้าง Employee hierarchy ที่มี Polymorphism
public class EmployeeHierarchy {
    
    abstract static class Employee {
        protected String name;
        protected double baseSalary;
        
        Employee(String name, double baseSalary) {
            this.name = name;
            this.baseSalary = baseSalary;
        }
        
        abstract double calculatePay();
        abstract String getRole();
        
        void printInfo() {
            System.out.printf("%-20s %-15s Pay: %,.2f%n",
                name, getRole(), calculatePay());
        }
    }
    
    static class FullTimeEmployee extends Employee {
        double benefits;
        
        FullTimeEmployee(String name, double salary, double benefits) {
            super(name, salary);
            this.benefits = benefits;
        }
        
        @Override
        public double calculatePay() { return baseSalary + benefits; }
        
        @Override
        public String getRole() { return "Full-Time"; }
    }
    
    static class PartTimeEmployee extends Employee {
        double hoursWorked;
        double hourlyRate;
        
        PartTimeEmployee(String name, double hoursWorked, double hourlyRate) {
            super(name, 0);
            this.hoursWorked = hoursWorked;
            this.hourlyRate = hourlyRate;
        }
        
        @Override
        public double calculatePay() { return hoursWorked * hourlyRate; }
        
        @Override
        public String getRole() { return "Part-Time"; }
    }
    
    static class Contractor extends Employee {
        int projectsCompleted;
        double ratePerProject;
        
        Contractor(String name, int projects, double rate) {
            super(name, 0);
            this.projectsCompleted = projects;
            this.ratePerProject = rate;
        }
        
        @Override
        public double calculatePay() { return projectsCompleted * ratePerProject; }
        
        @Override
        public String getRole() { return "Contractor"; }
    }
    
    public static void main(String[] args) {
        Employee[] employees = {
            new FullTimeEmployee("Alice Johnson", 45000, 5000),
            new FullTimeEmployee("Bob Smith", 55000, 7000),
            new PartTimeEmployee("Carol Davis", 80, 200),
            new PartTimeEmployee("David Brown", 60, 180),
            new Contractor("Eve Wilson", 3, 15000),
            new Contractor("Frank Miller", 2, 20000)
        };
        
        System.out.println("=== Payroll Report ===");
        System.out.printf("%-20s %-15s %s%n", "Name", "Role", "Pay");
        System.out.println("-".repeat(50));
        
        for (Employee emp : employees) {
            emp.printInfo();
        }
        
        System.out.println("-".repeat(50));
        double totalPayroll = 0;
        for (Employee emp : employees) {
            totalPayroll += emp.calculatePay();  // Polymorphic call
        }
        System.out.printf("Total Payroll: %,.2f%n", totalPayroll);
    }
}
```

---

## 12.8 สรุป Part 12

ในบทนี้คุณได้เรียนรู้:

✅ Compile-time vs Runtime Polymorphism  
✅ Dynamic Method Dispatch  
✅ Upcasting และ Downcasting  
✅ instanceof + Pattern Matching  
✅ Polymorphism กับ Abstract Classes  
✅ Polymorphism กับ Interfaces  
✅ Polymorphism กับ Collections  

---

*[← Part 11: Inheritance](./part-11-oop-inheritance.md) | [Part 13: Interfaces →](./part-13-interfaces.md)*
