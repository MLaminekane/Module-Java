# Collections Java

## Hiérarchie des Collections

La hiérarchie des collections en Java est organisée autour de plusieurs interfaces principales:

- **Collection**: Interface racine pour toutes les collections
  - **List**: Collection ordonnée qui peut contenir des doublons
  - **Set**: Collection sans doublons
  - **Queue**: Collection conçue pour le traitement des éléments avant leur suppression
  - **Deque**: Queue à double extrémité (insertion/suppression aux deux extrémités)
- **Map**: Stocke des paires clé-valeur (n'hérite pas de Collection)

## List

```java
import java.util.ArrayList;
import java.util.LinkedList;
import java.util.List;

// ArrayList - implémentation basée sur un tableau dynamique
List<String> arrayList = new ArrayList<>();
arrayList.add("Apple");
arrayList.add("Banana");
arrayList.add("Cherry");

// Accès par index - O(1)
String fruit = arrayList.get(1); // "Banana"

// Insertion/suppression - O(n) dans le pire cas
arrayList.add(1, "Blueberry"); // Insère à l'index 1
arrayList.remove(2); // Supprime l'élément à l'index 2

// Taille
int size = arrayList.size(); // 3

// Parcours
for (String item : arrayList) {
    System.out.println(item);
}

// LinkedList - implémentation basée sur une liste doublement chaînée
List<String> linkedList = new LinkedList<>();
linkedList.add("Dog");
linkedList.add("Cat");
linkedList.add("Bird");

// Avantages de LinkedList: insertion/suppression efficaces - O(1)
// Inconvénients: accès par index plus lent - O(n)

// Méthodes communes aux List
boolean contains = linkedList.contains("Cat"); // true
int index = linkedList.indexOf("Dog"); // 0
linkedList.clear(); // Vide la liste
boolean isEmpty = linkedList.isEmpty(); // true
```

## Set

```java
import java.util.HashSet;
import java.util.LinkedHashSet;
import java.util.Set;
import java.util.TreeSet;

// HashSet - implémentation basée sur une table de hachage
Set<String> hashSet = new HashSet<>();
hashSet.add("Apple");
hashSet.add("Banana");
hashSet.add("Apple"); // Ignoré car déjà présent
System.out.println(hashSet.size()); // 2

// Vérification de présence - O(1) en moyenne
boolean contains = hashSet.contains("Apple"); // true

// LinkedHashSet - maintient l'ordre d'insertion
Set<String> linkedHashSet = new LinkedHashSet<>();
linkedHashSet.add("Cat");
linkedHashSet.add("Dog");
linkedHashSet.add("Bird");
// Itération dans l'ordre d'insertion: Cat, Dog, Bird

// TreeSet - tri les éléments selon leur ordre naturel ou un Comparator
Set<String> treeSet = new TreeSet<>();
treeSet.add("Zebra");
treeSet.add("Elephant");
treeSet.add("Ant");
// Itération dans l'ordre trié: Ant, Elephant, Zebra

// TreeSet avec Comparator personnalisé
Set<String> lengthSortedSet = new TreeSet<>((s1, s2) -> s1.length() - s2.length());
lengthSortedSet.add("Dog");
lengthSortedSet.add("Elephant");
lengthSortedSet.add("Cat");
// Itération par longueur: Cat, Dog, Elephant
```

## Map

```java
import java.util.HashMap;
import java.util.LinkedHashMap;
import java.util.Map;
import java.util.TreeMap;

// HashMap - implémentation basée sur une table de hachage
Map<String, Integer> hashMap = new HashMap<>();
hashMap.put("Apple", 10);
hashMap.put("Banana", 20);
hashMap.put("Cherry", 30);

// Accès aux valeurs - O(1) en moyenne
int value = hashMap.get("Banana"); // 20
Integer defaultValue = hashMap.getOrDefault("Orange", 0); // 0

// Vérification
boolean containsKey = hashMap.containsKey("Apple"); // true
boolean containsValue = hashMap.containsValue(30); // true

// Suppression
hashMap.remove("Cherry");

// LinkedHashMap - maintient l'ordre d'insertion
Map<String, Integer> linkedHashMap = new LinkedHashMap<>();
linkedHashMap.put("First", 1);
linkedHashMap.put("Second", 2);
linkedHashMap.put("Third", 3);
// Itération dans l'ordre d'insertion

// TreeMap - trie les entrées par clé
Map<String, Integer> treeMap = new TreeMap<>();
treeMap.put("Zebra", 26);
treeMap.put("Ant", 1);
treeMap.put("Dog", 4);
// Itération dans l'ordre des clés: Ant, Dog, Zebra

// Parcours d'une Map
for (Map.Entry<String, Integer> entry : hashMap.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}

// Java 8+ - forEach
hashMap.forEach((key, val) -> System.out.println(key + ": " + val));

// Récupération des clés ou valeurs
Set<String> keys = hashMap.keySet();
Collection<Integer> values = hashMap.values();
```

## Queue et Deque

```java
import java.util.ArrayDeque;
import java.util.Deque;
import java.util.LinkedList;
import java.util.PriorityQueue;
import java.util.Queue;

// Queue - File (FIFO: First In, First Out)
Queue<String> queue = new LinkedList<>();
queue.offer("First");  // Ajoute à la fin
queue.offer("Second");
queue.offer("Third");

String head = queue.peek(); // "First" (sans retirer)
String removed = queue.poll(); // "First" (et retire)

// PriorityQueue - File avec priorité basée sur l'ordre naturel ou un Comparator
Queue<Integer> priorityQueue = new PriorityQueue<>();
priorityQueue.offer(5);
priorityQueue.offer(1);
priorityQueue.offer(3);

int highest = priorityQueue.poll(); // 1 (le plus petit)

// File avec priorité personnalisée (ordre décroissant)
Queue<Integer> reversePriorityQueue = new PriorityQueue<>(Comparator.reverseOrder());
reversePriorityQueue.offer(5);
reversePriorityQueue.offer(1);
reversePriorityQueue.offer(3);

int highest2 = reversePriorityQueue.poll(); // 5 (le plus grand)

// Deque - Double-ended queue (insertion/suppression aux deux extrémités)
Deque<String> deque = new ArrayDeque<>();

// Opérations en tête (comme une pile)
deque.push("First");    // Ajoute en tête
deque.push("Second");
String top = deque.pop(); // "Second" (LIFO: Last In, First Out)

// Opérations en queue (comme une file)
deque.addLast("Third");
deque.addLast("Fourth");
String first = deque.removeFirst(); // "First"
```

## Utilitaires de Collections

```java
import java.util.Arrays;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;

List<Integer> numbers = new ArrayList<>(Arrays.asList(5, 2, 8, 1, 3));

// Tri
Collections.sort(numbers); // [1, 2, 3, 5, 8]
Collections.sort(numbers, Collections.reverseOrder()); // [8, 5, 3, 2, 1]

// Recherche binaire (dans une liste triée)
Collections.sort(numbers); // [1, 2, 3, 5, 8]
int index = Collections.binarySearch(numbers, 3); // 2

// Min/Max
int min = Collections.min(numbers); // 1
int max = Collections.max(numbers); // 8

// Remplissage et copie
Collections.fill(numbers, 0); // [0, 0, 0, 0, 0]
List<Integer> destination = Arrays.asList(0, 0, 0, 0, 0);
Collections.copy(destination, Arrays.asList(1, 2, 3, 4, 5));

// Mélange
Collections.shuffle(numbers);

// Collections immuables
List<String> immutableList = Collections.unmodifiableList(new ArrayList<>(Arrays.asList("a", "b", "c")));
// immutableList.add("d"); // Lance UnsupportedOperationException

// Collections singleton
Set<String> singletonSet = Collections.singleton("alone");
List<Integer> singletonList = Collections.singletonList(42);
Map<String, Integer> singletonMap = Collections.singletonMap("key", 42);
```

## Java 8+ Stream API

```java
import java.util.Arrays;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;
import java.util.stream.Stream;

List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "Dave", "Eve", "Frank");

// Filtrage
List<String> longNames = names.stream()
    .filter(name -> name.length() > 4)
    .collect(Collectors.toList()); // [Alice, Charlie, Frank]

// Transformation (map)
List<Integer> nameLengths = names.stream()
    .map(String::length)
    .collect(Collectors.toList()); // [5, 3, 7, 4, 3, 5]

// Tri
List<String> sortedNames = names.stream()
    .sorted()
    .collect(Collectors.toList()); // [Alice, Bob, Charlie, Dave, Eve, Frank]

// Tri personnalisé
List<String> sortedByLength = names.stream()
    .sorted(Comparator.comparing(String::length))
    .collect(Collectors.toList()); // [Bob, Eve, Dave, Alice, Frank, Charlie]

// Réduction
int totalChars = names.stream()
    .mapToInt(String::length)
    .sum(); // 27

String concatenated = names.stream()
    .collect(Collectors.joining(", ")); // "Alice, Bob, Charlie, Dave, Eve, Frank"

// Statistiques
IntSummaryStatistics stats = names.stream()
    .mapToInt(String::length)
    .summaryStatistics();
System.out.println("Average: " + stats.getAverage());
System.out.println("Max: " + stats.getMax());

// Groupement
Map<Integer, List<String>> byLength = names.stream()
    .collect(Collectors.groupingBy(String::length));
// {3=[Bob, Eve], 4=[Dave], 5=[Alice, Frank], 7=[Charlie]}

// Parallel streams
boolean anyMatch = names.parallelStream()
    .anyMatch(name -> name.startsWith("C")); // true

// Création de streams
Stream<Integer> streamOfInts = Stream.of(1, 2, 3, 4, 5);
Stream<String> streamFromArray = Arrays.stream(new String[]{"a", "b", "c"});
Stream<Integer> infiniteStream = Stream.iterate(0, n -> n + 2); // 0, 2, 4, 6, ...
Stream<Integer> limitedStream = infiniteStream.limit(5); // 0, 2, 4, 6, 8
```

## Opérations sur les collections avec Java 10+

```java
// Java 9: méthodes de fabrique pour collections immuables
List<String> immutableList = List.of("a", "b", "c");
Set<Integer> immutableSet = Set.of(1, 2, 3);
Map<String, Integer> immutableMap = Map.of("one", 1, "two", 2);

// Java 10: var et méthodes copyOf
var numbers = List.of(1, 2, 3, 4, 5);
var copyOfNumbers = List.copyOf(numbers); // Copie immuable

// Java 10: Collectors.toUnmodifiableList/Set/Map
List<String> unmodifiable = names.stream()
    .filter(name -> name.length() > 4)
    .collect(Collectors.toUnmodifiableList());
```
