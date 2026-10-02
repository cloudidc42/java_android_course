# Part 16: Generics
## หลักสูตร Java & Android Development - ระดับ Intermediate

---

## 16.1 Generics คืออะไร?

Generics ช่วยให้เขียน code ที่ทำงานกับ type ต่างๆ ได้โดยปลอดภัย (type-safe)

```java
// Without generics (old way)
List oldList = new ArrayList();
oldList.add("Hello");
oldList.add(42);
String s = (String) oldList.get(0);  // Requires cast, risk of ClassCastException

// With generics
List<String> newList = new ArrayList<>();
newList.add("Hello");
// newList.add(42);  // Compile-time error - safe!
String s2 = newList.get(0);  // No cast needed
```

---

## 16.2 Generic Classes

```java
// Generic class with type parameter T
public class Box<T> {
    private T content;
    private String label;
    
    Box(T content, String label) {
        this.content = content;
        this.label = label;
    }
    
    T getContent() { return content; }
    void setContent(T newContent) { this.content = newContent; }
    String getLabel() { return label; }
    
    boolean isEmpty() { return content == null; }
    
    @Override
    public String toString() {
        return String.format("Box[%s: %s]", label, content);
    }
}

// Multiple type parameters
public class Pair<K, V> {
    private K key;
    private V value;
    
    Pair(K key, V value) {
        this.key = key;
        this.value = value;
    }
    
    K getKey() { return key; }
    V getValue() { return value; }
    
    static <A, B> Pair<A, B> of(A a, B b) { return new Pair<>(a, b); }
    
    @Override
    public String toString() { return "(" + key + ", " + value + ")"; }
}

// Triple
public class Triple<A, B, C> {
    final A first;
    final B second;
    final C third;
    
    Triple(A first, B second, C third) {
        this.first = first;
        this.second = second;
        this.third = third;
    }
    
    @Override
    public String toString() {
        return "(" + first + ", " + second + ", " + third + ")";
    }
}

public class GenericsDemo {
    public static void main(String[] args) {
        Box<String> stringBox = new Box<>("Hello", "greeting");
        Box<Integer> intBox = new Box<>(42, "number");
        Box<Double[]> arrayBox = new Box<>(new Double[]{1.1, 2.2, 3.3}, "doubles");
        
        System.out.println(stringBox);
        System.out.println(intBox);
        System.out.println(stringBox.getContent().toUpperCase());  // type-safe
        System.out.println(intBox.getContent() * 2);               // type-safe
        
        Pair<String, Integer> nameAge = Pair.of("Alice", 30);
        Pair<String, List<String>> person = Pair.of("Bob", Arrays.asList("Java", "Python"));
        
        System.out.println(nameAge);
        System.out.println(person);
        System.out.println("Age: " + nameAge.getValue() + 1);
        
        Triple<String, Integer, Double> student = new Triple<>("Alice", 20, 3.9);
        System.out.println(student);
    }
}
```

---

## 16.3 Generic Methods

```java
import java.util.*;
import java.util.function.*;

public class GenericMethods {
    
    // Generic method
    static <T> void swap(T[] arr, int i, int j) {
        T temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
    
    // Generic method with return type
    static <T> T getFirst(List<T> list) {
        if (list.isEmpty()) throw new NoSuchElementException("List is empty");
        return list.get(0);
    }
    
    static <T> T getLast(List<T> list) {
        if (list.isEmpty()) throw new NoSuchElementException("List is empty");
        return list.get(list.size() - 1);
    }
    
    // Generic pair
    static <K, V> Map<V, K> invertMap(Map<K, V> original) {
        Map<V, K> inverted = new HashMap<>();
        original.forEach((k, v) -> inverted.put(v, k));
        return inverted;
    }
    
    // Generic filter
    static <T> List<T> filter(List<T> list, Predicate<T> predicate) {
        List<T> result = new ArrayList<>();
        for (T item : list) {
            if (predicate.test(item)) result.add(item);
        }
        return result;
    }
    
    // Generic transform
    static <T, R> List<R> map(List<T> list, Function<T, R> mapper) {
        List<R> result = new ArrayList<>();
        for (T item : list) result.add(mapper.apply(item));
        return result;
    }
    
    // Generic reduce
    static <T> T reduce(List<T> list, T identity, BinaryOperator<T> accumulator) {
        T result = identity;
        for (T item : list) result = accumulator.apply(result, item);
        return result;
    }
    
    public static void main(String[] args) {
        Integer[] arr = {1, 2, 3, 4, 5};
        swap(arr, 0, 4);
        System.out.println("After swap: " + Arrays.toString(arr));
        
        String[] strArr = {"a", "b", "c"};
        swap(strArr, 0, 2);
        System.out.println("After swap: " + Arrays.toString(strArr));
        
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        System.out.println("First: " + getFirst(numbers));
        System.out.println("Last: " + getLast(numbers));
        
        List<Integer> evens = filter(numbers, n -> n % 2 == 0);
        System.out.println("Evens: " + evens);
        
        List<String> strings = map(numbers, n -> "Item" + n);
        System.out.println("Mapped: " + strings);
        
        int sum = reduce(numbers, 0, Integer::sum);
        System.out.println("Sum: " + sum);
        
        Map<String, Integer> scores = Map.of("Alice", 95, "Bob", 82, "Charlie", 78);
        Map<Integer, String> inverted = invertMap(scores);
        System.out.println("Inverted: " + inverted);
    }
}
```

