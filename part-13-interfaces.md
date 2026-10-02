# Part 13: Interfaces และ Abstract Classes
## หลักสูตร Java & Android Development - ระดับ Intermediate

---

## 13.1 Interface คืออะไร?

Interface คือ contract (สัญญา) ที่กำหนดว่า class ต้องมี method อะไรบ้าง

```java
public interface Flyable {
    // constants (public static final)
    double MAX_ALTITUDE = 12000.0;
    
    // abstract methods (public abstract)
    void takeOff();
    void land();
    void fly(int altitude);
    
    // default method (Java 8+)
    default void describeAbility() {
        System.out.println(getClass().getSimpleName() + " can fly!");
    }
    
    // static method (Java 8+)
    static boolean isValidAltitude(double alt) {
        return alt > 0 && alt <= MAX_ALTITUDE;
    }
    
    // private method (Java 9+)
    private void logFlight(String event) {
        System.out.println("[LOG] " + event);
    }
}
```

---

## 13.2 Implementing Interface

```java
public interface Swimmable {
    void swim();
    default void float_() { System.out.println(getClass().getSimpleName() + " floats"); }
}

public interface Runnable {
    void run();
    default int getSpeed() { return 10; }
}

// Multiple interface implementation
public class Duck implements Flyable, Swimmable, Runnable {
    private String name;
    
    Duck(String name) { this.name = name; }
    
    @Override public void takeOff() { System.out.println(name + " takes off!"); }
    @Override public void land() { System.out.println(name + " lands!"); }
    @Override public void fly(int altitude) {
        if (Flyable.isValidAltitude(altitude)) {
            System.out.printf("%s flies at %dm%n", name, altitude);
        }
    }
    @Override public void swim() { System.out.println(name + " swims!"); }
    @Override public void run() { System.out.println(name + " runs at " + getSpeed() + " km/h"); }
    
    @Override
    public int getSpeed() { return 5; }  // ducks run slower
}

public class Airplane implements Flyable {
    String flightNumber;
    int capacity;
    
    Airplane(String flightNumber, int capacity) {
        this.flightNumber = flightNumber;
        this.capacity = capacity;
    }
    
    @Override public void takeOff() { System.out.println("Flight " + flightNumber + " takes off!"); }
    @Override public void land() { System.out.println("Flight " + flightNumber + " lands!"); }
    @Override public void fly(int altitude) {
        System.out.printf("Flight %s cruising at %dm (capacity: %d)%n",
            flightNumber, altitude, capacity);
    }
}

public class Fish implements Swimmable {
    String species;
    Fish(String species) { this.species = species; }
    
    @Override
    public void swim() { System.out.println(species + " swims in the ocean!"); }
}

public class InterfaceDemo {
    static void makeItFly(Flyable f) {
        f.takeOff();
        f.fly(1000);
        f.land();
    }
    
    public static void main(String[] args) {
        Duck duck = new Duck("Donald");
        Airplane plane = new Airplane("TG123", 300);
        Fish fish = new Fish("Nemo");
        
        System.out.println("=== Flying ===");
        makeItFly(duck);
        makeItFly(plane);
        
        System.out.println("\n=== Swimming ===");
        duck.swim();
        fish.swim();
        
        System.out.println("\n=== Default methods ===");
        duck.describeAbility();
        duck.float_();
        duck.run();
        
        // Interface type variable
        Flyable[] flyers = {duck, plane};
        Swimmable[] swimmers = {duck, fish};
        
        System.out.println("\n=== All flyers ===");
        for (Flyable f : flyers) f.describeAbility();
    }
}
```

---

## 13.3 Functional Interfaces (Java 8+)

