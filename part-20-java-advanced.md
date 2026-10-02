# Part 20: Java Advanced Features
## หลักสูตร Java & Android Development - ระดับ Intermediate

---

## 20.1 Enum (Enumeration)

```java
public class EnumDemo {
    
    // Basic enum
    enum Day {
        MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;
        
        boolean isWeekend() {
            return this == SATURDAY || this == SUNDAY;
        }
        
        boolean isWeekday() { return !isWeekend(); }
    }
    
    // Enum with fields and methods
    enum Planet {
        MERCURY(3.303e+23, 2.4397e6),
        VENUS  (4.869e+24, 6.0518e6),
        EARTH  (5.976e+24, 6.37814e6),
        MARS   (6.421e+23, 3.3972e6);
        
        private final double mass;    // kg
        private final double radius;  // m
        static final double G = 6.67300E-11;
        
        Planet(double mass, double radius) {
            this.mass = mass;
            this.radius = radius;
        }
        
        double surfaceGravity() {
            return G * mass / (radius * radius);
        }
        
        double surfaceWeight(double otherMass) {
            return otherMass * surfaceGravity();
        }
    }
    
    // Enum with abstract method
    enum Operation {
        PLUS("+")   { @Override double apply(double x, double y) { return x + y; } },
        MINUS("-")  { @Override double apply(double x, double y) { return x - y; } },
        TIMES("*")  { @Override double apply(double x, double y) { return x * y; } },
        DIVIDE("/") { @Override double apply(double x, double y) { return x / y; } };
        
        private final String symbol;
        Operation(String symbol) { this.symbol = symbol; }
        
        abstract double apply(double x, double y);
        
        @Override
        public String toString() { return symbol; }
    }
    
    // Enum with interface
    enum Status implements java.util.function.Predicate<String> {
        ACTIVE {
            @Override public boolean test(String s) { return "active".equals(s); }
            @Override public String getDescription() { return "Currently active"; }
        },
        INACTIVE {
            @Override public boolean test(String s) { return "inactive".equals(s); }
            @Override public String getDescription() { return "Not active"; }
        };
        
        abstract String getDescription();
    }
    
    public static void main(String[] args) {
        // Basic usage
        Day today = Day.WEDNESDAY;
        System.out.println(today + " is weekend: " + today.isWeekend());
        
        for (Day d : Day.values()) {
            System.out.printf("  %-10s ordinal=%d weekend=%b%n",
                d, d.ordinal(), d.isWeekend());
        }
        
        // From string
        Day friday = Day.valueOf("FRIDAY");
        System.out.println("Parsed: " + friday);
        
        // Planet
        double earthWeight = 75.0;
        double mass = earthWeight / Planet.EARTH.surfaceGravity();
        System.out.println("\nWeight on planets:");
        for (Planet p : Planet.values()) {
            System.out.printf("  %-8s %.2f N%n", p, p.surfaceWeight(mass));
        }
        
        // Operation
        System.out.println("\nOperations:");
        double x = 10, y = 3;
        for (Operation op : Operation.values()) {
            System.out.printf("  %.0f %s %.0f = %.2f%n", x, op, y, op.apply(x, y));
        }
        
        // EnumSet / EnumMap
        java.util.EnumSet<Day> workdays = java.util.EnumSet.range(Day.MONDAY, Day.FRIDAY);
        System.out.println("\nWorkdays: " + workdays);
        
        java.util.EnumMap<Day, String> schedule = new java.util.EnumMap<>(Day.class);
        schedule.put(Day.MONDAY, "Team meeting");
        schedule.put(Day.WEDNESDAY, "Code review");
        schedule.put(Day.FRIDAY, "Sprint demo");
        schedule.forEach((day, event) -> System.out.println("  " + day + ": " + event));
    }
}
```

---

## 20.2 Records (Java 16+)

