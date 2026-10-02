# Part 15: Collections Framework
## หลักสูตร Java & Android Development - ระดับ Intermediate

---

## 15.1 Collections Framework Overview

```
Collection
├── List (ordered, allows duplicates)
│   ├── ArrayList     - dynamic array, fast random access
│   ├── LinkedList    - doubly linked, fast insert/delete
│   └── Vector        - thread-safe (legacy)
├── Set (no duplicates)
│   ├── HashSet       - no order, fast O(1)
│   ├── LinkedHashSet - insertion order
│   └── TreeSet       - sorted order
└── Queue
    ├── LinkedList    - also implements Queue
    ├── PriorityQueue - sorted by priority
    └── ArrayDeque    - double-ended queue

Map (key-value pairs, separate hierarchy)
├── HashMap          - no order, fast O(1)
├── LinkedHashMap    - insertion order
├── TreeMap          - sorted by key
└── Hashtable        - thread-safe (legacy)
```

---

## 15.2 ArrayList

```java
import java.util.*;
import java.util.stream.*;

public class ArrayListDemo {
    public static void main(String[] args) {
        // --- Creation ---
        ArrayList<String> names = new ArrayList<>();
        ArrayList<Integer> numbers = new ArrayList<>(Arrays.asList(5, 3, 8, 1, 9, 2));
        List<String> immutable = List.of("A", "B", "C");  // Java 9+, unmodifiable
        List<String> mutable = new ArrayList<>(List.of("X", "Y", "Z")); // modifiable copy
        
        // --- Adding ---
        names.add("Alice");
        names.add("Bob");
        names.add("Charlie");
        names.add(1, "Anna");      // insert at index 1
        names.addAll(List.of("Dave", "Eve"));
        System.out.println("Names: " + names);
        
        // --- Accessing ---
        System.out.println("First: " + names.get(0));
        System.out.println("Last: " + names.get(names.size() - 1));
        System.out.println("Index of Bob: " + names.indexOf("Bob"));
        System.out.println("Contains Dave: " + names.contains("Dave"));
        
        // --- Modifying ---
        names.set(0, "Alicia");
        names.remove("Anna");
        names.remove(names.size() - 1);  // remove by index
        System.out.println("After modify: " + names);
        
        // --- Iterating ---
        System.out.print("forEach: ");
        for (String name : names) System.out.print(name + " ");
        System.out.println();
        
        names.forEach(System.out::println);
        
        // --- Sorting ---
        Collections.sort(numbers);
        System.out.println("Sorted: " + numbers);
        
        Collections.sort(numbers, Comparator.reverseOrder());
        System.out.println("Reversed: " + numbers);
        
        // --- Searching ---
        Collections.sort(numbers);
        int idx = Collections.binarySearch(numbers, 5);
        System.out.println("Index of 5: " + idx);
        
        // --- Sublist ---
        List<Integer> sub = numbers.subList(1, 4);  // [1, 4)
        System.out.println("Sublist [1,4): " + sub);
        
        // --- Convert ---
        String[] arr = names.toArray(new String[0]);
        System.out.println("Array: " + Arrays.toString(arr));
        
        // --- Filter and collect ---
        List<String> longNames = names.stream()
            .filter(n -> n.length() > 4)
            .collect(Collectors.toList());
        System.out.println("Names > 4 chars: " + longNames);
        
        // --- Statistics ---
        System.out.println("Max: " + Collections.max(numbers));
        System.out.println("Min: " + Collections.min(numbers));
        System.out.println("Sum: " + numbers.stream().mapToInt(Integer::intValue).sum());
    }
}
```

---

## 15.3 LinkedList

