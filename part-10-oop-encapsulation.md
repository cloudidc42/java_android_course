# Part 10: OOP - Encapsulation
## หลักสูตร Java & Android Development - ระดับ Intermediate

---

## 10.1 Encapsulation คืออะไร?

Encapsulation คือการห่อหุ้มข้อมูล (fields) และวิธีการใช้งาน (methods) ไว้ใน class เดียว และควบคุมการเข้าถึงด้วย access modifiers

**ทำไมต้องใช้ Encapsulation?**
- ป้องกันการแก้ไขข้อมูลโดยตรง
- ตรวจสอบความถูกต้องของข้อมูล
- ซ่อน implementation detail
- เปลี่ยน implementation ได้โดยไม่กระทบผู้ใช้

---

## 10.2 Access Modifiers

| Modifier | Class | Package | Subclass | World |
|----------|-------|---------|----------|-------|
| `public` | ✓ | ✓ | ✓ | ✓ |
| `protected` | ✓ | ✓ | ✓ | ✗ |
| *(default)* | ✓ | ✓ | ✗ | ✗ |
| `private` | ✓ | ✗ | ✗ | ✗ |

```java
package com.example;

public class AccessExample {
    public int publicField = 1;      // ทุกที่เข้าถึงได้
    protected int protectedField = 2; // package + subclass
    int defaultField = 3;            // ใน package เดียวกัน
    private int privateField = 4;    // เฉพาะใน class นี้
    
    public void publicMethod() { }
    protected void protectedMethod() { }
    void defaultMethod() { }
    private void privateMethod() { }
}
```

---

## 10.3 Getters และ Setters

```java
public class Employee {
    // private fields - ห้ามเข้าถึงโดยตรง
    private int id;
    private String name;
    private double salary;
    private int age;
    private String email;
    private String department;
    
    // Constructor
    public Employee(int id, String name, double salary) {
        this.id = id;
        setName(name);    // ใช้ setter เพื่อ validate
        setSalary(salary);
    }
    
    // Getters - อ่านค่า
    public int getId() { return id; }
    public String getName() { return name; }
    public double getSalary() { return salary; }
    public int getAge() { return age; }
    public String getEmail() { return email; }
    public String getDepartment() { return department; }
    
    // Setters - กำหนดค่าพร้อม validate
    public void setName(String name) {
        if (name == null || name.trim().isEmpty()) {
            throw new IllegalArgumentException("ชื่อต้องไม่ว่าง");
        }
        this.name = name.trim();
    }
    
    public void setSalary(double salary) {
        if (salary < 15000) {
            throw new IllegalArgumentException("เงินเดือนต้องไม่น้อยกว่า 15,000");
        }
        this.salary = salary;
    }
    
    public void setAge(int age) {
        if (age < 18 || age > 65) {
            throw new IllegalArgumentException("อายุต้องอยู่ระหว่าง 18-65");
        }
        this.age = age;
    }
    
    public void setEmail(String email) {
        if (email != null && !email.matches("[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}")) {
            throw new IllegalArgumentException("Email ไม่ถูกต้อง");
        }
        this.email = email;
    }
    
    public void setDepartment(String department) {
        this.department = department;
    }
    
    // Business methods
    public void giveRaise(double percentage) {
        if (percentage <= 0) throw new IllegalArgumentException("% ต้องมากกว่า 0");
        double raise = salary * percentage / 100;
        salary += raise;
        System.out.printf("ขึ้นเงินเดือน %.1f%% = %.2f บาท (ใหม่: %.2f)%n",
            percentage, raise, salary);
    }
    
    public boolean isEligibleForBonus() {
        return salary < 50000;  // ตัวอย่างกฎ bonus
    }
    
    @Override
    public String toString() {
        return String.format("Employee{id=%d, name='%s', salary=%.2f, dept='%s'}",
            id, name, salary, department);
    }
    
    public static void main(String[] args) {
        Employee emp = new Employee(1, "Alice Smith", 25000);
        emp.setAge(28);
        emp.setEmail("alice@company.com");
        emp.setDepartment("Engineering");
        
        System.out.println(emp);
        System.out.println("Eligible for bonus: " + emp.isEligibleForBonus());
        
        emp.giveRaise(10);
        System.out.println("New salary: " + emp.getSalary());
        
        // Test validation
        try {
            emp.setSalary(-1000);
        } catch (IllegalArgumentException e) {
            System.out.println("Error: " + e.getMessage());
        }
        
        try {
            emp.setEmail("not-an-email");
        } catch (IllegalArgumentException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}
```

