# Part 24: JDBC Database
## หลักสูตร Java & Android Development - ระดับ Advanced

---

## 24.1 JDBC Overview

JDBC (Java Database Connectivity) คือ API สำหรับเชื่อมต่อฐานข้อมูล

**Dependencies (Maven):**
```xml
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>8.1.0</version>
</dependency>
<!-- หรือ H2 (in-memory, สำหรับ testing) -->
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>2.2.220</version>
</dependency>
```

---

## 24.2 Basic JDBC Operations

```java
import java.sql.*;

public class JDBCBasics {
    
    // Connection string
    static final String URL = "jdbc:mysql://localhost:3306/testdb";
    static final String USER = "root";
    static final String PASSWORD = "password";
    
    // สำหรับ H2 in-memory:
    // static final String URL = "jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1";
    // static final String USER = "sa";
    // static final String PASSWORD = "";
    
    static Connection getConnection() throws SQLException {
        return DriverManager.getConnection(URL, USER, PASSWORD);
    }
    
    // Create table
    static void createTable() {
        String sql = """
            CREATE TABLE IF NOT EXISTS students (
                id      INT PRIMARY KEY AUTO_INCREMENT,
                name    VARCHAR(100) NOT NULL,
                email   VARCHAR(100) UNIQUE,
                grade   DOUBLE,
                created TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            )
            """;
        
        try (Connection conn = getConnection();
             Statement stmt = conn.createStatement()) {
            stmt.executeUpdate(sql);
            System.out.println("Table created");
        } catch (SQLException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
    
    // INSERT
    static void insertStudent(String name, String email, double grade) {
        String sql = "INSERT INTO students (name, email, grade) VALUES (?, ?, ?)";
        
        try (Connection conn = getConnection();
             PreparedStatement ps = conn.prepareStatement(sql,
                 Statement.RETURN_GENERATED_KEYS)) {
            
            ps.setString(1, name);
            ps.setString(2, email);
            ps.setDouble(3, grade);
            
            int rows = ps.executeUpdate();
            
            if (rows > 0) {
                ResultSet keys = ps.getGeneratedKeys();
                if (keys.next()) {
                    System.out.println("Inserted ID: " + keys.getInt(1));
                }
            }
        } catch (SQLException e) {
            System.out.println("Insert error: " + e.getMessage());
        }
    }
    
    // SELECT
    static void listStudents() {
        String sql = "SELECT * FROM students ORDER BY id";
        
        try (Connection conn = getConnection();
             Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery(sql)) {
            
            System.out.println("\n=== Students ===");
            System.out.printf("%-5s %-15s %-25s %5s%n", "ID", "Name", "Email", "Grade");
            System.out.println("-".repeat(55));
            
            while (rs.next()) {
                System.out.printf("%-5d %-15s %-25s %5.1f%n",
                    rs.getInt("id"),
                    rs.getString("name"),
                    rs.getString("email"),
                    rs.getDouble("grade"));
            }
        } catch (SQLException e) {
            System.out.println("Query error: " + e.getMessage());
        }
    }
    
    // SELECT with WHERE
    static void findByGrade(double minGrade) {
        String sql = "SELECT * FROM students WHERE grade >= ? ORDER BY grade DESC";
        
        try (Connection conn = getConnection();
             PreparedStatement ps = conn.prepareStatement(sql)) {
            
            ps.setDouble(1, minGrade);
            
            try (ResultSet rs = ps.executeQuery()) {
                System.out.printf("\nStudents with grade >= %.1f:%n", minGrade);
                while (rs.next()) {
                    System.out.printf("  %s (%.1f)%n",
                        rs.getString("name"),
                        rs.getDouble("grade"));
                }
            }
        } catch (SQLException e) {
            System.out.println("Query error: " + e.getMessage());
        }
    }
    
    // UPDATE
    static void updateGrade(int id, double newGrade) {
        String sql = "UPDATE students SET grade = ? WHERE id = ?";
        
        try (Connection conn = getConnection();
             PreparedStatement ps = conn.prepareStatement(sql)) {
            
            ps.setDouble(1, newGrade);
            ps.setInt(2, id);
            
            int rows = ps.executeUpdate();
            System.out.println("Updated " + rows + " row(s)");
        } catch (SQLException e) {
            System.out.println("Update error: " + e.getMessage());
        }
    }
    
    // DELETE
    static void deleteStudent(int id) {
        String sql = "DELETE FROM students WHERE id = ?";
        
        try (Connection conn = getConnection();
             PreparedStatement ps = conn.prepareStatement(sql)) {
            
            ps.setInt(1, id);
            int rows = ps.executeUpdate();
            System.out.println("Deleted " + rows + " row(s)");
        } catch (SQLException e) {
            System.out.println("Delete error: " + e.getMessage());
        }
    }
    
    public static void main(String[] args) {
        createTable();
        
        insertStudent("Alice Johnson", "alice@example.com", 3.9);
        insertStudent("Bob Smith",     "bob@example.com",   3.5);
        insertStudent("Charlie Brown", "charlie@example.com", 3.2);
        insertStudent("Diana Prince",  "diana@example.com",  3.8);
        
        listStudents();
        findByGrade(3.6);
        
        updateGrade(2, 3.7);
        listStudents();
        
        deleteStudent(3);
        listStudents();
    }
}
```

