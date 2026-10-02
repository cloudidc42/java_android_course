# Part 18: Lambda Expressions
## หลักสูตร Java & Android Development - ระดับ Intermediate

---

## 18.1 Lambda คืออะไร?

Lambda expression คือ anonymous function ที่เขียนได้กระชับ เหมาะสำหรับ functional interfaces

```java
// Anonymous class (ก่อน Java 8)
Runnable old = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello");
    }
};

// Lambda (Java 8+)
Runnable lambda = () -> System.out.println("Hello");

// Syntax: (parameters) -> body
// Single statement:
Runnable r1 = () -> System.out.println("Hi");
// Multiple statements:
Runnable r2 = () -> {
    String msg = "Hello World";
    System.out.println(msg);
};
```

---

## 18.2 Lambda Syntax

```java
import java.util.*;
import java.util.function.*;

public class LambdaSyntax {
    
    @FunctionalInterface
    interface StringPrinter { void print(String s); }
    
    @FunctionalInterface
    interface Calculator { int calculate(int a, int b); }
    
    @FunctionalInterface
    interface Transformer { String transform(String s); }
    
    public static void main(String[] args) {
        // No parameters
        Runnable noParam = () -> System.out.println("No params");
        noParam.run();
        
        // One parameter (parens optional)
        StringPrinter printer = s -> System.out.println(">>> " + s);
        printer.print("Hello Lambda");
        
        // Two parameters
        Calculator add = (a, b) -> a + b;
        Calculator multiply = (a, b) -> a * b;
        System.out.println("3+4 = " + add.calculate(3, 4));
        System.out.println("3*4 = " + multiply.calculate(3, 4));
        
        // Multi-line body
        Calculator complex = (a, b) -> {
            int sum = a + b;
            int product = a * b;
            return sum + product;
        };
        System.out.println("Complex(3,4) = " + complex.calculate(3, 4));
        
        // With explicit types
        Calculator divide = (int a, int b) -> b != 0 ? a / b : 0;
        System.out.println("10/3 = " + divide.calculate(10, 3));
        System.out.println("10/0 = " + divide.calculate(10, 0));
        
        // Returning lambda from method
        Transformer makeUpperCase = s -> s.toUpperCase();
        Transformer addExcl = s -> s + "!";
        System.out.println(makeUpperCase.transform("hello"));
        System.out.println(addExcl.transform("wow"));
    }
}
```

---

## 18.3 Method References

```java
import java.util.*;
import java.util.function.*;
import java.util.stream.*;

public class MethodReferences {
    
    static class StringUtils {
        static String reverse(String s) { return new StringBuilder(s).reverse().toString(); }
        static boolean isLong(String s) { return s.length() > 5; }
        static String trim(String s) { return s.strip(); }
    }
    
    static class Printer {
        void print(String s) { System.out.println("[PRINT] " + s); }
        void printUpperCase(String s) { System.out.println(s.toUpperCase()); }
    }
    
    static void process(String value, Consumer<String> consumer) {
        consumer.accept(value);
    }
    
    public static void main(String[] args) {
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "Dave", "Eve");
        
        // 1. Static method reference: ClassName::staticMethod
        Function<String, String> reverser = StringUtils::reverse;
        System.out.println(reverser.apply("Hello"));  // olleH
        
        names.stream()
            .filter(StringUtils::isLong)  // static method ref as Predicate
            .map(StringUtils::reverse)
            .forEach(System.out::println);
        
        // 2. Instance method reference on instance: instance::method
        Printer myPrinter = new Printer();
        Consumer<String> printFunc = myPrinter::print;
        printFunc.accept("Hello");
        
        names.forEach(myPrinter::printUpperCase);
        
        // 3. Instance method reference on type: ClassName::instanceMethod
        Function<String, String> upper = String::toUpperCase;
        Function<String, Integer> len = String::length;
        Predicate<String> empty = String::isEmpty;
        
        names.stream()
            .map(String::toUpperCase)        // type method ref
            .sorted(String::compareTo)       // type method ref for comparator
            .forEach(System.out::println);
        
        // 4. Constructor reference: ClassName::new
        Supplier<ArrayList<String>> listFactory = ArrayList::new;
        ArrayList<String> list = listFactory.get();
        list.add("item");
        System.out.println("From factory: " + list);
        
        Function<String, StringBuilder> sbFactory = StringBuilder::new;
        StringBuilder sb = sbFactory.apply("Hello");
        System.out.println("StringBuilder: " + sb);
        
        // Comparison with lambda
        // Lambda:           names.forEach(s -> System.out.println(s));
        // Method reference: names.forEach(System.out::println);
        
        System.out.println("\n=== All names ===");
        names.forEach(System.out::println);
    }
}
```

---

## 18.4 Built-in Functional Interfaces

