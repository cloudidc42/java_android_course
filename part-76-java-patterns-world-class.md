# Part 76: World-Class Java Patterns
## หลักสูตร Java & Android Development - ระดับ World-Class

---

## 76.1 Functional Interfaces & Lambdas ขั้นสูง

```java
// Custom functional interfaces
@FunctionalInterface
public interface TriFunction<A, B, C, R> {
    R apply(A a, B b, C c);
}

@FunctionalInterface
public interface ThrowingSupplier<T> {
    T get() throws Exception;
}

// Utility: wrap checked exception in unchecked
public static <T> java.util.function.Supplier<T> wrap(ThrowingSupplier<T> supplier) {
    return () -> {
        try { return supplier.get(); }
        catch (Exception e) { throw new RuntimeException(e); }
    };
}

// Method references
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

// Instance method reference
names.stream().map(String::toLowerCase).collect(Collectors.toList());

// Static method reference
names.stream().map(MyUtils::process).collect(Collectors.toList());

// Constructor reference
names.stream().map(StringBuilder::new).collect(Collectors.toList());

// Composition
java.util.function.Function<String, String> trim = String::trim;
java.util.function.Function<String, String> lower = String::toLowerCase;
java.util.function.Function<String, String> normalize = trim.andThen(lower);

names.stream().map(normalize).collect(Collectors.toList());
```

---

## 76.2 Builder Pattern with Validation

```java
// EmailMessage.java - immutable builder
public final class EmailMessage {
    
    private final String to;
    private final String from;
    private final String subject;
    private final String body;
    private final List<String> cc;
    private final List<String> attachments;
    private final boolean isHtml;
    private final int priority;  // 1=high, 3=normal, 5=low
    
    private EmailMessage(Builder builder) {
        this.to          = builder.to;
        this.from        = builder.from;
        this.subject     = builder.subject;
        this.body        = builder.body;
        this.cc          = Collections.unmodifiableList(builder.cc);
        this.attachments = Collections.unmodifiableList(builder.attachments);
        this.isHtml      = builder.isHtml;
        this.priority    = builder.priority;
    }
    
    public static Builder builder(String to, String from) {
        return new Builder(to, from);
    }
    
    public static class Builder {
        private final String to;
        private final String from;
        private String subject = "";
        private String body = "";
        private List<String> cc = new ArrayList<>();
        private List<String> attachments = new ArrayList<>();
        private boolean isHtml = false;
        private int priority = 3;
        
        private Builder(String to, String from) {
            if (!isValidEmail(to))   throw new IllegalArgumentException("Invalid 'to': " + to);
            if (!isValidEmail(from)) throw new IllegalArgumentException("Invalid 'from': " + from);
            this.to = to;
            this.from = from;
        }
        
        public Builder subject(String subject) {
            this.subject = Objects.requireNonNull(subject);
            return this;
        }
        
        public Builder body(String body) {
            this.body = Objects.requireNonNull(body);
            return this;
        }
        
        public Builder htmlBody(String html) {
            this.body = html;
            this.isHtml = true;
            return this;
        }
        
        public Builder cc(String... emails) {
            for (String email : emails) {
                if (isValidEmail(email)) cc.add(email);
            }
            return this;
        }
        
        public Builder attach(String filePath) {
            attachments.add(filePath);
            return this;
        }
        
        public Builder priority(int p) {
            if (p < 1 || p > 5) throw new IllegalArgumentException("Priority 1-5");
            this.priority = p;
            return this;
        }
        
        public EmailMessage build() {
            if (subject.isEmpty()) throw new IllegalStateException("Subject required");
            if (body.isEmpty())    throw new IllegalStateException("Body required");
            return new EmailMessage(this);
        }
        
        private boolean isValidEmail(String email) {
            return email != null && email.contains("@");
        }
    }
    
    // Getters
    public String       getTo()          { return to; }
    public String       getFrom()        { return from; }
    public String       getSubject()     { return subject; }
    public String       getBody()        { return body; }
    public List<String> getCc()          { return cc; }
    public List<String> getAttachments() { return attachments; }
    public boolean      isHtml()         { return isHtml; }
    public int          getPriority()    { return priority; }
}

// Usage
EmailMessage email = EmailMessage
    .builder("customer@example.com", "shop@example.com")
    .subject("Order Confirmation #12345")
    .htmlBody("<h1>Thank you!</h1><p>Your order is confirmed.</p>")
    .cc("manager@example.com")
    .priority(2)
    .build();
```

