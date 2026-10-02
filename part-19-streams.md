# Part 19: Stream API
## หลักสูตร Java & Android Development - ระดับ Intermediate

---

## 19.1 Stream คืออะไร?

Stream คือ pipeline ของ operations บน sequence of elements

```
Source -> [Intermediate ops] -> Terminal op
List   -> filter -> map -> sorted -> collect
```

**ลักษณะสำคัญ:**
- Lazy evaluation (ทำงานเมื่อ terminal operation ถูกเรียก)
- ใช้ครั้งเดียว (consume once)
- ไม่แก้ไข source
- รองรับ parallel processing

---

## 19.2 Creating Streams

```java
import java.util.*;
import java.util.stream.*;
import java.util.Arrays;

public class CreatingStreams {
    public static void main(String[] args) {
        // From Collection
        List<String> list = Arrays.asList("a", "b", "c");
        Stream<String> fromList = list.stream();
        
        // From Array
        String[] arr = {"x", "y", "z"};
        Stream<String> fromArr = Arrays.stream(arr);
        
        // Stream.of()
        Stream<Integer> ofInts = Stream.of(1, 2, 3, 4, 5);
        
        // Stream.iterate() - infinite stream
        Stream<Integer> evens = Stream.iterate(0, n -> n + 2);
        evens.limit(5).forEach(n -> System.out.print(n + " "));
        System.out.println();
        
        // Stream.generate() - infinite stream
        Stream<Double> randoms = Stream.generate(Math::random);
        randoms.limit(3).forEach(d -> System.out.printf("%.3f ", d));
        System.out.println();
        
        // IntStream, LongStream, DoubleStream (primitives)
        IntStream range = IntStream.range(1, 6);        // 1,2,3,4,5
        IntStream rangeClosed = IntStream.rangeClosed(1, 5); // 1,2,3,4,5
        range.forEach(n -> System.out.print(n + " "));
        System.out.println();
        
        // From String
        IntStream chars = "Hello".chars();
        chars.forEach(c -> System.out.print((char) c + " "));
        System.out.println();
        
        // Empty stream
        Stream<String> empty = Stream.empty();
        System.out.println("Empty: " + empty.count());
        
        // Concatenate streams
        Stream<Integer> s1 = Stream.of(1, 2, 3);
        Stream<Integer> s2 = Stream.of(4, 5, 6);
        Stream<Integer> combined = Stream.concat(s1, s2);
        combined.forEach(n -> System.out.print(n + " "));
        System.out.println();
    }
}
```

---

## 19.3 Intermediate Operations