---

## 10.4 Immutable Classes

```java
public final class Money {
    private final double amount;
    private final String currency;
    
    // Immutable: ค่าเปลี่ยนไม่ได้หลังสร้าง
    public Money(double amount, String currency) {
        if (amount < 0) throw new IllegalArgumentException("จำนวนเงินต้องไม่ติดลบ");
        this.amount = amount;
        this.currency = currency;
    }
    
    public double getAmount() { return amount; }
    public String getCurrency() { return currency; }
    
    // Return new object แทนการแก้ไข
    public Money add(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new IllegalArgumentException("สกุลเงินต้องเหมือนกัน");
        }
        return new Money(this.amount + other.amount, this.currency);
    }
    
    public Money subtract(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new IllegalArgumentException("สกุลเงินต้องเหมือนกัน");
        }
        if (this.amount < other.amount) {
            throw new IllegalArgumentException("เงินไม่พอ");
        }
        return new Money(this.amount - other.amount, this.currency);
    }
    
    public Money multiply(double factor) {
        return new Money(this.amount * factor, this.currency);
    }
    
    @Override
    public String toString() {
        return String.format("%.2f %s", amount, currency);
    }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (!(obj instanceof Money)) return false;
        Money other = (Money) obj;
        return this.amount == other.amount && this.currency.equals(other.currency);
    }
    
    @Override
    public int hashCode() {
        return java.util.Objects.hash(amount, currency);
    }
    
    public static void main(String[] args) {
        Money price = new Money(100, "THB");
        Money tax = new Money(7, "THB");
        Money total = price.add(tax);
        
        System.out.println("ราคา: " + price);
        System.out.println("ภาษี: " + tax);
        System.out.println("รวม: " + total);
        System.out.println("price unchanged: " + price);  // ยังเป็น 100
        
        Money discounted = total.multiply(0.9);
        System.out.println("ลด 10%: " + discounted);
    }
}
```

---

## 10.5 Builder Pattern

```java
public class UserProfile {
    // Required fields
    private final String username;
    private final String email;
    
    // Optional fields
    private final String firstName;
    private final String lastName;
    private final int age;
    private final String phone;
    private final String address;
    private final String bio;
    private final boolean emailVerified;
    
    // Private constructor - ใช้ผ่าน Builder เท่านั้น
    private UserProfile(Builder builder) {
        this.username = builder.username;
        this.email = builder.email;
        this.firstName = builder.firstName;
        this.lastName = builder.lastName;
        this.age = builder.age;
        this.phone = builder.phone;
        this.address = builder.address;
        this.bio = builder.bio;
        this.emailVerified = builder.emailVerified;
    }
    
    // Getters
    public String getUsername() { return username; }
    public String getEmail() { return email; }
    public String getFirstName() { return firstName; }
    public String getLastName() { return lastName; }
    public int getAge() { return age; }
    public String getFullName() { return firstName + " " + lastName; }
    
    @Override
    public String toString() {
        return String.format("UserProfile{username='%s', email='%s', name='%s', age=%d, verified=%b}",
            username, email, getFullName(), age, emailVerified);
    }
    
    // Static Builder class
    public static class Builder {
        // Required
        private final String username;
        private final String email;
        
        // Optional (with defaults)
        private String firstName = "";
        private String lastName = "";
        private int age = 0;
        private String phone = "";
        private String address = "";
        private String bio = "";
        private boolean emailVerified = false;
        
        public Builder(String username, String email) {
            if (username == null || username.isEmpty())
                throw new IllegalArgumentException("Username ต้องไม่ว่าง");
            if (!email.matches("[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}"))
                throw new IllegalArgumentException("Email ไม่ถูกต้อง");
            this.username = username;
            this.email = email;
        }
        
        public Builder firstName(String firstName) {
            this.firstName = firstName;
            return this;
        }
        
        public Builder lastName(String lastName) {
            this.lastName = lastName;
            return this;
        }
        
        public Builder age(int age) {
            if (age < 0 || age > 150) throw new IllegalArgumentException("อายุไม่ถูกต้อง");
            this.age = age;
            return this;
        }
        
        public Builder phone(String phone) {
            this.phone = phone;
            return this;
        }
        
        public Builder address(String address) {
            this.address = address;
            return this;
        }
        
        public Builder bio(String bio) {
            this.bio = bio;
            return this;
        }
        
        public Builder emailVerified(boolean verified) {
            this.emailVerified = verified;
            return this;
        }
        
        public UserProfile build() {
            return new UserProfile(this);
        }
    }
    
    public static void main(String[] args) {
        // สร้าง profile ด้วย Builder
        UserProfile user1 = new UserProfile.Builder("alice99", "alice@example.com")
            .firstName("Alice")
            .lastName("Johnson")
            .age(25)
            .phone("0812345678")
            .emailVerified(true)
            .build();
        
        UserProfile user2 = new UserProfile.Builder("bob_dev", "bob@dev.com")
            .firstName("Bob")
            .lastName("Smith")
            .age(30)
            .bio("Software developer passionate about Java")
            .build();
        
        System.out.println(user1);
        System.out.println(user2);
        
        // Validation
        try {
            new UserProfile.Builder("", "test@test.com").build();
        } catch (IllegalArgumentException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}
```

