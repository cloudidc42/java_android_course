# Part 21: Multithreading & Concurrency
## หลักสูตร Java & Android Development - ระดับ Advanced

---

## 21.1 Thread พื้นฐาน

```java
public class ThreadBasics {
    
    // วิธีที่ 1: extends Thread
    static class MyThread extends Thread {
        private String name;
        private int iterations;
        
        MyThread(String name, int iterations) {
            super(name);
            this.name = name;
            this.iterations = iterations;
        }
        
        @Override
        public void run() {
            for (int i = 1; i <= iterations; i++) {
                System.out.printf("[%s] iteration %d (thread: %s)%n",
                    name, i, Thread.currentThread().getName());
                try { Thread.sleep(100); }
                catch (InterruptedException e) { Thread.currentThread().interrupt(); break; }
            }
            System.out.println("[" + name + "] finished");
        }
    }
    
    // วิธีที่ 2: implements Runnable (preferred)
    static class Counter implements Runnable {
        private String label;
        private int count;
        
        Counter(String label, int count) {
            this.label = label;
            this.count = count;
        }
        
        @Override
        public void run() {
            for (int i = 1; i <= count; i++) {
                System.out.printf("[%s] count=%d%n", label, i);
                try { Thread.sleep(50); }
                catch (InterruptedException e) { return; }
            }
        }
    }
    
    public static void main(String[] args) throws InterruptedException {
        System.out.println("Main thread: " + Thread.currentThread().getName());
        
        // extends Thread
        MyThread t1 = new MyThread("T1", 3);
        MyThread t2 = new MyThread("T2", 3);
        t1.start();
        t2.start();
        
        // implements Runnable
        Thread t3 = new Thread(new Counter("C1", 3), "CounterThread");
        t3.start();
        
        // Lambda Runnable (Java 8+)
        Thread t4 = new Thread(() -> {
            for (int i = 0; i < 3; i++) {
                System.out.println("[Lambda] " + i);
                try { Thread.sleep(75); }
                catch (InterruptedException e) { return; }
            }
        }, "LambdaThread");
        t4.start();
        
        // join - wait for thread to finish
        t1.join();
        t2.join();
        t3.join();
        t4.join();
        
        System.out.println("All threads finished");
        
        // Thread info
        System.out.println("Active threads: " + Thread.activeCount());
        System.out.println("Current thread priority: " + Thread.currentThread().getPriority());
    }
}
```

---

## 21.2 Thread Safety and Synchronization

