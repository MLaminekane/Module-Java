# Programmation Orientée Objet en Java

## Classes et Objets

```java
// Définition d'une classe
public class Person {
    // Attributs (variables d'instance)
    private String name;
    private int age;

    // Constructeur
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // Constructeur par défaut
    public Person() {
        this("Unknown", 0);
    }

    // Getters et Setters
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        if (age >= 0) {
            this.age = age;
        }
    }

    // Méthode d'instance
    public void displayInfo() {
        System.out.println("Name: " + name + ", Age: " + age);
    }

    // Méthode statique (de classe)
    public static Person createAdult(String name) {
        return new Person(name, 18);
    }

    // Redéfinition de méthode de Object
    @Override
    public String toString() {
        return "Person [name=" + name + ", age=" + age + "]";
    }
}

// Utilisation
Person person1 = new Person("Alice", 30);
Person person2 = new Person();
Person adult = Person.createAdult("Bob");

person1.displayInfo();
System.out.println(person2);
```

## Héritage

```java
// Classe de base (parent)
public class Animal {
    protected String name;

    public Animal(String name) {
        this.name = name;
    }

    public void eat() {
        System.out.println(name + " is eating");
    }

    public void makeSound() {
        System.out.println("Some generic sound");
    }
}

// Classe dérivée (enfant)
public class Dog extends Animal {
    private String breed;

    public Dog(String name, String breed) {
        super(name); // Appel du constructeur parent
        this.breed = breed;
    }

    // Redéfinition d'une méthode
    @Override
    public void makeSound() {
        System.out.println(name + " barks");
    }

    // Nouvelle méthode spécifique à Dog
    public void fetch() {
        System.out.println(name + " is fetching");
    }
}

// Utilisation
Animal animal = new Animal("Generic Animal");
Dog dog = new Dog("Rex", "German Shepherd");

animal.makeSound(); // "Some generic sound"
dog.makeSound();    // "Rex barks"
dog.eat();          // Hérité de Animal
dog.fetch();        // Spécifique à Dog

// Polymorphisme
Animal polymorphic = new Dog("Max", "Beagle");
polymorphic.makeSound(); // "Max barks" (méthode de Dog)
// polymorphic.fetch(); // Erreur de compilation
```

## Interfaces

```java
// Définition d'une interface
public interface Drawable {
    void draw(); // Méthode abstraite implicite

    // Méthode par défaut (depuis Java 8)
    default void displayInfo() {
        System.out.println("Drawing a " + getClass().getSimpleName());
    }

    // Méthode statique (depuis Java 8)
    static void printVersion() {
        System.out.println("Drawable API v1.0");
    }
}

// Implémentation d'une interface
public class Circle implements Drawable {
    private double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    @Override
    public void draw() {
        System.out.println("Drawing a circle with radius " + radius);
    }
}

// Classe implémentant plusieurs interfaces
public class Rectangle implements Drawable, Resizable {
    private double width;
    private double height;

    // Implémentation des méthodes...
}

// Utilisation
Drawable circle = new Circle(5.0);
circle.draw();       // "Drawing a circle with radius 5.0"
circle.displayInfo(); // Méthode par défaut
Drawable.printVersion(); // Méthode statique
```

## Classes et méthodes abstraites

```java
// Classe abstraite
public abstract class Shape {
    protected String color;

    public Shape(String color) {
        this.color = color;
    }

    // Méthode concrète
    public String getColor() {
        return color;
    }

    // Méthode abstraite (à implémenter par les sous-classes)
    public abstract double getArea();
}

// Implémentation de classe abstraite
public class Square extends Shape {
    private double side;

    public Square(String color, double side) {
        super(color);
        this.side = side;
    }

    @Override
    public double getArea() {
        return side * side;
    }
}

// Utilisation
// Shape shape = new Shape("Red"); // Erreur: impossible d'instancier une classe abstraite
Shape square = new Square("Blue", 5.0);
System.out.println("Area: " + square.getArea());
System.out.println("Color: " + square.getColor());
```

