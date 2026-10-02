# Part 29: Logging
## หลักสูตร Java & Android Development - ระดับ Advanced

---

## 29.1 Logging Framework

Java มี logging framework หลายตัว:
- `java.util.logging` (JUL) - built-in แต่จำกัด
- **Log4j 2** - powerful, popular
- **SLF4J + Logback** - de-facto standard

---

## 29.2 SLF4J + Logback

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>2.0.9</version>
</dependency>
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.4.11</version>
</dependency>
```

```xml
<!-- src/main/resources/logback.xml -->
<configuration>
    
    <!-- Console appender -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    
    <!-- File appender -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/app.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/app.%d{yyyy-MM-dd}.log.gz</fileNamePattern>
            <maxHistory>30</maxHistory>
            <totalSizeCap>1GB</totalSizeCap>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger - %msg%n</pattern>
        </encoder>
    </appender>
    
    <!-- Loggers -->
    <logger name="com.example" level="DEBUG"/>
    <logger name="com.zaxxer.hikari" level="WARN"/>
    
    <!-- Root logger -->
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
    </root>
    
</configuration>
```

---

## 29.3 Using SLF4J

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class LoggingDemo {
    
    // One logger per class - static final
    private static final Logger log = LoggerFactory.getLogger(LoggingDemo.class);
    
    public void demonstrateLogging() {
        // Log levels: TRACE < DEBUG < INFO < WARN < ERROR
        log.trace("Trace: very detailed, execution flow");
        log.debug("Debug: diagnostic info for developers");
        log.info("Info: normal application events");
        log.warn("Warning: something unexpected but not critical");
        log.error("Error: something failed");
        
        // Parameterized logging (efficient - no string concat if level disabled)
        String user = "Alice";
        int orderId = 12345;
        log.info("User {} placed order {}", user, orderId);
        
        // Check level before expensive computation
        if (log.isDebugEnabled()) {
            log.debug("Expensive debug: {}", computeDebugInfo());
        }
        
        // Exception logging
        try {
            riskyOperation();
        } catch (Exception e) {
            log.error("Operation failed for user {}", user, e);
        }
        
        // MDC (Mapped Diagnostic Context) for correlation
        org.slf4j.MDC.put("requestId", "req-" + System.currentTimeMillis());
        org.slf4j.MDC.put("userId", "user-123");
        
        log.info("Processing request");
        
        // Always clear MDC after request
        org.slf4j.MDC.clear();
    }
    
    private String computeDebugInfo() { return "expensive-computation-result"; }
    private void riskyOperation() throws Exception { throw new Exception("Test error"); }
    
    public static void main(String[] args) {
        new LoggingDemo().demonstrateLogging();
    }
}
```

---

## 29.4 Structured Logging

```java
import org.slf4j.*;

public class StructuredLogging {
    private static final Logger log = LoggerFactory.getLogger(StructuredLogging.class);
    
    // Log with context for distributed tracing
    static void processOrder(String orderId, String userId, double amount) {
        MDC.put("orderId", orderId);
        MDC.put("userId", userId);
        
        try {
            log.info("Order processing started, amount={}", amount);
            
            // Simulate processing
            validateOrder(orderId, amount);
            chargePayment(userId, amount);
            fulfillOrder(orderId);
            
            log.info("Order completed successfully");
        } catch (Exception e) {
            log.error("Order processing failed", e);
        } finally {
            MDC.remove("orderId");
            MDC.remove("userId");
        }
    }
    
    static void validateOrder(String orderId, double amount) {
        log.debug("Validating order, amount={}", amount);
        if (amount <= 0) throw new IllegalArgumentException("Invalid amount");
    }
    
    static void chargePayment(String userId, double amount) {
        log.info("Charging payment, userId={}, amount={}", userId, amount);
    }
    
    static void fulfillOrder(String orderId) {
        log.info("Fulfilling order, orderId={}", orderId);
    }
    
    public static void main(String[] args) {
        processOrder("ORD-001", "USR-123", 299.99);
        processOrder("ORD-002", "USR-456", -50.0);  // will fail
    }
}
```

