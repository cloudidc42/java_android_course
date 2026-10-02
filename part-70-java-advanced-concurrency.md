# Part 70: Java Advanced Concurrency
## หลักสูตร Java & Android Development - ระดับ World-Class

---

## 70.1 ThreadPoolExecutor ขั้นสูง

```java
// CustomThreadPoolExecutor.java
public class AppThreadPool {
    
    // CPU-bound tasks (e.g. image processing)
    public static final ExecutorService CPU_EXECUTOR = new ThreadPoolExecutor(
        Runtime.getRuntime().availableProcessors(),      // corePoolSize
        Runtime.getRuntime().availableProcessors() * 2,  // maximumPoolSize
        60, TimeUnit.SECONDS,                            // keepAliveTime
        new LinkedBlockingQueue<>(128),                  // work queue
        new ThreadFactory() {
            private final AtomicInteger count = new AtomicInteger(1);
            @Override
            public Thread newThread(Runnable r) {
                Thread t = new Thread(r, "cpu-pool-" + count.getAndIncrement());
                t.setPriority(Thread.NORM_PRIORITY);
                return t;
            }
        },
        new ThreadPoolExecutor.CallerRunsPolicy()  // rejection policy
    );
    
    // IO-bound tasks (network, disk)
    public static final ExecutorService IO_EXECUTOR = new ThreadPoolExecutor(
        4, 64,
        60, TimeUnit.SECONDS,
        new SynchronousQueue<>(),  // no queue - create new threads immediately
        r -> new Thread(r, "io-thread"),
        new ThreadPoolExecutor.CallerRunsPolicy()
    );
    
    // Single-thread for sequential writes
    public static final ExecutorService DISK_EXECUTOR =
        Executors.newSingleThreadExecutor(r -> new Thread(r, "disk-writer"));
    
    // Scheduled tasks
    public static final ScheduledExecutorService SCHEDULER =
        Executors.newScheduledThreadPool(2, r -> new Thread(r, "scheduler"));
}

// Usage
AppThreadPool.CPU_EXECUTOR.execute(() -> processImage(bitmap));
AppThreadPool.IO_EXECUTOR.execute(() -> fetchDataFromNetwork());

// Scheduled task
AppThreadPool.SCHEDULER.scheduleAtFixedRate(
    () -> syncData(),
    0, 30, TimeUnit.MINUTES
);
```

---

## 70.2 CompletableFuture Chains

```java
// Complex async pipeline
public CompletableFuture<List<Product>> fetchProductsWithDiscounts(String category) {
    return CompletableFuture
        // Step 1: Fetch products
        .supplyAsync(() -> apiService.getProducts(category), AppThreadPool.IO_EXECUTOR)
        
        // Step 2: Filter available
        .thenApplyAsync(products ->
            products.stream()
                .filter(p -> p.isAvailable() && p.getStock() > 0)
                .collect(Collectors.toList()),
            AppThreadPool.CPU_EXECUTOR)
        
        // Step 3: Fetch discounts for each (parallel)
        .thenComposeAsync(products -> {
            List<CompletableFuture<Product>> futures = products.stream()
                .map(p -> CompletableFuture.supplyAsync(
                    () -> applyDiscount(p), AppThreadPool.IO_EXECUTOR))
                .collect(Collectors.toList());
            
            return CompletableFuture.allOf(futures.toArray(new CompletableFuture[0]))
                .thenApply(v -> futures.stream()
                    .map(CompletableFuture::join)
                    .collect(Collectors.toList()));
        })
        
        // Step 4: Sort by price
        .thenApply(products -> products.stream()
            .sorted(Comparator.comparingDouble(Product::getDiscountedPrice))
            .collect(Collectors.toList()))
        
        // Handle errors
        .exceptionally(e -> {
            Log.e("Products", "Failed to load products", e);
            return Collections.emptyList();
        });
}

// Wait for multiple independent tasks
CompletableFuture<List<User>> usersFuture = fetchUsers();
CompletableFuture<List<Product>> productsFuture = fetchProducts();
CompletableFuture<Discount[]> discountsFuture = fetchDiscounts();

CompletableFuture.allOf(usersFuture, productsFuture, discountsFuture)
    .thenRun(() -> {
        List<User> users = usersFuture.join();
        List<Product> products = productsFuture.join();
        Discount[] discounts = discountsFuture.join();
        buildDashboard(users, products, discounts);
    });
```

---

## 70.3 CountDownLatch, CyclicBarrier, Semaphore