```java
import java.util.*;

public class LinkedListDemo {
    public static void main(String[] args) {
        LinkedList<String> queue = new LinkedList<>();
        
        // --- As Queue (FIFO) ---
        queue.offer("Task1");   // add to tail
        queue.offer("Task2");
        queue.offer("Task3");
        
        System.out.println("Queue: " + queue);
        System.out.println("Peek (no remove): " + queue.peek());
        System.out.println("Poll (remove): " + queue.poll());
        System.out.println("After poll: " + queue);
        
        // --- As Stack (LIFO) ---
        LinkedList<String> stack = new LinkedList<>();
        stack.push("Page1");  // addFirst
        stack.push("Page2");
        stack.push("Page3");
        
        System.out.println("\nStack: " + stack);
        System.out.println("Pop: " + stack.pop());  // removeFirst
        System.out.println("After pop: " + stack);
        
        // --- As Deque (both ends) ---
        LinkedList<Integer> deque = new LinkedList<>();
        deque.addFirst(1);
        deque.addLast(2);
        deque.addFirst(0);
        deque.addLast(3);
        
        System.out.println("\nDeque: " + deque);
        System.out.println("First: " + deque.getFirst());
        System.out.println("Last: " + deque.getLast());
        deque.removeFirst();
        deque.removeLast();
        System.out.println("After removes: " + deque);
        
        // --- Performance note ---
        // LinkedList: fast O(1) add/remove at ends
        // ArrayList: fast O(1) random access by index
    }
}
```

---

## 15.4 HashSet, LinkedHashSet, TreeSet

```java
import java.util.*;

public class SetDemo {
    public static void main(String[] args) {
        // --- HashSet (no order, no duplicates) ---
        Set<String> hashSet = new HashSet<>();
        hashSet.add("Banana");
        hashSet.add("Apple");
        hashSet.add("Cherry");
        hashSet.add("Apple");   // duplicate - ignored
        hashSet.add("Date");
        System.out.println("HashSet: " + hashSet);  // random order
        System.out.println("Contains Apple: " + hashSet.contains("Apple"));
        
        // --- LinkedHashSet (insertion order) ---
        Set<String> linkedSet = new LinkedHashSet<>();
        linkedSet.add("Banana");
        linkedSet.add("Apple");
        linkedSet.add("Cherry");
        linkedSet.add("Apple");  // duplicate - ignored
        linkedSet.add("Date");
        System.out.println("LinkedHashSet: " + linkedSet);  // insertion order
        
        // --- TreeSet (sorted) ---
        TreeSet<String> treeSet = new TreeSet<>();
        treeSet.add("Banana");
        treeSet.add("Apple");
        treeSet.add("Cherry");
        treeSet.add("Date");
        System.out.println("TreeSet: " + treeSet);      // alphabetical
        System.out.println("First: " + treeSet.first());
        System.out.println("Last: " + treeSet.last());
        System.out.println("HeadSet < Cherry: " + treeSet.headSet("Cherry"));
        System.out.println("TailSet >= Cherry: " + treeSet.tailSet("Cherry"));
        
        // --- Set operations ---
        Set<Integer> setA = new HashSet<>(Arrays.asList(1, 2, 3, 4, 5));
        Set<Integer> setB = new HashSet<>(Arrays.asList(4, 5, 6, 7, 8));
        
        // Union
        Set<Integer> union = new HashSet<>(setA);
        union.addAll(setB);
        System.out.println("\nUnion: " + new TreeSet<>(union));
        
        // Intersection
        Set<Integer> intersection = new HashSet<>(setA);
        intersection.retainAll(setB);
        System.out.println("Intersection: " + new TreeSet<>(intersection));
        
        // Difference (A - B)
        Set<Integer> difference = new HashSet<>(setA);
        difference.removeAll(setB);
        System.out.println("Difference A-B: " + new TreeSet<>(difference));
        
        // Removing duplicates from a list
        List<String> withDups = Arrays.asList("a", "b", "a", "c", "b", "d");
        List<String> unique = new ArrayList<>(new LinkedHashSet<>(withDups));
        System.out.println("\nWith duplicates: " + withDups);
        System.out.println("Without duplicates: " + unique);
    }
}
```

---

## 15.5 HashMap, LinkedHashMap, TreeMap