```java
import java.util.*;
import java.util.Objects;

public class RecordsDemo {
    
    // Basic record
    record Point(double x, double y) {
        // Compact constructor (validation)
        Point {
            if (Double.isNaN(x) || Double.isNaN(y)) {
                throw new IllegalArgumentException("Coordinates cannot be NaN");
            }
        }
        
        // Additional methods
        double distanceTo(Point other) {
            double dx = this.x - other.x, dy = this.y - other.y;
            return Math.sqrt(dx*dx + dy*dy);
        }
        
        Point translate(double dx, double dy) {
            return new Point(x + dx, y + dy);
        }
        
        // Static factory
        static Point origin() { return new Point(0, 0); }
    }
    
    // Record with generic
    record Range<T extends Comparable<T>>(T min, T max) {
        Range {
            if (min.compareTo(max) > 0) {
                throw new IllegalArgumentException("min must be <= max");
            }
        }
        
        boolean contains(T value) {
            return value.compareTo(min) >= 0 && value.compareTo(max) <= 0;
        }
        
        T size() {
            if (min instanceof Integer) {
                return (T)(Integer)((Integer)max - (Integer)min);
            }
            return max;  // simplified
        }
    }
    
    // Record in data processing
    record Person(String firstName, String lastName, int age) {
        String fullName() { return firstName + " " + lastName; }
        boolean isAdult() { return age >= 18; }
        
        // Override toString for custom format
        @Override
        public String toString() {
            return String.format("%s %s (age %d)", firstName, lastName, age);
        }
    }
    
    public static void main(String[] args) {
        // Point
        Point p1 = new Point(3, 4);
        Point p2 = new Point(0, 0);
        
        System.out.println("p1: " + p1);          // auto toString
        System.out.println("x=" + p1.x());         // accessor
        System.out.println("Distance: " + p1.distanceTo(p2));
        
        Point p3 = p1.translate(1, 1);
        System.out.println("Translated: " + p3);
        
        // Records are immutable - can't change fields
        // p1.x = 5;  // ERROR
        
        // Range
        Range<Integer> intRange = new Range<>(1, 10);
        System.out.println("\nRange: " + intRange);
        System.out.println("Contains 5: " + intRange.contains(5));
        System.out.println("Contains 15: " + intRange.contains(15));
        
        Range<String> strRange = new Range<>("apple", "mango");
        System.out.println("Contains banana: " + strRange.contains("banana"));
        System.out.println("Contains orange: " + strRange.contains("orange"));
        
        // Person records in collection
        List<Person> people = Arrays.asList(
            new Person("Alice", "Smith", 30),
            new Person("Bob", "Jones", 17),
            new Person("Charlie", "Brown", 25)
        );
        
        people.stream()
            .filter(Person::isAdult)
            .sorted(Comparator.comparing(Person::lastName))
            .forEach(System.out::println);
        
        // Records work as map keys (hashCode/equals auto-generated)
        Map<Point, String> locations = new HashMap<>();
        locations.put(new Point(0, 0), "Origin");
        locations.put(new Point(1, 0), "East");
        locations.put(new Point(0, 1), "North");
        
        System.out.println("\nLocations: " + locations.get(new Point(0, 0)));
    }
}
```

---

## 20.3 Sealed Classes (Java 17+)

```java
public class SealedDemo {
    
    // Sealed class: only listed classes can extend it
    sealed interface Shape permits Circle, Rectangle, Triangle {}
    
    record Circle(double radius) implements Shape {
        double area() { return Math.PI * radius * radius; }
    }
    
    record Rectangle(double width, double height) implements Shape {
        double area() { return width * height; }
    }
    
    record Triangle(double base, double height) implements Shape {
        double area() { return 0.5 * base * height; }
    }
    
    // Pattern matching with sealed classes (exhaustive switch)
    static double getArea(Shape shape) {
        return switch (shape) {
            case Circle c    -> c.area();
            case Rectangle r -> r.area();
            case Triangle t  -> t.area();
            // No default needed - compiler knows all cases
        };
    }
    
    static String describe(Shape shape) {
        return switch (shape) {
            case Circle c    -> String.format("Circle with radius %.1f", c.radius());
            case Rectangle r -> String.format("Rectangle %.1f x %.1f", r.width(), r.height());
            case Triangle t  -> String.format("Triangle base=%.1f h=%.1f", t.base(), t.height());
        };
    }
    
    // Sealed class with abstract methods
    sealed abstract class Expr permits Num, Add, Mul, Neg {}
    
    record Num(double value) extends Expr {}
    record Add(Expr left, Expr right) extends Expr {}
    record Mul(Expr left, Expr right) extends Expr {}
    record Neg(Expr expr) extends Expr {}
    
    double eval(Expr expr) {
        return switch (expr) {
            case Num n    -> n.value();
            case Add a    -> eval(a.left()) + eval(a.right());
            case Mul m    -> eval(m.left()) * eval(m.right());
            case Neg n    -> -eval(n.expr());
        };
    }
    
    public static void main(String[] args) {
        Shape[] shapes = {
            new Circle(5),
            new Rectangle(4, 6),
            new Triangle(3, 8)
        };
        
        for (Shape s : shapes) {
            System.out.printf("%-35s area=%.2f%n", describe(s), getArea(s));
        }
    }
}
```

