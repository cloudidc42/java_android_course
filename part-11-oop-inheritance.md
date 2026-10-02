# Part 11: OOP - Inheritance
## หลักสูตร Java & Android Development - ระดับ Intermediate

---

## 11.1 Inheritance คืออะไร?

Inheritance (การสืบทอด) คือความสามารถของ class ลูก (subclass) ในการรับคุณสมบัติ (fields) และพฤติกรรม (methods) จาก class แม่ (superclass)

```
Animal (superclass)
  ├── Dog
  ├── Cat
  └── Bird
```

**ประโยชน์:**
- Reuse code ที่มีอยู่แล้ว
- ลดการเขียนซ้ำ
- สร้าง hierarchy ที่เข้าใจง่าย

---

## 11.2 Inheritance พื้นฐาน

```java
// Superclass
public class Animal {
    String name;
    int age;
    String sound;
    
    Animal(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    void makeSound() {
        System.out.println(name + " says: " + sound);
    }
    
    void eat(String food) {
        System.out.println(name + " eats " + food);
    }
    
    void sleep() {
        System.out.println(name + " is sleeping...");
    }
    
    String getInfo() {
        return String.format("%s (age: %d)", name, age);
    }
}

// Subclass - extends ใช้ในการสืบทอด
public class Dog extends Animal {
    String breed;
    boolean isVaccinated;
    
    Dog(String name, int age, String breed) {
        super(name, age);  // เรียก constructor ของ Animal
        this.breed = breed;
        this.sound = "Woof!";
        this.isVaccinated = false;
    }
    
    void fetch() {
        System.out.println(name + " fetches the ball!");
    }
    
    void vaccinate() {
        isVaccinated = true;
        System.out.println(name + " has been vaccinated");
    }
    
    @Override
    String getInfo() {
        return super.getInfo() + String.format(", breed: %s, vaccinated: %b",
            breed, isVaccinated);
    }
}

public class Cat extends Animal {
    boolean isIndoor;
    
    Cat(String name, int age, boolean isIndoor) {
        super(name, age);
        this.isIndoor = isIndoor;
        this.sound = "Meow!";
    }
    
    void purr() {
        System.out.println(name + " purrs...");
    }
    
    void climb() {
        System.out.println(name + " climbs the tree");
    }
}

public class Bird extends Animal {
    double wingspan;
    boolean canFly;
    
    Bird(String name, int age, double wingspan, boolean canFly) {
        super(name, age);
        this.wingspan = wingspan;
        this.canFly = canFly;
        this.sound = "Tweet!";
    }
    
    void fly() {
        if (canFly) {
            System.out.println(name + " flies with wingspan " + wingspan + "m");
        } else {
            System.out.println(name + " cannot fly");
        }
    }
}

public class Main {
    public static void main(String[] args) {
        Dog dog = new Dog("Buddy", 3, "Golden Retriever");
        Cat cat = new Cat("Whiskers", 2, true);
        Bird eagle = new Bird("Eagle", 5, 2.1, true);
        Bird penguin = new Bird("Pingu", 4, 0.5, false);
        
        // Inherited methods
        dog.makeSound();
        dog.eat("bone");
        dog.sleep();
        
        // Subclass specific methods
        dog.fetch();
        dog.vaccinate();
        
        cat.makeSound();
        cat.purr();
        
        eagle.fly();
        penguin.fly();
        
        // Overridden method
        System.out.println(dog.getInfo());
        
        // Polymorphism - Animal array holds all subtypes
        Animal[] animals = {dog, cat, eagle, penguin};
        System.out.println("\n=== All animals ===");
        for (Animal a : animals) {
            System.out.println(a.getInfo());
            a.makeSound();
        }
    }
}
```

---

## 11.3 super Keyword