```java
import java.util.*;
import java.util.stream.*;

public class MapDemo {
    public static void main(String[] args) {
        // --- HashMap ---
        Map<String, Integer> scores = new HashMap<>();
        scores.put("Alice", 95);
        scores.put("Bob", 82);
        scores.put("Charlie", 78);
        scores.put("Alice", 97);  // update existing key
        
        System.out.println("Scores: " + scores);
        System.out.println("Alice: " + scores.get("Alice"));
        System.out.println("Dave (default): " + scores.getOrDefault("Dave", 0));
        System.out.println("Contains Bob: " + scores.containsKey("Bob"));
        
        // putIfAbsent
        scores.putIfAbsent("Dave", 88);
        scores.putIfAbsent("Alice", 50);  // won't update, already exists
        System.out.println("After putIfAbsent: " + scores);
        
        // computeIfAbsent / compute
        Map<String, List<String>> groupedBy = new HashMap<>();
        String[] words = {"apple", "banana", "avocado", "blueberry", "cherry", "apricot"};
        for (String word : words) {
            String key = String.valueOf(word.charAt(0));
            groupedBy.computeIfAbsent(key, k -> new ArrayList<>()).add(word);
        }
        System.out.println("\nGrouped by first letter: " + new TreeMap<>(groupedBy));
        
        // merge
        Map<String, Integer> wordCount = new HashMap<>();
        String[] sentence = "the cat sat on the mat the cat".split(" ");
        for (String word : sentence) {
            wordCount.merge(word, 1, Integer::sum);
        }
        System.out.println("\nWord count: " + new TreeMap<>(wordCount));
        
        // --- Iterating ---
        System.out.println("\n=== Iterating ===");
        for (Map.Entry<String, Integer> entry : scores.entrySet()) {
            System.out.printf("  %s: %d%n", entry.getKey(), entry.getValue());
        }
        
        scores.forEach((k, v) -> System.out.printf("  %s = %d%n", k, v));
        
        // Keys and values
        System.out.println("Keys: " + scores.keySet());
        System.out.println("Values: " + scores.values());
        
        // --- LinkedHashMap (insertion order) ---
        Map<String, String> capitals = new LinkedHashMap<>();
        capitals.put("Thailand", "Bangkok");
        capitals.put("Japan", "Tokyo");
        capitals.put("France", "Paris");
        System.out.println("\nCapitals (insertion order): " + capitals);
        
        // --- TreeMap (sorted by key) ---
        TreeMap<String, Integer> sorted = new TreeMap<>(scores);
        System.out.println("\nSorted scores: " + sorted);
        System.out.println("First key: " + sorted.firstKey());
        System.out.println("Last key: " + sorted.lastKey());
        System.out.println("HeadMap (< D): " + sorted.headMap("D"));
        
        // --- Sorting map by value ---
        System.out.println("\nSorted by score desc:");
        scores.entrySet().stream()
            .sorted(Map.Entry.<String, Integer>comparingByValue().reversed())
            .forEach(e -> System.out.printf("  %s: %d%n", e.getKey(), e.getValue()));
    }
}
```

---

## 15.6 PriorityQueue

```java
import java.util.*;

public class PriorityQueueDemo {
    
    static class Task implements Comparable<Task> {
        String name;
        int priority;  // 1=highest
        String description;
        
        Task(String name, int priority, String description) {
            this.name = name;
            this.priority = priority;
            this.description = description;
        }
        
        @Override
        public int compareTo(Task other) {
            return Integer.compare(this.priority, other.priority);  // min heap = highest priority first
        }
        
        @Override
        public String toString() {
            return String.format("[P%d] %s: %s", priority, name, description);
        }
    }
    
    public static void main(String[] args) {
        // --- Natural ordering (min heap) ---
        PriorityQueue<Integer> minHeap = new PriorityQueue<>();
        minHeap.offer(5);
        minHeap.offer(1);
        minHeap.offer(3);
        minHeap.offer(2);
        minHeap.offer(4);
        
        System.out.print("MinHeap poll order: ");
        while (!minHeap.isEmpty()) System.out.print(minHeap.poll() + " ");
        System.out.println();
        
        // --- Max heap using comparator ---
        PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
        maxHeap.addAll(Arrays.asList(5, 1, 3, 2, 4));
        
        System.out.print("MaxHeap poll order: ");
        while (!maxHeap.isEmpty()) System.out.print(maxHeap.poll() + " ");
        System.out.println();
        
        // --- Task scheduling ---
        PriorityQueue<Task> taskQueue = new PriorityQueue<>();
        taskQueue.offer(new Task("Fix bug", 2, "Critical production bug"));
        taskQueue.offer(new Task("Code review", 4, "Review PR #123"));
        taskQueue.offer(new Task("Deploy", 1, "Emergency deploy"));
        taskQueue.offer(new Task("Meeting", 3, "Sprint planning"));
        taskQueue.offer(new Task("Documentation", 5, "Update README"));
        
        System.out.println("\n=== Task Queue (by priority) ===");
        while (!taskQueue.isEmpty()) {
            System.out.println(taskQueue.poll());
        }
    }
}
```