---

## 16.4 Bounded Type Parameters

```java
import java.util.*;

public class BoundedGenerics {
    
    // Upper bound: T must be Number or subclass
    static <T extends Number> double sum(List<T> list) {
        double total = 0;
        for (T item : list) total += item.doubleValue();
        return total;
    }
    
    static <T extends Number & Comparable<T>> T max(List<T> list) {
        if (list.isEmpty()) throw new NoSuchElementException();
        T result = list.get(0);
        for (T item : list) {
            if (item.compareTo(result) > 0) result = item;
        }
        return result;
    }
    
    // Generic class with bound
    static class NumberBox<T extends Number> {
        private T value;
        
        NumberBox(T value) { this.value = value; }
        
        double getDoubleValue() { return value.doubleValue(); }
        boolean isNegative() { return value.doubleValue() < 0; }
        boolean isZero() { return value.doubleValue() == 0; }
        
        NumberBox<Double> toDouble() { return new NumberBox<>(value.doubleValue()); }
    }
    
    // Comparable bound
    static <T extends Comparable<T>> T clamp(T value, T min, T max) {
        if (value.compareTo(min) < 0) return min;
        if (value.compareTo(max) > 0) return max;
        return value;
    }
    
    static <T extends Comparable<T>> void bubbleSort(T[] arr) {
        int n = arr.length;
        for (int i = 0; i < n - 1; i++) {
            for (int j = 0; j < n - i - 1; j++) {
                if (arr[j].compareTo(arr[j+1]) > 0) {
                    T temp = arr[j]; arr[j] = arr[j+1]; arr[j+1] = temp;
                }
            }
        }
    }
    
    public static void main(String[] args) {
        List<Integer> ints = Arrays.asList(1, 2, 3, 4, 5);
        List<Double> doubles = Arrays.asList(1.5, 2.5, 3.5);
        
        System.out.println("Sum of ints: " + sum(ints));
        System.out.println("Sum of doubles: " + sum(doubles));
        System.out.println("Max of ints: " + max(ints));
        
        NumberBox<Integer> intBox = new NumberBox<>(42);
        NumberBox<Double> dblBox = new NumberBox<>(-3.14);
        
        System.out.println("IntBox: " + intBox.getDoubleValue());
        System.out.println("DblBox negative: " + dblBox.isNegative());
        
        // Clamp
        System.out.println("Clamp 5 in [1,10]: " + clamp(5, 1, 10));
        System.out.println("Clamp -3 in [1,10]: " + clamp(-3, 1, 10));
        System.out.println("Clamp 15 in [1,10]: " + clamp(15, 1, 10));
        System.out.println("Clamp 'cat' in [a,z]: " + clamp("cat", "a", "z"));
        
        Integer[] arr = {5, 2, 8, 1, 9, 3};
        bubbleSort(arr);
        System.out.println("Sorted: " + Arrays.toString(arr));
        
        String[] words = {"banana", "apple", "cherry"};
        bubbleSort(words);
        System.out.println("Sorted words: " + Arrays.toString(words));
    }
}
```

---

## 16.5 Wildcards

