# Part 93: Java Generics ขั้นสูง
## หลักสูตร Java & Android Development - ระดับ World-Class

---

## 93.1 Bounded Type Parameters

```java
// Upper bound: T must extend Number
public static <T extends Number> double sum(List<T> list) {
    return list.stream().mapToDouble(Number::doubleValue).sum();
}

// Works with Integer, Double, Long, etc.
List<Integer> ints    = Arrays.asList(1, 2, 3);
List<Double>  doubles = Arrays.asList(1.1, 2.2, 3.3);
sum(ints);    // 6.0
sum(doubles); // 6.6

// Multiple bounds
public static <T extends Comparable<T> & Cloneable> T max(T a, T b) {
    return a.compareTo(b) >= 0 ? a : b;
}

// Wildcard: ? extends T (upper bounded - read only)
// Safe to read: all elements are AT LEAST Number
public static double totalArea(List<? extends Shape> shapes) {
    return shapes.stream().mapToDouble(Shape::area).sum();
}

// Wildcard: ? super T (lower bounded - write only)
// Safe to write: all elements are AT MOST Number's supertype
public static void addNumbers(List<? super Integer> list) {
    list.add(1);
    list.add(2);
    list.add(3);
}

// PECS: Producer Extends, Consumer Super
// Producer (source): use ? extends T  → you read FROM it
// Consumer (sink):   use ? super T    → you write TO it
public static <T> void copy(List<? extends T> src, List<? super T> dst) {
    for (T item : src) dst.add(item);
}
```

---

## 93.2 Generic Classes & Interfaces

```java
// Generic Result wrapper (used throughout Android apps)
public sealed class Result<T> {
    
    public static final class Success<T> extends Result<T> {
        public final T data;
        public Success(T data) { this.data = data; }
    }
    
    public static final class Error<T> extends Result<T> {
        public final String message;
        public final Throwable cause;
        
        public Error(String message, Throwable cause) {
            this.message = message;
            this.cause   = cause;
        }
    }
    
    public static final class Loading<T> extends Result<T> {}
    
    // Factory methods
    public static <T> Result<T> success(T data)      { return new Success<>(data); }
    public static <T> Result<T> error(String msg, Throwable e) { return new Error<>(msg, e); }
    public static <T> Result<T> loading()            { return new Loading<>(); }
    
    // Map: transform success value
    public <R> Result<R> map(java.util.function.Function<T, R> mapper) {
        if (this instanceof Success) {
            return success(mapper.apply(((Success<T>) this).data));
        } else if (this instanceof Error) {
            Error<T> err = (Error<T>) this;
            return error(err.message, err.cause);
        }
        return loading();
    }
    
    public boolean isSuccess() { return this instanceof Success; }
    public boolean isError()   { return this instanceof Error; }
    public boolean isLoading() { return this instanceof Loading; }
    
    // Get data or throw
    public T getOrThrow() {
        if (this instanceof Success) return ((Success<T>) this).data;
        if (this instanceof Error) throw new RuntimeException(((Error<T>) this).message);
        throw new IllegalStateException("Still loading");
    }
    
    public T getOrDefault(T defaultValue) {
        return isSuccess() ? ((Success<T>) this).data : defaultValue;
    }
}

// Generic UseCase
public abstract class UseCase<P, R> {
    
    public abstract CompletableFuture<Result<R>> execute(P params);
    
    // Convenience: no params
    public CompletableFuture<Result<R>> execute() {
        return execute(null);
    }
}

// Concrete use case
public class GetProductsUseCase extends UseCase<String, List<Product>> {
    
    private final ProductRepository repository;
    
    public GetProductsUseCase(ProductRepository repository) {
        this.repository = repository;
    }
    
    @Override
    public CompletableFuture<Result<List<Product>>> execute(String categoryId) {
        return CompletableFuture.supplyAsync(() -> {
            try {
                List<Product> products = repository.getProductsSync(categoryId);
                return Result.success(products);
            } catch (Exception e) {
                return Result.error(e.getMessage(), e);
            }
        });
    }
}
```

---

## 93.3 Generic Utilities

```java
public class GenericUtils {
    
    // Safely cast with Optional
    @SuppressWarnings("unchecked")
    public static <T> Optional<T> safeCast(Object obj, Class<T> type) {
        if (type.isInstance(obj)) return Optional.of((T) obj);
        return Optional.empty();
    }
    
    // Paginated result wrapper
    public static class Page<T> {
        private final List<T> items;
        private final int     pageNumber;
        private final int     pageSize;
        private final long    totalElements;
        
        public Page(List<T> items, int pageNumber, int pageSize, long totalElements) {
            this.items = Collections.unmodifiableList(items);
            this.pageNumber = pageNumber;
            this.pageSize = pageSize;
            this.totalElements = totalElements;
        }
        
        public long getTotalPages() {
            return (totalElements + pageSize - 1) / pageSize;
        }
        
        public boolean hasNext()     { return pageNumber < getTotalPages() - 1; }
        public boolean hasPrevious() { return pageNumber > 0; }
        public boolean isFirst()     { return pageNumber == 0; }
        public boolean isLast()      { return !hasNext(); }
        
        public List<T>  getItems()        { return items; }
        public int      getPageNumber()   { return pageNumber; }
        public int      getPageSize()     { return pageSize; }
        public long     getTotalElements(){ return totalElements; }
        
        // Map items
        public <R> Page<R> map(java.util.function.Function<T, R> mapper) {
            List<R> mapped = items.stream().map(mapper).collect(Collectors.toList());
            return new Page<>(mapped, pageNumber, pageSize, totalElements);
        }
    }
    
    // Pair
    public static class Pair<A, B> {
        public final A first;
        public final B second;
        
        private Pair(A first, B second) { this.first = first; this.second = second; }
        
        public static <A, B> Pair<A, B> of(A a, B b) { return new Pair<>(a, b); }
        
        public <R> Pair<R, B> mapFirst(java.util.function.Function<A, R> f) {
            return Pair.of(f.apply(first), second);
        }
        
        @Override
        public String toString() { return "(" + first + ", " + second + ")"; }
    }
}
```

---

## 93.4 สรุป Part 93

ในบทนี้คุณได้เรียนรู้:

✅ Upper bounded generics (extends)  
✅ Multiple bounds  
✅ Wildcards (? extends, ? super)  
✅ PECS: Producer Extends, Consumer Super  
✅ Sealed Result<T> class (Success/Error/Loading)  
✅ Generic UseCase base class  
✅ Paginated Page<T> wrapper  
✅ Generic Pair<A,B>  

---

*[← Part 92: Architecture Patterns](./part-92-architecture-patterns.md) | [Part 94: Production-Ready Error Handling →](./part-94-error-handling.md)*
