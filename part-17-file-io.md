# Part 17: File I/O
## หลักสูตร Java & Android Development - ระดับ Intermediate

---

## 17.1 File I/O Overview

Java มี 2 ชุด API หลักสำหรับ File I/O:
- **java.io** - stream-based (older)
- **java.nio.file** - modern NIO.2 (Java 7+, recommended)

---

## 17.2 java.nio.file.Files (Modern API)

```java
import java.nio.file.*;
import java.io.*;
import java.util.*;

public class NIOFilesDemo {
    public static void main(String[] args) throws IOException {
        Path dir = Path.of("demo_files");
        Path file = dir.resolve("hello.txt");
        
        // --- Create directory ---
        Files.createDirectories(dir);
        System.out.println("Created dir: " + dir.toAbsolutePath());
        
        // --- Write file ---
        String content = "Hello World!\nThis is Java NIO\nLine 3";
        Files.writeString(file, content);
        System.out.println("Written to: " + file);
        
        // --- Read file ---
        String readBack = Files.readString(file);
        System.out.println("Read:\n" + readBack);
        
        // --- Read all lines ---
        List<String> lines = Files.readAllLines(file);
        System.out.println("Lines: " + lines.size());
        lines.forEach(l -> System.out.println("  > " + l));
        
        // --- Append to file ---
        Files.writeString(file, "\nAppended line", StandardOpenOption.APPEND);
        System.out.println("After append: " + Files.readString(file));
        
        // --- File info ---
        System.out.println("Size: " + Files.size(file) + " bytes");
        System.out.println("Exists: " + Files.exists(file));
        System.out.println("Readable: " + Files.isReadable(file));
        System.out.println("Writable: " + Files.isWritable(file));
        
        // --- Copy file ---
        Path copy = dir.resolve("hello_copy.txt");
        Files.copy(file, copy, StandardCopyOption.REPLACE_EXISTING);
        
        // --- Move/Rename ---
        Path renamed = dir.resolve("hello_renamed.txt");
        Files.move(copy, renamed, StandardCopyOption.REPLACE_EXISTING);
        
        // --- List directory ---
        System.out.println("\nFiles in " + dir + ":");
        try (var stream = Files.list(dir)) {
            stream.forEach(p -> System.out.println("  " + p.getFileName()));
        }
        
        // --- Delete ---
        Files.deleteIfExists(renamed);
        Files.deleteIfExists(file);
        Files.deleteIfExists(dir);
        System.out.println("Cleaned up");
    }
}
```

---

## 17.3 Reading and Writing Different Formats

```java
import java.nio.file.*;
import java.io.*;
import java.util.*;

public class FileFormats {
    
    static Path tempDir = Path.of("file_examples");
    
    static void setup() throws IOException {
        Files.createDirectories(tempDir);
    }
    
    // Write CSV
    static void writeCSV(String filename, List<String[]> data) throws IOException {
        StringBuilder sb = new StringBuilder();
        for (String[] row : data) {
            sb.append(String.join(",", row)).append("\n");
        }
        Files.writeString(tempDir.resolve(filename), sb.toString());
    }
    
    // Read CSV
    static List<String[]> readCSV(String filename) throws IOException {
        List<String[]> result = new ArrayList<>();
        for (String line : Files.readAllLines(tempDir.resolve(filename))) {
            if (!line.isBlank()) result.add(line.split(","));
        }
        return result;
    }
    
    // Write Properties
    static void writeProperties(String filename, Map<String, String> props) throws IOException {
        StringBuilder sb = new StringBuilder();
        props.forEach((k, v) -> sb.append(k).append("=").append(v).append("\n"));
        Files.writeString(tempDir.resolve(filename), sb.toString());
    }
    
    // Read Properties
    static Map<String, String> readProperties(String filename) throws IOException {
        Map<String, String> props = new LinkedHashMap<>();
        for (String line : Files.readAllLines(tempDir.resolve(filename))) {
            line = line.trim();
            if (!line.isBlank() && !line.startsWith("#")) {
                int idx = line.indexOf('=');
                if (idx > 0) {
                    props.put(line.substring(0, idx).trim(), line.substring(idx + 1).trim());
                }
            }
        }
        return props;
    }
    
    // Write JSON (simple)
    static void writeJSON(String filename, Map<String, Object> data) throws IOException {
        StringBuilder json = new StringBuilder("{\n");
        data.forEach((k, v) -> {
            json.append("  \"").append(k).append("\": ");
            if (v instanceof String) json.append("\"").append(v).append("\"");
            else json.append(v);
            json.append(",\n");
        });
        if (json.length() > 2) json.setLength(json.length() - 2);
        json.append("\n}");
        Files.writeString(tempDir.resolve(filename), json.toString());
    }
    
    public static void main(String[] args) throws IOException {
        setup();
        
        // --- CSV ---
        List<String[]> students = new ArrayList<>();
        students.add(new String[]{"ID", "Name", "GPA"});
        students.add(new String[]{"S001", "Alice", "3.9"});
        students.add(new String[]{"S002", "Bob", "3.5"});
        students.add(new String[]{"S003", "Charlie", "3.7"});
        
        writeCSV("students.csv", students);
        
        System.out.println("=== CSV Content ===");
        List<String[]> read = readCSV("students.csv");
        for (String[] row : read) {
            System.out.println(Arrays.toString(row));
        }
        
        // --- Properties ---
        Map<String, String> config = new LinkedHashMap<>();
        config.put("app.name", "MyApp");
        config.put("app.version", "1.0.0");
        config.put("db.host", "localhost");
        config.put("db.port", "3306");
        config.put("db.name", "mydb");
        
        writeProperties("config.properties", config);
        
        System.out.println("\n=== Properties ===");
        Map<String, String> loadedConfig = readProperties("config.properties");
        loadedConfig.forEach((k, v) -> System.out.println(k + " = " + v));
        
        // --- JSON ---
        Map<String, Object> person = new LinkedHashMap<>();
        person.put("name", "Alice");
        person.put("age", 30);
        person.put("city", "Bangkok");
        person.put("score", 99.5);
        
        writeJSON("person.json", person);
        System.out.println("\n=== JSON ===");
        System.out.println(Files.readString(tempDir.resolve("person.json")));
        
        // Cleanup
        try (var stream = Files.walk(tempDir)) {
            stream.sorted(Comparator.reverseOrder()).forEach(p -> {
                try { Files.delete(p); } catch (IOException e) { /* ignore */ }
            });
        }
    }
}
```

