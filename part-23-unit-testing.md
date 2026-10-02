# Part 23: Unit Testing with JUnit 5
## หลักสูตร Java & Android Development - ระดับ Advanced

---

## 23.1 JUnit 5 Setup

**pom.xml (Maven):**
```xml
<dependencies>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.10.0</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.mockito</groupId>
        <artifactId>mockito-core</artifactId>
        <version>5.3.1</version>
        <scope>test</scope>
    </dependency>
</dependencies>
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.1.2</version>
        </plugin>
    </plugins>
</build>
```

---

## 23.2 Basic JUnit 5

```java
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;

// Code to test
class Calculator {
    int add(int a, int b) { return a + b; }
    int subtract(int a, int b) { return a - b; }
    int multiply(int a, int b) { return a * b; }
    double divide(double a, double b) {
        if (b == 0) throw new ArithmeticException("Division by zero");
        return a / b;
    }
    
    boolean isPrime(int n) {
        if (n < 2) return false;
        for (int i = 2; i <= Math.sqrt(n); i++) {
            if (n % i == 0) return false;
        }
        return true;
    }
}

class CalculatorTest {
    
    private Calculator calc;
    
    @BeforeEach  // run before each test
    void setUp() {
        calc = new Calculator();
    }
    
    @AfterEach   // run after each test
    void tearDown() {
        // cleanup if needed
    }
    
    @BeforeAll   // run once before all tests
    static void setUpClass() {
        System.out.println("Starting Calculator tests...");
    }
    
    @AfterAll    // run once after all tests
    static void tearDownClass() {
        System.out.println("Calculator tests finished.");
    }
    
    @Test
    @DisplayName("Add two positive numbers")
    void testAdd() {
        assertEquals(5, calc.add(2, 3));
        assertEquals(10, calc.add(7, 3));
        assertEquals(0, calc.add(-3, 3));
    }
    
    @Test
    void testSubtract() {
        assertEquals(2, calc.subtract(5, 3));
        assertEquals(-1, calc.subtract(2, 3));
    }
    
    @Test
    void testMultiply() {
        assertEquals(15, calc.multiply(3, 5));
        assertEquals(0, calc.multiply(0, 100));
        assertEquals(-6, calc.multiply(-2, 3));
    }
    
    @Test
    void testDivide() {
        assertEquals(2.0, calc.divide(10, 5), 0.001);
        assertEquals(3.333, calc.divide(10, 3), 0.001);
    }
    
    @Test
    @DisplayName("Division by zero throws exception")
    void testDivideByZero() {
        ArithmeticException ex = assertThrows(
            ArithmeticException.class,
            () -> calc.divide(10, 0)
        );
        assertEquals("Division by zero", ex.getMessage());
    }
    
    @Test
    void testIsPrime() {
        // assertTrue / assertFalse
        assertTrue(calc.isPrime(2));
        assertTrue(calc.isPrime(3));
        assertTrue(calc.isPrime(17));
        assertTrue(calc.isPrime(97));
        
        assertFalse(calc.isPrime(1));
        assertFalse(calc.isPrime(4));
        assertFalse(calc.isPrime(100));
    }
    
    @Test
    @Disabled("Not implemented yet")
    void testSqrt() {
        // This test is skipped
    }
    
    @Test
    void testMultipleAssertions() {
        // assertAll: runs all assertions even if some fail
        assertAll("calculator operations",
            () -> assertEquals(5, calc.add(2, 3)),
            () -> assertEquals(1, calc.subtract(3, 2)),
            () -> assertEquals(6, calc.multiply(2, 3))
        );
    }
    
    @Test
    void testTimeout() {
        // Test must complete within 100ms
        assertTimeout(java.time.Duration.ofMillis(100), () -> {
            int result = calc.add(1, 2);
            assertEquals(3, result);
        });
    }
}
```

---

## 23.3 Parameterized Tests

