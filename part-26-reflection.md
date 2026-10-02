# Part 26: Reflection และ Annotations
## หลักสูตร Java & Android Development - ระดับ Advanced

---

## 26.1 Reflection พื้นฐาน

```java
import java.lang.reflect.*;
import java.util.*;

public class ReflectionBasics {
    
    static class Person {
        private String name;
        protected int age;
        public String city;
        
        Person() {}
        
        Person(String name, int age) {
            this.name = name;
            this.age = age;
        }
        
        public String getName() { return name; }
        private void setName(String name) { this.name = name; }
        
        public String greet(String greeting) {
            return greeting + ", I'm " + name + " from " + city;
        }
        
        @Override
        public String toString() {
            return "Person{name=" + name + ", age=" + age + "}";
        }
    }
    
    public static void main(String[] args) throws Exception {
        // --- Get Class object ---
        Class<?> clazz = Person.class;
        Class<?> clazz2 = Class.forName("ReflectionBasics$Person");
        
        System.out.println("Class: " + clazz.getName());
        System.out.println("Simple: " + clazz.getSimpleName());
        System.out.println("Package: " + clazz.getPackageName());
        System.out.println("Superclass: " + clazz.getSuperclass().getSimpleName());
        
        // --- Inspect constructors ---
        System.out.println("\n=== Constructors ===");
        for (Constructor<?> c : clazz.getDeclaredConstructors()) {
            System.out.println("  " + c);
        }
        
        // --- Inspect fields ---
        System.out.println("\n=== Fields ===");
        for (Field f : clazz.getDeclaredFields()) {
            System.out.printf("  %-15s %s %s%n",
                f.getType().getSimpleName(),
                f.getName(),
                Modifier.toString(f.getModifiers()));
        }
        
        // --- Inspect methods ---
        System.out.println("\n=== Methods ===");
        for (Method m : clazz.getDeclaredMethods()) {
            System.out.printf("  %s %s(%s)%n",
                m.getReturnType().getSimpleName(),
                m.getName(),
                Arrays.stream(m.getParameterTypes())
                    .map(Class::getSimpleName)
                    .reduce("", (a, b) -> a.isEmpty() ? b : a + ", " + b));
        }
        
        // --- Create instance via reflection ---
        Constructor<?> noArgCtor = clazz.getDeclaredConstructor();
        Person p1 = (Person) noArgCtor.newInstance();
        
        Constructor<?> argCtor = clazz.getDeclaredConstructor(String.class, int.class);
        Person p2 = (Person) argCtor.newInstance("Alice", 30);
        System.out.println("\nCreated: " + p2);
        
        // --- Access private field ---
        Field nameField = clazz.getDeclaredField("name");
        nameField.setAccessible(true);  // bypass access control
        nameField.set(p1, "Bob");
        System.out.println("After set name: " + nameField.get(p1));
        
        // --- Invoke private method ---
        Method setNameMethod = clazz.getDeclaredMethod("setName", String.class);
        setNameMethod.setAccessible(true);
        setNameMethod.invoke(p1, "Charlie");
        System.out.println("After setName: " + p1.getName());
        
        // --- Invoke public method ---
        Method greetMethod = clazz.getMethod("greet", String.class);
        p2.city = "Bangkok";
        String result = (String) greetMethod.invoke(p2, "Sawadee");
        System.out.println("Greet: " + result);
        
        // --- Array reflection ---
        Class<?> intArr = int[].class;
        System.out.println("\nint[] component: " + intArr.getComponentType());
        System.out.println("is array: " + intArr.isArray());
        
        Object arr = Array.newInstance(int.class, 5);
        Array.set(arr, 0, 10);
        Array.set(arr, 1, 20);
        System.out.println("arr[0]: " + Array.get(arr, 0));
    }
}
```

---

## 26.2 Custom Annotations