```java
public class ThreadSafety {
    
    // NOT thread-safe
    static class UnsafeCounter {
        private int count = 0;
        
        void increment() { count++; }  // read-modify-write: not atomic!
        int getCount() { return count; }
    }
    
    // Thread-safe with synchronized
    static class SyncCounter {
        private int count = 0;
        
        synchronized void increment() { count++; }
        synchronized int getCount() { return count; }
    }
    
    // Thread-safe with AtomicInteger
    static class AtomicCounter {
        private java.util.concurrent.atomic.AtomicInteger count =
            new java.util.concurrent.atomic.AtomicInteger(0);
        
        void increment() { count.incrementAndGet(); }
        int getCount() { return count.get(); }
    }
    
    static void testCounter(String name, Runnable inc, java.util.function.Supplier<Integer> get)
            throws InterruptedException {
        Thread[] threads = new Thread[100];
        for (int i = 0; i < threads.length; i++) {
            threads[i] = new Thread(() -> {
                for (int j = 0; j < 1000; j++) inc.run();
            });
        }
        
        long start = System.currentTimeMillis();
        for (Thread t : threads) t.start();
        for (Thread t : threads) t.join();
        long time = System.currentTimeMillis() - start;
        
        System.out.printf("%-15s expected=100000 actual=%-8d time=%dms%n",
            name, get.get(), time);
    }
    
    // synchronized block (more granular)
    static class BankAccount {
        private double balance;
        private String id;
        private final Object lock = new Object();
        
        BankAccount(String id, double balance) { this.id = id; this.balance = balance; }
        
        void deposit(double amount) {
            synchronized (lock) {
                double prev = balance;
                balance += amount;
                System.out.printf("[%s] deposit %.0f: %.0f -> %.0f%n",
                    id, amount, prev, balance);
            }
        }
        
        void withdraw(double amount) {
            synchronized (lock) {
                if (amount > balance) throw new IllegalStateException("Insufficient funds");
                double prev = balance;
                balance -= amount;
                System.out.printf("[%s] withdraw %.0f: %.0f -> %.0f%n",
                    id, amount, prev, balance);
            }
        }
        
        synchronized double getBalance() { return balance; }
    }
    
    public static void main(String[] args) throws InterruptedException {
        UnsafeCounter unsafe = new UnsafeCounter();
        SyncCounter sync = new SyncCounter();
        AtomicCounter atomic = new AtomicCounter();
        
        testCounter("Unsafe",  unsafe::increment,  unsafe::getCount);
        testCounter("Sync",    sync::increment,    sync::getCount);
        testCounter("Atomic",  atomic::increment,  atomic::getCount);
        
        // Bank account test
        BankAccount acc = new BankAccount("ACC001", 10000);
        Thread[] workers = new Thread[10];
        for (int i = 0; i < workers.length; i++) {
            final int tid = i;
            workers[i] = new Thread(() -> {
                acc.deposit(100);
                acc.withdraw(50);
            }, "Worker-" + tid);
        }
        for (Thread t : workers) t.start();
        for (Thread t : workers) t.join();
        System.out.printf("Final balance: %.0f (expected: %.0f)%n",
            acc.getBalance(), 10000 + 100*10 - 50*10.0);
    }
}
```

---

## 21.3 ExecutorService

```java
import java.util.concurrent.*;
import java.util.*;
import java.util.stream.*;

public class ExecutorServiceDemo {
    
    static int computeSquare(int n) {
        try { Thread.sleep(10); } catch (InterruptedException e) {}
        return n * n;
    }
    
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        // Fixed thread pool
        ExecutorService executor = Executors.newFixedThreadPool(4);
        
        // Submit Runnable (no return value)
        executor.submit(() -> System.out.println("Task 1 by " + Thread.currentThread().getName()));
        executor.submit(() -> System.out.println("Task 2 by " + Thread.currentThread().getName()));
        
        // Submit Callable (with return value)
        Future<Integer> future = executor.submit(() -> {
            Thread.sleep(100);
            return 42;
        });
        
        System.out.println("Waiting for result...");
        System.out.println("Result: " + future.get());  // blocks until done
        
        // Multiple futures
        List<Future<Integer>> futures = new ArrayList<>();
        for (int i = 1; i <= 10; i++) {
            final int n = i;
            futures.add(executor.submit(() -> computeSquare(n)));
        }
        
        System.out.print("Squares: ");
        for (Future<Integer> f : futures) {
            System.out.print(f.get() + " ");
        }
        System.out.println();
        
        // invokeAll - submit all and wait for all
        List<Callable<String>> tasks = IntStream.rangeClosed(1, 5)
            .mapToObj(i -> (Callable<String>) () -> {
                Thread.sleep(50 * i);
                return "Task " + i + " done";
            })
            .collect(Collectors.toList());
        
        List<Future<String>> results = executor.invokeAll(tasks);
        System.out.println("invokeAll results:");
        for (Future<String> f : results) System.out.println("  " + f.get());
        
        // invokeAny - return first completed
        String first = executor.invokeAny(tasks);
        System.out.println("invokeAny first: " + first);
        
        // Shutdown
        executor.shutdown();
        executor.awaitTermination(5, TimeUnit.SECONDS);
        System.out.println("Executor shut down");
        
        // ScheduledExecutorService
        ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(2);
        
        scheduler.schedule(() -> System.out.println("One-shot after 200ms"),
            200, TimeUnit.MILLISECONDS);
        
        ScheduledFuture<?> periodic = scheduler.scheduleAtFixedRate(
            () -> System.out.println("Periodic: " + System.currentTimeMillis()),
            0, 100, TimeUnit.MILLISECONDS);
        
        Thread.sleep(350);
        periodic.cancel(false);
        scheduler.shutdown();
    }
}
```