```java
import org.junit.jupiter.params.*;
import org.junit.jupiter.params.provider.*;
import static org.junit.jupiter.api.Assertions.*;

class StringUtils {
    static boolean isPalindrome(String s) {
        if (s == null) return false;
        String clean = s.toLowerCase().replaceAll("[^a-z0-9]", "");
        return clean.equals(new StringBuilder(clean).reverse().toString());
    }
    
    static int countVowels(String s) {
        return (int) s.toLowerCase().chars().filter("aeiou"::indexOf).count();
    }
    
    static String capitalize(String s) {
        if (s == null || s.isEmpty()) return s;
        return Character.toUpperCase(s.charAt(0)) + s.substring(1).toLowerCase();
    }
}

class StringUtilsTest {
    
    @ParameterizedTest
    @ValueSource(strings = {"racecar", "level", "madam", "A man a plan a canal Panama"})
    void testIsPalindrome_valid(String input) {
        assertTrue(StringUtils.isPalindrome(input), input + " should be palindrome");
    }
    
    @ParameterizedTest
    @ValueSource(strings = {"hello", "world", "java"})
    void testIsPalindrome_invalid(String input) {
        assertFalse(StringUtils.isPalindrome(input));
    }
    
    @ParameterizedTest
    @CsvSource({
        "hello,  2",
        "world,  1",
        "aeiou,  5",
        "rhythm, 0",
        "Java,   2"
    })
    void testCountVowels(String input, int expected) {
        assertEquals(expected, StringUtils.countVowels(input));
    }
    
    @ParameterizedTest
    @MethodSource("capitalizeProvider")
    void testCapitalize(String input, String expected) {
        assertEquals(expected, StringUtils.capitalize(input));
    }
    
    static java.util.stream.Stream<Arguments> capitalizeProvider() {
        return java.util.stream.Stream.of(
            Arguments.of("hello",   "Hello"),
            Arguments.of("WORLD",   "World"),
            Arguments.of("java",    "Java"),
            Arguments.of("",        ""),
            Arguments.of(null,      null)
        );
    }
    
    @ParameterizedTest
    @EnumSource(value = java.time.DayOfWeek.class,
                names = {"SATURDAY", "SUNDAY"})
    void testWeekend(java.time.DayOfWeek day) {
        // Example: testing with enum values
        int value = day.getValue();
        assertTrue(value >= 6, day + " should have value >= 6");
    }
}
```

---

## 23.4 Testing with Mockito

```java
import org.mockito.*;
import org.mockito.junit.jupiter.*;
import org.junit.jupiter.api.*;
import org.junit.jupiter.api.extension.*;
import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;

// Interfaces and classes to test
interface UserRepository {
    Optional<User> findById(int id);
    List<User> findAll();
    User save(User user);
    boolean existsById(int id);
}

interface EmailService {
    void sendWelcomeEmail(String email);
}

record User(int id, String name, String email) {}

class UserService {
    private final UserRepository repo;
    private final EmailService emailService;
    
    UserService(UserRepository repo, EmailService emailService) {
        this.repo = repo;
        this.emailService = emailService;
    }
    
    User getUserById(int id) {
        return repo.findById(id)
            .orElseThrow(() -> new RuntimeException("User " + id + " not found"));
    }
    
    User createUser(String name, String email) {
        if (name == null || name.isBlank()) throw new IllegalArgumentException("Name required");
        if (!email.contains("@")) throw new IllegalArgumentException("Invalid email");
        
        User user = new User(0, name, email);
        User saved = repo.save(user);
        emailService.sendWelcomeEmail(email);
        return saved;
    }
    
    List<User> getAllUsers() {
        return repo.findAll();
    }
}

@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    
    @Mock
    private UserRepository userRepo;
    
    @Mock
    private EmailService emailService;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    void testGetUser_found() {
        User mockUser = new User(1, "Alice", "alice@example.com");
        when(userRepo.findById(1)).thenReturn(Optional.of(mockUser));
        
        User result = userService.getUserById(1);
        
        assertEquals("Alice", result.name());
        assertEquals("alice@example.com", result.email());
        verify(userRepo, times(1)).findById(1);
    }
    
    @Test
    void testGetUser_notFound() {
        when(userRepo.findById(99)).thenReturn(Optional.empty());
        
        assertThrows(RuntimeException.class, () -> userService.getUserById(99));
        verify(userRepo).findById(99);
    }
    
    @Test
    void testCreateUser_success() {
        User savedUser = new User(1, "Bob", "bob@example.com");
        when(userRepo.save(any(User.class))).thenReturn(savedUser);
        
        User result = userService.createUser("Bob", "bob@example.com");
        
        assertNotNull(result);
        assertEquals("Bob", result.name());
        
        verify(userRepo, times(1)).save(any(User.class));
        verify(emailService, times(1)).sendWelcomeEmail("bob@example.com");
    }
    
    @Test
    void testCreateUser_invalidName() {
        assertThrows(IllegalArgumentException.class,
            () -> userService.createUser("", "test@example.com"));
        
        verifyNoInteractions(userRepo, emailService);
    }
    
    @Test
    void testCreateUser_invalidEmail() {
        assertThrows(IllegalArgumentException.class,
            () -> userService.createUser("Alice", "not-an-email"));
        
        verifyNoInteractions(emailService);
    }
    
    @Test
    void testGetAllUsers() {
        List<User> mockUsers = Arrays.asList(
            new User(1, "Alice", "alice@example.com"),
            new User(2, "Bob", "bob@example.com")
        );
        when(userRepo.findAll()).thenReturn(mockUsers);
        
        List<User> result = userService.getAllUsers();
        
        assertEquals(2, result.size());
        assertEquals("Alice", result.get(0).name());
        verify(userRepo).findAll();
    }
    
    @Test
    void testEmailServiceNotCalledOnInvalidUser() {
        userService.createUser("Charlie", "invalid-email"); // will throw
        verify(emailService, never()).sendWelcomeEmail(anyString());
    }
}
```

