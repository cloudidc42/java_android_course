# Part 22: Design Patterns
## หลักสูตร Java & Android Development - ระดับ Advanced

---

## 22.1 Creational Patterns

### Singleton Pattern

```java
public class SingletonDemo {
    
    // Thread-safe Singleton with double-checked locking
    static class DatabasePool {
        private static volatile DatabasePool instance;
        private List<String> connections = new ArrayList<>();
        private int maxConnections;
        
        private DatabasePool(int max) {
            this.maxConnections = max;
            for (int i = 0; i < max; i++) {
                connections.add("conn-" + i);
            }
            System.out.println("DatabasePool created with " + max + " connections");
        }
        
        static DatabasePool getInstance() {
            if (instance == null) {
                synchronized (DatabasePool.class) {
                    if (instance == null) {
                        instance = new DatabasePool(10);
                    }
                }
            }
            return instance;
        }
        
        synchronized String getConnection() {
            if (connections.isEmpty()) throw new RuntimeException("No connections available");
            return connections.remove(0);
        }
        
        synchronized void releaseConnection(String conn) {
            connections.add(conn);
        }
        
        int availableConnections() { return connections.size(); }
    }
    
    // Enum Singleton (simplest thread-safe way)
    enum AppConfig {
        INSTANCE;
        
        private final Map<String, String> config = new HashMap<>();
        
        AppConfig() {
            config.put("app.name", "MyApp");
            config.put("db.host", "localhost");
            config.put("version", "1.0.0");
        }
        
        String get(String key) { return config.getOrDefault(key, ""); }
        void set(String key, String value) { config.put(key, value); }
    }
    
    public static void main(String[] args) {
        DatabasePool pool1 = DatabasePool.getInstance();
        DatabasePool pool2 = DatabasePool.getInstance();
        System.out.println("Same instance: " + (pool1 == pool2));
        
        String conn = pool1.getConnection();
        System.out.println("Got: " + conn);
        System.out.println("Available: " + pool1.availableConnections());
        pool1.releaseConnection(conn);
        System.out.println("After release: " + pool1.availableConnections());
        
        // Enum singleton
        System.out.println(AppConfig.INSTANCE.get("app.name"));
        AppConfig.INSTANCE.set("theme", "dark");
        System.out.println(AppConfig.INSTANCE.get("theme"));
    }
}
```

### Factory Pattern

```java
public class FactoryDemo {
    
    interface Notification {
        void send(String recipient, String message);
        String getType();
    }
    
    static class EmailNotification implements Notification {
        @Override
        public void send(String recipient, String message) {
            System.out.printf("[EMAIL] To: %s | Message: %s%n", recipient, message);
        }
        @Override
        public String getType() { return "EMAIL"; }
    }
    
    static class SMSNotification implements Notification {
        @Override
        public void send(String recipient, String message) {
            System.out.printf("[SMS] To: %s | Message: %s%n", recipient, message);
        }
        @Override
        public String getType() { return "SMS"; }
    }
    
    static class PushNotification implements Notification {
        @Override
        public void send(String recipient, String message) {
            System.out.printf("[PUSH] To: %s | Message: %s%n", recipient, message);
        }
        @Override
        public String getType() { return "PUSH"; }
    }
    
    // Factory Method
    static class NotificationFactory {
        static Notification create(String type) {
            return switch (type.toUpperCase()) {
                case "EMAIL" -> new EmailNotification();
                case "SMS"   -> new SMSNotification();
                case "PUSH"  -> new PushNotification();
                default -> throw new IllegalArgumentException("Unknown type: " + type);
            };
        }
    }
    
    // Abstract Factory
    interface UIFactory {
        Button createButton();
        TextField createTextField();
    }
    
    interface Button { void click(); String getStyle(); }
    interface TextField { String getText(); void setText(String t); String getStyle(); }
    
    static class MaterialButton implements Button {
        @Override public void click() { System.out.println("[Material] Button clicked"); }
        @Override public String getStyle() { return "Material Design"; }
    }
    
    static class MaterialTextField implements TextField {
        private String text = "";
        @Override public String getText() { return text; }
        @Override public void setText(String t) { this.text = t; }
        @Override public String getStyle() { return "Material TextField"; }
    }
    
    static class FluentButton implements Button {
        @Override public void click() { System.out.println("[Fluent] Button clicked"); }
        @Override public String getStyle() { return "Fluent Design"; }
    }
    
    static class FluentTextField implements TextField {
        private String text = "";
        @Override public String getText() { return text; }
        @Override public void setText(String t) { this.text = t; }
        @Override public String getStyle() { return "Fluent TextField"; }
    }
    
    static class MaterialFactory implements UIFactory {
        @Override public Button createButton() { return new MaterialButton(); }
        @Override public TextField createTextField() { return new MaterialTextField(); }
    }
    
    static class FluentFactory implements UIFactory {
        @Override public Button createButton() { return new FluentButton(); }
        @Override public TextField createTextField() { return new FluentTextField(); }
    }
    
    static void buildUI(UIFactory factory) {
        Button btn = factory.createButton();
        TextField tf = factory.createTextField();
        
        System.out.println("Button: " + btn.getStyle());
        System.out.println("TextField: " + tf.getStyle());
        btn.click();
        tf.setText("Hello");
        System.out.println("Text: " + tf.getText());
    }
    
    public static void main(String[] args) {
        // Factory Method
        String[] types = {"EMAIL", "SMS", "PUSH"};
        for (String type : types) {
            Notification n = NotificationFactory.create(type);
            n.send("user@example.com", "Hello World");
        }
        
        // Abstract Factory
        System.out.println("\n=== Material UI ===");
        buildUI(new MaterialFactory());
        
        System.out.println("\n=== Fluent UI ===");
        buildUI(new FluentFactory());
    }
}
```

