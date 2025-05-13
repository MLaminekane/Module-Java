# Gestion des Exceptions en Java

## Hiérarchie des exceptions

En Java, toutes les exceptions héritent de la classe `Throwable`:

- **Throwable**: Classe de base pour toutes les exceptions
  - **Error**: Erreurs graves du système (OutOfMemoryError, StackOverflowError)
  - **Exception**: Problèmes récupérables
    - **RuntimeException**: Exceptions non vérifiées (NullPointerException, ArrayIndexOutOfBoundsException)
    - **Autres exceptions**: Exceptions vérifiées (IOException, SQLException)

## Try-Catch-Finally

```java
import java.io.FileReader;
import java.io.IOException;

public class ExceptionDemo {
    public static void main(String[] args) {
        FileReader reader = null;
        try {
            reader = new FileReader("file.txt");
            // Code qui peut générer une exception
            int character = reader.read();
        } catch (IOException e) {
            // Gestion de l'exception
            System.err.println("Erreur de lecture: " + e.getMessage());
            e.printStackTrace();
        } finally {
            // Bloc exécuté dans tous les cas
            try {
                if (reader != null) {
                    reader.close();
                }
            } catch (IOException e) {
                System.err.println("Erreur lors de la fermeture: " + e.getMessage());
            }
        }
    }
}
```

## Try-with-resources (Java 7+)

```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;

public class TryWithResourcesDemo {
    public static void main(String[] args) {
        // Les ressources sont automatiquement fermées
        try (FileReader fr = new FileReader("file.txt");
             BufferedReader reader = new BufferedReader(fr)) {
            String line = reader.readLine();
            System.out.println(line);
        } catch (IOException e) {
            System.err.println("Erreur: " + e.getMessage());
        }
        // Pas besoin de finally pour fermer les ressources
    }
}
```

## Multi-catch (Java 7+)

```java
import java.io.FileNotFoundException;
import java.io.IOException;
import java.sql.SQLException;

public class MultiCatchDemo {
    public static void main(String[] args) {
        try {
            // Code pouvant générer plusieurs types d'exceptions
            if (Math.random() > 0.5) {
                throw new IOException("IO Error");
            } else {
                throw new SQLException("SQL Error");
            }
        } catch (IOException | SQLException e) {
            // Gestion commune pour plusieurs types d'exceptions
            System.err.println("Error: " + e.getMessage());
        }
    }
}
```

## Exceptions vérifiées vs non vérifiées

```java
import java.io.FileNotFoundException;
import java.io.FileReader;

public class CheckedVsUnchecked {
    // Méthode avec exception vérifiée (doit être déclarée ou gérée)
    public static void readFile(String filename) throws FileNotFoundException {
        FileReader reader = new FileReader(filename);
        // ...
    }

    // Méthode avec exception non vérifiée (RuntimeException)
    public static int divide(int a, int b) {
        return a / b; // Peut lancer ArithmeticException si b == 0
    }

    public static void main(String[] args) {
        // Pour les exceptions vérifiées, on doit soit:
        // 1. Utiliser try-catch
        try {
            readFile("nonexistent.txt");
        } catch (FileNotFoundException e) {
            System.err.println("Le fichier n'existe pas");
        }

        // 2. Ou propager l'exception
        // main() throws FileNotFoundException

        // Pour les exceptions non vérifiées, le try-catch est optionnel
        try {
            int result = divide(10, 0);
        } catch (ArithmeticException e) {
            System.err.println("Division par zéro");
        }

        // Sans try-catch, l'exception se propage et peut terminer le programme
        // int result = divide(10, 0);
    }
}
```

## Lancer des exceptions

```java
public class ThrowDemo {
    public static void validateAge(int age) {
        if (age < 0) {
            throw new IllegalArgumentException("L'âge ne peut pas être négatif");
        }
        if (age < 18) {
            throw new RuntimeException("Doit être majeur");
        }
    }

    public static void main(String[] args) {
        try {
            validateAge(-5);
        } catch (IllegalArgumentException e) {
            System.err.println(e.getMessage());
        }

        try {
            validateAge(15);
        } catch (RuntimeException e) {
            System.err.println(e.getMessage());
        }
    }
}
```

## Exceptions personnalisées