```java
import java.lang.annotation.*;
import java.lang.reflect.*;

// --- Defining Annotations ---

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@interface Test {
    String name() default "";
    boolean skip() default false;
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
@interface Validate {
    boolean required() default true;
    int minLength() default 0;
    int maxLength() default Integer.MAX_VALUE;
    String pattern() default "";
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@interface Entity {
    String tableName();
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
@interface Column {
    String name() default "";
    boolean nullable() default true;
    boolean unique() default false;
}

// --- Using Annotations ---

@Entity(tableName = "users")
class User {
    @Column(name = "user_id", nullable = false)
    @Validate(required = true)
    int id;
    
    @Column(name = "user_name", nullable = false, unique = true)
    @Validate(required = true, minLength = 3, maxLength = 50)
    String username;
    
    @Column(name = "email_addr", nullable = false)
    @Validate(required = true, pattern = ".*@.*\\..*")
    String email;
    
    @Column(nullable = true)
    String bio;
    
    User(int id, String username, String email, String bio) {
        this.id = id;
        this.username = username;
        this.email = email;
        this.bio = bio;
    }
}

// --- Processing Annotations ---

public class AnnotationProcessor {
    
    // Validation processor
    static List<String> validate(Object obj) throws Exception {
        List<String> errors = new ArrayList<>();
        Class<?> clazz = obj.getClass();
        
        for (Field field : clazz.getDeclaredFields()) {
            if (!field.isAnnotationPresent(Validate.class)) continue;
            
            Validate v = field.getAnnotation(Validate.class);
            field.setAccessible(true);
            Object value = field.get(obj);
            String fieldName = field.getName();
            
            if (v.required() && (value == null || value.toString().isBlank())) {
                errors.add(fieldName + " is required");
                continue;
            }
            
            if (value instanceof String s) {
                if (s.length() < v.minLength()) {
                    errors.add(fieldName + " must be >= " + v.minLength() + " chars");
                }
                if (s.length() > v.maxLength()) {
                    errors.add(fieldName + " must be <= " + v.maxLength() + " chars");
                }
                if (!v.pattern().isEmpty() && !s.matches(v.pattern())) {
                    errors.add(fieldName + " has invalid format");
                }
            }
        }
        return errors;
    }
    
    // ORM SQL generator
    static String generateInsertSQL(Object obj) throws Exception {
        Class<?> clazz = obj.getClass();
        
        if (!clazz.isAnnotationPresent(Entity.class)) {
            throw new IllegalArgumentException("Not an entity: " + clazz.getSimpleName());
        }
        
        Entity entity = clazz.getAnnotation(Entity.class);
        List<String> columns = new ArrayList<>();
        List<String> values = new ArrayList<>();
        
        for (Field field : clazz.getDeclaredFields()) {
            if (!field.isAnnotationPresent(Column.class)) continue;
            
            Column col = field.getAnnotation(Column.class);
            field.setAccessible(true);
            Object value = field.get(obj);
            
            String colName = col.name().isEmpty() ? field.getName() : col.name();
            columns.add(colName);
            
            if (value == null) values.add("NULL");
            else if (value instanceof String) values.add("'" + value + "'");
            else values.add(String.valueOf(value));
        }
        
        return String.format("INSERT INTO %s (%s) VALUES (%s)",
            entity.tableName(),
            String.join(", ", columns),
            String.join(", ", values));
    }
    
    // Simple test runner
    static void runTests(Class<?> clazz) throws Exception {
        Object instance = clazz.getDeclaredConstructor().newInstance();
        int passed = 0, failed = 0, skipped = 0;
        
        for (Method method : clazz.getDeclaredMethods()) {
            if (!method.isAnnotationPresent(Test.class)) continue;
            
            Test test = method.getAnnotation(Test.class);
            String testName = test.name().isEmpty() ? method.getName() : test.name();
            
            if (test.skip()) {
                System.out.println("[SKIP] " + testName);
                skipped++;
                continue;
            }
            
            try {
                method.invoke(instance);
                System.out.println("[PASS] " + testName);
                passed++;
            } catch (Exception e) {
                System.out.println("[FAIL] " + testName + ": " + e.getCause().getMessage());
                failed++;
            }
        }
        
        System.out.printf("\nResults: %d passed, %d failed, %d skipped%n", passed, failed, skipped);
    }
    
    public static void main(String[] args) throws Exception {
        // Validation
        User validUser = new User(1, "alice123", "alice@example.com", "Developer");
        User invalidUser = new User(0, "ab", "not-email", null);
        
        System.out.println("=== Validation ===");
        List<String> errors1 = validate(validUser);
        System.out.println("Valid user errors: " + (errors1.isEmpty() ? "none" : errors1));
        
        List<String> errors2 = validate(invalidUser);
        System.out.println("Invalid user errors: " + errors2);
        
        // SQL Generation
        System.out.println("\n=== SQL Generation ===");
        System.out.println(generateInsertSQL(validUser));
        
        // Test runner
        System.out.println("\n=== Test Runner ===");
        runTests(SampleTests.class);
    }
    
    // Sample test class
    static class SampleTests {
        @Test(name = "Addition test")
        void testAdd() {
            assert 2 + 2 == 4 : "2+2 should be 4";
        }
        
        @Test(name = "Failing test")
        void testFail() {
            if (true) throw new AssertionError("Intentional failure");
        }
        
        @Test(name = "Skipped test", skip = true)
        void testSkipped() {
            throw new RuntimeException("Should not run");
        }
        
        @Test(name = "String test")
        void testString() {
            assert "hello".toUpperCase().equals("HELLO") : "Uppercase failed";
        }
    }
}
```

---

## 26.3 สรุป Part 26

ในบทนี้คุณได้เรียนรู้:

✅ Reflection: Class, Field, Method, Constructor  
✅ Access private members (setAccessible)  
✅ Dynamic instance creation  
✅ Custom annotations (@Retention, @Target)  
✅ Annotation processing at runtime  
✅ Simple ORM and test runner  

---

*[← Part 25: Networking](./part-25-networking.md) | [Part 27: JVM Internals →](./part-27-jvm-internals.md)*
