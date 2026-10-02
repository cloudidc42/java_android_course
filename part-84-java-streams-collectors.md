# Part 84: Java Streams & Collectors ขั้นสูง
## หลักสูตร Java & Android Development - ระดับ World-Class

---

## 84.1 Custom Collectors

```java
// Collect into custom result
public class StreamAdvanced {
    
    // Standard collectors review
    List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "Dave", "Eve");
    
    // Grouping
    Map<Integer, List<String>> byLength = names.stream()
        .collect(Collectors.groupingBy(String::length));
    // {3=[Bob, Eve], 4=[Dave], 5=[Alice], 7=[Charlie]}
    
    // Counting per group
    Map<Integer, Long> countByLength = names.stream()
        .collect(Collectors.groupingBy(String::length, Collectors.counting()));
    
    // Partitioning (splits into true/false)
    Map<Boolean, List<String>> partition = names.stream()
        .collect(Collectors.partitioningBy(s -> s.length() > 4));
    
    // Joining
    String joined = names.stream()
        .collect(Collectors.joining(", ", "[", "]"));
    // [Alice, Bob, Charlie, Dave, Eve]
    
    // toMap - careful of duplicates!
    Map<String, Integer> nameLengthMap = names.stream()
        .collect(Collectors.toMap(
            name -> name,
            String::length,
            (existing, replacement) -> existing  // merge function for duplicates
        ));
    
    // Custom collector: sum of squares
    Collector<Integer, int[], Long> sumOfSquares = Collector.of(
        () -> new int[1],                   // supplier
        (arr, n) -> arr[0] += n * n,        // accumulator
        (a, b) -> { a[0] += b[0]; return a; }, // combiner
        arr -> (long) arr[0]                // finisher
    );
    
    long result = IntStream.rangeClosed(1, 5)
        .boxed()
        .collect(sumOfSquares);
    // 1² + 2² + 3² + 4² + 5² = 55
    
    // Downstream collectors
    Map<String, Double> avgPriceByCategory = products.stream()
        .collect(Collectors.groupingBy(
            Product::getCategory,
            Collectors.averagingDouble(Product::getPrice)));
    
    // Statistics
    Map<String, DoubleSummaryStatistics> statsByCategory = products.stream()
        .collect(Collectors.groupingBy(
            Product::getCategory,
            Collectors.summarizingDouble(Product::getPrice)));
    // statsByCategory.get("Electronics").getAverage()
    
    // Nested grouping
    Map<String, Map<Boolean, List<Product>>> grouped = products.stream()
        .collect(Collectors.groupingBy(
            Product::getCategory,
            Collectors.partitioningBy(Product::isInStock)));
}
```

---

## 84.2 FlatMap, Distinct, Sorted

```java
// FlatMap: flatten nested lists
List<List<String>> nested = Arrays.asList(
    Arrays.asList("a", "b", "c"),
    Arrays.asList("d", "e"),
    Arrays.asList("f", "g", "h")
);

List<String> flat = nested.stream()
    .flatMap(Collection::stream)
    .collect(Collectors.toList());
// [a, b, c, d, e, f, g, h]

// Get all unique tags from all products
List<String> allTags = products.stream()
    .flatMap(p -> p.getTags().stream())
    .distinct()
    .sorted()
    .collect(Collectors.toList());

// Sorted with Comparator
List<Product> sorted = products.stream()
    .sorted(Comparator.comparing(Product::getCategory)
        .thenComparing(Comparator.comparingDouble(Product::getPrice).reversed()))
    .collect(Collectors.toList());

// Multi-level sort: category asc, price desc, name asc
Comparator<Product> comparator = Comparator
    .comparing(Product::getCategory)
    .thenComparing(Comparator.comparingDouble(Product::getPrice).reversed())
    .thenComparing(Product::getName);

products.sort(comparator);
```

---

## 84.3 Stream of Primitives

```java
// IntStream, LongStream, DoubleStream (avoid boxing overhead)
int[] numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

IntSummaryStatistics stats = IntStream.of(numbers).summaryStatistics();
System.out.println("Sum: " + stats.getSum());
System.out.println("Avg: " + stats.getAverage());
System.out.println("Min: " + stats.getMin());
System.out.println("Max: " + stats.getMax());

// Range
int sumOdd = IntStream.rangeClosed(1, 100)
    .filter(n -> n % 2 != 0)
    .sum();  // 2500

// Generate
DoubleStream random = DoubleStream.generate(Math::random).limit(5);

// Iterate
IntStream fibonacci = IntStream.iterate(new int[]{0, 1}, f -> new int[]{f[1], f[0] + f[1]})
    .mapToInt(f -> f[0])
    .limit(10);
// 0, 1, 1, 2, 3, 5, 8, 13, 21, 34

// Convert to boxed stream
List<Integer> list = IntStream.range(0, 10).boxed().collect(Collectors.toList());

// mapToInt / mapToObj
OptionalInt maxLength = names.stream()
    .mapToInt(String::length)
    .max();

double[] prices = products.stream()
    .mapToDouble(Product::getPrice)
    .toArray();
```

---

## 84.4 Optional Best Practices

```java
// Optional - avoid NullPointerException
Optional<User> findUser(String id) {
    return Optional.ofNullable(userRepository.getById(id));
}

// Chain Optional operations
String city = findUser("123")
    .map(User::getAddress)
    .map(Address::getCity)
    .orElse("Unknown");

// orElseGet (lazy - only calls if empty)
User user = findUser("123")
    .orElseGet(() -> createDefaultUser());

// orElseThrow
User user2 = findUser("123")
    .orElseThrow(() -> new UserNotFoundException("User 123 not found"));

// filter in Optional
Optional<User> premiumUser = findUser("123")
    .filter(u -> "premium".equals(u.getSubscriptionType()));

// ifPresent
findUser("123").ifPresent(u -> sendWelcomeEmail(u.getEmail()));

// ifPresentOrElse (Java 9+)
findUser("123").ifPresentOrElse(
    u -> sendWelcomeEmail(u.getEmail()),
    () -> Log.d("User", "User not found"));

// flatMap (for nested optionals)
Optional<String> email = findUser("123")
    .flatMap(u -> Optional.ofNullable(u.getVerifiedEmail()));

// stream (Java 9+) - zero or one element
long activeCount = userIds.stream()
    .map(this::findUser)
    .flatMap(Optional::stream)  // unpack non-empty optionals
    .filter(User::isActive)
    .count();
```

---

## 84.5 สรุป Part 84

ในบทนี้คุณได้เรียนรู้:

✅ groupingBy, partitioningBy, joining, toMap  
✅ Custom Collector (Collector.of)  
✅ Nested/downstream collectors  
✅ flatMap (flatten nested lists)  
✅ Multi-level Comparator  
✅ IntStream, LongStream, DoubleStream  
✅ Optional chaining (map, flatMap, filter, orElse)  
✅ Optional.stream() for flatMap  

---

*[← Part 83: Bluetooth & NFC](./part-83-android-bluetooth-nfc.md) | [Part 85: Android App Shortcuts →](./part-85-android-shortcuts.md)*
