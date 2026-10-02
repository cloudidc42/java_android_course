# Part 14: Exception Handling
## หลักสูตร Java & Android Development - ระดับ Intermediate

---

## 14.1 Exception คืออะไร?

Exception คือ event ที่เกิดขึ้นระหว่างการทำงานของโปรแกรมที่ทำให้ flow ปกติถูกขัดจังหวะ

```
Throwable
├── Error (ร้ายแรง, ไม่ควร catch)
│   ├── OutOfMemoryError
│   ├── StackOverflowError
│   └── VirtualMachineError
└── Exception
    ├── RuntimeException (Unchecked)
    │   ├── NullPointerException
    │   ├── ArrayIndexOutOfBoundsException
    │   ├── ClassCastException
    │   ├── ArithmeticException
    │   └── IllegalArgumentException
    └── Checked Exceptions
        ├── IOException
        ├── SQLException
        └── FileNotFoundException
```

---

## 14.2 try-catch-finally พื้นฐาน

```java
import java.util.*;

public class BasicExceptions {
    
    public static void main(String[] args) {
        // --- Basic try-catch ---
        try {
            int result = 10 / 0;
        } catch (ArithmeticException e) {
            System.out.println("Cannot divide by zero: " + e.getMessage());
        }
        
        // --- Multiple catch blocks ---
        int[] numbers = {1, 2, 3};
        String text = null;
        
        try {
            System.out.println(numbers[5]);       // ArrayIndexOutOfBoundsException
            System.out.println(text.length());    // NullPointerException
        } catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("Array index error: " + e.getMessage());
        } catch (NullPointerException e) {
            System.out.println("Null pointer: " + e.getMessage());
        }
        
        // --- Multi-catch (Java 7+) ---
        try {
            String val = "abc";
            int num = Integer.parseInt(val);  // NumberFormatException
        } catch (NumberFormatException | IllegalArgumentException e) {
            System.out.println("Parse/Argument error: " + e.getMessage());
        }
        
        // --- finally block ---
        System.out.println("\n--- Finally block ---");
        try {
            System.out.println("In try");
            if (true) throw new RuntimeException("test");
            System.out.println("After throw (unreachable)");
        } catch (RuntimeException e) {
            System.out.println("In catch: " + e.getMessage());
        } finally {
            System.out.println("In finally (always runs)");
        }
        
        // --- Exception info ---
        try {
            Object obj = new Integer(5);
            String str = (String) obj;  // ClassCastException
        } catch (ClassCastException e) {
            System.out.println("Type: " + e.getClass().getName());
            System.out.println("Message: " + e.getMessage());
            // e.printStackTrace();  // full stack trace
        }
    }
}
```

---

## 14.3 Custom Exceptions