---

## 20.4 Text Blocks (Java 15+)

```java
public class TextBlocks {
    public static void main(String[] args) {
        // Traditional string vs text block
        String oldJson = "{\n" +
            "  \"name\": \"Alice\",\n" +
            "  \"age\": 30,\n" +
            "  \"city\": \"Bangkok\"\n" +
            "}";
        
        String newJson = """
            {
              "name": "Alice",
              "age": 30,
              "city": "Bangkok"
            }
            """;
        
        System.out.println("JSON:");
        System.out.println(newJson);
        
        // HTML
        String html = """
            <!DOCTYPE html>
            <html>
              <head><title>Hello</title></head>
              <body>
                <h1>Hello, World!</h1>
                <p>This is a text block</p>
              </body>
            </html>
            """;
        System.out.println("HTML:");
        System.out.println(html);
        
        // SQL
        String sql = """
            SELECT u.name, u.email, COUNT(o.id) as orders
            FROM users u
            LEFT JOIN orders o ON u.id = o.user_id
            WHERE u.created_at > '2024-01-01'
              AND u.active = true
            GROUP BY u.id
            ORDER BY orders DESC
            LIMIT 10
            """;
        System.out.println("SQL:");
        System.out.println(sql);
        
        // With formatting
        String template = """
            Dear %s,
            
            Your order #%d has been shipped.
            Total: %.2f THB
            
            Thank you!
            """;
        System.out.println(template.formatted("Alice", 1001, 2500.00));
        
        // Trailing whitespace control
        String noTrailing = """
            line1  \s
            line2
            """;  // \s preserves trailing spaces on line1
    }
}
```

---

## 20.5 Pattern Matching

```java
public class PatternMatching {
    
    sealed interface Animal permits Dog, Cat, Bird {}
    record Dog(String name, String breed) implements Animal {}
    record Cat(String name, boolean indoor) implements Animal {}
    record Bird(String name, double wingspan) implements Animal {}
    
    // Pattern matching instanceof (Java 16+)
    static void describe(Object obj) {
        if (obj instanceof String s) {
            System.out.println("String of length " + s.length() + ": " + s);
        } else if (obj instanceof Integer i && i > 0) {
            System.out.println("Positive integer: " + i);
        } else if (obj instanceof Double d) {
            System.out.printf("Double: %.2f%n", d);
        } else if (obj instanceof int[] arr) {
            System.out.println("Int array of length " + arr.length);
        } else {
            System.out.println("Unknown: " + obj);
        }
    }
    
    // Switch pattern matching (Java 21+)
    static String describeAnimal(Animal animal) {
        return switch (animal) {
            case Dog d when d.breed().equals("Labrador") ->
                d.name() + " is a friendly Labrador";
            case Dog d ->
                d.name() + " is a " + d.breed();
            case Cat c when c.indoor() ->
                c.name() + " is an indoor cat";
            case Cat c ->
                c.name() + " is an outdoor cat";
            case Bird b when b.wingspan() > 1.0 ->
                b.name() + " is a large bird";
            case Bird b ->
                b.name() + " is a small bird";
        };
    }
    
    // Deconstruction pattern (Java 21+)
    static void processAnimal(Animal animal) {
        switch (animal) {
            case Dog(String name, String breed) ->
                System.out.printf("Dog: %s (%s)%n", name, breed);
            case Cat(String name, boolean indoor) ->
                System.out.printf("Cat: %s (%s)%n", name, indoor ? "indoor" : "outdoor");
            case Bird(String name, double span) ->
                System.out.printf("Bird: %s (wingspan=%.1f)%n", name, span);
        }
    }
    
    public static void main(String[] args) {
        // Pattern matching instanceof
        Object[] objects = {"Hello", 42, -5, 3.14, new int[]{1,2,3}, null};
        for (Object obj : objects) describe(obj);
        
        // Animal switch
        Animal[] animals = {
            new Dog("Rex", "Labrador"),
            new Dog("Buddy", "Poodle"),
            new Cat("Kitty", true),
            new Cat("Tom", false),
            new Bird("Eagle", 2.5),
            new Bird("Sparrow", 0.3)
        };
        
        System.out.println("\n=== Animals ===");
        for (Animal a : animals) {
            System.out.println(describeAnimal(a));
        }
    }
}
```

