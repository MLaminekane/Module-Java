# Bases de Java

## Types primitifs

```java
// Types numériques
byte b = 127;                // 8 bits, -128 à 127
short s = 32767;             // 16 bits, -32768 à 32767
int i = 2147483647;          // 32 bits, -2147483648 à 2147483647
long l = 9223372036854775807L; // 64 bits, -9223372036854775808 à 9223372036854775807
float f = 3.14f;             // 32 bits, précision simple
double d = 3.14159265359;    // 64 bits, précision double

// Autres types
char c = 'A';                // 16 bits, caractère Unicode
boolean bool = true;         // true ou false
```

## Opérateurs

```java
// Arithmétiques
int sum = 5 + 3;       // 8
int diff = 5 - 3;      // 2
int product = 5 * 3;   // 15
int quotient = 5 / 2;  // 2 (division entière)
int remainder = 5 % 2; // 1 (modulo)

// Incrémentation/Décrémentation
int x = 5;
x++;                   // x = 6
x--;                   // x = 5

// Comparaison
boolean isEqual = (5 == 3);      // false
boolean isNotEqual = (5 != 3);   // true
boolean isGreater = (5 > 3);     // true
boolean isLessOrEqual = (5 <= 5); // true

// Logiques
boolean andResult = true && false; // false
boolean orResult = true || false;  // true
boolean notResult = !true;         // false
```

## Structures de contrôle

```java
// If-else
int age = 18;
if (age >= 18) {
    System.out.println("Majeur");
} else {
    System.out.println("Mineur");
}

// Switch
String day = "Monday";
switch (day) {
    case "Monday":
        System.out.println("Début de semaine");
        break;
    case "Friday":
        System.out.println("Fin de semaine");
        break;
    default:
        System.out.println("Milieu de semaine");
}

// Boucle for
for (int i = 0; i < 5; i++) {
    System.out.println(i);
}

// Boucle while
int count = 0;
while (count < 5) {
    System.out.println(count);
    count++;
}

// Boucle do-while
int j = 0;
do {
    System.out.println(j);
    j++;
} while (j < 5);

// For-each (depuis Java 5)
int[] numbers = {1, 2, 3, 4, 5};
for (int num : numbers) {
    System.out.println(num);
}
```

## Tableaux

```java
// Déclaration et initialisation
int[] array1 = new int[5];       // Tableau de 5 entiers initialisés à 0
int[] array2 = {1, 2, 3, 4, 5};  // Initialisation directe

// Tableaux multidimensionnels
int[][] matrix = {
    {1, 2, 3},
    {4, 5, 6},
    {7, 8, 9}
};

// Accès aux éléments
int element = array2[2];         // 3 (indexé à partir de 0)
int matrixElement = matrix[1][2]; // 6
```

## Méthodes

```java
// Déclaration de méthode
public static int add(int a, int b) {
    return a + b;
}

// Méthode avec paramètres variables (varargs)
public static int sum(int... numbers) {
    int total = 0;
    for (int num : numbers) {
        total += num;
    }
    return total;
}

// Appel de méthode
int result = add(5, 3);          // 8
int total = sum(1, 2, 3, 4, 5);  // 15
```

## Chaînes de caractères

```java
// Création
String s1 = "Hello";
String s2 = new String("World");

// Concaténation
String s3 = s1 + " " + s2;       // "Hello World"
String s4 = s1.concat(" ").concat(s2); // "Hello World"

// Méthodes utiles
int length = s1.length();        // 5
char firstChar = s1.charAt(0);   // 'H'
boolean startsWith = s1.startsWith("He"); // true
String upper = s1.toUpperCase(); // "HELLO"
String sub = s1.substring(1, 4); // "ell"
```

## Classes wrapper

```java
// Autoboxing/Unboxing (depuis Java 5)
Integer intObj = 42;             // Autoboxing (int -> Integer)
int primitive = intObj;          // Unboxing (Integer -> int)

// Conversion de types
String numStr = "42";
int num = Integer.parseInt(numStr); // String -> int
String backToStr = Integer.toString(num); // int -> String
```