```java
// Checked Exception (extends Exception)
public class InsufficientFundsException extends Exception {
    private double balance;
    private double amount;
    
    InsufficientFundsException(double balance, double amount) {
        super(String.format("Insufficient funds: balance=%.2f, requested=%.2f", balance, amount));
        this.balance = balance;
        this.amount = amount;
    }
    
    public double getBalance() { return balance; }
    public double getAmount() { return amount; }
    public double getShortfall() { return amount - balance; }
}

// Unchecked Exception (extends RuntimeException)
public class InvalidTransactionException extends RuntimeException {
    public enum Reason { NEGATIVE_AMOUNT, ZERO_AMOUNT, DAILY_LIMIT_EXCEEDED }
    
    private Reason reason;
    
    InvalidTransactionException(Reason reason, String message) {
        super(message);
        this.reason = reason;
    }
    
    public Reason getReason() { return reason; }
}

public class AccountNotFoundException extends RuntimeException {
    private String accountId;
    
    AccountNotFoundException(String accountId) {
        super("Account not found: " + accountId);
        this.accountId = accountId;
    }
    
    public String getAccountId() { return accountId; }
}

// BankAccount using custom exceptions
public class BankAccount {
    private String id;
    private String owner;
    private double balance;
    private double dailyWithdrawn;
    private static final double DAILY_LIMIT = 50000;
    
    BankAccount(String id, String owner, double initialBalance) {
        this.id = id;
        this.owner = owner;
        this.balance = initialBalance;
    }
    
    public void deposit(double amount) {
        if (amount <= 0) {
            throw new InvalidTransactionException(
                InvalidTransactionException.Reason.NEGATIVE_AMOUNT,
                "Deposit amount must be positive: " + amount
            );
        }
        balance += amount;
        System.out.printf("[%s] Deposited %.2f. Balance: %.2f%n", id, amount, balance);
    }
    
    public void withdraw(double amount) throws InsufficientFundsException {
        if (amount <= 0) {
            throw new InvalidTransactionException(
                InvalidTransactionException.Reason.NEGATIVE_AMOUNT,
                "Withdrawal amount must be positive"
            );
        }
        if (dailyWithdrawn + amount > DAILY_LIMIT) {
            throw new InvalidTransactionException(
                InvalidTransactionException.Reason.DAILY_LIMIT_EXCEEDED,
                "Daily limit of " + DAILY_LIMIT + " would be exceeded"
            );
        }
        if (amount > balance) {
            throw new InsufficientFundsException(balance, amount);  // checked
        }
        balance -= amount;
        dailyWithdrawn += amount;
        System.out.printf("[%s] Withdrawn %.2f. Balance: %.2f%n", id, amount, balance);
    }
    
    public double getBalance() { return balance; }
    
    @Override
    public String toString() {
        return String.format("Account[%s, owner=%s, balance=%.2f]", id, owner, balance);
    }
}

public class BankSystem {
    private Map<String, BankAccount> accounts = new HashMap<>();
    
    void createAccount(String id, String owner, double balance) {
        accounts.put(id, new BankAccount(id, owner, balance));
        System.out.println("Created: " + accounts.get(id));
    }
    
    BankAccount getAccount(String id) {
        BankAccount acc = accounts.get(id);
        if (acc == null) throw new AccountNotFoundException(id);
        return acc;
    }
    
    void transfer(String fromId, String toId, double amount) {
        try {
            BankAccount from = getAccount(fromId);
            BankAccount to = getAccount(toId);
            
            from.withdraw(amount);  // may throw InsufficientFundsException
            to.deposit(amount);
            
            System.out.printf("Transfer %.2f: %s -> %s successful%n", amount, fromId, toId);
            
        } catch (InsufficientFundsException e) {
            System.out.printf("Transfer failed: %s (shortfall: %.2f)%n",
                e.getMessage(), e.getShortfall());
        } catch (AccountNotFoundException e) {
            System.out.println("Transfer failed: " + e.getMessage());
        } catch (InvalidTransactionException e) {
            System.out.println("Invalid transaction: " + e.getReason() + " - " + e.getMessage());
        }
    }
    
    public static void main(String[] args) {
        BankSystem bank = new BankSystem();
        
        bank.createAccount("A001", "Alice", 10000);
        bank.createAccount("A002", "Bob", 5000);
        
        bank.transfer("A001", "A002", 3000);   // success
        bank.transfer("A002", "A001", 9000);   // insufficient funds
        bank.transfer("A001", "A999", 1000);   // account not found
        bank.transfer("A001", "A002", -100);   // invalid amount
        
        System.out.println("\n=== Final Balances ===");
        bank.accounts.values().forEach(System.out::println);
    }
}
```

---

## 14.4 Try-with-Resources