---

## 24.3 Transaction Management

```java
import java.sql.*;

public class TransactionDemo {
    
    static void transferFunds(Connection conn, int fromId, int toId, double amount)
            throws SQLException {
        
        conn.setAutoCommit(false);  // Start transaction
        
        try {
            // Check balance
            String checkSql = "SELECT balance FROM accounts WHERE id = ? FOR UPDATE";
            PreparedStatement checkStmt = conn.prepareStatement(checkSql);
            checkStmt.setInt(1, fromId);
            ResultSet rs = checkStmt.executeQuery();
            
            if (!rs.next()) throw new SQLException("Account " + fromId + " not found");
            double balance = rs.getDouble("balance");
            
            if (balance < amount) {
                throw new SQLException("Insufficient funds: " + balance);
            }
            
            // Debit
            String debitSql = "UPDATE accounts SET balance = balance - ? WHERE id = ?";
            PreparedStatement debitStmt = conn.prepareStatement(debitSql);
            debitStmt.setDouble(1, amount);
            debitStmt.setInt(2, fromId);
            debitStmt.executeUpdate();
            
            // Credit
            String creditSql = "UPDATE accounts SET balance = balance + ? WHERE id = ?";
            PreparedStatement creditStmt = conn.prepareStatement(creditSql);
            creditStmt.setDouble(1, amount);
            creditStmt.setInt(2, toId);
            int updated = creditStmt.executeUpdate();
            
            if (updated == 0) throw new SQLException("Account " + toId + " not found");
            
            conn.commit();  // Commit transaction
            System.out.printf("Transferred %.2f from %d to %d%n", amount, fromId, toId);
            
        } catch (SQLException e) {
            conn.rollback();  // Rollback on error
            System.out.println("Transfer failed: " + e.getMessage());
            throw e;
        } finally {
            conn.setAutoCommit(true);
        }
    }
    
    // Batch insert
    static void batchInsert(Connection conn, List<String[]> data) throws SQLException {
        String sql = "INSERT INTO logs (level, message, timestamp) VALUES (?, ?, NOW())";
        
        conn.setAutoCommit(false);
        PreparedStatement ps = conn.prepareStatement(sql);
        
        try {
            for (String[] row : data) {
                ps.setString(1, row[0]);
                ps.setString(2, row[1]);
                ps.addBatch();
            }
            
            int[] results = ps.executeBatch();
            conn.commit();
            System.out.println("Batch inserted " + results.length + " rows");
            
        } catch (SQLException e) {
            conn.rollback();
            throw e;
        } finally {
            conn.setAutoCommit(true);
        }
    }
}
```

