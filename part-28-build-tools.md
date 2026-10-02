# Part 28: Build Tools - Maven และ Gradle
## หลักสูตร Java & Android Development - ระดับ Advanced

---

## 28.1 Maven พื้นฐาน

Maven ใช้ XML configuration file ชื่อ `pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
    <modelVersion>4.0.0</modelVersion>
    
    <!-- Project coordinates -->
    <groupId>com.example</groupId>
    <artifactId>my-app</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>jar</packaging>
    
    <!-- Project info -->
    <name>My Java Application</name>
    <description>Sample Java App</description>
    
    <!-- Properties -->
    <properties>
        <java.version>21</java.version>
        <maven.compiler.source>${java.version}</maven.compiler.source>
        <maven.compiler.target>${java.version}</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        
        <!-- Dependency versions -->
        <junit.version>5.10.0</junit.version>
        <mockito.version>5.3.1</mockito.version>
        <jackson.version>2.15.2</jackson.version>
        <hikari.version>5.0.1</hikari.version>
        <h2.version>2.2.220</h2.version>
        <log4j.version>2.21.0</log4j.version>
    </properties>
    
    <!-- Dependencies -->
    <dependencies>
        <!-- JSON -->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>${jackson.version}</version>
        </dependency>
        
        <!-- Database -->
        <dependency>
            <groupId>com.zaxxer</groupId>
            <artifactId>HikariCP</artifactId>
            <version>${hikari.version}</version>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <version>${h2.version}</version>
            <scope>runtime</scope>
        </dependency>
        
        <!-- Logging -->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>${log4j.version}</version>
        </dependency>
        
        <!-- Testing -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.mockito</groupId>
            <artifactId>mockito-core</artifactId>
            <version>${mockito.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <!-- Build plugins -->
    <build>
        <plugins>
            <!-- Compiler -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.11.0</version>
                <configuration>
                    <release>${java.version}</release>
                    <compilerArgs>
                        <arg>--enable-preview</arg>
                    </compilerArgs>
                </configuration>
            </plugin>
            
            <!-- Test runner -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.1.2</version>
                <configuration>
                    <argLine>--enable-preview</argLine>
                    <includes>
                        <include>**/*Test.java</include>
                        <include>**/*Tests.java</include>
                    </includes>
                </configuration>
            </plugin>
            
            <!-- Executable JAR (fat jar) -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.5.0</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals><goal>shade</goal></goals>
                        <configuration>
                            <transformers>
                                <transformer implementation=
                                    "org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass>com.example.Main</mainClass>
                                </transformer>
                            </transformers>
                            <createDependencyReducedPom>false</createDependencyReducedPom>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

---

## 28.2 Maven คำสั่ง

```bash
# Lifecycle phases
mvn compile         # compile src/main/java
mvn test            # compile + run tests
mvn package         # compile + test + create JAR/WAR
mvn install         # package + install to local ~/.m2
mvn deploy          # install + upload to remote repo
mvn clean           # delete target/
mvn clean package   # clean then package

# Common options
mvn -DskipTests package      # skip tests
mvn -T 4 package             # parallel build (4 threads)
mvn dependency:tree          # show dependency tree
mvn dependency:resolve       # download all deps
mvn versions:display-dependency-updates  # check for updates

# Run specific test
mvn test -Dtest=CalculatorTest
mvn test -Dtest=CalculatorTest#testAdd

# Project structure
src/
├── main/
│   ├── java/        (source code)
│   └── resources/   (config files, properties)
└── test/
    ├── java/        (test code)
    └── resources/   (test config)
```

---

## 28.3 Gradle พื้นฐาน

Gradle ใช้ Groovy DSL หรือ Kotlin DSL

```groovy
// build.gradle (Groovy DSL)
plugins {
    id 'java'
    id 'application'
}

group = 'com.example'
version = '1.0.0'

java {
    sourceCompatibility = JavaVersion.VERSION_21
    targetCompatibility = JavaVersion.VERSION_21
}

application {
    mainClass = 'com.example.Main'
}

repositories {
    mavenCentral()
}

dependencies {
    // Main dependencies
    implementation 'com.fasterxml.jackson.core:jackson-databind:2.15.2'
    implementation 'com.zaxxer:HikariCP:5.0.1'
    implementation 'org.apache.logging.log4j:log4j-core:2.21.0'
    
    // Runtime only
    runtimeOnly 'com.h2database:h2:2.2.220'
    
    // Test dependencies
    testImplementation 'org.junit.jupiter:junit-jupiter:5.10.0'
    testImplementation 'org.mockito:mockito-core:5.3.1'
    testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
}