---

## 21.4 CompletableFuture (Java 8+)

```java
import java.util.concurrent.*;
import java.util.*;
import java.util.stream.*;

public class CompletableFutureDemo {
    
    static String fetchData(String url) {
        try { Thread.sleep(200); } catch (InterruptedException e) {}
        return "Data from " + url;
    }
    
    static int processData(String data) {
        try { Thread.sleep(100); } catch (InterruptedException e) {}
        return data.length();
    }
    
    static String formatResult(int size) {
        return "Processed " + size + " chars";
    }
    
    public static void main(String[] args) throws Exception {
        // Basic async operation
        CompletableFuture<String> cf = CompletableFuture.supplyAsync(
            () -> fetchData("http://api.example.com/users")
        );
        
        System.out.println("Doing other work while fetching...");
        Thread.sleep(50);
        System.out.println("Result: " + cf.get());
        
        // Chaining: thenApply, thenAccept, thenRun
        CompletableFuture.supplyAsync(() -> fetchData("http://example.com/data"))
            .thenApply(data -> processData(data))    // transform result
            .thenApply(size -> formatResult(size))   // transform again
            .thenAccept(result -> System.out.println("Final: " + result))  // consume
            .join();  // wait for completion
        
        // Error handling
        CompletableFuture.supplyAsync(() -> {
                if (Math.random() < 0.5) throw new RuntimeException("Fetch failed!");
                return "success";
            })
            .exceptionally(ex -> "Error: " + ex.getMessage())
            .thenAccept(System.out::println)
            .join();
        
        // Combine two futures
        CompletableFuture<String> user = CompletableFuture.supplyAsync(
            () -> fetchData("users/1"));
        CompletableFuture<String> profile = CompletableFuture.supplyAsync(
            () -> fetchData("profiles/1"));
        
        CompletableFuture<String> combined = user.thenCombine(profile,
            (u, p) -> u + " + " + p);
        System.out.println("Combined: " + combined.get());
        
        // Wait for all
        List<CompletableFuture<String>> fetches = List.of(
            CompletableFuture.supplyAsync(() -> fetchData("endpoint1")),
            CompletableFuture.supplyAsync(() -> fetchData("endpoint2")),
            CompletableFuture.supplyAsync(() -> fetchData("endpoint3"))
        );
        
        CompletableFuture<Void> allOf = CompletableFuture.allOf(
            fetches.toArray(new CompletableFuture[0]));
        allOf.join();
        
        System.out.println("All fetches done:");
        fetches.forEach(f -> System.out.println("  " + f.join()));
        
        // First completed
        CompletableFuture<String> anyOf = CompletableFuture.anyOf(
            CompletableFuture.supplyAsync(() -> { try {Thread.sleep(300);} catch(Exception e){} return "slow"; }),
            CompletableFuture.supplyAsync(() -> { try {Thread.sleep(100);} catch(Exception e){} return "fast"; }),
            CompletableFuture.supplyAsync(() -> { try {Thread.sleep(200);} catch(Exception e){} return "medium"; })
        ).thenApply(o -> (String)o);
        System.out.println("First done: " + anyOf.get());
    }
}
```

---

## 21.5 Locks and Concurrent Collections

