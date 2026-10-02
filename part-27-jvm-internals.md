# Part 27: JVM Internals และ Performance
## หลักสูตร Java & Android Development - ระดับ Advanced

---

## 27.1 JVM Architecture

```
JVM
├── Class Loader
│   ├── Bootstrap ClassLoader (rt.jar)
│   ├── Extension ClassLoader
│   └── Application ClassLoader
├── Runtime Data Areas
│   ├── Method Area (class definitions, static fields)
│   ├── Heap (objects)
│   │   ├── Young Generation (Eden + Survivor)
│   │   └── Old Generation (Tenured)
│   ├── Stack (per-thread, frames)
│   ├── PC Registers (per-thread)
│   └── Native Method Stack
└── Execution Engine
    ├── Interpreter
    ├── JIT Compiler
    └── Garbage Collector
```

---

## 27.2 Memory Management

```java
public class MemoryDemo {
    
    public static void main(String[] args) {
        // --- JVM Memory ---
        Runtime runtime = Runtime.getRuntime();
        
        long maxMemory  = runtime.maxMemory();    // max heap
        long totalMemory = runtime.totalMemory(); // current heap
        long freeMemory  = runtime.freeMemory();  // free heap
        long usedMemory  = totalMemory - freeMemory;
        
        System.out.println("=== JVM Memory ===");
        System.out.printf("Max Heap:   %,d MB%n", maxMemory / (1024*1024));
        System.out.printf("Total Heap: %,d MB%n", totalMemory / (1024*1024));
        System.out.printf("Used Heap:  %,d MB%n", usedMemory / (1024*1024));
        System.out.printf("Free Heap:  %,d MB%n", freeMemory / (1024*1024));
        
        // CPU cores
        System.out.println("CPU cores: " + runtime.availableProcessors());
        
        // --- Memory benchmark ---
        System.out.println("\n=== Memory Usage Test ===");
        
        long before = runtime.totalMemory() - runtime.freeMemory();
        
        // Allocate 1M strings
        String[] strings = new String[1_000_000];
        for (int i = 0; i < strings.length; i++) strings[i] = "str" + i;
        
        long after = runtime.totalMemory() - runtime.freeMemory();
        System.out.printf("1M strings used: %,d MB%n", (after - before) / (1024*1024));
        
        // Null out reference - eligible for GC
        strings = null;
        System.gc();  // suggest GC (not guaranteed)
        Thread.yield();
        
        long afterGC = runtime.totalMemory() - runtime.freeMemory();
        System.out.printf("After GC: %,d MB%n", afterGC / (1024*1024));
        
        // --- Object creation patterns ---
        measureCreation("String concat", () -> {
            String s = "";
            for (int i = 0; i < 1000; i++) s += "x";  // SLOW: creates 1000 String objects
        });
        
        measureCreation("StringBuilder", () -> {
            StringBuilder sb = new StringBuilder();
            for (int i = 0; i < 1000; i++) sb.append("x");  // FAST: single object
            sb.toString();
        });
    }
    
    static void measureCreation(String name, Runnable task) {
        long start = System.nanoTime();
        for (int i = 0; i < 1000; i++) task.run();
        long time = System.nanoTime() - start;
        System.out.printf("%-20s: %,d ms%n", name, time / 1_000_000);
    }
}
```

---

## 27.3 Performance Optimization

```java
import java.util.*;
import java.util.stream.*;

public class PerformanceOpt {
    
    // 1. String operations
    static void stringPerformance() {
        int N = 100_000;
        
        // Bad: String concatenation in loop
        long t1 = System.currentTimeMillis();
        String result = "";
        for (int i = 0; i < N; i++) result += i + ",";
        long t2 = System.currentTimeMillis();
        
        // Good: StringBuilder
        long t3 = System.currentTimeMillis();
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < N; i++) sb.append(i).append(",");
        String result2 = sb.toString();
        long t4 = System.currentTimeMillis();
        
        // Best: String.join / Collectors.joining
        long t5 = System.currentTimeMillis();
        String result3 = IntStream.range(0, N)
            .mapToObj(String::valueOf)
            .collect(Collectors.joining(","));
        long t6 = System.currentTimeMillis();
        
        System.out.printf("String concat:   %,d ms%n", t2 - t1);
        System.out.printf("StringBuilder:   %,d ms%n", t4 - t3);
        System.out.printf("Stream joining:  %,d ms%n", t6 - t5);
    }
    
    // 2. ArrayList vs LinkedList
    static void collectionPerformance() {
        int N = 100_000;
        List<Integer> arrayList = new ArrayList<>();
        List<Integer> linkedList = new LinkedList<>();
        
        // Random access
        for (int i = 0; i < N; i++) { arrayList.add(i); linkedList.add(i); }
        
        long t1 = System.nanoTime();
        for (int i = 0; i < N; i++) arrayList.get(i % N);
        long t2 = System.nanoTime();
        
        long t3 = System.nanoTime();
        for (int i = 0; i < N; i++) linkedList.get(i % 1000);  // limited for LinkedList
        long t4 = System.nanoTime();
        
        System.out.printf("ArrayList get:   %,d ns%n", (t2-t1)/N);
        System.out.printf("LinkedList get:  %,d ns%n", (t4-t3)/1000);
        
        // Insert at beginning
        List<Integer> al = new ArrayList<>();
        List<Integer> ll = new LinkedList<>();
        
        long t5 = System.nanoTime();
        for (int i = 0; i < 10000; i++) al.add(0, i);
        long t6 = System.nanoTime();
        
        long t7 = System.nanoTime();
        for (int i = 0; i < 10000; i++) ll.add(0, i);
        long t8 = System.nanoTime();
        
        System.out.printf("ArrayList insert@0: %,d ms%n", (t6-t5)/1_000_000);
        System.out.printf("LinkedList insert@0: %,d ms%n", (t8-t7)/1_000_000);
    }
    
    // 3. Autoboxing overhead
    static void autoboxingPerformance() {
        int N = 1_000_000;
        
        // Boxed Integer (slow)
        long t1 = System.currentTimeMillis();
        Integer sum1 = 0;
        for (int i = 0; i < N; i++) sum1 += i;  // autoboxing on each iteration
        long t2 = System.currentTimeMillis();
        
        // Primitive int (fast)
        long t3 = System.currentTimeMillis();
        int sum2 = 0;
        for (int i = 0; i < N; i++) sum2 += i;  // no boxing
        long t4 = System.currentTimeMillis();
        
        System.out.printf("Boxed Integer sum:    %,d ms%n", t2-t1);
        System.out.printf("Primitive int sum:    %,d ms%n", t4-t3);
    }
    
    // 4. Cache-friendly access
    static void cachePerformance() {
        int SIZE = 1000;
        int[][] matrix = new int[SIZE][SIZE];
        for (int i = 0; i < SIZE; i++)
            for (int j = 0; j < SIZE; j++) matrix[i][j] = i + j;
        
        // Row-major (cache-friendly)
        long t1 = System.nanoTime();
        long sum1 = 0;
        for (int i = 0; i < SIZE; i++)
            for (int j = 0; j < SIZE; j++) sum1 += matrix[i][j];
        long t2 = System.nanoTime();
        
        // Column-major (cache-unfriendly)
        long t3 = System.nanoTime();
        long sum2 = 0;
        for (int j = 0; j < SIZE; j++)
            for (int i = 0; i < SIZE; i++) sum2 += matrix[i][j];
        long t4 = System.nanoTime();
        
        System.out.printf("Row-major access:    %,d ms%n", (t2-t1)/1_000_000);
        System.out.printf("Column-major access: %,d ms%n", (t4-t3)/1_000_000);
    }
    
    public static void main(String[] args) {
        System.out.println("=== String Performance ===");
        stringPerformance();
        
        System.out.println("\n=== Collection Performance ===");
        collectionPerformance();
        
        System.out.println("\n=== Autoboxing Performance ===");
        autoboxingPerformance();
        
        System.out.println("\n=== Cache Performance ===");
        cachePerformance();
    }
}
```