```java
import java.util.*;
import java.util.stream.*;

public class IntermediateOps {
    
    record Person(String name, int age, String city, double salary) {}
    
    public static void main(String[] args) {
        List<Person> people = Arrays.asList(
            new Person("Alice", 30, "Bangkok", 50000),
            new Person("Bob",   25, "Bangkok", 35000),
            new Person("Charlie", 35, "CM",   60000),
            new Person("Diana", 28, "Phuket", 45000),
            new Person("Eve",   32, "Bangkok", 55000),
            new Person("Frank", 22, "CM",      28000),
            new Person("Grace", 40, "Bangkok", 80000),
            new Person("Henry", 27, "Phuket",  38000)
        );
        
        // --- filter ---
        System.out.println("=== filter (Bangkok) ===");
        people.stream()
            .filter(p -> p.city().equals("Bangkok"))
            .forEach(p -> System.out.println("  " + p.name()));
        
        // --- map ---
        System.out.println("\n=== map (names) ===");
        people.stream()
            .map(Person::name)
            .forEach(n -> System.out.print(n + " "));
        System.out.println();
        
        // --- mapToInt / mapToDouble ---
        int totalAge = people.stream()
            .mapToInt(Person::age)
            .sum();
        System.out.println("\nTotal age: " + totalAge);
        
        double avgSalary = people.stream()
            .mapToDouble(Person::salary)
            .average()
            .orElse(0);
        System.out.printf("Avg salary: %.0f%n", avgSalary);
        
        // --- flatMap ---
        List<List<Integer>> nested = Arrays.asList(
            Arrays.asList(1, 2, 3),
            Arrays.asList(4, 5),
            Arrays.asList(6, 7, 8, 9)
        );
        List<Integer> flat = nested.stream()
            .flatMap(Collection::stream)
            .collect(Collectors.toList());
        System.out.println("\nFlatted: " + flat);
        
        // flatMap with strings
        List<String> sentences = Arrays.asList("hello world", "java stream", "flatmap demo");
        List<String> words = sentences.stream()
            .flatMap(s -> Arrays.stream(s.split(" ")))
            .distinct()
            .sorted()
            .collect(Collectors.toList());
        System.out.println("Unique words: " + words);
        
        // --- sorted ---
        System.out.println("\n=== sorted by salary desc ===");
        people.stream()
            .sorted(Comparator.comparingDouble(Person::salary).reversed())
            .limit(3)
            .forEach(p -> System.out.printf("  %-10s %.0f%n", p.name(), p.salary()));
        
        // --- distinct ---
        List<Integer> nums = Arrays.asList(1, 2, 2, 3, 3, 3, 4, 1, 5);
        List<Integer> unique = nums.stream().distinct().collect(Collectors.toList());
        System.out.println("\nDistinct: " + unique);
        
        // --- limit / skip ---
        System.out.println("\n=== limit(3) ===");
        people.stream().limit(3).map(Person::name).forEach(System.out::println);
        
        System.out.println("=== skip(5) ===");
        people.stream().skip(5).map(Person::name).forEach(System.out::println);
        
        // --- peek (debug) ---
        System.out.println("\n=== peek (debug pipeline) ===");
        long count = people.stream()
            .filter(p -> p.age() > 28)
            .peek(p -> System.out.println("  After filter: " + p.name()))
            .map(Person::name)
            .peek(n -> System.out.println("  After map: " + n))
            .count();
        System.out.println("Count: " + count);
    }
}
```

---

## 19.4 Terminal Operations

```java
import java.util.*;
import java.util.stream.*;

public class TerminalOps {
    
    public static void main(String[] args) {
        List<Integer> nums = Arrays.asList(5, 2, 8, 1, 9, 3, 7, 4, 6, 10);
        
        // --- count ---
        long count = nums.stream().filter(n -> n > 5).count();
        System.out.println("Count > 5: " + count);
        
        // --- sum, average, min, max ---
        int sum = nums.stream().mapToInt(Integer::intValue).sum();
        OptionalDouble avg = nums.stream().mapToInt(Integer::intValue).average();
        OptionalInt max = nums.stream().mapToInt(Integer::intValue).max();
        OptionalInt min = nums.stream().mapToInt(Integer::intValue).min();
        
        System.out.printf("Sum:%d Avg:%.1f Max:%d Min:%d%n",
            sum, avg.getAsDouble(), max.getAsInt(), min.getAsInt());
        
        // --- findFirst / findAny ---
        Optional<Integer> first = nums.stream().filter(n -> n > 7).findFirst();
        first.ifPresent(n -> System.out.println("First > 7: " + n));
        
        // --- anyMatch / allMatch / noneMatch ---
        System.out.println("Any > 9: " + nums.stream().anyMatch(n -> n > 9));
        System.out.println("All > 0: " + nums.stream().allMatch(n -> n > 0));
        System.out.println("None < 0: " + nums.stream().noneMatch(n -> n < 0));
        
        // --- reduce ---
        int product = nums.stream().reduce(1, (a, b) -> a * b);
        System.out.println("Product: " + product);
        
        // reduce with identity
        String joined = Stream.of("a", "b", "c", "d")
            .reduce("", (a, b) -> a + b);
        System.out.println("Joined: " + joined);
        
        // --- forEach ---
        nums.stream()
            .filter(n -> n % 2 == 0)
            .sorted()
            .forEach(n -> System.out.print(n + " "));
        System.out.println();
        
        // --- toArray ---
        Integer[] array = nums.stream()
            .filter(n -> n > 5)
            .sorted()
            .toArray(Integer[]::new);
        System.out.println("Array: " + Arrays.toString(array));
        
        // --- collect ---
        List<Integer> evenList = nums.stream()
            .filter(n -> n % 2 == 0)
            .collect(Collectors.toList());
        System.out.println("Evens: " + evenList);
        
        Set<Integer> evenSet = nums.stream()
            .filter(n -> n % 2 == 0)
            .collect(Collectors.toSet());
        System.out.println("Even set: " + new TreeSet<>(evenSet));
        
        // joining
        String csv = Stream.of("apple", "banana", "cherry")
            .collect(Collectors.joining(", ", "[", "]"));
        System.out.println("CSV: " + csv);
        
        // summaryStatistics
        IntSummaryStatistics stats = nums.stream()
            .mapToInt(Integer::intValue)
            .summaryStatistics();
        System.out.println("Stats: " + stats);
    }
}
```

