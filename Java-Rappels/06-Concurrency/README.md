# Concurrence en Java

## Threads de base

```java
// Méthode 1: Étendre la classe Thread
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Thread en cours d'exécution: " + Thread.currentThread().getName());
        for (int i = 0; i < 5; i++) {
            System.out.println(Thread.currentThread().getName() + ": " + i);
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                System.out.println("Thread interrompu");
                return;
            }
        }
    }
}

// Méthode 2: Implémenter l'interface Runnable
class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Runnable en cours d'exécution: " + Thread.currentThread().getName());
        for (int i = 0; i < 5; i++) {
            System.out.println(Thread.currentThread().getName() + ": " + i);
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                System.out.println("Thread interrompu");
                return;
            }
        }
    }
}

public class ThreadBasics {
    public static void main(String[] args) {
        // Création et démarrage d'un thread via héritage
        MyThread thread1 = new MyThread();
        thread1.setName("MyThread");
        thread1.start();

        // Création et démarrage d'un thread via Runnable
        Thread thread2 = new Thread(new MyRunnable(), "MyRunnable");
        thread2.start();

        // Création d'un thread avec une expression lambda (Java 8+)
        Thread thread3 = new Thread(() -> {
            System.out.println("Lambda en cours d'exécution: " + Thread.currentThread().getName());
            for (int i = 0; i < 5; i++) {
                System.out.println(Thread.currentThread().getName() + ": " + i);
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    System.out.println("Thread interrompu");
                    return;
                }
            }
        }, "LambdaThread");

        thread3.start();

        // Attente de la fin des threads
        try {
            thread1.join();
            thread2.join();
            thread3.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("Tous les threads ont terminé");
    }
}
```

## Synchronisation

```java
class Counter {
    private int count = 0;

    // Méthode synchronisée
    public synchronized void increment() {
        count++;
    }

    // Bloc synchronisé
    public void incrementWithBlock() {
        synchronized(this) {
            count++;
        }
    }

    public int getCount() {
        return count;
    }
}

class Worker extends Thread {
    private Counter counter;
    private int increments;

    public Worker(Counter counter, String name, int increments) {
        super(name);
        this.counter = counter;
        this.increments = increments;
    }

    @Override
    public void run() {
        for (int i = 0; i < increments; i++) {
            counter.increment();
        }
    }
}

public class SynchronizationExample {
    public static void main(String[] args) throws InterruptedException {
        Counter counter = new Counter();
        int numThreads = 10;
        int incrementsPerThread = 1000;

        Thread[] threads = new Thread[numThreads];

        // Création et démarrage des threads
        for (int i = 0; i < numThreads; i++) {
            threads[i] = new Worker(counter, "Worker-" + i, incrementsPerThread);
            threads[i].start();
        }

        // Attente de la fin de tous les threads
        for (Thread thread : threads) {
            thread.join();
        }

        // Vérification du résultat
        System.out.println("Valeur finale attendue: " + (numThreads * incrementsPerThread));
        System.out.println("Valeur finale obtenue: " + counter.getCount());
    }
}
```

## Locks et Conditions

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.locks.Condition;

class BoundedBuffer {
    private final Lock lock = new ReentrantLock();
    private final Condition notFull = lock.newCondition();
    private final Condition notEmpty = lock.newCondition();

    private final Object[] items;
    private int putIndex, takeIndex, count;

    public BoundedBuffer(int capacity) {
        items = new Object[capacity];
    }