```java
import java.util.*;

public class WildcardDemo {
    
    // Unbounded wildcard: List<?>
    static void printList(List<?> list) {
        for (Object item : list) System.out.print(item + " ");
        System.out.println();
    }
    
    // Upper bounded: ? extends Number (read, can't write)
    static double sumNumbers(List<? extends Number> list) {
        double total = 0;
        for (Number n : list) total += n.doubleValue();
        return total;
    }
    
    // Lower bounded: ? super Integer (write, limited read)
    static void addIntegers(List<? super Integer> list, int count) {
        for (int i = 1; i <= count; i++) list.add(i);
    }
    
    // PECS: Producer Extends, Consumer Super
    static <T> void copy(List<? extends T> src, List<? super T> dst) {
        for (T item : src) dst.add(item);
    }
    
    public static void main(String[] args) {
        List<Integer> ints = Arrays.asList(1, 2, 3, 4, 5);
        List<Double> dbls = Arrays.asList(1.1, 2.2, 3.3);
        List<String> strs = Arrays.asList("a", "b", "c");
        
        System.out.print("Ints: "); printList(ints);
        System.out.print("Dbls: "); printList(dbls);
        System.out.print("Strs: "); printList(strs);
        
        System.out.println("Sum ints: " + sumNumbers(ints));
        System.out.println("Sum dbls: " + sumNumbers(dbls));
        
        List<Number> numbers = new ArrayList<>();
        addIntegers(numbers, 5);
        System.out.println("Added integers: " + numbers);
        
        List<Number> dest = new ArrayList<>();
        copy(ints, dest);
        System.out.println("Copied: " + dest);
    }
}
```

---

## 16.6 Generic Data Structures

```java
import java.util.*;

public class GenericDataStructures {
    
    // Generic Stack
    static class Stack<T> {
        private List<T> elements = new ArrayList<>();
        
        void push(T item) { elements.add(item); }
        
        T pop() {
            if (isEmpty()) throw new EmptyStackException();
            return elements.remove(elements.size() - 1);
        }
        
        T peek() {
            if (isEmpty()) throw new EmptyStackException();
            return elements.get(elements.size() - 1);
        }
        
        boolean isEmpty() { return elements.isEmpty(); }
        int size() { return elements.size(); }
        
        @Override
        public String toString() { return elements.toString(); }
    }
    
    // Generic Queue
    static class Queue<T> {
        private LinkedList<T> elements = new LinkedList<>();
        
        void enqueue(T item) { elements.addLast(item); }
        
        T dequeue() {
            if (isEmpty()) throw new NoSuchElementException("Queue is empty");
            return elements.removeFirst();
        }
        
        T peek() {
            if (isEmpty()) throw new NoSuchElementException("Queue is empty");
            return elements.getFirst();
        }
        
        boolean isEmpty() { return elements.isEmpty(); }
        int size() { return elements.size(); }
        
        @Override
        public String toString() { return elements.toString(); }
    }
    
    // Generic Binary Search Tree
    static class BST<T extends Comparable<T>> {
        private class Node {
            T data;
            Node left, right;
            Node(T data) { this.data = data; }
        }
        
        private Node root;
        
        void insert(T value) { root = insert(root, value); }
        
        private Node insert(Node node, T value) {
            if (node == null) return new Node(value);
            int cmp = value.compareTo(node.data);
            if (cmp < 0) node.left = insert(node.left, value);
            else if (cmp > 0) node.right = insert(node.right, value);
            return node;
        }
        
        boolean contains(T value) { return contains(root, value); }
        
        private boolean contains(Node node, T value) {
            if (node == null) return false;
            int cmp = value.compareTo(node.data);
            if (cmp < 0) return contains(node.left, value);
            if (cmp > 0) return contains(node.right, value);
            return true;
        }
        
        List<T> inOrder() {
            List<T> result = new ArrayList<>();
            inOrder(root, result);
            return result;
        }
        
        private void inOrder(Node node, List<T> result) {
            if (node == null) return;
            inOrder(node.left, result);
            result.add(node.data);
            inOrder(node.right, result);
        }
    }
    
    public static void main(String[] args) {
        // Stack usage
        Stack<String> stack = new Stack<>();
        stack.push("first");
        stack.push("second");
        stack.push("third");
        System.out.println("Stack: " + stack);
        System.out.println("Pop: " + stack.pop());
        System.out.println("Peek: " + stack.peek());
        
        // Queue usage
        Queue<Integer> queue = new Queue<>();
        for (int i = 1; i <= 5; i++) queue.enqueue(i * 10);
        System.out.println("\nQueue: " + queue);
        System.out.println("Dequeue: " + queue.dequeue());
        
        // BST usage
        BST<Integer> bst = new BST<>();
        int[] values = {5, 3, 8, 1, 4, 7, 9};
        for (int v : values) bst.insert(v);
        System.out.println("\nBST in-order: " + bst.inOrder());
        System.out.println("Contains 4: " + bst.contains(4));
        System.out.println("Contains 6: " + bst.contains(6));
        
        // String BST
        BST<String> wordBst = new BST<>();
        for (String w : new String[]{"banana", "apple", "cherry", "date"}) {
            wordBst.insert(w);
        }
        System.out.println("Words in-order: " + wordBst.inOrder());
    }
}
```