---

## 20.6 var Keyword (Java 10+)

```java
import java.util.*;
import java.util.stream.*;

public class VarDemo {
    
    public static void main(String[] args) {
        // var - infers type from right side
        var name = "Alice";              // String
        var age = 30;                    // int
        var price = 99.99;               // double
        var numbers = new ArrayList<Integer>(); // ArrayList<Integer>
        
        // With complex types (cleaner)
        var map = new HashMap<String, List<Integer>>();
        map.put("odds", List.of(1, 3, 5));
        map.put("evens", List.of(2, 4, 6));
        
        // In for-each
        for (var entry : map.entrySet()) {
            System.out.println(entry.getKey() + ": " + entry.getValue());
        }
        
        // With streams
        var result = Stream.of("a", "bb", "ccc")
            .filter(s -> s.length() > 1)
            .collect(Collectors.toList());
        System.out.println(result);
        
        // var can be used in for loops
        for (var i = 0; i < 3; i++) {
            System.out.print(i + " ");
        }
        System.out.println();
        
        // var cannot be used for:
        // - method parameters: void method(var x) {} // ERROR
        // - fields: var field = "hello"; // ERROR in class body
        // - null: var x = null; // ERROR - can't infer type
        // - without initializer: var x; // ERROR
        
        // Diamond inference with var
        var pairs = new AbstractMap.SimpleEntry<>("key", 42);
        System.out.println(pairs.getKey() + " = " + pairs.getValue());
        
        // try-with-resources
        try (var scanner = new Scanner(System.in)) {
            // scanner is Scanner
        }
    }
}
```

---

## 20.7 โปรแกรมตัวอย่าง: Modern Java Application