### Builder Pattern

```java
public class BuilderDemo {
    
    static class HttpRequest {
        private final String method;
        private final String url;
        private final Map<String, String> headers;
        private final Map<String, String> params;
        private final String body;
        private final int timeout;
        private final boolean followRedirects;
        
        private HttpRequest(Builder builder) {
            this.method = builder.method;
            this.url = builder.url;
            this.headers = Collections.unmodifiableMap(new HashMap<>(builder.headers));
            this.params = Collections.unmodifiableMap(new HashMap<>(builder.params));
            this.body = builder.body;
            this.timeout = builder.timeout;
            this.followRedirects = builder.followRedirects;
        }
        
        @Override
        public String toString() {
            StringBuilder sb = new StringBuilder();
            sb.append(method).append(" ").append(url);
            if (!params.isEmpty()) sb.append("?").append(
                params.entrySet().stream()
                    .map(e -> e.getKey() + "=" + e.getValue())
                    .collect(java.util.stream.Collectors.joining("&")));
            sb.append("\n");
            headers.forEach((k, v) -> sb.append(k).append(": ").append(v).append("\n"));
            if (body != null) sb.append("\n").append(body);
            return sb.toString();
        }
        
        static class Builder {
            private String method = "GET";
            private String url;
            private Map<String, String> headers = new HashMap<>();
            private Map<String, String> params = new HashMap<>();
            private String body;
            private int timeout = 30;
            private boolean followRedirects = true;
            
            Builder(String url) {
                if (url == null || url.isBlank()) throw new IllegalArgumentException("URL required");
                this.url = url;
            }
            
            Builder method(String method) { this.method = method.toUpperCase(); return this; }
            Builder get() { return method("GET"); }
            Builder post() { return method("POST"); }
            Builder put() { return method("PUT"); }
            Builder delete() { return method("DELETE"); }
            
            Builder header(String key, String value) { headers.put(key, value); return this; }
            Builder param(String key, String value) { params.put(key, value); return this; }
            Builder body(String body) { this.body = body; return this; }
            Builder timeout(int seconds) { this.timeout = seconds; return this; }
            Builder noRedirects() { this.followRedirects = false; return this; }
            Builder auth(String token) { return header("Authorization", "Bearer " + token); }
            Builder json() { return header("Content-Type", "application/json"); }
            
            HttpRequest build() { return new HttpRequest(this); }
        }
    }
    
    public static void main(String[] args) {
        // GET request
        HttpRequest get = new HttpRequest.Builder("https://api.example.com/users")
            .get()
            .header("Accept", "application/json")
            .param("page", "1")
            .param("limit", "20")
            .auth("mytoken123")
            .build();
        
        System.out.println("=== GET ===");
        System.out.println(get);
        
        // POST request
        HttpRequest post = new HttpRequest.Builder("https://api.example.com/users")
            .post()
            .json()
            .auth("mytoken123")
            .body("{\"name\":\"Alice\",\"email\":\"alice@example.com\"}")
            .timeout(60)
            .build();
        
        System.out.println("=== POST ===");
        System.out.println(post);
    }
}
```

---

## 22.2 Structural Patterns

### Decorator Pattern

