# Part 91: Java Reflection & Annotation Processing
## หลักสูตร Java & Android Development - ระดับ World-Class

---

## 91.1 Reflection Basics

```java
// Reflection = inspect and manipulate classes at runtime
public class ReflectionDemo {
    
    public static void main(String[] args) throws Exception {
        Class<?> clazz = Product.class;
        
        // Get class info
        System.out.println("Name: "        + clazz.getName());
        System.out.println("Simple name: " + clazz.getSimpleName());
        System.out.println("Package: "     + clazz.getPackageName());
        System.out.println("Superclass: "  + clazz.getSuperclass().getName());
        
        // Interfaces
        for (Class<?> iface : clazz.getInterfaces()) {
            System.out.println("Interface: " + iface.getName());
        }
        
        // Fields
        for (Field field : clazz.getDeclaredFields()) {
            System.out.printf("Field: %s %s (static=%b)%n",
                field.getType().getSimpleName(),
                field.getName(),
                Modifier.isStatic(field.getModifiers()));
        }
        
        // Methods
        for (Method method : clazz.getDeclaredMethods()) {
            System.out.printf("Method: %s %s(%s)%n",
                method.getReturnType().getSimpleName(),
                method.getName(),
                Arrays.stream(method.getParameterTypes())
                    .map(Class::getSimpleName)
                    .collect(Collectors.joining(", ")));
        }
        
        // Constructors
        for (Constructor<?> ctor : clazz.getDeclaredConstructors()) {
            System.out.println("Constructor: " + ctor);
        }
    }
    
    // Access private fields
    public static <T> void setField(Object obj, String fieldName, T value) throws Exception {
        Field field = obj.getClass().getDeclaredField(fieldName);
        field.setAccessible(true);  // bypass private access
        field.set(obj, value);
    }
    
    public static Object getField(Object obj, String fieldName) throws Exception {
        Field field = obj.getClass().getDeclaredField(fieldName);
        field.setAccessible(true);
        return field.get(obj);
    }
    
    // Invoke private method
    public static Object invokeMethod(Object obj, String methodName, Object... args) throws Exception {
        Class<?>[] paramTypes = Arrays.stream(args)
            .map(Object::getClass)
            .toArray(Class[]::new);
        
        Method method = obj.getClass().getDeclaredMethod(methodName, paramTypes);
        method.setAccessible(true);
        return method.invoke(obj, args);
    }
    
    // Create instance dynamically
    public static Object createInstance(String className, Object... constructorArgs) throws Exception {
        Class<?> clazz = Class.forName(className);
        Class<?>[] paramTypes = Arrays.stream(constructorArgs)
            .map(Object::getClass)
            .toArray(Class[]::new);
        
        Constructor<?> ctor = clazz.getConstructor(paramTypes);
        return ctor.newInstance(constructorArgs);
    }
}
```

---

## 91.2 Custom Annotations

```java
// Define custom annotations
@Retention(RetentionPolicy.RUNTIME)  // available at runtime via reflection
@Target(ElementType.FIELD)           // can annotate fields
public @interface Validate {
    boolean notNull()  default false;
    int minLength()    default 0;
    int maxLength()    default Integer.MAX_VALUE;
    String pattern()   default "";
    String message()   default "Validation failed";
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Log {
    boolean logArgs()   default true;
    boolean logResult() default false;
    String level()      default "DEBUG";
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Entity {
    String tableName() default "";
}

// Apply annotations
@Entity(tableName = "users")
public class User {
    @Validate(notNull = true, minLength = 2, maxLength = 50)
    private String name;
    
    @Validate(notNull = true, pattern = "^[\\w.-]+@[\\w.-]+\\.[a-z]{2,}$")
    private String email;
    
    @Validate(minLength = 8, message = "Password must be at least 8 characters")
    private String password;
    
    @Log(logArgs = false, logResult = true)
    public String getDisplayName() {
        return name.trim();
    }
}
```

---

## 91.3 Annotation Processor (Compile-time)

```java
// Validator.java - process @Validate at runtime
public class Validator {
    
    public static List<String> validate(Object obj) {
        List<String> errors = new ArrayList<>();
        Class<?> clazz = obj.getClass();
        
        for (Field field : clazz.getDeclaredFields()) {
            Validate annotation = field.getAnnotation(Validate.class);
            if (annotation == null) continue;
            
            field.setAccessible(true);
            Object value;
            try {
                value = field.get(obj);
            } catch (IllegalAccessException e) {
                continue;
            }
            
            String fieldName = field.getName();
            
            // notNull check
            if (annotation.notNull() && value == null) {
                errors.add(fieldName + ": must not be null");
                continue;
            }
            
            if (value instanceof String) {
                String str = (String) value;
                
                // minLength check
                if (str.length() < annotation.minLength()) {
                    errors.add(fieldName + ": " + annotation.message()
                        + " (min " + annotation.minLength() + " chars)");
                }
                
                // maxLength check
                if (str.length() > annotation.maxLength()) {
                    errors.add(fieldName + ": too long (max " + annotation.maxLength() + " chars)");
                }
                
                // pattern check
                if (!annotation.pattern().isEmpty() && !str.matches(annotation.pattern())) {
                    errors.add(fieldName + ": " + annotation.message() + " (invalid format)");
                }
            }
        }
        
        return errors;
    }
    
    public static void validateOrThrow(Object obj) {
        List<String> errors = validate(obj);
        if (!errors.isEmpty()) {
            throw new IllegalArgumentException("Validation failed:\n" + String.join("\n", errors));
        }
    }
}

// ObjectMapper.java - JSON to Object using reflection + annotations
public class SimpleJsonMapper {
    
    public static String toJson(Object obj) throws Exception {
        StringBuilder sb = new StringBuilder("{");
        boolean first = true;
        
        for (Field field : obj.getClass().getDeclaredFields()) {
            field.setAccessible(true);
            Object value = field.get(obj);
            
            if (!first) sb.append(",");
            first = false;
            
            sb.append("\"").append(field.getName()).append("\":");
            
            if (value == null) {
                sb.append("null");
            } else if (value instanceof String) {
                sb.append("\"").append(value).append("\"");
            } else {
                sb.append(value);
            }
        }
        
        return sb.append("}").toString();
    }
    
    // Usage:
    // User user = new User("Alice", "alice@example.com");
    // String json = SimpleJsonMapper.toJson(user);
    // {"name":"Alice","email":"alice@example.com"}
}

// Usage
User user = new User();
user.setName("A");  // too short
user.setEmail("invalid-email");
user.setPassword("1234");  // too short

List<String> errors = Validator.validate(user);
errors.forEach(System.out::println);
// name: Validation failed (min 2 chars)
// email: Validation failed (invalid format)
// password: Password must be at least 8 characters (min 8 chars)
```

---

## 91.4 สรุป Part 91

ในบทนี้คุณได้เรียนรู้:

✅ Reflection: getDeclaredFields, getDeclaredMethods  
✅ Access private fields (setAccessible)  
✅ Invoke private methods  
✅ Create instance dynamically  
✅ Custom annotations (@Retention, @Target)  
✅ Runtime annotation processing  
✅ Validator using reflection + @Validate  
✅ Simple JSON mapper via reflection  

---

*[← Part 90: Release](./part-90-android-release.md) | [Part 92: Android Architecture Patterns Comparison →](./part-92-architecture-patterns.md)*