```java
import java.io.*;

public class TryWithResources {
    
    // AutoCloseable resource
    static class DatabaseConnection implements AutoCloseable {
        String url;
        boolean connected;
        
        DatabaseConnection(String url) throws Exception {
            this.url = url;
            System.out.println("Connecting to: " + url);
            if (url.contains("invalid")) {
                throw new Exception("Cannot connect to: " + url);
            }
            this.connected = true;
            System.out.println("Connected!");
        }
        
        String query(String sql) {
            if (!connected) throw new IllegalStateException("Not connected");
            System.out.println("Executing: " + sql);
            return "Result of: " + sql;
        }
        
        @Override
        public void close() {
            connected = false;
            System.out.println("Connection to " + url + " closed");
        }
    }
    
    static class FileProcessor implements AutoCloseable {
        String filename;
        
        FileProcessor(String filename) {
            System.out.println("Opening: " + filename);
            this.filename = filename;
        }
        
        String read() { return "Content of " + filename; }
        
        @Override
        public void close() { System.out.println("Closing: " + filename); }
    }
    
    public static void main(String[] args) {
        // try-with-resources: auto close even if exception occurs
        System.out.println("=== Single resource ===");
        try (DatabaseConnection conn = new DatabaseConnection("jdbc:mysql://localhost/db")) {
            String result = conn.query("SELECT * FROM users");
            System.out.println(result);
        } catch (Exception e) {
            System.out.println("Error: " + e.getMessage());
        }
        // conn.close() called automatically here
        
        System.out.println("\n=== Multiple resources ===");
        try (
            DatabaseConnection conn = new DatabaseConnection("jdbc:mysql://localhost/db");
            FileProcessor file = new FileProcessor("output.txt")
        ) {
            conn.query("INSERT INTO log VALUES ('test')");
            System.out.println(file.read());
        } catch (Exception e) {
            System.out.println("Error: " + e.getMessage());
        }
        // Both closed in reverse order: file then conn
        
        System.out.println("\n=== Failed connection ===");
        try (DatabaseConnection conn = new DatabaseConnection("invalid://url")) {
            conn.query("SELECT 1");  // won't reach here
        } catch (Exception e) {
            System.out.println("Error caught: " + e.getMessage());
        }
    }
}
```

---

## 14.5 Exception Chaining

```java
public class ExceptionChaining {
    
    static class DataProcessingException extends Exception {
        DataProcessingException(String message, Throwable cause) {
            super(message, cause);  // chain with cause
        }
    }
    
    static String readConfig(String filename) throws IOException {
        // Simulate file not found
        throw new FileNotFoundException("Config file not found: " + filename);
    }
    
    static String parseConfig(String filename) throws DataProcessingException {
        try {
            return readConfig(filename);
        } catch (IOException e) {
            // Wrap lower-level exception in higher-level one
            throw new DataProcessingException("Failed to parse config: " + filename, e);
        }
    }
    
    static void initializeApp(String config) {
        try {
            String data = parseConfig(config);
            System.out.println("App initialized with: " + data);
        } catch (DataProcessingException e) {
            System.out.println("Init failed: " + e.getMessage());
            System.out.println("Caused by: " + e.getCause().getMessage());
            // e.printStackTrace();  // shows full chain
        }
    }
    
    public static void main(String[] args) {
        initializeApp("app.properties");
    }
}
```

---

## 14.6 Best Practices

```java
import java.util.*;

public class ExceptionBestPractices {
    
    // 1. Specific exceptions (don't catch Exception broadly)
    static void goodPractice1() {
        try {
            int[] arr = {1, 2, 3};
            System.out.println(arr[10]);
        } catch (ArrayIndexOutOfBoundsException e) {  // specific
            System.out.println("Array access error");
        }
    }
    
    // 2. Don't swallow exceptions
    static void goodPractice2() {
        try {
            String s = null;
            s.length();
        } catch (NullPointerException e) {
            System.out.println("Null pointer: " + e.getMessage());
            // At minimum: log or print - never empty catch
        }
    }
    
    // 3. Use Optional instead of null checks
    static Optional<String> findUser(int id) {
        Map<Integer, String> users = Map.of(1, "Alice", 2, "Bob");
        return Optional.ofNullable(users.get(id));
    }
    
    // 4. Validate input early
    static double divide(double a, double b) {
        if (b == 0) throw new ArithmeticException("Divisor cannot be zero");
        return a / b;
    }
    
    // 5. Exception hierarchy for APIs
    static class ServiceException extends RuntimeException {
        int statusCode;
        ServiceException(int code, String message) {
            super(message); this.statusCode = code;
        }
    }
    
    static class ValidationException extends ServiceException {
        String field;
        ValidationException(String field, String message) {
            super(400, message); this.field = field;
        }
    }
    
    static class NotFoundException extends ServiceException {
        NotFoundException(String resource) {
            super(404, resource + " not found");
        }
    }
    
    static void processRequest(String userId, String data) {
        if (userId == null || userId.isBlank()) {
            throw new ValidationException("userId", "User ID cannot be blank");
        }
        if (data == null) {
            throw new ValidationException("data", "Data cannot be null");
        }
        if (!userId.equals("123")) {
            throw new NotFoundException("User " + userId);
        }
        System.out.println("Processing data for user " + userId);
    }
    
    public static void main(String[] args) {
        // Optional usage
        findUser(1).ifPresent(u -> System.out.println("Found: " + u));
        findUser(99).ifPresentOrElse(
            u -> System.out.println("Found: " + u),
            () -> System.out.println("User not found")
        );
        
        // Input validation
        try { divide(10, 0); }
        catch (ArithmeticException e) { System.out.println(e.getMessage()); }
        
        System.out.println(divide(10, 3));
        
        // Service exceptions
        String[][] requests = {
            {"", "data"},       // validation error
            {"123", "mydata"},  // success
            {"999", "data"}     // not found
        };
        
        for (String[] req : requests) {
            try {
                processRequest(req[0], req[1]);
            } catch (ValidationException e) {
                System.out.printf("Validation [%d] field=%s: %s%n",
                    e.statusCode, e.field, e.getMessage());
            } catch (NotFoundException e) {
                System.out.printf("Not Found [%d]: %s%n", e.statusCode, e.getMessage());
            } catch (ServiceException e) {
                System.out.printf("Service Error [%d]: %s%n", e.statusCode, e.getMessage());
            }
        }
    }
}
```