```java
public class Vehicle {
    String brand;
    String model;
    int year;
    int speed;
    
    Vehicle(String brand, String model, int year) {
        this.brand = brand;
        this.model = model;
        this.year = year;
        this.speed = 0;
    }
    
    void accelerate(int amount) {
        speed += amount;
        System.out.println(brand + " accelerates to " + speed + " km/h");
    }
    
    void brake(int amount) {
        speed = Math.max(0, speed - amount);
        System.out.println(brand + " brakes to " + speed + " km/h");
    }
    
    String describe() {
        return year + " " + brand + " " + model;
    }
}

public class ElectricCar extends Vehicle {
    double batteryCapacity;  // kWh
    double currentCharge;    // percentage
    
    ElectricCar(String brand, String model, int year, double batteryCapacity) {
        super(brand, model, year);  // เรียก Vehicle constructor
        this.batteryCapacity = batteryCapacity;
        this.currentCharge = 100.0;
    }
    
    @Override
    void accelerate(int amount) {
        if (currentCharge < 10) {
            System.out.println("Battery low! Cannot accelerate");
            return;
        }
        super.accelerate(amount);  // เรียก Vehicle.accelerate()
        currentCharge -= amount * 0.1;  // drain battery
        System.out.printf("Battery: %.1f%%%n", currentCharge);
    }
    
    void charge(double hours) {
        double charged = hours * 50;  // 50 kWh/hour
        currentCharge = Math.min(100, currentCharge + charged / batteryCapacity * 100);
        System.out.printf("Charged to %.1f%%%n", currentCharge);
    }
    
    @Override
    String describe() {
        return super.describe() + String.format(" Electric (%.0fkWh)", batteryCapacity);
    }
    
    public static void main(String[] args) {
        ElectricCar tesla = new ElectricCar("Tesla", "Model 3", 2023, 82);
        
        System.out.println(tesla.describe());
        tesla.accelerate(60);
        tesla.accelerate(40);
        tesla.brake(30);
        tesla.charge(0.5);
    }
}
```

---

## 11.4 Method Overriding

```java
public class Shape {
    String color;
    
    Shape(String color) {
        this.color = color;
    }
    
    double area() {
        return 0;  // Base implementation
    }
    
    double perimeter() {
        return 0;
    }
    
    void draw() {
        System.out.println("Drawing a " + color + " " + getClass().getSimpleName());
    }
    
    @Override
    public String toString() {
        return String.format("%s %s: area=%.2f, perimeter=%.2f",
            color, getClass().getSimpleName(), area(), perimeter());
    }
}

public class Circle extends Shape {
    double radius;
    
    Circle(String color, double radius) {
        super(color);
        this.radius = radius;
    }
    
    @Override
    double area() {
        return Math.PI * radius * radius;
    }
    
    @Override
    double perimeter() {
        return 2 * Math.PI * radius;
    }
}

public class Rectangle extends Shape {
    double width, height;
    
    Rectangle(String color, double width, double height) {
        super(color);
        this.width = width;
        this.height = height;
    }
    
    @Override
    double area() {
        return width * height;
    }
    
    @Override
    double perimeter() {
        return 2 * (width + height);
    }
}

public class Triangle extends Shape {
    double a, b, c;  // sides
    
    Triangle(String color, double a, double b, double c) {
        super(color);
        this.a = a; this.b = b; this.c = c;
    }
    
    @Override
    double area() {
        double s = (a + b + c) / 2;
        return Math.sqrt(s * (s-a) * (s-b) * (s-c));
    }
    
    @Override
    double perimeter() {
        return a + b + c;
    }
}

public class ShapeDemo {
    static double totalArea(Shape[] shapes) {
        double total = 0;
        for (Shape s : shapes) total += s.area();
        return total;
    }
    
    public static void main(String[] args) {
        Shape[] shapes = {
            new Circle("red", 5),
            new Rectangle("blue", 4, 6),
            new Triangle("green", 3, 4, 5),
            new Circle("yellow", 3),
            new Rectangle("purple", 8, 2)
        };
        
        System.out.println("=== All Shapes ===");
        for (Shape s : shapes) {
            System.out.println(s);
            s.draw();
        }
        
        System.out.printf("\nTotal Area: %.2f%n", totalArea(shapes));
    }
}
```

---

## 11.5 final Keyword