---

## 17.4 BufferedReader and BufferedWriter

```java
import java.io.*;
import java.nio.file.*;
import java.util.*;

public class BufferedIODemo {
    
    public static void main(String[] args) throws IOException {
        Path file = Path.of("buffered_demo.txt");
        
        // --- BufferedWriter ---
        try (BufferedWriter writer = Files.newBufferedWriter(file)) {
            writer.write("Line 1: Hello from BufferedWriter");
            writer.newLine();
            writer.write("Line 2: Java File I/O");
            writer.newLine();
            writer.write("Line 3: Buffered is efficient");
            writer.newLine();
            
            // Write multiple lines
            List<String> moreLines = List.of("Line 4", "Line 5", "Line 6");
            for (String line : moreLines) {
                writer.write(line);
                writer.newLine();
            }
        }  // auto-close (flushes buffer)
        
        // --- BufferedReader ---
        System.out.println("=== Reading with BufferedReader ===");
        try (BufferedReader reader = Files.newBufferedReader(file)) {
            String line;
            int lineNum = 1;
            while ((line = reader.readLine()) != null) {
                System.out.printf("%3d: %s%n", lineNum++, line);
            }
        }
        
        // --- Read lines as Stream (Java 8+) ---
        System.out.println("\n=== Lines as Stream ===");
        try (var lines = Files.lines(file)) {
            lines
                .filter(l -> l.contains("Java") || l.contains("Buffered"))
                .map(String::toUpperCase)
                .forEach(System.out::println);
        }
        
        // --- Large file processing ---
        System.out.println("\n=== Large file simulation ===");
        Path largeFile = Path.of("large.txt");
        
        // Write 10000 lines
        try (BufferedWriter bw = Files.newBufferedWriter(largeFile)) {
            for (int i = 1; i <= 10000; i++) {
                bw.write(String.format("Record %05d: value=%d%n", i, i * 2));
            }
        }
        
        // Process line by line (memory efficient)
        long count = 0, sum = 0;
        try (BufferedReader br = Files.newBufferedReader(largeFile)) {
            String line;
            while ((line = br.readLine()) != null) {
                count++;
                int valueStart = line.lastIndexOf('=') + 1;
                sum += Long.parseLong(line.substring(valueStart).trim());
            }
        }
        System.out.printf("Lines: %d, Sum: %d%n", count, sum);
        
        // Cleanup
        Files.deleteIfExists(file);
        Files.deleteIfExists(largeFile);
    }
}
```

---

## 17.5 Serialization