---

## 14.7 แบบฝึกหัด: Student Grade System

```java
import java.util.*;

public class GradeSystem {
    
    static class InvalidGradeException extends Exception {
        double grade;
        InvalidGradeException(double grade) {
            super(String.format("Invalid grade: %.1f (must be 0-100)", grade));
            this.grade = grade;
        }
    }
    
    static class StudentNotFoundException extends RuntimeException {
        StudentNotFoundException(String id) { super("Student not found: " + id); }
    }
    
    static class Student {
        String id;
        String name;
        List<Double> grades = new ArrayList<>();
        
        Student(String id, String name) { this.id = id; this.name = name; }
        
        void addGrade(double grade) throws InvalidGradeException {
            if (grade < 0 || grade > 100) throw new InvalidGradeException(grade);
            grades.add(grade);
        }
        
        double getAverage() {
            if (grades.isEmpty()) throw new IllegalStateException("No grades recorded for " + name);
            return grades.stream().mapToDouble(Double::doubleValue).average().orElse(0);
        }
        
        String getLetterGrade() {
            double avg = getAverage();
            if (avg >= 80) return "A";
            if (avg >= 70) return "B";
            if (avg >= 60) return "C";
            if (avg >= 50) return "D";
            return "F";
        }
    }
    
    static Map<String, Student> students = new HashMap<>();
    
    static void addStudent(String id, String name) {
        students.put(id, new Student(id, name));
    }
    
    static void recordGrade(String id, double grade) {
        Student s = students.get(id);
        if (s == null) throw new StudentNotFoundException(id);
        
        try {
            s.addGrade(grade);
            System.out.printf("Recorded %.1f for %s%n", grade, s.name);
        } catch (InvalidGradeException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
    
    static void printReport() {
        System.out.println("\n=== Grade Report ===");
        System.out.printf("%-10s %-15s %-8s %s%n", "ID", "Name", "Average", "Grade");
        System.out.println("-".repeat(40));
        
        for (Student s : students.values()) {
            try {
                System.out.printf("%-10s %-15s %-8.1f %s%n",
                    s.id, s.name, s.getAverage(), s.getLetterGrade());
            } catch (IllegalStateException e) {
                System.out.printf("%-10s %-15s %-8s %s%n",
                    s.id, s.name, "N/A", "N/A");
            }
        }
    }
    
    public static void main(String[] args) {
        addStudent("S001", "Alice");
        addStudent("S002", "Bob");
        addStudent("S003", "Charlie");
        
        recordGrade("S001", 85);
        recordGrade("S001", 92);
        recordGrade("S001", 78);
        recordGrade("S002", 65);
        recordGrade("S002", 110);  // invalid
        recordGrade("S002", 72);
        recordGrade("S999", 80);   // not found - caught
        
        printReport();
    }
}
```

---

## 14.8 สรุป Part 14

ในบทนี้คุณได้เรียนรู้:

✅ Exception hierarchy (Checked vs Unchecked)  
✅ try-catch-finally  
✅ Multi-catch  
✅ Custom Exceptions  
✅ try-with-resources  
✅ Exception Chaining  
✅ Best Practices  

---

*[← Part 13: Interfaces](./part-13-interfaces.md) | [Part 15: Collections →](./part-15-collections.md)*