---

## 19.5 Collectors

```java
import java.util.*;
import java.util.stream.*;

public class CollectorsDemo {
    
    record Employee(String name, String dept, double salary, int year) {}
    
    public static void main(String[] args) {
        List<Employee> employees = Arrays.asList(
            new Employee("Alice",   "IT",    65000, 3),
            new Employee("Bob",     "Sales", 45000, 2),
            new Employee("Charlie", "IT",    72000, 5),
            new Employee("Diana",   "HR",    55000, 4),
            new Employee("Eve",     "Sales", 48000, 1),
            new Employee("Frank",   "IT",    58000, 2),
            new Employee("Grace",   "HR",    60000, 3),
            new Employee("Henry",   "Sales", 52000, 4)
        );
        
        // --- toList, toSet, toMap ---
        List<String> names = employees.stream()
            .map(Employee::name)
            .collect(Collectors.toList());
        
        Map<String, Double> salaryByName = employees.stream()
            .collect(Collectors.toMap(Employee::name, Employee::salary));
        System.out.println("Salary of Alice: " + salaryByName.get("Alice"));
        
        // --- groupingBy ---
        Map<String, List<Employee>> byDept = employees.stream()
            .collect(Collectors.groupingBy(Employee::dept));
        
        byDept.forEach((dept, emps) -> {
            System.out.printf("\n%s (%d employees):%n", dept, emps.size());
            emps.forEach(e -> System.out.printf("  %-10s %.0f%n", e.name(), e.salary()));
        });
        
        // --- groupingBy with downstream collector ---
        Map<String, Double> avgSalaryByDept = employees.stream()
            .collect(Collectors.groupingBy(Employee::dept,
                Collectors.averagingDouble(Employee::salary)));
        
        System.out.println("\n=== Avg salary by dept ===");
        avgSalaryByDept.entrySet().stream()
            .sorted(Map.Entry.<String, Double>comparingByValue().reversed())
            .forEach(e -> System.out.printf("  %-8s %.0f%n", e.getKey(), e.getValue()));
        
        // --- counting ---
        Map<String, Long> countByDept = employees.stream()
            .collect(Collectors.groupingBy(Employee::dept, Collectors.counting()));
        System.out.println("\nCount by dept: " + countByDept);
        
        // --- partitioningBy (split into 2 groups) ---
        Map<Boolean, List<Employee>> highPaid = employees.stream()
            .collect(Collectors.partitioningBy(e -> e.salary() >= 60000));
        
        System.out.println("\n=== High paid (>=60k) ===");
        highPaid.get(true).forEach(e -> System.out.println("  " + e.name()));
        System.out.println("=== Low paid (<60k) ===");
        highPaid.get(false).forEach(e -> System.out.println("  " + e.name()));
        
        // --- summarizingDouble ---
        DoubleSummaryStatistics salaryStats = employees.stream()
            .collect(Collectors.summarizingDouble(Employee::salary));
        System.out.printf("\nSalary: min=%.0f max=%.0f avg=%.0f%n",
            salaryStats.getMin(), salaryStats.getMax(), salaryStats.getAverage());
        
        // --- toUnmodifiableList (Java 10+) ---
        List<String> immutableNames = employees.stream()
            .map(Employee::name)
            .collect(Collectors.toUnmodifiableList());
        
        // --- joining ---
        String deptList = employees.stream()
            .map(Employee::dept)
            .distinct()
            .sorted()
            .collect(Collectors.joining(", "));
        System.out.println("Departments: " + deptList);
    }
}
```

---

## 19.6 Optional