```java
import java.util.*;
import java.util.function.*;

public class FunctionalInterfaces {
    
    // Custom functional interface
    @FunctionalInterface
    interface MathOperation {
        double operate(double a, double b);
    }
    
    @FunctionalInterface
    interface StringTransformer {
        String transform(String input);
    }
    
    @FunctionalInterface
    interface Validator<T> {
        boolean validate(T value);
        
        default Validator<T> and(Validator<T> other) {
            return value -> this.validate(value) && other.validate(value);
        }
        
        default Validator<T> or(Validator<T> other) {
            return value -> this.validate(value) || other.validate(value);
        }
    }
    
    static double calculate(double a, double b, MathOperation op) {
        return op.operate(a, b);
    }
    
    static String process(String s, StringTransformer t) {
        return t.transform(s);
    }
    
    public static void main(String[] args) {
        // Lambda expressions as functional interfaces
        MathOperation add = (a, b) -> a + b;
        MathOperation multiply = (a, b) -> a * b;
        MathOperation power = (a, b) -> Math.pow(a, b);
        
        System.out.println("5 + 3 = " + calculate(5, 3, add));
        System.out.println("5 * 3 = " + calculate(5, 3, multiply));
        System.out.println("2^10 = " + calculate(2, 10, power));
        
        StringTransformer reverse = s -> new StringBuilder(s).reverse().toString();
        StringTransformer shout = s -> s.toUpperCase() + "!!!";
        StringTransformer camelCase = s -> {
            String[] words = s.split(" ");
            StringBuilder sb = new StringBuilder(words[0].toLowerCase());
            for (int i = 1; i < words.length; i++) {
                sb.append(Character.toUpperCase(words[i].charAt(0)));
                sb.append(words[i].substring(1).toLowerCase());
            }
            return sb.toString();
        };
        
        String text = "Hello World";
        System.out.println(process(text, reverse));
        System.out.println(process(text, shout));
        System.out.println(process(text, camelCase));
        
        // Validator composition
        Validator<String> notEmpty = s -> !s.isEmpty();
        Validator<String> minLength = s -> s.length() >= 8;
        Validator<String> hasUpper = s -> s.chars().anyMatch(Character::isUpperCase);
        Validator<String> hasDigit = s -> s.chars().anyMatch(Character::isDigit);
        
        Validator<String> passwordValidator = notEmpty.and(minLength).and(hasUpper).and(hasDigit);
        
        String[] passwords = {"pass", "Password", "Password1", "p@ssword1"};
        for (String pwd : passwords) {
            System.out.printf("%-15s -> %s%n", pwd, 
                passwordValidator.validate(pwd) ? "VALID" : "INVALID");
        }
        
        // Java Built-in functional interfaces
        Function<String, Integer> strlen = String::length;
        Predicate<Integer> isEven = n -> n % 2 == 0;
        Consumer<String> printer = System.out::println;
        Supplier<List<String>> listFactory = ArrayList::new;
        BiFunction<Integer, Integer, Integer> max = Math::max;
        
        System.out.println("\n=== Built-in Interfaces ===");
        System.out.println("Length of 'Hello': " + strlen.apply("Hello"));
        System.out.println("Is 4 even: " + isEven.test(4));
        printer.accept("Printed via Consumer");
        List<String> newList = listFactory.get();
        System.out.println("New list: " + newList);
        System.out.println("Max(5,8): " + max.apply(5, 8));
    }
}
```

---

## 13.4 Abstract Class vs Interface

```java
// Abstract class - เมื่อมี state และ partial implementation
public abstract class AbstractAnimal {
    protected String name;      // state
    protected int age;          // state
    private static int count;   // class-level state
    
    AbstractAnimal(String name, int age) {
        this.name = name;
        this.age = age;
        count++;
    }
    
    // Abstract methods
    abstract String getSound();
    abstract String getMovementType();
    
    // Concrete methods with shared logic
    void sleep() {
        System.out.println(name + " is sleeping...");
    }
    
    void eat(String food) {
        System.out.printf("%s eats %s%n", name, food);
    }
    
    String describe() {
        return String.format("%s (age %d) - says '%s' and %s",
            name, age, getSound(), getMovementType());
    }
    
    static int getCount() { return count; }
}

// Interface - เมื่อต้องการ contract โดยไม่มี state
public interface Trainable {
    boolean teach(String command);
    boolean execute(String command);
    
    default void showTricks(String[] commands) {
        System.out.println("Showing tricks:");
        for (String cmd : commands) {
            boolean success = execute(cmd);
            System.out.printf("  %s: %s%n", cmd, success ? "✓" : "✗");
        }
    }
}

public interface Petable {
    void pet();
    default void cuddle() {
        pet();
        System.out.println("~ cuddle time ~");
    }
}

// Class can extend one abstract class + implement multiple interfaces
public class TrainedDog extends AbstractAnimal implements Trainable, Petable {
    private Map<String, Runnable> knownCommands = new HashMap<>();
    
    TrainedDog(String name, int age) {
        super(name, age);
    }
    
    @Override
    public String getSound() { return "Woof!"; }
    
    @Override
    public String getMovementType() { return "runs"; }
    
    @Override
    public boolean teach(String command) {
        knownCommands.put(command, () -> System.out.println(name + " does: " + command));
        System.out.println(name + " learned: " + command);
        return true;
    }
    
    @Override
    public boolean execute(String command) {
        Runnable action = knownCommands.get(command);
        if (action != null) { action.run(); return true; }
        System.out.println(name + " doesn't know: " + command);
        return false;
    }
    
    @Override
    public void pet() { System.out.println("Petting " + name + "... tail wag!"); }
}

public class AbstractVsInterface {
    public static void main(String[] args) {
        TrainedDog rex = new TrainedDog("Rex", 3);
        
        System.out.println(rex.describe());
        rex.eat("kibble");
        rex.sleep();
        
        rex.teach("sit");
        rex.teach("shake");
        rex.teach("roll over");
        
        rex.showTricks(new String[]{"sit", "shake", "fetch", "roll over"});
        rex.cuddle();
        
        System.out.println("Total animals: " + AbstractAnimal.getCount());
    }
}
```