---

## 76.3 Specification Pattern

```java
// Specification.java - complex business rules as reusable objects
public interface Specification<T> {
    boolean isSatisfiedBy(T candidate);
    
    default Specification<T> and(Specification<T> other) {
        return candidate -> this.isSatisfiedBy(candidate) && other.isSatisfiedBy(candidate);
    }
    
    default Specification<T> or(Specification<T> other) {
        return candidate -> this.isSatisfiedBy(candidate) || other.isSatisfiedBy(candidate);
    }
    
    default Specification<T> not() {
        return candidate -> !this.isSatisfiedBy(candidate);
    }
}

// Concrete specifications
public class ActiveUserSpec implements Specification<User> {
    @Override
    public boolean isSatisfiedBy(User user) {
        return user.isActive() && user.getLastLoginAt() != null;
    }
}

public class PremiumUserSpec implements Specification<User> {
    @Override
    public boolean isSatisfiedBy(User user) {
        return "premium".equals(user.getSubscriptionType());
    }
}

public class RecentlyRegisteredSpec implements Specification<User> {
    private final int daysThreshold;
    
    public RecentlyRegisteredSpec(int days) { this.daysThreshold = days; }
    
    @Override
    public boolean isSatisfiedBy(User user) {
        long thresholdMs = daysThreshold * 86_400_000L;
        return System.currentTimeMillis() - user.getCreatedAt() <= thresholdMs;
    }
}

// Compose specifications
Specification<User> activeSpec = new ActiveUserSpec();
Specification<User> premiumSpec = new PremiumUserSpec();
Specification<User> newSpec = new RecentlyRegisteredSpec(30);

// Active AND (Premium OR New)
Specification<User> targetUsers = activeSpec.and(premiumSpec.or(newSpec));

// Filter a list
List<User> eligible = users.stream()
    .filter(targetUsers::isSatisfiedBy)
    .collect(Collectors.toList());
```

---

## 76.4 Decorator Pattern ขั้นสูง

```java
// Cache decorator
public class CachedProductRepository implements ProductRepository {
    
    private final ProductRepository delegate;
    private final InMemoryCache<String, Product> cache;
    
    public CachedProductRepository(ProductRepository delegate) {
        this.delegate = delegate;
        this.cache = new InMemoryCache<>(5 * 60 * 1000);  // 5 min TTL
    }
    
    @Override
    public CompletableFuture<Product> getById(String id) {
        Product cached = cache.get(id);
        if (cached != null) {
            return CompletableFuture.completedFuture(cached);
        }
        return delegate.getById(id).thenApply(product -> {
            if (product != null) cache.put(id, product);
            return product;
        });
    }
    
    @Override
    public CompletableFuture<Void> save(Product product) {
        cache.put(product.getId(), product);  // update cache
        return delegate.save(product);
    }
}

// Logging decorator
public class LoggingProductRepository implements ProductRepository {
    
    private final ProductRepository delegate;
    
    public LoggingProductRepository(ProductRepository delegate) {
        this.delegate = delegate;
    }
    
    @Override
    public CompletableFuture<Product> getById(String id) {
        Log.d("Repo", "getById: " + id);
        long start = System.currentTimeMillis();
        return delegate.getById(id)
            .whenComplete((result, error) -> {
                long duration = System.currentTimeMillis() - start;
                if (error != null) {
                    Log.e("Repo", "getById failed: " + error.getMessage());
                } else {
                    Log.d("Repo", "getById took " + duration + "ms");
                }
            });
    }
}

// Compose decorators
ProductRepository repo = new LoggingProductRepository(
    new CachedProductRepository(
        new NetworkProductRepository(apiService)));
```

---

## 76.5 สรุป Part 76

ในบทนี้คุณได้เรียนรู้:

✅ Custom functional interfaces (@FunctionalInterface)  
✅ Checked exception wrapper  
✅ Method references (instance, static, constructor)  
✅ Function composition (andThen)  
✅ Builder with validation and immutability  
✅ Specification pattern (composable business rules)  
✅ Decorator pattern stacking (logging + caching)  

---

*[← Part 75: Real-World Project](./part-75-android-realworld-project.md) | [Part 77: Android Architecture Components Deep Dive →](./part-77-android-architecture-deep.md)*