```java
import java.util.*;
import java.util.stream.*;

public class OptionalDemo {
    
    static Optional<String> findUserEmail(int userId) {
        Map<Integer, String> db = Map.of(1, "alice@example.com", 2, "bob@example.com");
        return Optional.ofNullable(db.get(userId));
    }
    
    static Optional<String> getDomain(String email) {
        if (email == null || !email.contains("@")) return Optional.empty();
        return Optional.of(email.split("@")[1]);
    }
    
    public static void main(String[] args) {
        // Creating Optional
        Optional<String> present = Optional.of("Hello");
        Optional<String> empty = Optional.empty();
        Optional<String> nullable = Optional.ofNullable(null);
        
        // Checking
        System.out.println("Present: " + present.isPresent());
        System.out.println("Empty: " + empty.isEmpty());
        
        // Getting value safely
        System.out.println("Value: " + present.get());
        System.out.println("OrElse: " + empty.orElse("default"));
        System.out.println("OrElseGet: " + empty.orElseGet(() -> "computed default"));
        
        try {
            empty.orElseThrow(() -> new RuntimeException("No value!"));
        } catch (RuntimeException e) {
            System.out.println("Thrown: " + e.getMessage());
        }
        
        // ifPresent
        present.ifPresent(v -> System.out.println("Got: " + v));
        empty.ifPresent(v -> System.out.println("Won't print"));
        
        // ifPresentOrElse (Java 9+)
        empty.ifPresentOrElse(
            v -> System.out.println("Value: " + v),
            () -> System.out.println("No value present")
        );
        
        // map and flatMap
        Optional<Integer> length = present.map(String::length);
        System.out.println("Length: " + length);
        
        Optional<String> upper = present.map(String::toUpperCase);
        System.out.println("Upper: " + upper);
        
        // filter
        Optional<String> filtered = present.filter(s -> s.startsWith("H"));
        System.out.println("Filtered: " + filtered);
        
        // Chaining
        System.out.println("\n=== Chaining ===");
        for (int id : new int[]{1, 2, 3}) {
            String domain = findUserEmail(id)
                .flatMap(OptionalDemo::getDomain)
                .map(String::toUpperCase)
                .orElse("UNKNOWN");
            System.out.printf("User %d domain: %s%n", id, domain);
        }
        
        // stream() (Java 9+): convert to Stream
        List<Optional<String>> optionals = Arrays.asList(
            Optional.of("a"), Optional.empty(), Optional.of("b"),
            Optional.empty(), Optional.of("c")
        );
        
        List<String> values = optionals.stream()
            .flatMap(Optional::stream)  // filter empty, unwrap present
            .collect(Collectors.toList());
        System.out.println("Values: " + values);
    }
}
```

---

## 19.7 Parallel Streams

```java
import java.util.*;
import java.util.stream.*;

public class ParallelStreams {
    
    static long computeSequential(int n) {
        return LongStream.rangeClosed(1, n).sum();
    }
    
    static long computeParallel(int n) {
        return LongStream.rangeClosed(1, n).parallel().sum();
    }
    
    public static void main(String[] args) {
        int N = 100_000_000;
        
        long start, end;
        
        start = System.currentTimeMillis();
        long seqResult = computeSequential(N);
        end = System.currentTimeMillis();
        System.out.printf("Sequential: %d ms, result: %d%n", end - start, seqResult);
        
        start = System.currentTimeMillis();
        long parResult = computeParallel(N);
        end = System.currentTimeMillis();
        System.out.printf("Parallel:   %d ms, result: %d%n", end - start, parResult);
        
        // Parallel stream from collection
        List<Integer> bigList = IntStream.rangeClosed(1, 1_000_000)
            .boxed()
            .collect(Collectors.toList());
        
        // Sequential
        start = System.currentTimeMillis();
        long seqSum = bigList.stream().mapToLong(Integer::longValue).sum();
        end = System.currentTimeMillis();
        System.out.printf("Sequential list: %d ms%n", end - start);
        
        // Parallel
        start = System.currentTimeMillis();
        long parSum = bigList.parallelStream().mapToLong(Integer::longValue).sum();
        end = System.currentTimeMillis();
        System.out.printf("Parallel list:   %d ms%n", end - start);
        
        System.out.println("Results match: " + (seqSum == parSum));
        
        // WARNING: Parallel streams are NOT always faster
        // Best for: large data, CPU-intensive ops, independent elements
        // Avoid for: small data, I/O, stateful operations, ordered output needed
    }
}
```