## Encapsulation

```java
public class BankAccount {
    // Encapsulation: attributs privés
    private String accountNumber;
    private double balance;
    private static final double WITHDRAWAL_LIMIT = 1000.0;

    public BankAccount(String accountNumber, double initialBalance) {
        this.accountNumber = accountNumber;
        this.balance = initialBalance;
    }

    // Méthodes publiques pour accéder aux attributs privés
    public double getBalance() {
        return balance;
    }

    public String getAccountNumber() {
        return accountNumber;
    }

    // Logique métier encapsulée
    public boolean withdraw(double amount) {
        if (amount <= 0) {
            return false;
        }

        if (amount > WITHDRAWAL_LIMIT) {
            System.out.println("Exceeds withdrawal limit");
            return false;
        }

        if (amount > balance) {
            System.out.println("Insufficient funds");
            return false;
        }

        balance -= amount;
        return true;
    }

    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }
}
```

## Enumérations

```java
// Énumération simple
public enum Day {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}

// Énumération avec attributs et méthodes
public enum Season {
    WINTER(0), SPRING(10), SUMMER(30), FALL(20);

    private final int averageTemperature;

    Season(int averageTemperature) {
        this.averageTemperature = averageTemperature;
    }

    public int getAverageTemperature() {
        return averageTemperature;
    }

    public boolean isWarm() {
        return averageTemperature >= 20;
    }
}

// Utilisation
Day today = Day.MONDAY;
System.out.println(today); // "MONDAY"

Season summer = Season.SUMMER;
System.out.println("Temperature: " + summer.getAverageTemperature());
System.out.println("Is warm? " + summer.isWarm());

// Switch avec enum
switch (today) {
    case MONDAY:
    case TUESDAY:
        System.out.println("Start of week");
        break;
    case SATURDAY:
    case SUNDAY:
        System.out.println("Weekend");
        break;
    default:
        System.out.println("Mid-week");
}
```

## Classes internes

```java
public class Outer {
    private int outerField = 10;

    // Classe interne (inner class)
    public class Inner {
        public void display() {
            System.out.println("Outer field: " + outerField);
        }
    }

    // Classe interne statique (static nested class)
    public static class StaticNested {
        public void display() {
            // Ne peut pas accéder directement à outerField
            System.out.println("Static nested class");
        }
    }

    // Méthode avec classe locale
    public void method() {
        final int localVar = 20;

        // Classe locale
        class LocalClass {
            void display() {
                System.out.println("Local var: " + localVar);
                System.out.println("Outer field: " + outerField);
            }
        }

        LocalClass local = new LocalClass();
        local.display();
    }

    // Méthode retournant une classe anonyme
    public Runnable getRunnable() {
        return new Runnable() {
            @Override
            public void run() {
                System.out.println("Anonymous class running");
            }
        };
    }
}

// Utilisation
Outer outer = new Outer();

// Classe interne
Outer.Inner inner = outer.new Inner();
inner.display();

// Classe interne statique
Outer.StaticNested staticNested = new Outer.StaticNested();
staticNested.display();

// Classe locale
outer.method();

// Classe anonyme
Runnable runnable = outer.getRunnable();
runnable.run();
```

## Records (depuis Java 16)

```java
// Record - classe de données immutable
public record Point(int x, int y) {
    // Constructeur compact implicite

    // Méthodes supplémentaires
    public double distanceFromOrigin() {
        return Math.sqrt(x * x + y * y);
    }

    // Constructeur personnalisé
    public Point {
        if (x < 0 || y < 0) {
            throw new IllegalArgumentException("Coordinates cannot be negative");
        }
    }
}

// Utilisation
Point p = new Point(3, 4);
System.out.println(p.x()); // Getter généré automatiquement
System.out.println(p.y());
System.out.println(p.distanceFromOrigin()); // 5.0
System.out.println(p); // toString() généré automatiquement
```