```java
import java.util.*;
import java.util.function.*;

public class FunctionalInterfacesBuiltIn {
    
    public static void main(String[] args) {
        // --- Function<T, R>: T -> R ---
        Function<String, Integer> toLength = String::length;
        Function<Integer, String> toHex = Integer::toHexString;
        Function<String, String> trim = String::trim;
        
        // Compose: f.andThen(g) = g(f(x))
        Function<String, String> trimAndUpper = trim.andThen(String::toUpperCase);
        System.out.println(trimAndUpper.apply("  hello  "));  // HELLO
        
        // compose: f.compose(g) = f(g(x))
        Function<Integer, Integer> times2 = x -> x * 2;
        Function<Integer, Integer> plus3 = x -> x + 3;
        Function<Integer, Integer> times2ThenPlus3 = plus3.compose(times2);  // plus3(times2(x))
        System.out.println(times2ThenPlus3.apply(5));  // 13: (5*2)+3
        
        // --- Predicate<T>: T -> boolean ---
        Predicate<String> notEmpty = s -> !s.isEmpty();
        Predicate<String> hasAt = s -> s.contains("@");
        Predicate<String> longEnough = s -> s.length() >= 5;
        
        // Combine predicates
        Predicate<String> validEmail = notEmpty.and(hasAt).and(longEnough);
        
        String[] emails = {"", "a@b", "alice@example.com", "invalid"};
        for (String email : emails) {
            System.out.printf("%-20s -> %s%n", email, validEmail.test(email) ? "VALID" : "INVALID");
        }
        
        Predicate<Integer> isEven = n -> n % 2 == 0;
        Predicate<Integer> isNeg = n -> n < 0;
        Predicate<Integer> isOddOrNeg = isEven.negate().or(isNeg);
        
        for (int n : new int[]{-3, -2, 0, 1, 2, 3}) {
            System.out.printf("%3d oddOrNeg: %b%n", n, isOddOrNeg.test(n));
        }
        
        // --- Consumer<T>: T -> void ---
        Consumer<String> print = System.out::println;
        Consumer<String> printUpper = s -> System.out.println(s.toUpperCase());
        Consumer<String> both = print.andThen(printUpper);  // run both
        
        both.accept("hello");
        
        // --- Supplier<T>: () -> T ---
        Supplier<Double> random = Math::random;
        Supplier<List<String>> emptyList = ArrayList::new;
        Supplier<String> greeting = () -> "Hello at " + new java.util.Date();
        
        System.out.println("Random: " + random.get());
        System.out.println("Greeting: " + greeting.get());
        
        // --- BiFunction<T, U, R>: (T, U) -> R ---
        BiFunction<String, Integer, String> repeat = (s, n) -> s.repeat(n);
        System.out.println(repeat.apply("ab", 3));  // ababab
        
        // --- UnaryOperator<T>: T -> T ---
        UnaryOperator<String> addExcl = s -> s + "!";
        UnaryOperator<Integer> square = n -> n * n;
        System.out.println(addExcl.apply("wow"));
        System.out.println(square.apply(7));
        
        // --- BinaryOperator<T>: (T, T) -> T ---
        BinaryOperator<Integer> sum = Integer::sum;
        BinaryOperator<String> concat = String::concat;
        System.out.println(sum.apply(10, 20));
        System.out.println(concat.apply("Hello ", "World"));
    }
}
```

---

## 18.5 Lambda กับ Collections

```java
import java.util.*;
import java.util.stream.*;
import java.util.function.*;

public class LambdaWithCollections {
    
    record Person(String name, int age, String city) {}
    
    public static void main(String[] args) {
        List<Person> people = Arrays.asList(
            new Person("Alice", 30, "Bangkok"),
            new Person("Bob",   25, "Chiang Mai"),
            new Person("Charlie", 35, "Bangkok"),
            new Person("Diana", 28, "Phuket"),
            new Person("Eve",   32, "Bangkok"),
            new Person("Frank", 22, "Chiang Mai")
        );
        
        // --- Sort with lambda ---
        List<Person> byAge = new ArrayList<>(people);
        byAge.sort((a, b) -> Integer.compare(a.age(), b.age()));
        System.out.println("By age:");
        byAge.forEach(p -> System.out.printf("  %s (%d)%n", p.name(), p.age()));
        
        // Method reference version:
        byAge.sort(Comparator.comparing(Person::age));
        
        // --- removeIf ---
        List<Integer> nums = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10));
        nums.removeIf(n -> n % 3 == 0);
        System.out.println("\nAfter removeIf (% 3): " + nums);
        
        // --- replaceAll ---
        List<String> words = new ArrayList<>(Arrays.asList("hello", "world", "java"));
        words.replaceAll(String::toUpperCase);
        System.out.println("After replaceAll: " + words);
        
        // --- Map operations ---
        Map<String, Integer> scores = new HashMap<>(Map.of(
            "Alice", 85, "Bob", 72, "Charlie", 91
        ));
        
        scores.replaceAll((k, v) -> v + 5);  // add 5 bonus
        System.out.println("After bonus: " + new TreeMap<>(scores));
        
        scores.forEach((name, score) ->
            System.out.printf("  %s: %d (%s)%n", name, score,
                score >= 90 ? "A" : score >= 80 ? "B" : "C"));
        
        // --- Collectors with lambda ---
        Map<String, List<Person>> byCity = people.stream()
            .collect(Collectors.groupingBy(Person::city));
        
        System.out.println("\nBy city:");
        byCity.entrySet().stream()
            .sorted(Map.Entry.comparingByKey())
            .forEach(e -> {
                System.out.println("  " + e.getKey() + ":");
                e.getValue().forEach(p -> System.out.printf("    %s (age %d)%n", p.name(), p.age()));
            });
        
        // Statistics
        IntSummaryStatistics stats = people.stream()
            .mapToInt(Person::age)
            .summaryStatistics();
        System.out.printf("\nAge - min:%d max:%d avg:%.1f%n",
            stats.getMin(), stats.getMax(), stats.getAverage());
    }
}
```