---

## 13.5 Interface Default Methods และ Diamond Problem

```java
public interface A {
    default void hello() { System.out.println("Hello from A"); }
}

public interface B extends A {
    @Override
    default void hello() { System.out.println("Hello from B"); }
}

public interface C extends A {
    @Override
    default void hello() { System.out.println("Hello from C"); }
}

// D must override hello() because B and C both provide implementations
public class D implements B, C {
    @Override
    public void hello() {
        B.super.hello();  // Choose B's implementation
        System.out.println("Hello from D");
    }
}

public class E implements B {
    // No override needed - B's hello() is used
}

public class DiamondDemo {
    public static void main(String[] args) {
        D d = new D();
        d.hello();  // B's hello then D's
        
        E e = new E();
        e.hello();  // B's hello
        
        // Interface reference
        A a = new D();
        a.hello();  // Same: B then D
    }
}
```

---

## 13.6 Comparable and Comparator (Built-in Interfaces)

```java
import java.util.*;

public class ComparableDemo {
    
    // Comparable: natural ordering (implement in the class itself)
    static class Student implements Comparable<Student> {
        String name;
        double gpa;
        int year;
        
        Student(String name, double gpa, int year) {
            this.name = name;
            this.gpa = gpa;
            this.year = year;
        }
        
        @Override
        public int compareTo(Student other) {
            // Natural ordering: by GPA descending, then name ascending
            int gpaCompare = Double.compare(other.gpa, this.gpa);
            if (gpaCompare != 0) return gpaCompare;
            return this.name.compareTo(other.name);
        }
        
        @Override
        public String toString() {
            return String.format("%-15s GPA:%.2f Year:%d", name, gpa, year);
        }
    }
    
    public static void main(String[] args) {
        List<Student> students = new ArrayList<>(Arrays.asList(
            new Student("Alice",   3.9, 3),
            new Student("Bob",     3.5, 2),
            new Student("Charlie", 3.9, 4),
            new Student("Diana",   3.7, 1),
            new Student("Eve",     3.5, 3)
        ));
        
        System.out.println("=== Natural order (by GPA desc) ===");
        Collections.sort(students);
        students.forEach(System.out::println);
        
        // Comparator: custom ordering (external, flexible)
        Comparator<Student> byYear = Comparator.comparingInt(s -> s.year);
        Comparator<Student> byName = Comparator.comparing(s -> s.name);
        Comparator<Student> byYearThenName = byYear.thenComparing(byName);
        
        System.out.println("\n=== By year then name ===");
        students.sort(byYearThenName);
        students.forEach(System.out::println);
        
        System.out.println("\n=== By GPA ascending ===");
        students.sort(Comparator.comparingDouble((Student s) -> s.gpa));
        students.forEach(System.out::println);
        
        // TreeSet with natural ordering
        TreeSet<Student> sortedSet = new TreeSet<>(students);
        System.out.println("\n=== TreeSet (natural order) ===");
        sortedSet.forEach(System.out::println);
        
        // Min/Max
        Student topStudent = Collections.min(students);  // min by natural order = highest GPA
        System.out.println("\nTop student: " + topStudent);
    }
}
```

---

## 13.7 Iterable Interface

```java
import java.util.Iterator;

public class CustomCollection {
    
    static class NumberRange implements Iterable<Integer> {
        private final int start;
        private final int end;
        private final int step;
        
        NumberRange(int start, int end, int step) {
            this.start = start;
            this.end = end;
            this.step = step;
        }
        
        @Override
        public Iterator<Integer> iterator() {
            return new Iterator<Integer>() {
                int current = start;
                
                @Override
                public boolean hasNext() {
                    return step > 0 ? current <= end : current >= end;
                }
                
                @Override
                public Integer next() {
                    int value = current;
                    current += step;
                    return value;
                }
            };
        }
        
        // Factory methods
        static NumberRange from(int start, int end) {
            return new NumberRange(start, end, 1);
        }
        
        static NumberRange evens(int start, int end) {
            int s = start % 2 == 0 ? start : start + 1;
            return new NumberRange(s, end, 2);
        }
        
        static NumberRange countdown(int from) {
            return new NumberRange(from, 1, -1);
        }
    }
    
    public static void main(String[] args) {
        // for-each works because NumberRange implements Iterable
        System.out.print("1 to 10: ");
        for (int n : NumberRange.from(1, 10)) System.out.print(n + " ");
        
        System.out.print("\nEvens 1-20: ");
        for (int n : NumberRange.evens(1, 20)) System.out.print(n + " ");
        
        System.out.print("\nCountdown: ");
        for (int n : NumberRange.countdown(5)) System.out.print(n + " ");
        
        System.out.println("\nDone!");
        
        // Sum using for-each
        int sum = 0;
        for (int n : NumberRange.from(1, 100)) sum += n;
        System.out.println("Sum 1-100: " + sum);
    }
}
```