---

## 15.7 Collections Utility Methods

```java
import java.util.*;

public class CollectionsUtility {
    public static void main(String[] args) {
        List<Integer> nums = new ArrayList<>(Arrays.asList(3, 1, 4, 1, 5, 9, 2, 6, 5, 3));
        
        // Sort
        Collections.sort(nums);
        System.out.println("Sorted: " + nums);
        
        // Reverse
        Collections.reverse(nums);
        System.out.println("Reversed: " + nums);
        
        // Shuffle (random order)
        Collections.shuffle(nums, new Random(42));
        System.out.println("Shuffled: " + nums);
        
        // Min/Max
        System.out.println("Min: " + Collections.min(nums));
        System.out.println("Max: " + Collections.max(nums));
        
        // Frequency
        List<String> words = Arrays.asList("a", "b", "a", "c", "a", "b");
        System.out.println("Frequency of 'a': " + Collections.frequency(words, "a"));
        
        // Fill
        List<String> filled = new ArrayList<>(Collections.nCopies(5, "Java"));
        System.out.println("Filled: " + filled);
        
        // Swap
        Collections.swap(nums, 0, nums.size() - 1);
        System.out.println("After swap: " + nums);
        
        // Rotate
        List<Integer> rotated = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
        Collections.rotate(rotated, 2);
        System.out.println("Rotated by 2: " + rotated);
        
        // Unmodifiable collections
        List<String> mutable = new ArrayList<>(Arrays.asList("A", "B", "C"));
        List<String> unmod = Collections.unmodifiableList(mutable);
        
        try {
            unmod.add("D");  // throws UnsupportedOperationException
        } catch (UnsupportedOperationException e) {
            System.out.println("Cannot modify unmodifiable list");
        }
        
        // Synchronized collections (thread-safe)
        List<String> syncList = Collections.synchronizedList(new ArrayList<>());
        syncList.add("thread-safe");
        
        // disjoint (no common elements)
        Set<Integer> setA = new HashSet<>(Arrays.asList(1, 2, 3));
        Set<Integer> setB = new HashSet<>(Arrays.asList(4, 5, 6));
        Set<Integer> setC = new HashSet<>(Arrays.asList(3, 4, 5));
        System.out.println("A disjoint B: " + Collections.disjoint(setA, setB));
        System.out.println("A disjoint C: " + Collections.disjoint(setA, setC));
    }
}
```

---

## 15.8 โปรแกรมตัวอย่าง: Student Management System