```java
public class DecoratorDemo {
    
    interface Coffee {
        String getDescription();
        double getCost();
    }
    
    // Base component
    static class SimpleCoffee implements Coffee {
        @Override public String getDescription() { return "Coffee"; }
        @Override public double getCost() { return 20.0; }
    }
    
    // Abstract decorator
    static abstract class CoffeeDecorator implements Coffee {
        protected Coffee coffee;
        CoffeeDecorator(Coffee coffee) { this.coffee = coffee; }
    }
    
    // Concrete decorators
    static class MilkDecorator extends CoffeeDecorator {
        MilkDecorator(Coffee c) { super(c); }
        @Override public String getDescription() { return coffee.getDescription() + ", Milk"; }
        @Override public double getCost() { return coffee.getCost() + 10; }
    }
    
    static class SugarDecorator extends CoffeeDecorator {
        SugarDecorator(Coffee c) { super(c); }
        @Override public String getDescription() { return coffee.getDescription() + ", Sugar"; }
        @Override public double getCost() { return coffee.getCost() + 5; }
    }
    
    static class WhipDecorator extends CoffeeDecorator {
        WhipDecorator(Coffee c) { super(c); }
        @Override public String getDescription() { return coffee.getDescription() + ", Whipped Cream"; }
        @Override public double getCost() { return coffee.getCost() + 15; }
    }
    
    static class VanillaDecorator extends CoffeeDecorator {
        VanillaDecorator(Coffee c) { super(c); }
        @Override public String getDescription() { return coffee.getDescription() + ", Vanilla"; }
        @Override public double getCost() { return coffee.getCost() + 8; }
    }
    
    public static void main(String[] args) {
        Coffee coffee1 = new SimpleCoffee();
        System.out.printf("%-40s %.0f THB%n", coffee1.getDescription(), coffee1.getCost());
        
        Coffee coffee2 = new MilkDecorator(new SugarDecorator(new SimpleCoffee()));
        System.out.printf("%-40s %.0f THB%n", coffee2.getDescription(), coffee2.getCost());
        
        Coffee latte = new WhipDecorator(new VanillaDecorator(new MilkDecorator(
            new MilkDecorator(new SimpleCoffee()))));
        System.out.printf("%-40s %.0f THB%n", latte.getDescription(), latte.getCost());
        
        // Triple with extra whip
        Coffee venti = new WhipDecorator(new WhipDecorator(
            new SugarDecorator(new MilkDecorator(new SimpleCoffee()))));
        System.out.printf("%-40s %.0f THB%n", venti.getDescription(), venti.getCost());
    }
}
```

### Observer Pattern

```java
import java.util.*;

public class ObserverDemo {
    
    interface Observer<T> {
        void update(String event, T data);
    }
    
    static class EventEmitter<T> {
        private final Map<String, List<Observer<T>>> listeners = new HashMap<>();
        
        void on(String event, Observer<T> observer) {
            listeners.computeIfAbsent(event, k -> new ArrayList<>()).add(observer);
        }
        
        void off(String event, Observer<T> observer) {
            List<Observer<T>> list = listeners.get(event);
            if (list != null) list.remove(observer);
        }
        
        void emit(String event, T data) {
            List<Observer<T>> list = listeners.getOrDefault(event, Collections.emptyList());
            list.forEach(obs -> obs.update(event, data));
        }
    }
    
    // Real-world example: Stock market
    record Stock(String symbol, double price, double change) {}
    
    static class StockMarket extends EventEmitter<Stock> {
        private Map<String, Double> prices = new HashMap<>();
        
        void updatePrice(String symbol, double newPrice) {
            double oldPrice = prices.getOrDefault(symbol, newPrice);
            double change = newPrice - oldPrice;
            prices.put(symbol, newPrice);
            
            Stock stock = new Stock(symbol, newPrice, change);
            emit("price_change", stock);
            
            if (change > 0) emit("price_up", stock);
            else if (change < 0) emit("price_down", stock);
        }
    }
    
    public static void main(String[] args) {
        StockMarket market = new StockMarket();
        
        // Register observers
        market.on("price_change", (event, stock) ->
            System.out.printf("[Ticker] %s: %.2f (%+.2f)%n",
                stock.symbol(), stock.price(), stock.change()));
        
        market.on("price_up", (event, stock) ->
            System.out.printf("[Alert] %s UP by %.2f!%n", stock.symbol(), stock.change()));
        
        market.on("price_down", (event, stock) ->
            System.out.printf("[Alert] %s DOWN by %.2f!%n", stock.symbol(), stock.change()));
        
        // Portfolio observer
        Map<String, Integer> portfolio = Map.of("AAPL", 100, "GOOGL", 50);
        market.on("price_change", (event, stock) -> {
            int shares = portfolio.getOrDefault(stock.symbol(), 0);
            if (shares > 0) {
                double gain = stock.change() * shares;
                System.out.printf("[Portfolio] %s x%d = %+.0f THB%n",
                    stock.symbol(), shares, gain);
            }
        });
        
        // Simulate price changes
        System.out.println("=== Market Update ===");
        market.updatePrice("AAPL", 150.0);
        System.out.println();
        market.updatePrice("AAPL", 155.5);
        System.out.println();
        market.updatePrice("GOOGL", 2800.0);
        System.out.println();
        market.updatePrice("AAPL", 148.0);
    }
}
```