    public void put(Object item) throws InterruptedException {
        lock.lock();
        try {
            while (count == items.length) {
                // Buffer est plein, attendre
                notFull.await();
            }

            items[putIndex] = item;
            putIndex = (putIndex + 1) % items.length;
            count++;

            // Signaler qu'un élément est disponible
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
    }

    public Object take() throws InterruptedException {
        lock.lock();
        try {
            while (count == 0) {
                // Buffer est vide, attendre
                notEmpty.await();
            }

            Object item = items[takeIndex];
            items[takeIndex] = null;
            takeIndex = (takeIndex + 1) % items.length;
            count--;

            // Signaler qu'une place est disponible
            notFull.signal();
            return item;
        } finally {
            lock.unlock();
        }
    }
}

public class LockExample {
    public static void main(String[] args) {
        BoundedBuffer buffer = new BoundedBuffer(5);

        // Producteur
        Thread producer = new Thread(() -> {
            try {
                for (int i = 0; i < 10; i++) {
                    System.out.println("Producteur: production de l'élément " + i);
                    buffer.put(i);
                    Thread.sleep(100);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        // Consommateur
        Thread consumer = new Thread(() -> {
            try {
                for (int i = 0; i < 10; i++) {
                    Object item = buffer.take();
                    System.out.println("Consommateur: consommation de l'élément " + item);
                    Thread.sleep(200);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        producer.start();
        consumer.start();
    }
}
```

## ExecutorService et Thread Pools

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

public class ExecutorServiceExample {
    public static void main(String[] args) {
        // Création d'un pool de threads fixe
        ExecutorService fixedPool = Executors.newFixedThreadPool(3);

        // Soumission de tâches
        for (int i = 0; i < 10; i++) {
            final int taskId = i;
            fixedPool.submit(() -> {
                String threadName = Thread.currentThread().getName();
                System.out.println("Tâche " + taskId + " exécutée par " + threadName);
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                return "Résultat de la tâche " + taskId;
            });
        }

        // Arrêt de l'ExecutorService
        fixedPool.shutdown();
        try {
            if (!fixedPool.awaitTermination(5, TimeUnit.SECONDS)) {
                fixedPool.shutdownNow();
            }
        } catch (InterruptedException e) {
            fixedPool.shutdownNow();
        }

        System.out.println("Toutes les tâches sont terminées");

        // Autres types de pools
        ExecutorService singleThreadExecutor = Executors.newSingleThreadExecutor();
        ExecutorService cachedThreadPool = Executors.newCachedThreadPool();
        ExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(2);

        // N'oubliez pas de les arrêter après utilisation
        singleThreadExecutor.shutdown();
        cachedThreadPool.shutdown();
        scheduledThreadPool.shutdown();
    }
}
```

## Future, Callable et CompletableFuture

```java
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.*;

public class FutureExample {
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        ExecutorService executor = Executors.newFixedThreadPool(3);

        // Callable (avec valeur de retour)
        Callable<Integer> task = () -> {
            System.out.println("Calcul en cours...");
            Thread.sleep(2000);
            return 42;
        };

        // Soumission d'une tâche Callable et obtention d'un Future
        Future<Integer> future = executor.submit(task);

        // Vérification si la tâche est terminée
        System.out.println("La tâche est terminée? " + future.isDone());

        // Attente et récupération du résultat (bloquant)
        Integer result = future.get();
        System.out.println("Résultat: " + result);

        // Exécution de plusieurs tâches en parallèle
        List<Future<String>> futures = new ArrayList<>();
        for (int i = 0; i < 5; i++) {
            final int id = i;
            futures.add(executor.submit(() -> {
                Thread.sleep(1000);
                return "Résultat " + id;
            }));
        }

        // Récupération de tous les résultats
        for (Future<String> f : futures) {
            System.out.println(f.get());
        }

        // CompletableFuture (Java 8+)
        CompletableFuture<String> cf = CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "Étape 1";
        }).thenApply(s -> {
            return s + " -> Étape 2";
        }).thenApply(s -> {
            return s + " -> Étape 3";
        });

        // Attente du résultat final
        System.out.println("CompletableFuture: " + cf.get());

        // Composition de CompletableFuture
        CompletableFuture<String> cf1 = CompletableFuture.supplyAsync(() -> "Hello");
        CompletableFuture<String> cf2 = CompletableFuture.supplyAsync(() -> "World");

        CompletableFuture<String> combined = cf1.thenCombine(cf2, (s1, s2) -> s1 + " " + s2);
        System.out.println("Combiné: " + combined.get());

        executor.shutdown();
    }
}
```

## Collections concurrentes

```java
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.*;

public class ConcurrentCollectionsExample {
    public static void main(String[] args) throws InterruptedException {
        // Map standard (non thread-safe)
        Map<String, Integer> hashMap = new HashMap<>();

        // ConcurrentHashMap (thread-safe)
        ConcurrentMap<String, Integer> concurrentMap = new ConcurrentHashMap<>();

        // CopyOnWriteArrayList (thread-safe pour les itérations)
        CopyOnWriteArrayList<String> copyOnWriteList = new CopyOnWriteArrayList<>();

        // BlockingQueue (pour producteur-consommateur)
        BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(10);

        // Exemple d'utilisation de ConcurrentHashMap
        ExecutorService executor = Executors.newFixedThreadPool(3);

        // Plusieurs threads écrivant dans la map
        for (int i = 0; i < 10; i++) {
            final String key = "key" + i;
            executor.submit(() -> {
                concurrentMap.put(key, (int) (Math.random() * 100));
                System.out.println(Thread.currentThread().getName() + " a ajouté " + key);
            });
        }

        // Exemple d'utilisation de BlockingQueue
        // Producteur
        executor.submit(() -> {
            try {
                for (int i = 0; i < 5; i++) {
                    blockingQueue.put("Item " + i);
                    System.out.println("Produit: Item " + i);
                    Thread.sleep(100);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        // Consommateur
        executor.submit(() -> {
            try {
                for (int i = 0; i < 5; i++) {
                    String item = blockingQueue.take();
                    System.out.println("Consommé: " + item);
                    Thread.sleep(200);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        executor.shutdown();
        executor.awaitTermination(5, TimeUnit.SECONDS);

        // Affichage du contenu final de la map
        System.out.println("\nContenu final de la map:");
        concurrentMap.forEach((k, v) -> System.out.println(k + " -> " + v));
    }
}
```

## Synchroniseurs

```java
import java.util.concurrent.*;

public class SynchronizersExample {
    public static void main(String[] args) throws InterruptedException {
        // CountDownLatch: permet à un thread d'attendre que plusieurs autres threads terminent
        CountDownLatch startSignal = new CountDownLatch(1);
        CountDownLatch doneSignal = new CountDownLatch(3);

        for (int i = 0; i < 3; i++) {
            final int workerId = i;
            new Thread(() -> {
                try {
                    System.out.println("Worker " + workerId + " prêt");
                    startSignal.await(); // Attendre le signal de départ

                    System.out.println("Worker " + workerId + " commence");
                    Thread.sleep((long) (Math.random() * 1000));
                    System.out.println("Worker " + workerId + " termine");

                    doneSignal.countDown(); // Signaler la fin
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }).start();
        }

        Thread.sleep(1000); // Donner le temps aux workers de se préparer
        System.out.println("Signal de départ envoyé");
        startSignal.countDown(); // Donner le signal de départ

        doneSignal.await(); // Attendre que tous les workers terminent
        System.out.println("Tous les workers ont terminé");

        // CyclicBarrier: permet à plusieurs threads de s'attendre mutuellement à un point
        CyclicBarrier barrier = new CyclicBarrier(3, () -> {
            // Action à exécuter quand tous les threads atteignent la barrière
            System.out.println("Barrière franchie par tous les threads!");
        });

        for (int i = 0; i < 3; i++) {
            final int threadId = i;
            new Thread(() -> {
                try {
                    System.out.println("Thread " + threadId + " travaille");
                    Thread.sleep((long) (Math.random() * 2000));

                    System.out.println("Thread " + threadId + " attend à la barrière");
                    barrier.await(); // Attendre les autres threads

                    System.out.println("Thread " + threadId + " continue après la barrière");
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }).start();
        }

        // Semaphore: contrôle l'accès à une ressource limitée
        Semaphore semaphore = new Semaphore(2); // 2 permis disponibles

        for (int i = 0; i < 5; i++) {
            final int threadId = i;
            new Thread(() -> {
                try {
                    System.out.println("Thread " + threadId + " attend un permis");
                    semaphore.acquire(); // Acquérir un permis

                    System.out.println("Thread " + threadId + " a obtenu un permis");
                    Thread.sleep(1000);

                    System.out.println("Thread " + threadId + " libère le permis");
                    semaphore.release(); // Libérer le permis
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
}
```

## Atomic Variables

```java
import java.util.concurrent.atomic.*;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

public class AtomicVariablesExample {
    // Variables atomiques
    private static AtomicInteger atomicCounter = new AtomicInteger(0);
    private static AtomicLong atomicLong = new AtomicLong(0);
    private static AtomicBoolean atomicBoolean = new AtomicBoolean(false);
    private static AtomicReference<String> atomicReference = new AtomicReference<>("initial");

    // Compteur non atomique pour comparaison
    private static int nonAtomicCounter = 0;

    public static void main(String[] args) throws InterruptedException {
        ExecutorService executor = Executors.newFixedThreadPool(10);

        // Incrémentation atomique
        for (int i = 0; i < 1000; i++) {
            executor.submit(() -> {
                atomicCounter.incrementAndGet(); // Opération atomique
                nonAtomicCounter++; // Opération non atomique
            });
        }

        // Opérations atomiques complexes
        executor.submit(() -> {
            atomicLong.updateAndGet(x -> x + 100);
            atomicBoolean.compareAndSet(false, true);
            atomicReference.accumulateAndGet("updated", (prev, next) -> prev + "-" + next);
        });

        executor.shutdown();
        executor.awaitTermination(5, TimeUnit.SECONDS);

        System.out.println("Compteur atomique: " + atomicCounter.get());
        System.out.println("Compteur non atomique: " + nonAtomicCounter);
        System.out.println("AtomicLong: " + atomicLong.get());
        System.out.println("AtomicBoolean: " + atomicBoolean.get());
        System.out.println("AtomicReference: " + atomicReference.get());

        // AtomicIntegerArray
        AtomicIntegerArray atomicArray = new AtomicIntegerArray(10);
        atomicArray.set(5, 100);
        System.out.println("AtomicIntegerArray[5]: " + atomicArray.get(5));

        // Opérations conditionnelles
        atomicCounter.compareAndSet(1000, 2000); // Change la valeur seulement si elle est 1000
        System.out.println("Après compareAndSet: " + atomicCounter.get());
    }
}
```

## Fork/Join Framework

```java
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.RecursiveTask;
import java.util.Arrays;

// Tâche récursive qui retourne un résultat
class SumTask extends RecursiveTask<Long> {
    private static final int THRESHOLD = 1000;
    private final long[] array;
    private final int start;
    private final int end;

    public SumTask(long[] array, int start, int end) {
        this.array = array;
        this.start = start;
        this.end = end;
    }

    @Override
    protected Long compute() {
        int length = end - start;
        if (length <= THRESHOLD) {
            // Calcul direct pour les petits tableaux
            return computeDirectly();
        }

        // Division du problème en sous-tâches
        int middle = start + length / 2;

        SumTask leftTask = new SumTask(array, start, middle);
        SumTask rightTask = new SumTask(array, middle, end);

        // Fork: soumettre une sous-tâche pour exécution asynchrone
        leftTask.fork();

        // Compute: exécuter directement l'autre sous-tâche
        long rightResult = rightTask.compute();

        // Join: attendre et obtenir le résultat de la sous-tâche
        long leftResult = leftTask.join();

        // Combiner les résultats
        return leftResult + rightResult;
    }

    private long computeDirectly() {
        long sum = 0;
        for (int i = start; i < end; i++) {
            sum += array[i];
        }
        return sum;
    }
}

public class ForkJoinExample {
    public static void main(String[] args) {
        // Création d'un grand tableau
        long[] numbers = new long[100_000_000];
        Arrays.fill(numbers, 1);

        // Création du pool Fork/Join
        ForkJoinPool forkJoinPool = new ForkJoinPool();

        // Création de la tâche
        SumTask task = new SumTask(numbers, 0, numbers.length);

        // Exécution et mesure du temps
        long startTime = System.currentTimeMillis();
        long result = forkJoinPool.invoke(task);
        long endTime = System.currentTimeMillis();

        System.out.println("Somme calculée: " + result);
        System.out.println("Temps d'exécution: " + (endTime - startTime) + " ms");

        // Calcul séquentiel pour comparaison
        startTime = System.currentTimeMillis();
        long sum = 0;
        for (long num : numbers) {
            sum += num;
        }
        endTime = System.currentTimeMillis();

        System.out.println("Somme séquentielle: " + sum);
        System.out.println("Temps d'exécution séquentiel: " + (endTime - startTime) + " ms");
    }
}
```

## ThreadLocal

```java
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

public class ThreadLocalExample {
    // ThreadLocal pour stocker un format de date par thread
    private static final ThreadLocal<SimpleDateFormat> dateFormatThreadLocal =
        ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));

    // Variable d'instance normale (partagée entre threads)
    private static SimpleDateFormat sharedDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    public static String formatDate(Date date) {
        // Utilise l'instance ThreadLocal
        return dateFormatThreadLocal.get().format(date);
    }

    public static String formatDateShared(Date date) {
        // Utilise l'instance partagée (non thread-safe)
        return sharedDateFormat.format(date);
    }

    public static void main(String[] args) throws InterruptedException {
        ExecutorService executor = Executors.newFixedThreadPool(10);

        for (int i = 0; i < 100; i++) {
            final int id = i;
            executor.submit(() -> {
                // Utilisation de ThreadLocal (thread-safe)
                String formattedDate = formatDate(new Date());
                System.out.println("Thread " + id + " (ThreadLocal): " + formattedDate);

                // Utilisation de l'instance partagée (non thread-safe)
                // Peut causer des problèmes en cas d'accès concurrent
                String sharedFormattedDate = formatDateShared(new Date());
                System.out.println("Thread " + id + " (Shared): " + sharedFormattedDate);
            });
        }

        executor.shutdown();
        executor.awaitTermination(5, TimeUnit.SECONDS);

        // Nettoyage des ThreadLocal pour éviter les fuites mémoire
        dateFormatThreadLocal.remove();
    }
}
```

## Nouveautés Java 9+

```java
import java.util.List;
import java.util.concurrent.*;
import java.util.stream.Collectors;

public class Java9PlusConcurrency {
    public static void main(String[] args) throws Exception {
        // Java 9: Factory methods pour les collections immuables
        List<Integer> immutableList = List.of(1, 2, 3, 4, 5);

        // Java 9: CompletableFuture amélioré
        CompletableFuture<String> cf = CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            return "Résultat";
        });

        // Nouvelles méthodes
        cf.orTimeout(2, TimeUnit.SECONDS)        // Timeout avec exception
          .completeOnTimeout("Défaut", 3, TimeUnit.SECONDS); // Timeout avec valeur par défaut

        // Java 9: SubmissionPublisher pour Reactive Streams
        try (SubmissionPublisher<Integer> publisher = new SubmissionPublisher<>()) {
            // Abonné simple
            publisher.subscribe(new Flow.Subscriber<>() {
                private Flow.Subscription subscription;

                @Override
                public void onSubscribe(Flow.Subscription subscription) {
                    this.subscription = subscription;
                    subscription.request(1);
                }

                @Override
                public void onNext(Integer item) {
                    System.out.println("Reçu: " + item);
                    subscription.request(1);
                }

                @Override
                public void onError(Throwable throwable) {
                    throwable.printStackTrace();
                }

                @Override
                public void onComplete() {
                    System.out.println("Terminé");
                }
            });

            // Publication d'éléments
            for (int i = 0; i < 5; i++) {
                publisher.submit(i);
            }

            Thread.sleep(1000); // Attendre le traitement
        }

        // Java 12+: CompletableFuture.completeAsync
        CompletableFuture<String> asyncComplete = new CompletableFuture<>();
        CompletableFuture<String> resultFuture = asyncComplete.completeAsync(() -> {
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            return "Complété de manière asynchrone";
        });

        System.out.println(resultFuture.get());
    }
}
```