```java
// final class ห้าม extend
public final class ImmutablePoint {
    private final double x, y;
    
    ImmutablePoint(double x, double y) {
        this.x = x;
        this.y = y;
    }
    
    public double getX() { return x; }
    public double getY() { return y; }
    
    // final method ห้าม override
    final double distanceTo(ImmutablePoint other) {
        double dx = this.x - other.x;
        double dy = this.y - other.y;
        return Math.sqrt(dx*dx + dy*dy);
    }
    
    @Override
    public String toString() {
        return String.format("Point(%.2f, %.2f)", x, y);
    }
}

public class BaseClass {
    // final method ใน non-final class
    final void cannotOverride() {
        System.out.println("This cannot be overridden");
    }
    
    void canOverride() {
        System.out.println("This can be overridden");
    }
}

public class DerivedClass extends BaseClass {
    // cannotOverride() cannot be overridden
    
    @Override
    void canOverride() {
        System.out.println("Overridden!");
    }
}
```

---

## 11.6 Abstract Classes (เบื้องต้น)

```java
public abstract class Employee {
    protected String name;
    protected double baseSalary;
    
    Employee(String name, double baseSalary) {
        this.name = name;
        this.baseSalary = baseSalary;
    }
    
    // Abstract method: subclass ต้อง implement
    abstract double calculateBonus();
    abstract String getJobTitle();
    
    // Concrete method
    double getTotalCompensation() {
        return baseSalary + calculateBonus();
    }
    
    void printPaySlip() {
        System.out.println("=".repeat(35));
        System.out.printf("พนักงาน: %s (%s)%n", name, getJobTitle());
        System.out.printf("เงินเดือน: %,.2f บาท%n", baseSalary);
        System.out.printf("โบนัส:     %,.2f บาท%n", calculateBonus());
        System.out.printf("รวม:       %,.2f บาท%n", getTotalCompensation());
        System.out.println("=".repeat(35));
    }
}

public class SoftwareDeveloper extends Employee {
    int projectsCompleted;
    
    SoftwareDeveloper(String name, double baseSalary, int projectsCompleted) {
        super(name, baseSalary);
        this.projectsCompleted = projectsCompleted;
    }
    
    @Override
    double calculateBonus() {
        return projectsCompleted * 5000;
    }
    
    @Override
    String getJobTitle() { return "Software Developer"; }
}

public class Manager extends Employee {
    int teamSize;
    double performanceRating; // 1-5
    
    Manager(String name, double baseSalary, int teamSize, double rating) {
        super(name, baseSalary);
        this.teamSize = teamSize;
        this.performanceRating = rating;
    }
    
    @Override
    double calculateBonus() {
        return baseSalary * 0.2 * performanceRating;
    }
    
    @Override
    String getJobTitle() { return "Manager (Team: " + teamSize + ")"; }
}

public class Salesperson extends Employee {
    double salesAmount;
    
    Salesperson(String name, double baseSalary, double salesAmount) {
        super(name, baseSalary);
        this.salesAmount = salesAmount;
    }
    
    @Override
    double calculateBonus() {
        return salesAmount * 0.05;  // 5% commission
    }
    
    @Override
    String getJobTitle() { return "Salesperson"; }
}

public class CompanyPayroll {
    public static void main(String[] args) {
        Employee[] employees = {
            new SoftwareDeveloper("Alice", 50000, 3),
            new Manager("Bob", 80000, 10, 4.5),
            new Salesperson("Charlie", 25000, 500000)
        };
        
        for (Employee emp : employees) {
            emp.printPaySlip();
        }
        
        double totalCost = 0;
        for (Employee emp : employees) totalCost += emp.getTotalCompensation();
        System.out.printf("ค่าใช้จ่ายรวม: %,.2f บาท%n", totalCost);
    }
}
```

---

## 11.7 Inheritance Hierarchy ตัวอย่าง