---

## 27.4 GC Tuning Flags

```
# JVM flags for tuning:

# Heap size
-Xms512m          # initial heap
-Xmx2g            # max heap
-XX:MetaspaceSize=256m

# GC selection
-XX:+UseG1GC      # G1 (default Java 9+)
-XX:+UseZGC       # ZGC (low latency, Java 11+)
-XX:+UseShenandoahGC  # Shenandoah (low latency)

# GC logging
-verbose:gc
-XX:+PrintGCDetails
-Xlog:gc*:file=gc.log

# Performance
-server
-XX:+OptimizeStringConcat
-XX:+UseCompressedOops   # saves memory on 64-bit

# Monitoring
jstat -gc <pid> 1000     # GC stats every 1s
jmap -heap <pid>         # heap info
jstack <pid>             # thread dump
jconsole                 # GUI monitor
```

---

## 27.5 Profiling

```java
public class ProfilingDemo {
    
    // Simple profiler
    static class Profiler {
        private static final Map<String, Long> timings = new LinkedHashMap<>();
        private static final Map<String, Integer> calls = new LinkedHashMap<>();
        
        static <T> T time(String name, java.util.function.Supplier<T> task) {
            long start = System.nanoTime();
            T result = task.get();
            long elapsed = System.nanoTime() - start;
            timings.merge(name, elapsed, Long::sum);
            calls.merge(name, 1, Integer::sum);
            return result;
        }
        
        static void time(String name, Runnable task) {
            time(name, () -> { task.run(); return null; });
        }
        
        static void report() {
            System.out.println("\n=== Performance Report ===");
            System.out.printf("%-25s %8s %8s %10s%n", "Method", "Calls", "Total(ms)", "Avg(μs)");
            System.out.println("-".repeat(60));
            timings.forEach((name, total) -> {
                int n = calls.get(name);
                System.out.printf("%-25s %8d %8.1f %10.1f%n",
                    name, n, total / 1_000_000.0, total / 1000.0 / n);
            });
        }
        
        static void reset() { timings.clear(); calls.clear(); }
    }
    
    static int fibonacci(int n) {
        if (n <= 1) return n;
        return fibonacci(n - 1) + fibonacci(n - 2);
    }
    
    static int fibMemo(int n, int[] memo) {
        if (n <= 1) return n;
        if (memo[n] != 0) return memo[n];
        return memo[n] = fibMemo(n - 1, memo) + fibMemo(n - 2, memo);
    }
    
    public static void main(String[] args) {
        // Profile different fibonacci implementations
        for (int i = 0; i < 100; i++) {
            final int n = 35;
            Profiler.time("fibonacci(35) naive", () -> fibonacci(n));
            Profiler.time("fibonacci(35) memo", () -> fibMemo(n, new int[n + 1]));
        }
        
        Profiler.report();
    }
}
```

---

## 27.6 สรุป Part 27

ในบทนี้คุณได้เรียนรู้:

✅ JVM architecture (class loader, heap, stack)  
✅ Memory management  
✅ Performance optimization tips  
✅ String vs StringBuilder performance  
✅ ArrayList vs LinkedList tradeoffs  
✅ Autoboxing overhead  
✅ GC tuning flags  
✅ Simple profiler  

---

*[← Part 26: Reflection](./part-26-reflection.md) | [Part 28: Build Tools →](./part-28-build-tools.md)*