```java
import java.io.*;
import java.nio.file.*;
import java.util.*;

public class SerializationDemo {
    
    static class Student implements Serializable {
        private static final long serialVersionUID = 1L;
        
        private String name;
        private int age;
        private double gpa;
        private transient String tempData;  // not serialized
        
        Student(String name, int age, double gpa) {
            this.name = name;
            this.age = age;
            this.gpa = gpa;
            this.tempData = "temp_" + name;
        }
        
        @Override
        public String toString() {
            return String.format("Student{name=%s, age=%d, gpa=%.1f, temp=%s}",
                name, age, gpa, tempData);
        }
    }
    
    // Serialize to file
    static void serialize(Object obj, String filename) throws IOException {
        try (ObjectOutputStream oos = new ObjectOutputStream(
                Files.newOutputStream(Path.of(filename)))) {
            oos.writeObject(obj);
        }
        System.out.println("Serialized to: " + filename);
    }
    
    // Deserialize from file
    @SuppressWarnings("unchecked")
    static <T> T deserialize(String filename) throws IOException, ClassNotFoundException {
        try (ObjectInputStream ois = new ObjectInputStream(
                Files.newInputStream(Path.of(filename)))) {
            return (T) ois.readObject();
        }
    }
    
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        Student alice = new Student("Alice", 20, 3.9);
        System.out.println("Original: " + alice);
        
        // Serialize
        serialize(alice, "student.ser");
        
        // Deserialize
        Student loaded = deserialize("student.ser");
        System.out.println("Deserialized: " + loaded);
        // tempData is null because it was transient
        
        // Serialize list
        List<Student> students = new ArrayList<>();
        students.add(new Student("Bob", 21, 3.5));
        students.add(new Student("Charlie", 22, 3.8));
        
        serialize(students, "students.ser");
        
        List<Student> loadedStudents = deserialize("students.ser");
        System.out.println("\nLoaded list:");
        loadedStudents.forEach(System.out::println);
        
        // Cleanup
        Files.deleteIfExists(Path.of("student.ser"));
        Files.deleteIfExists(Path.of("students.ser"));
    }
}
```

---

## 17.6 Directory Operations

```java
import java.nio.file.*;
import java.nio.file.attribute.*;
import java.io.*;
import java.util.*;
import java.util.stream.*;

public class DirectoryOps {
    
    public static void main(String[] args) throws IOException {
        Path root = Path.of("dir_demo");
        
        // Create directory structure
        Files.createDirectories(root.resolve("src/main/java"));
        Files.createDirectories(root.resolve("src/test/java"));
        Files.createDirectories(root.resolve("resources"));
        
        Files.writeString(root.resolve("src/main/java/Main.java"), "public class Main {}");
        Files.writeString(root.resolve("src/main/java/Helper.java"), "public class Helper {}");
        Files.writeString(root.resolve("src/test/java/MainTest.java"), "class MainTest {}");
        Files.writeString(root.resolve("resources/config.txt"), "key=value");
        Files.writeString(root.resolve("README.md"), "# Project");
        
        // --- List directory ---
        System.out.println("=== Direct children of root ===");
        try (var stream = Files.list(root)) {
            stream.sorted().forEach(p -> {
                boolean isDir = Files.isDirectory(p);
                System.out.println("  " + (isDir ? "[D]" : "[F]") + " " + p.getFileName());
            });
        }
        
        // --- Walk entire tree ---
        System.out.println("\n=== All files (walk) ===");
        try (var stream = Files.walk(root)) {
            stream.sorted().forEach(p -> {
                int depth = root.relativize(p).getNameCount();
                String indent = "  ".repeat(depth);
                System.out.println(indent + p.getFileName());
            });
        }
        
        // --- Find specific files ---
        System.out.println("\n=== Java files ===");
        try (var stream = Files.walk(root)) {
            stream
                .filter(p -> p.toString().endsWith(".java"))
                .forEach(p -> System.out.println("  " + root.relativize(p)));
        }
        
        // --- Get file sizes ---
        System.out.println("\n=== File sizes ===");
        try (var stream = Files.walk(root)) {
            stream.filter(Files::isRegularFile)
                .forEach(p -> {
                    try {
                        System.out.printf("  %-40s %5d bytes%n",
                            root.relativize(p), Files.size(p));
                    } catch (IOException e) { /* ignore */ }
                });
        }
        
        // --- Total size ---
        long totalSize;
        try (var stream = Files.walk(root)) {
            totalSize = stream.filter(Files::isRegularFile)
                .mapToLong(p -> {
                    try { return Files.size(p); }
                    catch (IOException e) { return 0; }
                }).sum();
        }
        System.out.println("Total size: " + totalSize + " bytes");
        
        // --- Delete tree ---
        try (var stream = Files.walk(root)) {
            stream.sorted(Comparator.reverseOrder()).forEach(p -> {
                try { Files.delete(p); } catch (IOException e) { /* ignore */ }
            });
        }
        System.out.println("Deleted tree");
    }
}
```