---

## 29.5 Log4j 2

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.21.0</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-slf4j2-impl</artifactId>
    <version>2.21.0</version>
</dependency>
```

```xml
<!-- src/main/resources/log4j2.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
    
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{HH:mm:ss.SSS} %-5level %logger{36} - %msg%n"/>
        </Console>
        
        <RollingFile name="RollingFile" fileName="logs/app.log"
                     filePattern="logs/app-%d{yyyy-MM-dd}-%i.log.gz">
            <PatternLayout>
                <Pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level %logger - %msg%n</Pattern>
            </PatternLayout>
            <Policies>
                <TimeBasedTriggeringPolicy />
                <SizeBasedTriggeringPolicy size="10 MB"/>
            </Policies>
            <DefaultRolloverStrategy max="30"/>
        </RollingFile>
        
        <!-- Async appender for better performance -->
        <Async name="AsyncFile">
            <AppenderRef ref="RollingFile"/>
        </Async>
    </Appenders>
    
    <Loggers>
        <Logger name="com.example" level="DEBUG" additivity="false">
            <AppenderRef ref="Console"/>
            <AppenderRef ref="AsyncFile"/>
        </Logger>
        
        <Root level="INFO">
            <AppenderRef ref="Console"/>
        </Root>
    </Loggers>
    
</Configuration>
```

---

## 29.6 Logging Best Practices

```java
public class LoggingBestPractices {
    
    private static final Logger log = LoggerFactory.getLogger(LoggingBestPractices.class);
    
    // GOOD practices
    void goodPractices() {
        // 1. Use parameterized messages (not string concatenation)
        String name = "Alice";
        log.info("User {} logged in", name);       // GOOD
        // log.info("User " + name + " logged in"); // BAD - always builds string
        
        // 2. Log at appropriate level
        log.trace("Method entry");                  // execution trace
        log.debug("Processing item {}", 42);        // developer diagnostic
        log.info("Order {} created", "ORD-001");    // business event
        log.warn("Retry attempt {}/3", 2);          // unexpected but recoverable
        log.error("Payment failed", new Exception()); // serious error
        
        // 3. Always include exception as last parameter
        try { throw new RuntimeException("test"); }
        catch (Exception e) {
            log.error("Failed to process", e);      // GOOD: includes stack trace
            // log.error("Failed: " + e.getMessage()); // BAD: loses stack trace
        }
        
        // 4. Use MDC for request context
        MDC.put("traceId", generateTraceId());
        try {
            processRequest();
        } finally {
            MDC.clear();  // IMPORTANT: clear in finally block
        }
        
        // 5. Avoid sensitive data in logs
        // log.info("Password: {}", password);  // NEVER do this
        // log.info("CardNumber: {}", card);    // NEVER do this
        log.info("User authenticated, userId={}", userId());
    }
    
    // BAD: logging in tight loops
    void badLoopLogging() {
        for (int i = 0; i < 1_000_000; i++) {
            // log.debug("Processing item {}", i);  // Creates huge log files!
        }
        // Instead:
        log.info("Processing 1,000,000 items...");
        // ... do processing ...
        log.info("Processing completed");
    }
    
    String generateTraceId() { return "trace-" + System.currentTimeMillis(); }
    void processRequest() { log.info("Request processed"); }
    String userId() { return "user-123"; }
}
```

---

## 29.7 สรุป Part 29

ในบทนี้คุณได้เรียนรู้:

✅ SLF4J API (facade pattern)  
✅ Logback configuration (appenders, patterns)  
✅ Log4j 2 configuration  
✅ Log levels (TRACE, DEBUG, INFO, WARN, ERROR)  
✅ Parameterized logging  
✅ MDC (Mapped Diagnostic Context)  
✅ Structured logging  
✅ Logging best practices  

---

*[← Part 28: Build Tools](./part-28-build-tools.md) | [Part 30: Security →](./part-30-security.md)*