---

## 10.6 Record Classes (Java 16+)

```java
public class RecordDemo {
    
    // Record: immutable data class อัตโนมัติ
    record Point(double x, double y) {
        // Compact constructor สำหรับ validate
        Point {
            if (Double.isNaN(x) || Double.isNaN(y)) {
                throw new IllegalArgumentException("Coordinates cannot be NaN");
            }
        }
        
        // Custom methods
        double distanceTo(Point other) {
            double dx = this.x - other.x;
            double dy = this.y - other.y;
            return Math.sqrt(dx*dx + dy*dy);
        }
        
        Point translate(double dx, double dy) {
            return new Point(x + dx, y + dy);
        }
        
        static Point origin() {
            return new Point(0, 0);
        }
    }
    
    record Rectangle(Point topLeft, double width, double height) {
        double area() { return width * height; }
        double perimeter() { return 2 * (width + height); }
        Point center() { return new Point(topLeft.x() + width/2, topLeft.y() + height/2); }
    }
    
    record Person(String name, int age, String email) implements Comparable<Person> {
        Person {
            if (age < 0) throw new IllegalArgumentException("อายุต้องไม่ติดลบ");
        }
        
        @Override
        public int compareTo(Person other) {
            return Integer.compare(this.age, other.age);
        }
    }
    
    public static void main(String[] args) {
        Point p1 = new Point(0, 0);
        Point p2 = new Point(3, 4);
        
        System.out.println("p1: " + p1);              // Point[x=0.0, y=0.0]
        System.out.println("p2: " + p2);
        System.out.printf("Distance: %.2f%n", p1.distanceTo(p2));  // 5.0
        
        Point p3 = p1.translate(1, 2);
        System.out.println("p3 (translated): " + p3);
        
        // Records are equal by value
        System.out.println(new Point(1, 2).equals(new Point(1, 2)));  // true
        
        Rectangle rect = new Rectangle(new Point(0, 0), 10, 5);
        System.out.printf("Area: %.1f, Perimeter: %.1f%n", rect.area(), rect.perimeter());
        System.out.println("Center: " + rect.center());
        
        Person alice = new Person("Alice", 25, "alice@example.com");
        System.out.println(alice.name() + " is " + alice.age());
    }
}
```

---

## 10.7 โปรแกรมตัวอย่าง: Product Catalog