```java
import java.util.concurrent.*;
import java.util.concurrent.locks.*;
import java.util.*;

public class ConcurrentTools {
    
    // ReadWriteLock: multiple readers, single writer
    static class Cache<K, V> {
        private final Map<K, V> data = new HashMap<>();
        private final ReadWriteLock lock = new ReentrantReadWriteLock();
        
        V get(K key) {
            lock.readLock().lock();  // Multiple threads can read simultaneously
            try { return data.get(key); }
            finally { lock.readLock().unlock(); }
        }
        
        void put(K key, V value) {
            lock.writeLock().lock();  // Only one thread writes at a time
            try { data.put(key, value); }
            finally { lock.writeLock().unlock(); }
        }
        
        int size() {
            lock.readLock().lock();
            try { return data.size(); }
            finally { lock.readLock().unlock(); }
        }
    }
    
    // BlockingQueue: producer-consumer pattern
    static class ProducerConsumer {
        private final BlockingQueue<Integer> queue;
        private volatile boolean running = true;
        
        ProducerConsumer(int capacity) {
            queue = new LinkedBlockingQueue<>(capacity);
        }
        
        Thread makeProducer(String name, int count) {
            return new Thread(() -> {
                for (int i = 1; i <= count; i++) {
                    try {
                        queue.put(i);  // blocks if queue full
                        System.out.printf("[%s] produced: %d (queue size: %d)%n",
                            name, i, queue.size());
                        Thread.sleep(50);
                    } catch (InterruptedException e) { return; }
                }
                running = false;
            }, name);
        }
        
        Thread makeConsumer(String name) {
            return new Thread(() -> {
                while (running || !queue.isEmpty()) {
                    try {
                        Integer item = queue.poll(100, TimeUnit.MILLISECONDS);
                        if (item != null) {
                            System.out.printf("[%s] consumed: %d%n", name, item);
                        }
                    } catch (InterruptedException e) { return; }
                }
            }, name);
        }
    }
    
    public static void main(String[] args) throws InterruptedException {
        // Cache demo
        Cache<String, Integer> cache = new Cache<>();
        
        Thread writer = new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                cache.put("key" + i, i * 10);
                try { Thread.sleep(50); } catch (InterruptedException e) {}
            }
        });
        
        Thread reader = new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                System.out.println("Cache size: " + cache.size());
                try { Thread.sleep(30); } catch (InterruptedException e) {}
            }
        });
        
        writer.start(); reader.start();
        writer.join(); reader.join();
        
        System.out.println("Final cache: " + cache.size() + " entries");
        
        // Producer-Consumer
        System.out.println("\n=== Producer-Consumer ===");
        ProducerConsumer pc = new ProducerConsumer(3);
        Thread producer = pc.makeProducer("Producer", 6);
        Thread consumer1 = pc.makeConsumer("Consumer1");
        Thread consumer2 = pc.makeConsumer("Consumer2");
        
        producer.start(); consumer1.start(); consumer2.start();
        producer.join(); consumer1.join(); consumer2.join();
        
        // ConcurrentHashMap
        System.out.println("\n=== ConcurrentHashMap ===");
        ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
        
        Thread[] adders = new Thread[10];
        for (int i = 0; i < adders.length; i++) {
            final int tid = i;
            adders[i] = new Thread(() -> {
                map.put("key" + tid, tid * 10);
                map.merge("total", 1, Integer::sum);
            });
        }
        
        for (Thread t : adders) t.start();
        for (Thread t : adders) t.join();
        
        System.out.println("Map size: " + map.size());
        System.out.println("Total merges: " + map.get("total"));
    }
}
```

---

## 21.6 สรุป Part 21

ในบทนี้คุณได้เรียนรู้:

✅ Thread creation (extends Thread, implements Runnable, lambda)  
✅ Thread synchronization (synchronized, locks)  
✅ AtomicInteger  
✅ ExecutorService (fixed pool, scheduled)  
✅ Future and Callable  
✅ CompletableFuture  
✅ Concurrent collections (BlockingQueue, ConcurrentHashMap)  

---

*[← Part 20: Advanced Features](./part-20-java-advanced.md) | [Part 22: Design Patterns →](./part-22-design-patterns.md)*