---

## 17.7 โปรแกรมตัวอย่าง: Simple Note App

```java
import java.nio.file.*;
import java.io.*;
import java.util.*;
import java.time.*;
import java.time.format.*;

public class NoteApp {
    
    static final Path NOTES_DIR = Path.of("notes");
    static final DateTimeFormatter FORMATTER = DateTimeFormatter.ofPattern("yyyy-MM-dd_HH-mm-ss");
    
    record Note(String id, String title, String content, LocalDateTime created) {}
    
    static void init() throws IOException {
        Files.createDirectories(NOTES_DIR);
    }
    
    static Note create(String title, String content) throws IOException {
        String id = LocalDateTime.now().format(FORMATTER);
        String filename = id + ".txt";
        String fileContent = "Title: " + title + "\n" +
                            "Created: " + id + "\n" +
                            "---\n" + content;
        Files.writeString(NOTES_DIR.resolve(filename), fileContent);
        System.out.println("Created note: " + filename);
        return new Note(id, title, content, LocalDateTime.now());
    }
    
    static List<Note> listNotes() throws IOException {
        List<Note> notes = new ArrayList<>();
        try (var stream = Files.list(NOTES_DIR)) {
            stream.filter(p -> p.toString().endsWith(".txt"))
                .sorted()
                .forEach(p -> {
                    try {
                        List<String> lines = Files.readAllLines(p);
                        if (lines.size() >= 3) {
                            String title = lines.get(0).replace("Title: ", "");
                            String id = lines.get(1).replace("Created: ", "");
                            StringBuilder sb = new StringBuilder();
                            for (int i = 3; i < lines.size(); i++) sb.append(lines.get(i)).append("\n");
                            notes.add(new Note(id, title, sb.toString().trim(), null));
                        }
                    } catch (IOException e) { /* ignore */ }
                });
        }
        return notes;
    }
    
    static Optional<Note> findByTitle(String title) throws IOException {
        return listNotes().stream()
            .filter(n -> n.title().toLowerCase().contains(title.toLowerCase()))
            .findFirst();
    }
    
    static void delete(String id) throws IOException {
        Path file = NOTES_DIR.resolve(id + ".txt");
        if (Files.deleteIfExists(file)) {
            System.out.println("Deleted note: " + id);
        } else {
            System.out.println("Note not found: " + id);
        }
    }
    
    static void cleanup() throws IOException {
        try (var stream = Files.walk(NOTES_DIR)) {
            stream.sorted(Comparator.reverseOrder())
                .forEach(p -> { try { Files.delete(p); } catch (IOException e) {} });
        }
    }
    
    public static void main(String[] args) throws IOException, InterruptedException {
        init();
        
        // Create notes
        Note n1 = create("Java Tips", "Use var for type inference in Java 10+");
        Thread.sleep(100);
        Note n2 = create("Android Tips", "Use ViewBinding instead of findViewById");
        Thread.sleep(100);
        Note n3 = create("Git Tips", "Use git stash to save work in progress");
        
        // List all
        System.out.println("\n=== All Notes ===");
        List<Note> notes = listNotes();
        notes.forEach(n -> System.out.printf("[%s] %s%n", n.id(), n.title()));
        
        // Find by title
        System.out.println("\n=== Search 'Java' ===");
        findByTitle("Java").ifPresent(n -> {
            System.out.println("Found: " + n.title());
            System.out.println("Content: " + n.content());
        });
        
        // Delete one
        System.out.println("\n=== Delete ===");
        if (!notes.isEmpty()) delete(notes.get(0).id());
        
        System.out.println("\n=== After delete ===");
        listNotes().forEach(n -> System.out.printf("[%s] %s%n", n.id(), n.title()));
        
        cleanup();
        System.out.println("\nCleaned up all notes");
    }
}
```

---

## 17.8 สรุป Part 17

ในบทนี้คุณได้เรียนรู้:

✅ java.nio.file.Files API  
✅ Read/Write text files  
✅ Directory operations  
✅ BufferedReader/Writer  
✅ Serialization  
✅ CSV, Properties, JSON formats  
✅ Walking directory trees  

---

*[← Part 16: Generics](./part-16-generics.md) | [Part 18: Lambda Expressions →](./part-18-lambda.md)*
