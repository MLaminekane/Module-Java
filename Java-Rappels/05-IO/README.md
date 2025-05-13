# Entrées/Sorties (I/O) en Java

## Flux d'entrée/sortie de base

Java fournit un ensemble complet de classes pour gérer les entrées/sorties via des flux (streams).

### Flux d'octets

```java
import java.io.*;

public class ByteStreamExample {
    public static void main(String[] args) {
        // Écriture d'octets dans un fichier
        try (FileOutputStream fos = new FileOutputStream("data.bin")) {
            byte[] data = {65, 66, 67, 68, 69}; // ASCII pour "ABCDE"
            fos.write(data);
            System.out.println("Données écrites avec succès");
        } catch (IOException e) {
            e.printStackTrace();
        }

        // Lecture d'octets depuis un fichier
        try (FileInputStream fis = new FileInputStream("data.bin")) {
            int value;
            System.out.print("Données lues: ");
            while ((value = fis.read()) != -1) {
                System.out.print((char) value); // Conversion en caractère
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### Flux de caractères

```java
import java.io.*;

public class CharacterStreamExample {
    public static void main(String[] args) {
        // Écriture de caractères dans un fichier
        try (FileWriter writer = new FileWriter("text.txt")) {
            writer.write("Bonjour, monde!");
            System.out.println("Texte écrit avec succès");
        } catch (IOException e) {
            e.printStackTrace();
        }

        // Lecture de caractères depuis un fichier
        try (FileReader reader = new FileReader("text.txt")) {
            int character;
            System.out.print("Texte lu: ");
            while ((character = reader.read()) != -1) {
                System.out.print((char) character);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## Flux tamponnés

Les flux tamponnés améliorent les performances en réduisant le nombre d'accès au système.

```java
import java.io.*;

public class BufferedStreamExample {
    public static void main(String[] args) {
        // Écriture avec tampon
        try (BufferedWriter writer = new BufferedWriter(new FileWriter("buffered.txt"))) {
            writer.write("Première ligne");
            writer.newLine(); // Ajoute un saut de ligne
            writer.write("Deuxième ligne");
            System.out.println("Texte écrit avec succès");
        } catch (IOException e) {
            e.printStackTrace();
        }

        // Lecture avec tampon
        try (BufferedReader reader = new BufferedReader(new FileReader("buffered.txt"))) {
            String line;
            System.out.println("Contenu lu ligne par ligne:");
            while ((line = reader.readLine()) != null) {
                System.out.println(line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## Lecture et écriture d'objets (Sérialisation)

```java
import java.io.*;

// La classe doit implémenter Serializable
class Person implements Serializable {
    // serialVersionUID pour contrôler la compatibilité des versions
    private static final long serialVersionUID = 1L;

    private String name;
    private int age;
    // Les champs transient ne sont pas sérialisés
    private transient String temporaryData;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
        this.temporaryData = "Cette donnée ne sera pas sérialisée";
    }

    @Override
    public String toString() {
        return "Person [name=" + name + ", age=" + age +
               ", temporaryData=" + temporaryData + "]";
    }
}

public class SerializationExample {
    public static void main(String[] args) {
        // Sérialisation (écriture d'objets)
        try (ObjectOutputStream oos = new ObjectOutputStream(
                new FileOutputStream("person.ser"))) {
            Person person = new Person("Alice", 30);
            oos.writeObject(person);
            System.out.println("Objet sérialisé avec succès");
        } catch (IOException e) {
            e.printStackTrace();
        }

        // Désérialisation (lecture d'objets)
        try (ObjectInputStream ois = new ObjectInputStream(
                new FileInputStream("person.ser"))) {
            Person person = (Person) ois.readObject();
            System.out.println("Objet désérialisé: " + person);
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```

## Fichiers et répertoires avec java.io.File

```java
import java.io.File;
import java.io.IOException;

public class FileExample {
    public static void main(String[] args) {
        // Création d'un objet File
        File file = new File("example.txt");

        // Vérification si le fichier existe
        if (!file.exists()) {
            try {
                // Création d'un nouveau fichier
                boolean created = file.createNewFile();
                System.out.println("Fichier créé: " + created);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

        // Informations sur le fichier
        System.out.println("Nom: " + file.getName());
        System.out.println("Chemin absolu: " + file.getAbsolutePath());
        System.out.println("Taille: " + file.length() + " octets");
        System.out.println("Est un répertoire: " + file.isDirectory());
        System.out.println("Est un fichier: " + file.isFile());

        // Manipulation de répertoires
        File dir = new File("myDirectory");
        if (!dir.exists()) {
            boolean created = dir.mkdir(); // ou mkdirs() pour créer les répertoires parents
            System.out.println("Répertoire créé: " + created);
        }

        // Liste des fichiers dans un répertoire
        File currentDir = new File(".");
        System.out.println("Contenu du répertoire courant:");
        File[] files = currentDir.listFiles();
        if (files != null) {
            for (File f : files) {
                System.out.println(f.getName() + (f.isDirectory() ? " (dir)" : ""));
            }
        }

        // Suppression de fichier
        // file.delete();
    }
}
```

## API NIO (New I/O) - Java 7+

```java
import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.nio.file.*;
import java.util.List;

public class NioExample {
    public static void main(String[] args) {
        // Chemins
        Path path = Paths.get("nio_example.txt");
        System.out.println("Chemin: " + path);
        System.out.println("Chemin absolu: " + path.toAbsolutePath());
        System.out.println("Nom du fichier: " + path.getFileName());

        // Écriture de fichier
        try {
            Files.write(path,
                        "Contenu écrit avec NIO\nDeuxième ligne".getBytes(),
                        StandardOpenOption.CREATE);
            System.out.println("Fichier écrit avec succès");
        } catch (IOException e) {
            e.printStackTrace();
        }

        // Lecture de fichier
        try {
            byte[] bytes = Files.readAllBytes(path);
            String content = new String(bytes, StandardCharsets.UTF_8);
            System.out.println("Contenu lu:\n" + content);

            // Lecture ligne par ligne
            List<String> lines = Files.readAllLines(path, StandardCharsets.UTF_8);
            System.out.println("Nombre de lignes: " + lines.size());
        } catch (IOException e) {
            e.printStackTrace();
        }

        // Copie de fichier
        Path destination = Paths.get("nio_copy.txt");
        try {
            Files.copy(path, destination, StandardCopyOption.REPLACE_EXISTING);
            System.out.println("Fichier copié avec succès");
        } catch (IOException e) {
            e.printStackTrace();
        }

        // Déplacement/renommage de fichier
        Path newPath = Paths.get("nio_renamed.txt");
        try {
            Files.move(destination, newPath, StandardCopyOption.REPLACE_EXISTING);
            System.out.println("Fichier déplacé avec succès");
        } catch (IOException e) {
            e.printStackTrace();
        }

        // Vérification d'existence
        System.out.println("Le fichier existe: " + Files.exists(path));

        // Création de répertoires
        Path dirPath = Paths.get("nio_directory/subdirectory");
        try {
            Files.createDirectories(dirPath);
            System.out.println("Répertoires créés avec succès");
        } catch (IOException e) {
            e.printStackTrace();
        }

        // Parcourir un répertoire
        try {
            Path dir = Paths.get(".");
            System.out.println("Contenu du répertoire courant:");
            try (DirectoryStream<Path> stream = Files.newDirectoryStream(dir)) {
                for (Path entry : stream) {
                    System.out.println(entry.getFileName());
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## Parcours de répertoire récursif avec NIO

```java
import java.io.IOException;
import java.nio.file.*;
import java.nio.file.attribute.BasicFileAttributes;

public class FileWalkExample {
    public static void main(String[] args) {
        Path startDir = Paths.get(".");

        // SimpleFileVisitor pour parcourir l'arborescence
        try {
            Files.walkFileTree(startDir, new SimpleFileVisitor<Path>() {
                @Override
                public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) {
                    System.out.println("Fichier: " + file);
                    return FileVisitResult.CONTINUE;
                }

                @Override
                public FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes attrs) {
                    System.out.println("Répertoire: " + dir);
                    return FileVisitResult.CONTINUE;
                }

                @Override
                public FileVisitResult visitFileFailed(Path file, IOException exc) {
                    System.err.println("Erreur lors de la visite du fichier: " + file);
                    return FileVisitResult.CONTINUE;
                }
            });
        } catch (IOException e) {
            e.printStackTrace();
        }

        // Recherche de fichiers avec un motif glob
        try {
            System.out.println("\nRecherche de fichiers .txt:");
            PathMatcher matcher = FileSystems.getDefault().getPathMatcher("glob:*.txt");

            Files.walk(startDir)
                 .filter(path -> matcher.matches(path.getFileName()))
                 .forEach(System.out::println);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## Surveillance de répertoire avec WatchService

```java
import java.io.IOException;
import java.nio.file.*;

public class WatchServiceExample {
    public static void main(String[] args) {
        Path dir = Paths.get(".");

        try {
            WatchService watchService = FileSystems.getDefault().newWatchService();

            // Enregistrer les événements à surveiller
            dir.register(watchService,
                         StandardWatchEventKinds.ENTRY_CREATE,
                         StandardWatchEventKinds.ENTRY_DELETE,
                         StandardWatchEventKinds.ENTRY_MODIFY);

            System.out.println("Surveillance du répertoire: " + dir);
            System.out.println("Créez, modifiez ou supprimez des fichiers pour voir les événements...");

            while (true) {
                WatchKey key;
                try {
                    // Attendre qu'un événement se produise
                    key = watchService.take();
                } catch (InterruptedException e) {
                    return;
                }

                // Traiter les événements
                for (WatchEvent<?> event : key.pollEvents()) {
                    WatchEvent.Kind<?> kind = event.kind();

                    if (kind == StandardWatchEventKinds.OVERFLOW) {
                        continue;
                    }

                    @SuppressWarnings("unchecked")
                    WatchEvent<Path> pathEvent = (WatchEvent<Path>) event;
                    Path filename = pathEvent.context();

                    System.out.println(kind.name() + ": " + filename);
                }

                // Réinitialiser la clé pour recevoir d'autres événements
                boolean valid = key.reset();
                if (!valid) {
                    break;
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## Canaux et tampons NIO

```java
import java.io.IOException;
import java.io.RandomAccessFile;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;
import java.nio.charset.StandardCharsets;

public class ChannelExample {
    public static void main(String[] args) {
        try {
            // Écriture avec un canal
            RandomAccessFile file = new RandomAccessFile("channel_example.txt", "rw");
            FileChannel channel = file.getChannel();

            String text = "Texte écrit avec un canal NIO";
            ByteBuffer buffer = ByteBuffer.wrap(text.getBytes(StandardCharsets.UTF_8));

            channel.write(buffer);
            channel.close();
            file.close();

            System.out.println("Écriture terminée");

            // Lecture avec un canal
            RandomAccessFile readFile = new RandomAccessFile("channel_example.txt", "r");
            FileChannel readChannel = readFile.getChannel();

            // Créer un tampon pour la lecture
            ByteBuffer readBuffer = ByteBuffer.allocate(1024);
            int bytesRead = readChannel.read(readBuffer);

            // Préparer le tampon pour la lecture
            readBuffer.flip();

            // Lire les données du tampon
            byte[] bytes = new byte[bytesRead];
            readBuffer.get(bytes);
            String content = new String(bytes, StandardCharsets.UTF_8);

            System.out.println("Contenu lu: " + content);

            readChannel.close();
            readFile.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## Entrées/Sorties asynchrones (NIO.2)

```java
import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.AsynchronousFileChannel;
import java.nio.channels.CompletionHandler;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.StandardOpenOption;
import java.util.concurrent.Future;

public class AsyncIOExample {
    public static void main(String[] args) {
        Path path = Paths.get("async_example.txt");

        // Écriture asynchrone
        try {
            AsynchronousFileChannel fileChannel = AsynchronousFileChannel.open(
                    path,
                    StandardOpenOption.WRITE,
                    StandardOpenOption.CREATE);

            ByteBuffer buffer = ByteBuffer.wrap("Écriture asynchrone".getBytes());

            // Méthode 1: Utilisation de Future
            Future<Integer> operation = fileChannel.write(buffer, 0);
            while (!operation.isDone()) {
                System.out.println("En attente de l'opération d'écriture...");
                Thread.sleep(100);
            }

            System.out.println("Écriture terminée: " + operation.get() + " octets écrits");
            fileChannel.close();

            // Méthode 2: Utilisation de CompletionHandler
            AsynchronousFileChannel readChannel = AsynchronousFileChannel.open(
                    path, StandardOpenOption.READ);

            ByteBuffer readBuffer = ByteBuffer.allocate(1024);

            readChannel.read(readBuffer, 0, readBuffer, new CompletionHandler<Integer, ByteBuffer>() {
                @Override
                public void completed(Integer result, ByteBuffer attachment) {
                    System.out.println("Lecture terminée: " + result + " octets lus");
                    attachment.flip();
                    byte[] data = new byte[attachment.limit()];
                    attachment.get(data);
                    System.out.println("Contenu: " + new String(data));
                    try {
                        readChannel.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }

                @Override
                public void failed(Throwable exc, ByteBuffer attachment) {
                    System.err.println("Erreur de lecture: " + exc.getMessage());
                    try {
                        readChannel.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            });

            // Attendre un moment pour que l'opération asynchrone se termine
            Thread.sleep(1000);

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