---

## 18.6 Closure และ Variable Capture

```java
public class LambdaCapture {
    
    static Runnable createCounter() {
        // Lambda captures effectively final variable
        // (variable that doesn't change after assignment)
        int start = 0;  // effectively final
        return () -> System.out.println("Count: " + start);
    }
    
    static java.util.function.Supplier<Integer> makeAdder(int n) {
        return () -> n + 10;  // captures n
    }
    
    public static void main(String[] args) {
        // Effectively final
        String greeting = "Hello";
        Runnable r = () -> System.out.println(greeting + " World");
        r.run();
        
        // Cannot reassign captured variable:
        // greeting = "Hi";  // ERROR: not effectively final after this
        
        // Use array for mutable state
        int[] count = {0};  // array is effectively final (reference doesn't change)
        Runnable counter = () -> {
            count[0]++;
            System.out.println("Count: " + count[0]);
        };
        counter.run();
        counter.run();
        counter.run();
        
        // Closure
        var add5 = makeAdder(5);
        var add10 = makeAdder(10);
        System.out.println("add5: " + add5.get());   // 15
        System.out.println("add10: " + add10.get()); // 20
        
        // Lambda capturing instance field
        new java.util.ArrayList<String>() {{
            add("Hello");
            // 'this' refers to ArrayList here
            forEach(s -> System.out.println(this.size() + ": " + s));
        }};
    }
}
```

---

## 18.7 โปรแกรมตัวอย่าง: Event System

```java
import java.util.*;
import java.util.function.*;

public class EventSystem {
    
    interface EventHandler<T> {
        void handle(T event);
    }
    
    static class EventBus<T> {
        private final Map<String, List<EventHandler<T>>> handlers = new HashMap<>();
        
        void subscribe(String eventType, EventHandler<T> handler) {
            handlers.computeIfAbsent(eventType, k -> new ArrayList<>()).add(handler);
            System.out.println("Subscribed to: " + eventType);
        }
        
        void publish(String eventType, T data) {
            System.out.println("\n[EVENT] " + eventType + ": " + data);
            List<EventHandler<T>> list = handlers.getOrDefault(eventType, Collections.emptyList());
            list.forEach(h -> h.handle(data));
        }
        
        void unsubscribe(String eventType) {
            handlers.remove(eventType);
        }
    }
    
    record UserEvent(String userId, String action, String details) {
        @Override public String toString() {
            return String.format("User[%s] %s: %s", userId, action, details);
        }
    }
    
    public static void main(String[] args) {
        EventBus<UserEvent> bus = new EventBus<>();
        
        // Subscribe with lambdas
        bus.subscribe("LOGIN", event -> {
            System.out.println("  [Logger] User logged in: " + event.userId());
        });
        
        bus.subscribe("LOGIN", event -> {
            System.out.println("  [Security] Login from: " + event.details());
        });
        
        bus.subscribe("PURCHASE", event -> {
            System.out.println("  [Inventory] Update stock for: " + event.details());
        });
        
        bus.subscribe("PURCHASE", event -> {
            System.out.printf("  [Email] Send receipt to %s%n", event.userId());
        });
        
        bus.subscribe("LOGOUT", event ->
            System.out.println("  [Session] Clear session for: " + event.userId()));
        
        // Publish events
        bus.publish("LOGIN", new UserEvent("U001", "login", "IP:192.168.1.1"));
        bus.publish("PURCHASE", new UserEvent("U001", "purchase", "Product:iPhone15"));
        bus.publish("LOGOUT", new UserEvent("U001", "logout", ""));
        bus.publish("UNKNOWN", new UserEvent("U002", "unknown", "test"));
    }
}
```

---

## 18.8 สรุป Part 18

ในบทนี้คุณได้เรียนรู้:

✅ Lambda expression syntax  
✅ Method references (4 types)  
✅ Functional interfaces (Function, Predicate, Consumer, Supplier)  
✅ Lambda composition  
✅ Lambda กับ Collections  
✅ Variable capture (closure)  

---

*[← Part 17: File I/O](./part-17-file-io.md) | [Part 19: Stream API →](./part-19-streams.md)*
