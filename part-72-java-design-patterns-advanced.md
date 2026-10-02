# Part 72: Advanced Design Patterns
## หลักสูตร Java & Android Development - ระดับ World-Class

---

## 72.1 Repository Pattern (ขั้นสูง)

```java
// Generic repository interface
public interface Repository<T, ID> {
    LiveData<T> findById(ID id);
    LiveData<List<T>> findAll();
    CompletableFuture<T> save(T entity);
    CompletableFuture<Void> delete(ID id);
    CompletableFuture<List<T>> findAllByFilter(Predicate<T> filter);
}

// BaseRepository - common implementation
public abstract class BaseRepository<T, ID> implements Repository<T, ID> {
    
    protected final ExecutorService executor = AppThreadPool.IO_EXECUTOR;
    
    @Override
    public CompletableFuture<List<T>> findAllByFilter(Predicate<T> filter) {
        return CompletableFuture.supplyAsync(() -> {
            List<T> all = findAllSync();
            return all.stream().filter(filter).collect(Collectors.toList());
        }, executor);
    }
    
    protected abstract List<T> findAllSync();
}
```

---

## 72.2 Event Bus Pattern

```java
// EventBus.java - decoupled communication between components
public class EventBus {
    
    private static EventBus instance;
    
    // Map: event type → list of handlers
    private final ConcurrentHashMap<Class<?>, List<EventHandler<?>>> handlers =
        new ConcurrentHashMap<>();
    
    private final Handler mainHandler = new Handler(Looper.getMainLooper());
    
    public static EventBus getInstance() {
        if (instance == null) instance = new EventBus();
        return instance;
    }
    
    @SuppressWarnings("unchecked")
    public <T> void subscribe(Class<T> eventType, EventHandler<T> handler) {
        handlers.computeIfAbsent(eventType, k -> new CopyOnWriteArrayList<>()).add(handler);
    }
    
    public <T> void unsubscribe(Class<T> eventType, EventHandler<T> handler) {
        List<EventHandler<?>> list = handlers.get(eventType);
        if (list != null) list.remove(handler);
    }
    
    @SuppressWarnings("unchecked")
    public <T> void publish(T event) {
        List<EventHandler<?>> list = handlers.get(event.getClass());
        if (list == null) return;
        
        for (EventHandler<?> handler : list) {
            EventHandler<T> typedHandler = (EventHandler<T>) handler;
            if (Looper.myLooper() == Looper.getMainLooper()) {
                typedHandler.handle(event);
            } else {
                mainHandler.post(() -> typedHandler.handle(event));
            }
        }
    }
    
    @FunctionalInterface
    public interface EventHandler<T> {
        void handle(T event);
    }
}

// Events
public class UserLoggedInEvent {
    public final String userId;
    public final String email;
    UserLoggedInEvent(String userId, String email) {
        this.userId = userId;
        this.email  = email;
    }
}

public class CartItemAddedEvent {
    public final String productId;
    public final int quantity;
    CartItemAddedEvent(String productId, int quantity) {
        this.productId = productId;
        this.quantity  = quantity;
    }
}

// Subscribe
EventBus.getInstance().subscribe(UserLoggedInEvent.class, event -> {
    updateUI("Welcome " + event.email);
    AnalyticsHelper.get().setUserId(event.userId);
});

// Publish
EventBus.getInstance().publish(new UserLoggedInEvent(user.getId(), user.getEmail()));
```

---

## 72.3 State Machine Pattern