```java
// Multi-level inheritance
public class LivingBeing {
    boolean alive = true;
    
    void breathe() { System.out.println("Breathing..."); }
    void grow() { System.out.println("Growing..."); }
}

public class Mammal extends LivingBeing {
    boolean warmBlooded = true;
    int heartRate;
    
    Mammal(int heartRate) {
        this.heartRate = heartRate;
    }
    
    void nurse() { System.out.println("Nursing young..."); }
}

public class Primate extends Mammal {
    int fingers = 10;
    boolean thumbOpposable = true;
    
    Primate() { super(80); }
    
    void grasp() { System.out.println("Grasping with hands..."); }
}

public class Human extends Primate {
    String language;
    int iq;
    
    Human(String language, int iq) {
        super();
        this.language = language;
        this.iq = iq;
    }
    
    void speak() { System.out.println("Speaking in " + language + "..."); }
    void think() { System.out.println("Thinking (IQ: " + iq + ")..."); }
    
    public static void main(String[] args) {
        Human human = new Human("Thai", 120);
        
        // Methods from all levels
        human.breathe();   // LivingBeing
        human.nurse();     // Mammal
        human.grasp();     // Primate
        human.speak();     // Human
        human.think();     // Human
        
        System.out.println("Heart rate: " + human.heartRate);  // Mammal
        System.out.println("Fingers: " + human.fingers);       // Primate
        System.out.println("Alive: " + human.alive);           // LivingBeing
        
        // instanceof checks
        System.out.println(human instanceof Human);      // true
        System.out.println(human instanceof Primate);    // true
        System.out.println(human instanceof Mammal);     // true
        System.out.println(human instanceof LivingBeing); // true
    }
}
```

---

## 11.8 โปรแกรมตัวอย่าง: Vehicle Fleet

```java
import java.util.*;

public class VehicleFleet {
    
    abstract static class Vehicle {
        protected String id;
        protected String make;
        protected String model;
        protected int year;
        protected double fuelLevel; // 0-100%
        
        Vehicle(String id, String make, String model, int year) {
            this.id = id;
            this.make = make;
            this.model = model;
            this.year = year;
            this.fuelLevel = 100;
        }
        
        abstract double calculateFuelCost(double km);
        abstract String getType();
        
        boolean canTravel(double km) {
            return calculateFuelCost(km) <= fuelLevel;
        }
        
        void refuel(double amount) {
            fuelLevel = Math.min(100, fuelLevel + amount);
            System.out.printf("[%s] Refueled to %.1f%%%n", id, fuelLevel);
        }
        
        String getStatus() {
            return String.format("[%s] %d %s %s (%s) Fuel:%.1f%%",
                id, year, make, model, getType(), fuelLevel);
        }
    }
    
    static class GasCar extends Vehicle {
        double fuelEfficiency; // km per liter
        
        GasCar(String id, String make, String model, int year, double efficiency) {
            super(id, make, model, year);
            this.fuelEfficiency = efficiency;
        }
        
        @Override
        double calculateFuelCost(double km) {
            return (km / fuelEfficiency) / (fuelLevel / 100 * 50); // simplified
        }
        
        @Override
        String getType() { return "Gasoline"; }
    }
    
    static class ElectricVehicle extends Vehicle {
        double batteryKwh;
        double efficiencyKmPerKwh;
        
        ElectricVehicle(String id, String make, String model, int year,
                         double batteryKwh, double efficiency) {
            super(id, make, model, year);
            this.batteryKwh = batteryKwh;
            this.efficiencyKmPerKwh = efficiency;
        }
        
        double getRange() {
            return batteryKwh * efficiencyKmPerKwh * fuelLevel / 100;
        }
        
        @Override
        double calculateFuelCost(double km) {
            double kwhNeeded = km / efficiencyKmPerKwh;
            return kwhNeeded / batteryKwh * 100;
        }
        
        @Override
        String getType() { return "Electric"; }
        
        @Override
        String getStatus() {
            return super.getStatus() + String.format(" Range:%.0fkm", getRange());
        }
    }
    
    static class Truck extends Vehicle {
        double payload; // tons
        int axles;
        
        Truck(String id, String make, String model, int year, double payload, int axles) {
            super(id, make, model, year);
            this.payload = payload;
            this.axles = axles;
        }
        
        @Override
        double calculateFuelCost(double km) {
            return km * 0.4; // 0.4 liter/km simplified
        }
        
        @Override
        String getType() { return "Truck (" + axles + "-axle)"; }
    }
    
    // Fleet management
    static List<Vehicle> fleet = new ArrayList<>();
    
    static void addVehicle(Vehicle v) {
        fleet.add(v);
        System.out.println("Added: " + v.getStatus());
    }
    
    static void showFleet() {
        System.out.println("\n=== Fleet Status ===");
        fleet.forEach(v -> System.out.println(v.getStatus()));
        
        long electric = fleet.stream().filter(v -> v instanceof ElectricVehicle).count();
        long gasoline = fleet.stream().filter(v -> v instanceof GasCar).count();
        long trucks = fleet.stream().filter(v -> v instanceof Truck).count();
        
        System.out.printf("Total: %d (EV: %d, Gas: %d, Truck: %d)%n",
            fleet.size(), electric, gasoline, trucks);
    }
    
    public static void main(String[] args) {
        addVehicle(new GasCar("G001", "Toyota", "Camry", 2022, 15));
        addVehicle(new GasCar("G002", "Honda", "Civic", 2021, 18));
        addVehicle(new ElectricVehicle("E001", "Tesla", "Model 3", 2023, 82, 6.5));
        addVehicle(new ElectricVehicle("E002", "BYD", "Atto 3", 2023, 60, 6.0));
        addVehicle(new Truck("T001", "Isuzu", "D-Max", 2020, 2.5, 2));
        
        showFleet();
        
        System.out.println("\n=== Electric Vehicles ===");
        for (Vehicle v : fleet) {
            if (v instanceof ElectricVehicle ev) {
                System.out.printf("%s - Range: %.0f km%n", ev.id, ev.getRange());
            }
        }
    }
}
```