---

## 22.3 Behavioral Patterns

### Strategy Pattern

```java
import java.util.*;

public class StrategyDemo {
    
    // Sort strategy
    interface SortStrategy {
        <T extends Comparable<T>> void sort(List<T> list);
        String getName();
    }
    
    static class BubbleSort implements SortStrategy {
        @Override
        public <T extends Comparable<T>> void sort(List<T> list) {
            int n = list.size();
            for (int i = 0; i < n - 1; i++) {
                for (int j = 0; j < n - i - 1; j++) {
                    if (list.get(j).compareTo(list.get(j + 1)) > 0) {
                        T temp = list.get(j);
                        list.set(j, list.get(j + 1));
                        list.set(j + 1, temp);
                    }
                }
            }
        }
        @Override public String getName() { return "Bubble Sort"; }
    }
    
    static class QuickSort implements SortStrategy {
        @Override
        public <T extends Comparable<T>> void sort(List<T> list) {
            Collections.sort(list);  // simplified (uses Timsort)
        }
        @Override public String getName() { return "Quick Sort (Collections.sort)"; }
    }
    
    static class ReverseSort implements SortStrategy {
        @Override
        public <T extends Comparable<T>> void sort(List<T> list) {
            list.sort(Comparator.reverseOrder());
        }
        @Override public String getName() { return "Reverse Sort"; }
    }
    
    static class Sorter<T extends Comparable<T>> {
        private SortStrategy strategy;
        private List<T> data;
        
        Sorter(List<T> data) {
            this.data = new ArrayList<>(data);
            this.strategy = new QuickSort();
        }
        
        void setStrategy(SortStrategy strategy) { this.strategy = strategy; }
        
        List<T> sort() {
            List<T> copy = new ArrayList<>(data);
            strategy.sort(copy);
            return copy;
        }
        
        void benchmark() {
            List<T> copy = new ArrayList<>(data);
            long start = System.nanoTime();
            strategy.sort(copy);
            long time = System.nanoTime() - start;
            System.out.printf("[%s] Time: %.3f ms, Result: %s...%n",
                strategy.getName(), time / 1_000_000.0,
                copy.subList(0, Math.min(5, copy.size())));
        }
    }
    
    // Pricing strategy
    interface PricingStrategy {
        double calculatePrice(double basePrice, int quantity);
        String getDescription();
    }
    
    static class RegularPricing implements PricingStrategy {
        @Override public double calculatePrice(double base, int qty) { return base * qty; }
        @Override public String getDescription() { return "Regular (no discount)"; }
    }
    
    static class BulkDiscountPricing implements PricingStrategy {
        @Override
        public double calculatePrice(double base, int qty) {
            if (qty >= 100) return base * qty * 0.75;
            if (qty >= 50)  return base * qty * 0.85;
            if (qty >= 10)  return base * qty * 0.90;
            return base * qty;
        }
        @Override public String getDescription() { return "Bulk Discount"; }
    }
    
    static class MemberPricing implements PricingStrategy {
        private double discount;
        MemberPricing(double discountPercent) { this.discount = discountPercent / 100; }
        @Override public double calculatePrice(double base, int qty) { return base * qty * (1 - discount); }
        @Override public String getDescription() { return "Member " + (int)(discount * 100) + "% off"; }
    }
    
    public static void main(String[] args) {
        // Sort strategy
        List<Integer> data = Arrays.asList(5, 2, 8, 1, 9, 3, 7, 4, 6, 10);
        Sorter<Integer> sorter = new Sorter<>(data);
        
        for (SortStrategy strategy : new SortStrategy[]{new BubbleSort(), new QuickSort(), new ReverseSort()}) {
            sorter.setStrategy(strategy);
            sorter.benchmark();
        }
        
        // Pricing strategy
        System.out.println("\n=== Pricing Strategies ===");
        double basePrice = 100.0;
        int[] quantities = {5, 10, 50, 100};
        PricingStrategy[] strategies = {
            new RegularPricing(),
            new BulkDiscountPricing(),
            new MemberPricing(15)
        };
        
        System.out.printf("%-25s", "Strategy");
        for (int qty : quantities) System.out.printf(" qty=%-8d", qty);
        System.out.println();
        System.out.println("-".repeat(65));
        
        for (PricingStrategy ps : strategies) {
            System.out.printf("%-25s", ps.getDescription());
            for (int qty : quantities) {
                System.out.printf(" %,8.0f  ", ps.calculatePrice(basePrice, qty));
            }
            System.out.println();
        }
    }
}
```