```java
import java.util.*;
import java.util.stream.*;
import java.util.function.*;
import java.time.*;

public class ModernJavaApp {
    
    // Records for data
    record Product(String id, String name, String category, double price, int stock) {
        boolean isAvailable() { return stock > 0; }
        boolean isExpensive() { return price > 1000; }
    }
    
    record Order(String orderId, String customerId, List<OrderItem> items, LocalDate date) {
        double total() { return items.stream().mapToDouble(OrderItem::subtotal).sum(); }
        int itemCount() { return items.stream().mapToInt(OrderItem::quantity).sum(); }
    }
    
    record OrderItem(Product product, int quantity) {
        double subtotal() { return product.price() * quantity; }
    }
    
    // Sealed type for order status
    sealed interface OrderStatus permits Pending, Confirmed, Shipped, Delivered, Cancelled {}
    record Pending(LocalDate created) implements OrderStatus {}
    record Confirmed(LocalDate confirmed) implements OrderStatus {}
    record Shipped(LocalDate shipped, String trackingNumber) implements OrderStatus {}
    record Delivered(LocalDate delivered) implements OrderStatus {}
    record Cancelled(String reason) implements OrderStatus {}
    
    static String getStatusMessage(OrderStatus status) {
        return switch (status) {
            case Pending p -> "Pending since " + p.created();
            case Confirmed c -> "Confirmed on " + c.confirmed();
            case Shipped s -> "Shipped on " + s.shipped() + " (tracking: " + s.trackingNumber() + ")";
            case Delivered d -> "Delivered on " + d.delivered();
            case Cancelled c -> "Cancelled: " + c.reason();
        };
    }
    
    public static void main(String[] args) {
        // Create products
        var products = List.of(
            new Product("P001", "iPhone 15", "Electronics", 35900, 50),
            new Product("P002", "iPad Pro",  "Electronics", 42900, 30),
            new Product("P003", "AirPods",   "Accessories", 7490,  100),
            new Product("P004", "MacBook",   "Electronics", 79900, 20),
            new Product("P005", "Watch",     "Accessories", 15900, 60),
            new Product("P006", "Case",      "Accessories", 990,   0)  // out of stock
        );
        
        // Filter and analyze
        System.out.println("=== Available Electronics ===");
        products.stream()
            .filter(p -> p.isAvailable() && p.category().equals("Electronics"))
            .sorted(Comparator.comparingDouble(Product::price))
            .forEach(p -> System.out.printf("  %-12s %,8.0f THB (stock: %d)%n",
                p.name(), p.price(), p.stock()));
        
        // Category analysis
        System.out.println("\n=== Category Summary ===");
        products.stream()
            .collect(Collectors.groupingBy(Product::category))
            .forEach((cat, prods) -> {
                var stats = prods.stream().mapToDouble(Product::price).summaryStatistics();
                System.out.printf("  %-15s count=%d avg=%,.0f min=%,.0f max=%,.0f%n",
                    cat, prods.size(), stats.getAverage(), stats.getMin(), stats.getMax());
            });
        
        // Create orders
        var orders = List.of(
            new Order("O001", "C001",
                List.of(
                    new OrderItem(products.get(0), 1),
                    new OrderItem(products.get(2), 2)
                ),
                LocalDate.of(2024, 1, 15)),
            new Order("O002", "C002",
                List.of(new OrderItem(products.get(3), 1)),
                LocalDate.of(2024, 1, 16)),
            new Order("O003", "C001",
                List.of(
                    new OrderItem(products.get(4), 1),
                    new OrderItem(products.get(2), 1)
                ),
                LocalDate.of(2024, 1, 20))
        );
        
        System.out.println("\n=== Orders ===");
        orders.forEach(o ->
            System.out.printf("  Order %s: %d items, total %,.0f THB%n",
                o.orderId(), o.itemCount(), o.total()));
        
        // Order statuses
        var statuses = Map.of(
            "O001", (OrderStatus) new Shipped(LocalDate.of(2024, 1, 17), "TH1234567"),
            "O002", (OrderStatus) new Delivered(LocalDate.of(2024, 1, 18)),
            "O003", (OrderStatus) new Confirmed(LocalDate.of(2024, 1, 20))
        );
        
        System.out.println("\n=== Order Statuses ===");
        orders.forEach(o -> {
            var status = statuses.getOrDefault(o.orderId(), new Pending(o.date()));
            System.out.printf("  %s: %s%n", o.orderId(), getStatusMessage(status));
        });
        
        // Revenue analysis
        double totalRevenue = orders.stream().mapToDouble(Order::total).sum();
        System.out.printf("\nTotal Revenue: %,.0f THB%n", totalRevenue);
        
        // Top customers
        System.out.println("\n=== Customer Spending ===");
        orders.stream()
            .collect(Collectors.groupingBy(Order::customerId,
                Collectors.summingDouble(Order::total)))
            .entrySet().stream()
            .sorted(Map.Entry.<String, Double>comparingByValue().reversed())
            .forEach(e -> System.out.printf("  %s: %,.0f THB%n", e.getKey(), e.getValue()));
    }
}
```

---

## 20.8 สรุป Part 20

ในบทนี้คุณได้เรียนรู้:

✅ Enum - constants, methods, abstract methods  
✅ Records - immutable data classes (Java 16+)  
✅ Sealed classes (Java 17+)  
✅ Text blocks (Java 15+)  
✅ Pattern matching (Java 16/21+)  
✅ var keyword (Java 10+)  
✅ Modern Java application architecture  

---

## จบ Module 2: Java Intermediate

**Module 2 ครอบคลุม:**
- Parts 11-20: Inheritance, Polymorphism, Interfaces, Exceptions, Collections, Generics, File I/O, Lambda, Streams, Advanced Features

**Module 3 ต่อไป (Parts 21-40):**
- Multithreading & Concurrency
- Design Patterns
- JUnit Testing
- JDBC Database
- Networking
- Reflection & Annotations
- JVM Internals

---

*[← Part 19: Streams](./part-19-streams.md) | [Part 21: Multithreading →](./part-21-multithreading.md)*