---

## 24.4 DAO Pattern

```java
import java.sql.*;
import java.util.*;

public class DAOPattern {
    
    // Entity
    record Student(int id, String name, String email, double gpa) {}
    
    // DAO Interface
    interface StudentDAO {
        Optional<Student> findById(int id);
        List<Student> findAll();
        List<Student> findByGpaGreaterThan(double minGpa);
        Student save(Student student);
        boolean update(Student student);
        boolean deleteById(int id);
        int count();
    }
    
    // DAO Implementation
    static class StudentDAOImpl implements StudentDAO {
        private final Connection conn;
        
        StudentDAOImpl(Connection conn) { this.conn = conn; }
        
        @Override
        public Optional<Student> findById(int id) {
            try (PreparedStatement ps = conn.prepareStatement(
                    "SELECT * FROM students WHERE id = ?")) {
                ps.setInt(1, id);
                ResultSet rs = ps.executeQuery();
                if (rs.next()) return Optional.of(mapRow(rs));
                return Optional.empty();
            } catch (SQLException e) { throw new RuntimeException(e); }
        }
        
        @Override
        public List<Student> findAll() {
            List<Student> list = new ArrayList<>();
            try (Statement stmt = conn.createStatement();
                 ResultSet rs = stmt.executeQuery("SELECT * FROM students ORDER BY id")) {
                while (rs.next()) list.add(mapRow(rs));
            } catch (SQLException e) { throw new RuntimeException(e); }
            return list;
        }
        
        @Override
        public List<Student> findByGpaGreaterThan(double minGpa) {
            List<Student> list = new ArrayList<>();
            try (PreparedStatement ps = conn.prepareStatement(
                    "SELECT * FROM students WHERE gpa >= ? ORDER BY gpa DESC")) {
                ps.setDouble(1, minGpa);
                ResultSet rs = ps.executeQuery();
                while (rs.next()) list.add(mapRow(rs));
            } catch (SQLException e) { throw new RuntimeException(e); }
            return list;
        }
        
        @Override
        public Student save(Student student) {
            String sql = "INSERT INTO students (name, email, gpa) VALUES (?, ?, ?)";
            try (PreparedStatement ps = conn.prepareStatement(sql,
                    Statement.RETURN_GENERATED_KEYS)) {
                ps.setString(1, student.name());
                ps.setString(2, student.email());
                ps.setDouble(3, student.gpa());
                ps.executeUpdate();
                
                ResultSet keys = ps.getGeneratedKeys();
                if (keys.next()) {
                    return new Student(keys.getInt(1), student.name(), student.email(), student.gpa());
                }
                throw new RuntimeException("Insert failed - no key");
            } catch (SQLException e) { throw new RuntimeException(e); }
        }
        
        @Override
        public boolean update(Student student) {
            String sql = "UPDATE students SET name=?, email=?, gpa=? WHERE id=?";
            try (PreparedStatement ps = conn.prepareStatement(sql)) {
                ps.setString(1, student.name());
                ps.setString(2, student.email());
                ps.setDouble(3, student.gpa());
                ps.setInt(4, student.id());
                return ps.executeUpdate() > 0;
            } catch (SQLException e) { throw new RuntimeException(e); }
        }
        
        @Override
        public boolean deleteById(int id) {
            try (PreparedStatement ps = conn.prepareStatement(
                    "DELETE FROM students WHERE id = ?")) {
                ps.setInt(1, id);
                return ps.executeUpdate() > 0;
            } catch (SQLException e) { throw new RuntimeException(e); }
        }
        
        @Override
        public int count() {
            try (Statement stmt = conn.createStatement();
                 ResultSet rs = stmt.executeQuery("SELECT COUNT(*) FROM students")) {
                if (rs.next()) return rs.getInt(1);
                return 0;
            } catch (SQLException e) { throw new RuntimeException(e); }
        }
        
        private Student mapRow(ResultSet rs) throws SQLException {
            return new Student(
                rs.getInt("id"),
                rs.getString("name"),
                rs.getString("email"),
                rs.getDouble("gpa")
            );
        }
    }
    
    // Service layer uses DAO
    static class StudentService {
        private final StudentDAO dao;
        
        StudentService(StudentDAO dao) { this.dao = dao; }
        
        Student enroll(String name, String email, double gpa) {
            if (gpa < 0 || gpa > 4.0) throw new IllegalArgumentException("Invalid GPA");
            return dao.save(new Student(0, name, email, gpa));
        }
        
        List<Student> getHonorRoll() { return dao.findByGpaGreaterThan(3.5); }
        
        void printReport() {
            List<Student> all = dao.findAll();
            System.out.printf("Total students: %d%n", all.size());
            System.out.println("=== Honor Roll ===");
            getHonorRoll().forEach(s ->
                System.out.printf("  %-15s %.2f%n", s.name(), s.gpa()));
        }
    }
    
    public static void main(String[] args) throws SQLException {
        // For demo, we'd normally use real DB connection
        // This shows the pattern structure
        System.out.println("DAO Pattern demonstrated - connect to real DB to run");
        System.out.println("StudentDAO methods: findById, findAll, save, update, deleteById");
    }
}
```