```java
// OrderStateMachine.java
public class OrderStateMachine {
    
    public enum State {
        PENDING, CONFIRMED, PROCESSING, SHIPPED, DELIVERED, CANCELLED
    }
    
    public enum Event {
        CONFIRM, START_PROCESSING, SHIP, DELIVER, CANCEL
    }
    
    private State currentState;
    private final List<StateChangeListener> listeners = new ArrayList<>();
    
    // Transition table: (currentState, event) → nextState
    private static final Map<Pair<State, Event>, State> TRANSITIONS = new HashMap<>();
    
    static {
        TRANSITIONS.put(Pair.of(State.PENDING, Event.CONFIRM),       State.CONFIRMED);
        TRANSITIONS.put(Pair.of(State.PENDING, Event.CANCEL),        State.CANCELLED);
        TRANSITIONS.put(Pair.of(State.CONFIRMED, Event.START_PROCESSING), State.PROCESSING);
        TRANSITIONS.put(Pair.of(State.CONFIRMED, Event.CANCEL),      State.CANCELLED);
        TRANSITIONS.put(Pair.of(State.PROCESSING, Event.SHIP),       State.SHIPPED);
        TRANSITIONS.put(Pair.of(State.SHIPPED, Event.DELIVER),       State.DELIVERED);
    }
    
    public OrderStateMachine() {
        this.currentState = State.PENDING;
    }
    
    public boolean transition(Event event) {
        State nextState = TRANSITIONS.get(Pair.of(currentState, event));
        if (nextState == null) return false;  // invalid transition
        
        State previousState = currentState;
        currentState = nextState;
        notifyListeners(previousState, event, nextState);
        return true;
    }
    
    public State getState() { return currentState; }
    
    public boolean canTransition(Event event) {
        return TRANSITIONS.containsKey(Pair.of(currentState, event));
    }
    
    public void addStateChangeListener(StateChangeListener listener) {
        listeners.add(listener);
    }
    
    private void notifyListeners(State from, Event event, State to) {
        for (StateChangeListener l : listeners) l.onStateChanged(from, event, to);
    }
    
    public interface StateChangeListener {
        void onStateChanged(State from, Event event, State to);
    }
    
    // Helper: check if terminal state
    public boolean isTerminal() {
        return currentState == State.DELIVERED || currentState == State.CANCELLED;
    }
}

// Usage
OrderStateMachine machine = new OrderStateMachine();
machine.addStateChangeListener((from, event, to) -> {
    Log.d("Order", from + " --[" + event + "]--> " + to);
    updateOrderUI(to);
});

machine.transition(Event.CONFIRM);        // PENDING → CONFIRMED
machine.transition(Event.START_PROCESSING); // CONFIRMED → PROCESSING
machine.transition(Event.SHIP);            // PROCESSING → SHIPPED
```

---

## 72.4 Chain of Responsibility

```java
// ValidationChain.java - chained validators
public abstract class Validator<T> {
    
    private Validator<T> next;
    
    public Validator<T> setNext(Validator<T> next) {
        this.next = next;
        return next;
    }
    
    public ValidationResult validate(T input) {
        ValidationResult result = doValidate(input);
        if (!result.isValid()) return result;
        if (next != null) return next.validate(input);
        return ValidationResult.valid();
    }
    
    protected abstract ValidationResult doValidate(T input);
}

// Concrete validators
public class NotEmptyValidator extends Validator<String> {
    private final String fieldName;
    
    NotEmptyValidator(String fieldName) { this.fieldName = fieldName; }
    
    @Override
    protected ValidationResult doValidate(String input) {
        if (input == null || input.trim().isEmpty()) {
            return ValidationResult.invalid(fieldName + " ต้องไม่ว่างเปล่า");
        }
        return ValidationResult.valid();
    }
}

public class MinLengthValidator extends Validator<String> {
    private final int minLength;
    
    MinLengthValidator(int minLength) { this.minLength = minLength; }
    
    @Override
    protected ValidationResult doValidate(String input) {
        if (input.length() < minLength) {
            return ValidationResult.invalid("ต้องมีอย่างน้อย " + minLength + " ตัวอักษร");
        }
        return ValidationResult.valid();
    }
}

public class EmailFormatValidator extends Validator<String> {
    @Override
    protected ValidationResult doValidate(String input) {
        if (!android.util.Patterns.EMAIL_ADDRESS.matcher(input).matches()) {
            return ValidationResult.invalid("รูปแบบอีเมลไม่ถูกต้อง");
        }
        return ValidationResult.valid();
    }
}

// Build chain
Validator<String> emailValidator = new NotEmptyValidator("อีเมล");
emailValidator.setNext(new EmailFormatValidator());

Validator<String> passwordValidator = new NotEmptyValidator("รหัสผ่าน")
    .setNext(new MinLengthValidator(8));

// Use
ValidationResult emailResult = emailValidator.validate(email);
ValidationResult pwResult = passwordValidator.validate(password);
```

---

## 72.5 สรุป Part 72

ในบทนี้คุณได้เรียนรู้:

✅ Generic Repository pattern  
✅ Event Bus (decoupled pub/sub)  
✅ State Machine (transition table)  
✅ Chain of Responsibility (validation chain)  
✅ Real-world usage ของแต่ละ pattern  

---

*[← Part 71: Reactive](./part-71-java-reactive.md) | [Part 73: Android App Performance →](./part-73-android-app-performance.md)*