```java
import java.util.*;
import java.util.stream.*;

public class StudentManagement {
    
    record Student(String id, String name, String major, int year, double gpa) {}
    
    static List<Student> students = new ArrayList<>(Arrays.asList(
        new Student("S001", "Alice Johnson", "CS",   3, 3.9),
        new Student("S002", "Bob Smith",     "Math", 2, 3.5),
        new Student("S003", "Charlie Brown", "CS",   1, 3.2),
        new Student("S004", "Diana Prince",  "CS",   4, 3.8),
        new Student("S005", "Eve Wilson",    "Phys", 2, 3.7),
        new Student("S006", "Frank Miller",  "Math", 3, 3.3),
        new Student("S007", "Grace Lee",     "CS",   2, 3.6),
        new Student("S008", "Henry Ford",    "Phys", 4, 3.1)
    ));
    
    // Find by ID
    static Optional<Student> findById(String id) {
        return students.stream().filter(s -> s.id().equals(id)).findFirst();
    }
    
    // Get by major
    static List<Student> getByMajor(String major) {
        return students.stream()
            .filter(s -> s.major().equals(major))
            .sorted(Comparator.comparingDouble(Student::gpa).reversed())
            .collect(Collectors.toList());
    }
    
    // Group by major
    static Map<String, List<Student>> groupByMajor() {
        return students.stream()
            .collect(Collectors.groupingBy(Student::major));
    }
    
    // Average GPA per major
    static Map<String, Double> avgGpaByMajor() {
        return students.stream()
            .collect(Collectors.groupingBy(Student::major,
                Collectors.averagingDouble(Student::gpa)));
    }
    
    // Top N students
    static List<Student> topStudents(int n) {
        return students.stream()
            .sorted(Comparator.comparingDouble(Student::gpa).reversed())
            .limit(n)
            .collect(Collectors.toList());
    }
    
    // Honor roll (GPA >= 3.7)
    static List<Student> honorRoll() {
        return students.stream()
            .filter(s -> s.gpa() >= 3.7)
            .sorted(Comparator.comparing(Student::name))
            .collect(Collectors.toList());
    }
    
    static void printStudentList(String title, List<Student> list) {
        System.out.println("\n=== " + title + " ===");
        System.out.printf("%-6s %-18s %-6s %4s  %s%n", "ID", "Name", "Major", "Year", "GPA");
        System.out.println("-".repeat(50));
        list.forEach(s -> System.out.printf("%-6s %-18s %-6s %4d  %.2f%n",
            s.id(), s.name(), s.major(), s.year(), s.gpa()));
    }
    
    public static void main(String[] args) {
        printStudentList("All Students", students);
        printStudentList("CS Students", getByMajor("CS"));
        printStudentList("Top 3 Students", topStudents(3));
        printStudentList("Honor Roll (GPA >= 3.7)", honorRoll());
        
        System.out.println("\n=== Average GPA by Major ===");
        avgGpaByMajor().entrySet().stream()
            .sorted(Map.Entry.<String, Double>comparingByValue().reversed())
            .forEach(e -> System.out.printf("%-8s: %.2f%n", e.getKey(), e.getValue()));
        
        System.out.println("\n=== Student count by major ===");
        groupByMajor().forEach((major, studs) ->
            System.out.printf("%-8s: %d students%n", major, studs.size()));
        
        // Find specific student
        findById("S003").ifPresent(s ->
            System.out.println("\nFound: " + s.name() + " (GPA: " + s.gpa() + ")"));
        
        findById("S999").ifPresentOrElse(
            s -> System.out.println("Found: " + s),
            () -> System.out.println("\nStudent S999 not found")
        );
        
        // Statistics
        DoubleSummaryStatistics stats = students.stream()
            .mapToDouble(Student::gpa)
            .summaryStatistics();
        System.out.printf("\nGPA Stats - Min:%.2f Max:%.2f Avg:%.2f%n",
            stats.getMin(), stats.getMax(), stats.getAverage());
    }
}
```

---

## 15.9 สรุป Part 15

ในบทนี้คุณได้เรียนรู้:

✅ Collections Framework hierarchy  
✅ ArrayList - dynamic array  
✅ LinkedList - queue/stack/deque  
✅ Set - HashSet, LinkedHashSet, TreeSet  
✅ Map - HashMap, LinkedHashMap, TreeMap  
✅ PriorityQueue  
✅ Collections utility methods  
✅ เมื่อใช้ collection ไหน  

---

*[← Part 14: Exception Handling](./part-14-exceptions.md) | [Part 16: Generics →](./part-16-generics.md)*