### Command Pattern

```java
import java.util.*;

public class CommandDemo {
    
    interface Command {
        void execute();
        void undo();
        String getDescription();
    }
    
    // Document editor
    static class TextDocument {
        private StringBuilder content = new StringBuilder();
        
        void insertAt(int pos, String text) {
            content.insert(pos, text);
        }
        
        void deleteAt(int pos, int length) {
            content.delete(pos, pos + length);
        }
        
        String getContent() { return content.toString(); }
    }
    
    static class InsertCommand implements Command {
        private TextDocument doc;
        private int position;
        private String text;
        
        InsertCommand(TextDocument doc, int pos, String text) {
            this.doc = doc; this.position = pos; this.text = text;
        }
        
        @Override public void execute() { doc.insertAt(position, text); }
        @Override public void undo() { doc.deleteAt(position, text.length()); }
        @Override public String getDescription() { return "Insert '" + text + "' at " + position; }
    }
    
    static class DeleteCommand implements Command {
        private TextDocument doc;
        private int position;
        private int length;
        private String deletedText;
        
        DeleteCommand(TextDocument doc, int pos, int len) {
            this.doc = doc; this.position = pos; this.length = len;
        }
        
        @Override
        public void execute() {
            deletedText = doc.getContent().substring(position, position + length);
            doc.deleteAt(position, length);
        }
        
        @Override
        public void undo() { doc.insertAt(position, deletedText); }
        
        @Override
        public String getDescription() { return "Delete " + length + " chars at " + position; }
    }
    
    // Command history for undo/redo
    static class CommandHistory {
        private Deque<Command> undoStack = new ArrayDeque<>();
        private Deque<Command> redoStack = new ArrayDeque<>();
        
        void execute(Command cmd) {
            cmd.execute();
            undoStack.push(cmd);
            redoStack.clear();
            System.out.println("Executed: " + cmd.getDescription());
        }
        
        void undo() {
            if (undoStack.isEmpty()) { System.out.println("Nothing to undo"); return; }
            Command cmd = undoStack.pop();
            cmd.undo();
            redoStack.push(cmd);
            System.out.println("Undone: " + cmd.getDescription());
        }
        
        void redo() {
            if (redoStack.isEmpty()) { System.out.println("Nothing to redo"); return; }
            Command cmd = redoStack.pop();
            cmd.execute();
            undoStack.push(cmd);
            System.out.println("Redone: " + cmd.getDescription());
        }
    }
    
    public static void main(String[] args) {
        TextDocument doc = new TextDocument();
        CommandHistory history = new CommandHistory();
        
        history.execute(new InsertCommand(doc, 0, "Hello"));
        System.out.println("Doc: '" + doc.getContent() + "'");
        
        history.execute(new InsertCommand(doc, 5, " World"));
        System.out.println("Doc: '" + doc.getContent() + "'");
        
        history.execute(new InsertCommand(doc, 0, ">>> "));
        System.out.println("Doc: '" + doc.getContent() + "'");
        
        history.undo();
        System.out.println("After undo: '" + doc.getContent() + "'");
        
        history.undo();
        System.out.println("After undo: '" + doc.getContent() + "'");
        
        history.redo();
        System.out.println("After redo: '" + doc.getContent() + "'");
        
        history.execute(new DeleteCommand(doc, 5, 6));
        System.out.println("After delete: '" + doc.getContent() + "'");
        
        history.undo();
        System.out.println("After undo delete: '" + doc.getContent() + "'");
    }
}
```

---

## 22.4 สรุป Part 22

ในบทนี้คุณได้เรียนรู้:

✅ Singleton (thread-safe, enum)  
✅ Factory Method  
✅ Abstract Factory  
✅ Builder Pattern  
✅ Decorator Pattern  
✅ Observer Pattern  
✅ Strategy Pattern  
✅ Command Pattern  

---

*[← Part 21: Multithreading](./part-21-multithreading.md) | [Part 23: Unit Testing →](./part-23-unit-testing.md)*