---

## 24.5 Connection Pooling with HikariCP

```java
// pom.xml:
// <dependency>
//     <groupId>com.zaxxer</groupId>
//     <artifactId>HikariCP</artifactId>
//     <version>5.0.1</version>
// </dependency>

import com.zaxxer.hikari.*;
import java.sql.*;

public class ConnectionPooling {
    
    static HikariDataSource createPool() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:mysql://localhost:3306/testdb");
        config.setUsername("root");
        config.setPassword("password");
        
        // Pool settings
        config.setMaximumPoolSize(10);         // max connections
        config.setMinimumIdle(2);              // minimum idle
        config.setConnectionTimeout(30000);    // 30s timeout
        config.setIdleTimeout(600000);         // 10min idle
        config.setMaxLifetime(1800000);        // 30min max life
        
        // Performance
        config.addDataSourceProperty("cachePrepStmts", "true");
        config.addDataSourceProperty("prepStmtCacheSize", "250");
        config.addDataSourceProperty("useServerPrepStmts", "true");
        
        return new HikariDataSource(config);
    }
    
    static class Repository {
        private final HikariDataSource pool;
        
        Repository(HikariDataSource pool) { this.pool = pool; }
        
        List<String> findAllNames() throws SQLException {
            // Connection is borrowed from pool and returned on close
            try (Connection conn = pool.getConnection();
                 Statement stmt = conn.createStatement();
                 ResultSet rs = stmt.executeQuery("SELECT name FROM students")) {
                
                List<String> names = new ArrayList<>();
                while (rs.next()) names.add(rs.getString("name"));
                return names;
            }
        }
    }
    
    public static void main(String[] args) {
        System.out.println("Connection pooling with HikariCP");
        System.out.println("Create: HikariDataSource ds = createPool()");
        System.out.println("Use:    try (Connection conn = ds.getConnection()) { ... }");
        System.out.println("Close:  ds.close();");
    }
}
```

---

## 24.6 สรุป Part 24

ในบทนี้คุณได้เรียนรู้:

✅ JDBC connection  
✅ CRUD operations (INSERT, SELECT, UPDATE, DELETE)  
✅ PreparedStatement (prevent SQL injection)  
✅ Transaction management  
✅ Batch operations  
✅ DAO Pattern  
✅ Connection Pooling (HikariCP)  

---

*[← Part 23: Unit Testing](./part-23-unit-testing.md) | [Part 25: Networking →](./part-25-networking.md)*