```java
import java.util.*;

public class ProductCatalog {
    
    public static class Product {
        private final String id;
        private String name;
        private String description;
        private double price;
        private int stock;
        private String category;
        private final List<String> tags;
        
        public Product(String id, String name, double price, int stock, String category) {
            if (id == null || id.isEmpty()) throw new IllegalArgumentException("ID required");
            if (price < 0) throw new IllegalArgumentException("Price must be >= 0");
            if (stock < 0) throw new IllegalArgumentException("Stock must be >= 0");
            
            this.id = id;
            this.name = name;
            this.price = price;
            this.stock = stock;
            this.category = category;
            this.tags = new ArrayList<>();
        }
        
        // Getters
        public String getId() { return id; }
        public String getName() { return name; }
        public double getPrice() { return price; }
        public int getStock() { return stock; }
        public String getCategory() { return category; }
        public List<String> getTags() { return Collections.unmodifiableList(tags); }
        public boolean isAvailable() { return stock > 0; }
        
        // Setters with validation
        public void setName(String name) {
            if (name == null || name.trim().isEmpty())
                throw new IllegalArgumentException("Name required");
            this.name = name;
        }
        
        public void setPrice(double price) {
            if (price < 0) throw new IllegalArgumentException("Price must be >= 0");
            this.price = price;
        }
        
        public void setStock(int stock) {
            if (stock < 0) throw new IllegalArgumentException("Stock must be >= 0");
            this.stock = stock;
        }
        
        public void addTag(String tag) {
            if (!tags.contains(tag)) tags.add(tag.toLowerCase());
        }
        
        public boolean sell(int quantity) {
            if (quantity <= 0) throw new IllegalArgumentException("Quantity must be > 0");
            if (stock < quantity) return false;
            stock -= quantity;
            return true;
        }
        
        public void restock(int quantity) {
            if (quantity <= 0) throw new IllegalArgumentException("Quantity must be > 0");
            stock += quantity;
        }
        
        public void applyDiscount(double percent) {
            if (percent <= 0 || percent >= 100)
                throw new IllegalArgumentException("Discount must be 0-100");
            price *= (1 - percent / 100);
        }
        
        @Override
        public String toString() {
            return String.format("[%s] %-25s %8.2f บาท (stock: %3d) [%s]",
                id, name, price, stock, category);
        }
    }
    
    private final Map<String, Product> products;
    
    public ProductCatalog() {
        this.products = new LinkedHashMap<>();
    }
    
    public void addProduct(Product p) {
        if (products.containsKey(p.getId()))
            throw new IllegalArgumentException("Product ID " + p.getId() + " มีแล้ว");
        products.put(p.getId(), p);
    }
    
    public Optional<Product> findById(String id) {
        return Optional.ofNullable(products.get(id));
    }
    
    public List<Product> findByCategory(String category) {
        List<Product> result = new ArrayList<>();
        for (Product p : products.values()) {
            if (p.getCategory().equalsIgnoreCase(category)) result.add(p);
        }
        return result;
    }
    
    public List<Product> findByPriceRange(double min, double max) {
        List<Product> result = new ArrayList<>();
        for (Product p : products.values()) {
            if (p.getPrice() >= min && p.getPrice() <= max) result.add(p);
        }
        return result;
    }
    
    public List<Product> getAvailable() {
        List<Product> result = new ArrayList<>();
        for (Product p : products.values()) {
            if (p.isAvailable()) result.add(p);
        }
        return result;
    }
    
    public void showAll() {
        System.out.println("=".repeat(65));
        System.out.println("Product Catalog");
        System.out.println("=".repeat(65));
        products.values().forEach(System.out::println);
        System.out.println("-".repeat(65));
        System.out.println("รวม " + products.size() + " รายการ");
    }
    
    public static void main(String[] args) {
        ProductCatalog catalog = new ProductCatalog();
        
        Product p1 = new Product("P001", "iPhone 15 Pro", 49900, 15, "Smartphone");
        p1.addTag("apple"); p1.addTag("smartphone"); p1.addTag("ios");
        
        Product p2 = new Product("P002", "Samsung Galaxy S24", 35900, 20, "Smartphone");
        p2.addTag("samsung"); p2.addTag("smartphone"); p2.addTag("android");
        
        Product p3 = new Product("P003", "iPad Air", 22900, 8, "Tablet");
        Product p4 = new Product("P004", "MacBook Pro M3", 89900, 5, "Laptop");
        Product p5 = new Product("P005", "AirPods Pro 2", 9900, 30, "Audio");
        
        catalog.addProduct(p1); catalog.addProduct(p2);
        catalog.addProduct(p3); catalog.addProduct(p4); catalog.addProduct(p5);
        
        catalog.showAll();
        
        System.out.println("\n--- Smartphones ---");
        catalog.findByCategory("Smartphone").forEach(System.out::println);
        
        System.out.println("\n--- ราคา 10,000-50,000 ---");
        catalog.findByPriceRange(10000, 50000).forEach(System.out::println);
        
        // ซื้อสินค้า
        System.out.println("\n--- ทดสอบการซื้อ ---");
        if (p1.sell(3)) System.out.println("ขาย iPhone 15 Pro 3 เครื่อง");
        System.out.println("iPhone stock remaining: " + p1.getStock());
        
        // ลดราคา
        p2.applyDiscount(10);
        System.out.println("Samsung after 10% discount: " + p2.getPrice());
    }
}
```

---

## 10.8 สรุป Part 10

ในบทนี้คุณได้เรียนรู้:

✅ Encapsulation และ Access Modifiers  
✅ Getters และ Setters  
✅ Validation ใน setters  
✅ Immutable Classes  
✅ Builder Pattern  
✅ Record Classes (Java 16+)  

---

*[← Part 09: OOP Basics](./part-09-oop-basics.md) | [Part 11: Inheritance →](./part-11-oop-inheritance.md)*