---

## 16.7 Generic Result Type

```java
import java.util.*;
import java.util.function.*;

public class ResultType {
    
    // Result<T> - either success or failure
    public static class Result<T> {
        private final T value;
        private final Exception error;
        private final boolean success;
        
        private Result(T value, Exception error, boolean success) {
            this.value = value;
            this.error = error;
            this.success = success;
        }
        
        static <T> Result<T> success(T value) {
            return new Result<>(value, null, true);
        }
        
        static <T> Result<T> failure(Exception error) {
            return new Result<>(null, error, false);
        }
        
        static <T> Result<T> failure(String message) {
            return failure(new RuntimeException(message));
        }
        
        boolean isSuccess() { return success; }
        boolean isFailure() { return !success; }
        
        T getValue() {
            if (!success) throw new IllegalStateException("Result is a failure");
            return value;
        }
        
        Exception getError() {
            if (success) throw new IllegalStateException("Result is a success");
            return error;
        }
        
        T getOrDefault(T defaultValue) { return success ? value : defaultValue; }
        
        <R> Result<R> map(Function<T, R> mapper) {
            if (!success) return Result.failure(error);
            try { return Result.success(mapper.apply(value)); }
            catch (Exception e) { return Result.failure(e); }
        }
        
        void ifSuccess(Consumer<T> action) { if (success) action.accept(value); }
        void ifFailure(Consumer<Exception> action) { if (!success) action.accept(error); }
        
        @Override
        public String toString() {
            return success ? "Success(" + value + ")" : "Failure(" + error.getMessage() + ")";
        }
    }
    
    // Example usage
    static Result<Integer> parseInt(String s) {
        try { return Result.success(Integer.parseInt(s)); }
        catch (NumberFormatException e) { return Result.failure("Not a number: " + s); }
    }
    
    static Result<Double> safeDivide(double a, double b) {
        if (b == 0) return Result.failure("Division by zero");
        return Result.success(a / b);
    }
    
    public static void main(String[] args) {
        // parse
        Result<Integer> r1 = parseInt("42");
        Result<Integer> r2 = parseInt("abc");
        
        r1.ifSuccess(v -> System.out.println("Parsed: " + v));
        r2.ifFailure(e -> System.out.println("Error: " + e.getMessage()));
        
        System.out.println("Default: " + r2.getOrDefault(0));
        
        // chain operations
        Result<Double> result = parseInt("10")
            .map(n -> n * 2)
            .map(n -> n + 0.5);
        
        System.out.println("Chain result: " + result);
        
        // divide
        System.out.println(safeDivide(10, 3));
        System.out.println(safeDivide(10, 0));
        
        // Array of results
        String[] inputs = {"1", "abc", "2", "xyz", "3"};
        List<Integer> validNumbers = new ArrayList<>();
        
        for (String input : inputs) {
            parseInt(input)
                .ifSuccess(validNumbers::add)
                .ifFailure(e -> System.out.println("Skip: " + e.getMessage()));
        }
        System.out.println("Valid numbers: " + validNumbers);
    }
}
```

---

## 16.8 สรุป Part 16

ในบทนี้คุณได้เรียนรู้:

✅ Generic classes และ type parameters  
✅ Generic methods  
✅ Bounded type parameters (extends, super)  
✅ Wildcards (?, extends, super)  
✅ PECS principle  
✅ Generic data structures  
✅ Generic Result type  

---

*[← Part 15: Collections](./part-15-collections.md) | [Part 17: File I/O →](./part-17-file-io.md)*