---

## 23.5 Test Lifecycle Annotations

```java
import org.junit.jupiter.api.*;

class LifecycleDemo {
    
    @BeforeAll
    static void initAll() {
        System.out.println("[BeforeAll] Runs once before all tests");
    }
    
    @AfterAll
    static void teardownAll() {
        System.out.println("[AfterAll] Runs once after all tests");
    }
    
    @BeforeEach
    void init() {
        System.out.println("[BeforeEach] Runs before each test");
    }
    
    @AfterEach
    void teardown() {
        System.out.println("[AfterEach] Runs after each test");
    }
    
    @Test
    @DisplayName("First test")
    void test1() {
        System.out.println("[Test 1]");
        Assertions.assertTrue(true);
    }
    
    @Test
    @DisplayName("Second test")
    void test2() {
        System.out.println("[Test 2]");
        Assertions.assertEquals(4, 2 + 2);
    }
    
    @Nested
    @DisplayName("Inner test class")
    class InnerTests {
        
        @BeforeEach
        void innerSetup() {
            System.out.println("[Inner BeforeEach]");
        }
        
        @Test
        void innerTest() {
            System.out.println("[Inner Test]");
        }
    }
    
    @Test
    @Tag("slow")
    void slowTest() {
        // Tagged tests can be filtered in CI
        System.out.println("[Slow test]");
    }
    
    @Test
    @EnabledOnOs(OS.LINUX)
    void linuxOnlyTest() {
        System.out.println("[Linux only]");
    }
}
```

---

## 23.6 Testing Best Practices

```java
// Good test structure: Arrange-Act-Assert (AAA)
class AAAPattern {
    
    @Test
    void testBankWithdrawal() {
        // ARRANGE: set up the test
        BankAccount account = new BankAccount("TEST001", 1000.0);
        double withdrawAmount = 500.0;
        
        // ACT: perform the action
        account.withdraw(withdrawAmount);
        
        // ASSERT: verify the result
        assertEquals(500.0, account.getBalance());
    }
    
    @Test
    void testBankWithdrawal_insufficient() {
        // ARRANGE
        BankAccount account = new BankAccount("TEST002", 100.0);
        
        // ACT + ASSERT
        assertThrows(IllegalStateException.class,
            () -> account.withdraw(200.0));
        
        // ASSERT side effects
        assertEquals(100.0, account.getBalance(), "Balance should be unchanged");
    }
    
    // Test naming: should_expectedBehavior_when_condition
    @Test
    void shouldReturnNegativeBalance_whenOverdrawn() { /* ... */ }
    
    @Test
    void shouldThrowException_whenWithdrawingMoreThanBalance() { /* ... */ }
}
```

---

## 23.7 สรุป Part 23

ในบทนี้คุณได้เรียนรู้:

✅ JUnit 5 setup (Maven)  
✅ @Test, @BeforeEach, @AfterEach, @BeforeAll, @AfterAll  
✅ Assertions (assertEquals, assertTrue, assertThrows)  
✅ Parameterized tests  
✅ Mockito (mocking, stubbing, verification)  
✅ Test lifecycle  
✅ AAA pattern  

---

*[← Part 22: Design Patterns](./part-22-design-patterns.md) | [Part 24: JDBC Database →](./part-24-jdbc.md)*