test {
    useJUnitPlatform()
    testLogging {
        events 'passed', 'skipped', 'failed'
    }
}

// Fat jar task
jar {
    manifest {
        attributes 'Main-Class': application.mainClass
    }
    
    from {
        configurations.runtimeClasspath.collect {
            it.isDirectory() ? it : zipTree(it)
        }
    }
    
    duplicatesStrategy = DuplicatesStrategy.EXCLUDE
}
```

---

## 28.4 Gradle Kotlin DSL

```kotlin
// build.gradle.kts (Kotlin DSL - recommended for new projects)
plugins {
    id("java")
    id("application")
}

group = "com.example"
version = "1.0.0"

java {
    sourceCompatibility = JavaVersion.VERSION_21
    targetCompatibility = JavaVersion.VERSION_21
}

application {
    mainClass.set("com.example.Main")
}

repositories {
    mavenCentral()
}

val jacksonVersion = "2.15.2"
val junitVersion = "5.10.0"

dependencies {
    implementation("com.fasterxml.jackson.core:jackson-databind:$jacksonVersion")
    implementation("com.zaxxer:HikariCP:5.0.1")
    
    runtimeOnly("com.h2database:h2:2.2.220")
    
    testImplementation("org.junit.jupiter:junit-jupiter:$junitVersion")
    testImplementation("org.mockito:mockito-core:5.3.1")
    testRuntimeOnly("org.junit.platform:junit-platform-launcher")
}

tasks.test {
    useJUnitPlatform()
    testLogging {
        events("passed", "skipped", "failed")
    }
}
```

---

## 28.5 Gradle คำสั่ง

```bash
# Tasks
./gradlew tasks              # list all tasks
./gradlew build              # compile + test + package
./gradlew test               # run tests
./gradlew run                # run application
./gradlew jar                # create JAR
./gradlew clean              # delete build/
./gradlew clean build        # clean then build

# Filtering
./gradlew test --tests "*.CalculatorTest"
./gradlew test --tests "*.CalculatorTest.testAdd"

# Info
./gradlew dependencies       # show dependency tree
./gradlew --version          # Gradle version
./gradlew properties         # project properties

# Parallel
./gradlew build --parallel --max-workers=4

# Gradle wrapper (commit gradlew to repo)
gradle wrapper               # generate wrapper
./gradlew wrapper --gradle-version 8.5   # upgrade
```

---

## 28.6 Maven vs Gradle

```
Feature           Maven               Gradle
-------------------------------------------------------
Config language   XML (verbose)       Groovy/Kotlin DSL
Build speed       Slower              Faster (incremental)
Incremental       Limited             Yes (up-to-date checks)
Caching           Limited             Build cache (local+remote)
Flexibility       Convention-based    Highly customizable
Learning curve    Lower               Higher
Android           Less common         Official (required)
Enterprise        Very common         Growing
IDE Support       Excellent           Excellent

คำแนะนำ:
- Java project ทั่วไป: ใช้ Maven (conventional, widely understood)
- Android project:     ต้องใช้ Gradle
- Performance-critical: ใช้ Gradle (faster builds)
```

---

## 28.7 Project Structure

```
my-app/
├── pom.xml (Maven) or build.gradle (Gradle)
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/example/
│   │   │       ├── Main.java
│   │   │       ├── service/
│   │   │       │   └── UserService.java
│   │   │       ├── repository/
│   │   │       │   └── UserRepository.java
│   │   │       └── model/
│   │   │           └── User.java
│   │   └── resources/
│   │       ├── application.properties
│   │       └── log4j2.xml
│   └── test/
│       ├── java/
│       │   └── com/example/
│       │       └── service/
│       │           └── UserServiceTest.java
│       └── resources/
│           └── test.properties
├── .gitignore
└── README.md

# .gitignore
target/          # Maven build output
build/           # Gradle build output
.gradle/         # Gradle cache
*.class
*.jar
!gradle-wrapper.jar
.idea/
*.iml
```

---

## 28.8 สรุป Part 28

ในบทนี้คุณได้เรียนรู้:

✅ Maven pom.xml (dependencies, plugins, properties)  
✅ Maven lifecycle (compile, test, package, install)  
✅ Gradle Groovy DSL  
✅ Gradle Kotlin DSL  
✅ Gradle tasks  
✅ Maven vs Gradle comparison  
✅ Standard project structure  

---

*[← Part 27: JVM Internals](./part-27-jvm-internals.md) | [Part 29: Logging →](./part-29-logging.md)*