```java
// CountDownLatch: รอให้ทุก task เสร็จก่อน proceed
public void loadDashboardData() throws InterruptedException {
    CountDownLatch latch = new CountDownLatch(3);
    
    AtomicReference<List<User>> users = new AtomicReference<>();
    AtomicReference<Stats> stats = new AtomicReference<>();
    AtomicReference<List<Notification>> notifications = new AtomicReference<>();
    
    AppThreadPool.IO_EXECUTOR.execute(() -> {
        try { users.set(apiService.getUsers()); }
        catch (Exception e) { users.set(Collections.emptyList()); }
        finally { latch.countDown(); }
    });
    
    AppThreadPool.IO_EXECUTOR.execute(() -> {
        try { stats.set(apiService.getStats()); }
        catch (Exception e) { stats.set(new Stats()); }
        finally { latch.countDown(); }
    });
    
    AppThreadPool.IO_EXECUTOR.execute(() -> {
        try { notifications.set(apiService.getNotifications()); }
        catch (Exception e) { notifications.set(Collections.emptyList()); }
        finally { latch.countDown(); }
    });
    
    latch.await(30, TimeUnit.SECONDS);  // wait max 30s
    buildDashboard(users.get(), stats.get(), notifications.get());
}

// Semaphore: จำกัดจำนวน concurrent requests
public class RateLimitedApiClient {
    private final Semaphore semaphore = new Semaphore(5);  // max 5 concurrent
    
    public Response callApi(String url) throws Exception {
        semaphore.acquire();
        try {
            return httpClient.get(url);
        } finally {
            semaphore.release();
        }
    }
}

// CyclicBarrier: รอให้ทุก worker ถึง checkpoint พร้อมกัน
public void processInParallel(List<Item> items) throws Exception {
    int threadCount = 4;
    CyclicBarrier barrier = new CyclicBarrier(threadCount, () ->
        Log.d("Worker", "All workers completed phase"));
    
    List<List<Item>> partitions = partition(items, threadCount);
    
    for (List<Item> partition : partitions) {
        AppThreadPool.CPU_EXECUTOR.execute(() -> {
            try {
                processPartition(partition);
                barrier.await();  // wait for all workers
                mergeResults();
            } catch (Exception e) {
                Thread.currentThread().interrupt();
            }
        });
    }
}
```

---

## 70.4 Atomic Operations & Lock-free

```java
// AtomicInteger, AtomicLong, AtomicReference
public class StatisticsCounter {
    
    private final AtomicLong totalRequests = new AtomicLong(0);
    private final AtomicLong failedRequests = new AtomicLong(0);
    private final AtomicLong totalLatency = new AtomicLong(0);
    
    public void recordRequest(boolean success, long latencyMs) {
        totalRequests.incrementAndGet();
        totalLatency.addAndGet(latencyMs);
        if (!success) failedRequests.incrementAndGet();
    }
    
    public double getAverageLatency() {
        long total = totalRequests.get();
        return total > 0 ? (double) totalLatency.get() / total : 0;
    }
    
    public double getFailureRate() {
        long total = totalRequests.get();
        return total > 0 ? (double) failedRequests.get() / total : 0;
    }
}

// ConcurrentHashMap for thread-safe cache
public class InMemoryCache<K, V> {
    private final ConcurrentHashMap<K, CacheEntry<V>> cache = new ConcurrentHashMap<>();
    private final long defaultTtlMs;
    
    public InMemoryCache(long defaultTtlMs) { this.defaultTtlMs = defaultTtlMs; }
    
    public void put(K key, V value) {
        cache.put(key, new CacheEntry<>(value, System.currentTimeMillis() + defaultTtlMs));
    }
    
    public V get(K key) {
        CacheEntry<V> entry = cache.get(key);
        if (entry == null) return null;
        if (entry.isExpired()) { cache.remove(key); return null; }
        return entry.value;
    }
    
    public V getOrCompute(K key, java.util.function.Supplier<V> supplier) {
        return cache.compute(key, (k, existing) -> {
            if (existing != null && !existing.isExpired()) return existing;
            return new CacheEntry<>(supplier.get(), System.currentTimeMillis() + defaultTtlMs);
        }).value;
    }
    
    static class CacheEntry<V> {
        final V    value;
        final long expiryTime;
        
        CacheEntry(V value, long expiryTime) {
            this.value = value;
            this.expiryTime = expiryTime;
        }
        
        boolean isExpired() { return System.currentTimeMillis() > expiryTime; }
    }
}
```

---

## 70.5 สรุป Part 70

ในบทนี้คุณได้เรียนรู้:

✅ ThreadPoolExecutor (CPU, IO, disk, scheduled)  
✅ CompletableFuture chains (thenApplyAsync, thenComposeAsync, allOf)  
✅ CountDownLatch (รอทุก task เสร็จ)  
✅ Semaphore (rate limiting)  
✅ CyclicBarrier (checkpoint sync)  
✅ Atomic operations (AtomicLong, AtomicInteger)  
✅ ConcurrentHashMap with TTL cache  

---

*[← Part 69: Offline-First](./part-69-android-offline-first.md) | [Part 71: Java Reactive Programming →](./part-71-java-reactive.md)*