---

## 13.8 โปรแกรมตัวอย่าง: Plugin System

```java
import java.util.*;
import java.util.function.*;

public class PluginSystem {
    
    interface Plugin {
        String getName();
        String getVersion();
        void initialize(Map<String, Object> config);
        void execute(String input, Consumer<String> output);
        default void shutdown() { System.out.println(getName() + " shutting down"); }
    }
    
    interface TransformPlugin extends Plugin {
        String transform(String input);
        
        @Override
        default void execute(String input, Consumer<String> output) {
            output.accept(transform(input));
        }
    }
    
    static class UpperCasePlugin implements TransformPlugin {
        @Override public String getName() { return "UpperCase"; }
        @Override public String getVersion() { return "1.0"; }
        @Override public void initialize(Map<String, Object> config) {}
        
        @Override
        public String transform(String input) { return input.toUpperCase(); }
    }
    
    static class WordCountPlugin implements Plugin {
        @Override public String getName() { return "WordCount"; }
        @Override public String getVersion() { return "2.1"; }
        @Override public void initialize(Map<String, Object> config) {}
        
        @Override
        public void execute(String input, Consumer<String> output) {
            String[] words = input.trim().split("\\s+");
            output.accept("Words: " + words.length + ", Chars: " + input.length());
        }
    }
    
    static class ReversePlugin implements TransformPlugin {
        @Override public String getName() { return "Reverse"; }
        @Override public String getVersion() { return "1.0"; }
        @Override public void initialize(Map<String, Object> config) {}
        
        @Override
        public String transform(String input) {
            return new StringBuilder(input).reverse().toString();
        }
    }
    
    static class PluginManager {
        private final Map<String, Plugin> plugins = new LinkedHashMap<>();
        
        void register(Plugin p) {
            p.initialize(new HashMap<>());
            plugins.put(p.getName(), p);
            System.out.printf("Registered: %s v%s%n", p.getName(), p.getVersion());
        }
        
        void execute(String pluginName, String input) {
            Plugin p = plugins.get(pluginName);
            if (p == null) { System.out.println("Plugin not found: " + pluginName); return; }
            System.out.printf("[%s] Input: %s%n", pluginName, input);
            p.execute(input, result -> System.out.printf("[%s] Output: %s%n", pluginName, result));
        }
        
        void executeAll(String input) {
            System.out.println("\n=== Running all plugins on: " + input + " ===");
            plugins.values().forEach(p -> {
                p.execute(input, result -> System.out.printf("  [%s] -> %s%n", p.getName(), result));
            });
        }
        
        void listPlugins() {
            System.out.println("\n=== Installed Plugins ===");
            plugins.values().forEach(p ->
                System.out.printf("  %-15s v%s%n", p.getName(), p.getVersion()));
        }
    }
    
    public static void main(String[] args) {
        PluginManager manager = new PluginManager();
        manager.register(new UpperCasePlugin());
        manager.register(new WordCountPlugin());
        manager.register(new ReversePlugin());
        
        manager.listPlugins();
        
        String text = "Hello World from Java";
        manager.executeAll(text);
        
        manager.execute("UpperCase", "hello world");
        manager.execute("Reverse", "abcde");
        manager.execute("InvalidPlugin", "test");
    }
}
```

---

## 13.9 สรุป Part 13

ในบทนี้คุณได้เรียนรู้:

✅ Interface การประกาศและใช้งาน  
✅ Multiple Interface Implementation  
✅ Functional Interfaces และ Lambda  
✅ Default Methods และ Diamond Problem  
✅ Abstract Class vs Interface  
✅ Comparable และ Comparator  
✅ Iterable Interface  

---

*[← Part 12: Polymorphism](./part-12-oop-polymorphism.md) | [Part 14: Exception Handling →](./part-14-exceptions.md)*