```java
// Exception vérifiée personnalisée
public class InsufficientFundsException extends Exception {
    private double amount;

    public InsufficientFundsException(double amount) {
        super("Fonds insuffisants: manque " + amount + "€");
        this.amount = amount;
    }

    public double getAmount() {
        return amount;
    }
}

// Exception non vérifiée personnalisée
public class InvalidAccountException extends RuntimeException {
    private String accountId;

    public InvalidAccountException(String accountId) {
        super("Compte invalide: " + accountId);
        this.accountId = accountId;
    }

    public String getAccountId() {
        return accountId;
    }
}

// Utilisation
public class BankAccount {
    private String accountId;
    private double balance;

    public BankAccount(String accountId, double initialBalance) {
        if (accountId == null || accountId.trim().isEmpty()) {
            throw new InvalidAccountException("ID vide");
        }
        this.accountId = accountId;
        this.balance = initialBalance;
    }

    public void withdraw(double amount) throws InsufficientFundsException {
        if (amount > balance) {
            throw new InsufficientFundsException(amount - balance);
        }
        balance -= amount;
    }

    public static void main(String[] args) {
        try {
            BankAccount account = new BankAccount("123", 1000);
            account.withdraw(1500); // Génère InsufficientFundsException
        } catch (InsufficientFundsException e) {
            System.err.println(e.getMessage());
            System.err.println("Montant manquant: " + e.getAmount() + "€");
        }

        try {
            BankAccount invalidAccount = new BankAccount("", 100); // Génère InvalidAccountException
        } catch (InvalidAccountException e) {
            System.err.println(e.getMessage());
        }
    }
}
```

## Chaînage d'exceptions

```java
import java.sql.SQLException;

public class ExceptionChainingDemo {
    public static void updateRecord() throws ServiceException {
        try {
            // Simulation d'une exception SQL
            throw new SQLException("Erreur de base de données");
        } catch (SQLException e) {
            // Encapsulation de l'exception dans une exception de niveau supérieur
            throw new ServiceException("Échec de la mise à jour", e);
        }
    }

    public static void main(String[] args) {
        try {
            updateRecord();
        } catch (ServiceException e) {
            System.err.println("Service error: " + e.getMessage());

            // Accès à la cause originale
            Throwable cause = e.getCause();
            if (cause != null) {
                System.err.println("Caused by: " + cause.getMessage());
            }

            // Affichage de la pile d'appels complète
            e.printStackTrace();
        }
    }
}

// Exception personnalisée pour la couche service
class ServiceException extends Exception {
    public ServiceException(String message) {
        super(message);
    }

    public ServiceException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

## Bonnes pratiques

```java
public class ExceptionBestPractices {
    // 1. Utiliser des exceptions spécifiques
    public void method1() throws SpecificException {
        // ...
    }

    // 2. Ne pas attraper Exception ou Throwable (trop générique)
    public void badPractice() {
        try {
            // ...
        } catch (Exception e) { // Éviter
            // Trop générique
        }
    }

    // 3. Ne pas ignorer les exceptions
    public void doNotIgnoreExceptions() {
        try {
            // ...
        } catch (IOException e) {
            // Au minimum, journaliser l'exception
            logger.error("Error occurred", e);
            // Ou la propager
            throw new RuntimeException(e);
        }
    }

    // 4. Fermer les ressources correctement
    public void closeResources() {
        // Préférer try-with-resources
        try (InputStream is = new FileInputStream("file.txt")) {
            // ...
        } catch (IOException e) {
            // Gestion de l'exception
        }
    }

    // 5. Documenter les exceptions
    /**
     * Description de la méthode.
     *
     * @param param Description du paramètre
     * @return Description de la valeur de retour
     * @throws IOException Si une erreur d'E/S se produit
     */
    public String documentedMethod(String param) throws IOException {
        // ...
        return result;
    }
}
```

## Nouveautés Java 9+

```java
// Java 9: Amélioration de try-with-resources
public class Java9TryWithResources {
    public void oldStyle() throws IOException {
        InputStream inputStream = new FileInputStream("file.txt");
        try (InputStream is = inputStream) { // Java 7-8
            // Utilisation de is
        }
    }

    public void newStyle() throws IOException {
        InputStream inputStream = new FileInputStream("file.txt");
        try (inputStream) { // Java 9+
            // Utilisation directe de inputStream
        }
    }
}
```