---

## 11.9 แบบฝึกหัด Part 11

### แบบฝึกหัดที่ 1: Animal Kingdom

```java
public class AnimalKingdom {
    
    abstract static class Animal {
        protected String name;
        protected String habitat;
        
        Animal(String name, String habitat) {
            this.name = name;
            this.habitat = habitat;
        }
        
        abstract String getSound();
        abstract void move();
        
        void introduce() {
            System.out.printf("I'm %s, I live in %s and go '%s'%n",
                name, habitat, getSound());
        }
    }
    
    static class Lion extends Animal {
        Lion(String name) { super(name, "Savanna"); }
        
        @Override
        public String getSound() { return "ROAR!"; }
        
        @Override
        public void move() { System.out.println(name + " runs at 80 km/h"); }
        
        void hunt() { System.out.println(name + " hunts prey"); }
    }
    
    static class Eagle extends Animal {
        double wingspan;
        Eagle(String name, double wingspan) {
            super(name, "Mountains");
            this.wingspan = wingspan;
        }
        
        @Override
        public String getSound() { return "SCREECH!"; }
        
        @Override
        public void move() { System.out.printf("%s flies with %.1fm wingspan%n", name, wingspan); }
    }
    
    static class Dolphin extends Animal {
        Dolphin(String name) { super(name, "Ocean"); }
        
        @Override
        public String getSound() { return "Click-click!"; }
        
        @Override
        public void move() { System.out.println(name + " swims at 40 km/h"); }
        
        void echolocate() { System.out.println(name + " uses echolocation"); }
    }
    
    public static void main(String[] args) {
        Animal[] animals = {
            new Lion("Simba"),
            new Eagle("Zeus", 2.5),
            new Dolphin("Flipper")
        };
        
        for (Animal a : animals) {
            a.introduce();
            a.move();
            System.out.println();
        }
    }
}
```

---

## 11.10 สรุป Part 11

ในบทนี้คุณได้เรียนรู้:

✅ Inheritance พื้นฐาน (extends)  
✅ super keyword  
✅ Method Overriding (@Override)  
✅ final class/method  
✅ Abstract classes  
✅ Multi-level inheritance  
✅ instanceof operator  

---

*[← Part 10: Encapsulation](./part-10-oop-encapsulation.md) | [Part 12: Polymorphism →](./part-12-oop-polymorphism.md)*