---

## 19.8 โปรแกรมตัวอย่าง: Sales Report

```java
import java.util.*;
import java.util.stream.*;
import java.time.*;

public class SalesReport {
    
    record Sale(String id, String product, String category,
                double amount, String region, LocalDate date) {}
    
    static List<Sale> generateData() {
        String[] products = {"iPhone", "Samsung", "MacBook", "Dell XPS", "AirPods", "Sony WH"};
        String[] categories = {"Phone", "Phone", "Laptop", "Laptop", "Accessory", "Accessory"};
        String[] regions = {"North", "South", "East", "West", "Central"};
        Random rand = new Random(42);
        
        List<Sale> sales = new ArrayList<>();
        for (int i = 1; i <= 200; i++) {
            int prodIdx = rand.nextInt(products.length);
            sales.add(new Sale(
                "S" + String.format("%04d", i),
                products[prodIdx],
                categories[prodIdx],
                (rand.nextInt(50) + 1) * 1000.0,
                regions[rand.nextInt(regions.length)],
                LocalDate.of(2024, rand.nextInt(12) + 1, rand.nextInt(28) + 1)
            ));
        }
        return sales;
    }
    
    public static void main(String[] args) {
        List<Sale> sales = generateData();
        
        // Total revenue
        double total = sales.stream().mapToDouble(Sale::amount).sum();
        System.out.printf("Total Revenue: %,.0f%n%n", total);
        
        // Revenue by category
        System.out.println("=== Revenue by Category ===");
        sales.stream()
            .collect(Collectors.groupingBy(Sale::category,
                Collectors.summingDouble(Sale::amount)))
            .entrySet().stream()
            .sorted(Map.Entry.<String, Double>comparingByValue().reversed())
            .forEach(e -> System.out.printf("  %-12s %,.0f%n", e.getKey(), e.getValue()));
        
        // Revenue by region
        System.out.println("\n=== Revenue by Region ===");
        sales.stream()
            .collect(Collectors.groupingBy(Sale::region,
                Collectors.summingDouble(Sale::amount)))
            .entrySet().stream()
            .sorted(Map.Entry.<String, Double>comparingByValue().reversed())
            .forEach(e -> System.out.printf("  %-10s %,.0f%n", e.getKey(), e.getValue()));
        
        // Top 5 products
        System.out.println("\n=== Top 5 Products ===");
        sales.stream()
            .collect(Collectors.groupingBy(Sale::product,
                Collectors.summingDouble(Sale::amount)))
            .entrySet().stream()
            .sorted(Map.Entry.<String, Double>comparingByValue().reversed())
            .limit(5)
            .forEach(e -> System.out.printf("  %-15s %,.0f%n", e.getKey(), e.getValue()));
        
        // Monthly trend
        System.out.println("\n=== Monthly Revenue ===");
        sales.stream()
            .collect(Collectors.groupingBy(s -> s.date().getMonth(),
                Collectors.summingDouble(Sale::amount)))
            .entrySet().stream()
            .sorted(Map.Entry.comparingByKey())
            .forEach(e -> {
                int bars = (int)(e.getValue() / 50000);
                System.out.printf("  %-10s %s %,.0f%n",
                    e.getKey(), "█".repeat(bars), e.getValue());
            });
        
        // High-value sales (>= 40000)
        long highValueCount = sales.stream()
            .filter(s -> s.amount() >= 40000)
            .count();
        System.out.printf("\nHigh-value sales (>=40k): %d (%.1f%%)%n",
            highValueCount, highValueCount * 100.0 / sales.size());
    }
}
```

---

## 19.9 สรุป Part 19

ในบทนี้คุณได้เรียนรู้:

✅ Stream pipeline (Source → Intermediate → Terminal)  
✅ Creating streams  
✅ Intermediate ops: filter, map, flatMap, sorted, distinct  
✅ Terminal ops: collect, reduce, count, findFirst  
✅ Collectors: groupingBy, partitioningBy, joining  
✅ Optional  
✅ Parallel streams  

---

*[← Part 18: Lambda](./part-18-lambda.md) | [Part 20: Java Advanced Features →](./part-20-java-advanced.md)*
