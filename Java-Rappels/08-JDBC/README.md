# JDBC - Java Database Connectivity

JDBC est l'API standard de Java pour se connecter aux bases de données relationnelles. Elle permet d'exécuter des requêtes SQL et de manipuler les données dans une base de données.

## Configuration et connexion

### 1. Ajouter le driver JDBC

Chaque base de données nécessite son propre driver JDBC. Voici comment ajouter un driver dans un projet Maven :

```xml
<!-- MySQL -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.33</version>
</dependency>

<!-- PostgreSQL -->
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.6.0</version>
</dependency>

<!-- H2 Database (pour les tests) -->
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>2.2.220</version>
</dependency>
```

### 2. Établir une connexion

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class JdbcConnectionExample {
    public static void main(String[] args) {
        // URL de connexion, nom d'utilisateur et mot de passe
        String url = "jdbc:mysql://localhost:3306/ma_base";
        String username = "root";
        String password = "password";

        // Établir une connexion
        try (Connection connection = DriverManager.getConnection(url, username, password)) {
            System.out.println("Connexion établie avec succès!");

            // Vérifier si la connexion est fermée
            if (!connection.isClosed()) {
                System.out.println("La connexion est ouverte");
            }

            // La connexion est automatiquement fermée grâce au try-with-resources
        } catch (SQLException e) {
            System.err.println("Erreur de connexion: " + e.getMessage());
            e.printStackTrace();
        }
    }
}
```

### URLs de connexion pour différentes bases de données

```
MySQL      : jdbc:mysql://localhost:3306/database_name
PostgreSQL : jdbc:postgresql://localhost:5432/database_name
Oracle     : jdbc:oracle:thin:@localhost:1521:database_name
SQL Server : jdbc:sqlserver://localhost:1433;databaseName=database_name
H2 (memory): jdbc:h2:mem:database_name
H2 (file)  : jdbc:h2:file:./data/database_name
SQLite     : jdbc:sqlite:database_file.db
```

## Exécution de requêtes SQL

### 1. Requêtes SELECT (lecture)

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class JdbcSelectExample {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/ma_base";
        String username = "root";
        String password = "password";

        try (
            Connection connection = DriverManager.getConnection(url, username, password);
            Statement statement = connection.createStatement();
            ResultSet resultSet = statement.executeQuery("SELECT id, nom, email FROM utilisateurs")
        ) {
            // Parcourir les résultats
            while (resultSet.next()) {
                int id = resultSet.getInt("id");
                String nom = resultSet.getString("nom");
                String email = resultSet.getString("email");

                System.out.println("Utilisateur #" + id + ": " + nom + " (" + email + ")");
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

### 2. Requêtes INSERT, UPDATE, DELETE (écriture)

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.sql.Statement;

public class JdbcUpdateExample {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/ma_base";
        String username = "root";
        String password = "password";

        try (
            Connection connection = DriverManager.getConnection(url, username, password);
            Statement statement = connection.createStatement()
        ) {
            // INSERT
            int rowsInserted = statement.executeUpdate(
                "INSERT INTO utilisateurs (nom, email) VALUES ('Jean Dupont', 'jean@exemple.com')"
            );
            System.out.println(rowsInserted + " utilisateur(s) inséré(s)");

            // UPDATE
            int rowsUpdated = statement.executeUpdate(
                "UPDATE utilisateurs SET email = 'jean.dupont@exemple.com' WHERE nom = 'Jean Dupont'"
            );
            System.out.println(rowsUpdated + " utilisateur(s) mis à jour");

            // DELETE
            int rowsDeleted = statement.executeUpdate(
                "DELETE FROM utilisateurs WHERE nom = 'Jean Dupont'"
            );
            System.out.println(rowsDeleted + " utilisateur(s) supprimé(s)");

        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

### 3. Requêtes préparées (sécurité contre les injections SQL)

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class JdbcPreparedStatementExample {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/ma_base";
        String username = "root";
        String password = "password";

        // Paramètres de recherche
        String searchName = "Dupont";

        try (Connection connection = DriverManager.getConnection(url, username, password)) {
            // Requête préparée avec un paramètre
            String sql = "SELECT id, nom, email FROM utilisateurs WHERE nom LIKE ?";

            try (PreparedStatement preparedStatement = connection.prepareStatement(sql)) {
                // Définir les paramètres (indexés à partir de 1)
                preparedStatement.setString(1, "%" + searchName + "%");

                // Exécuter la requête
                try (ResultSet resultSet = preparedStatement.executeQuery()) {
                    while (resultSet.next()) {
                        int id = resultSet.getInt("id");
                        String nom = resultSet.getString("nom");
                        String email = resultSet.getString("email");

                        System.out.println("Utilisateur #" + id + ": " + nom + " (" + email + ")");
                    }
                }
            }

            // Exemple d'insertion avec requête préparée
            String insertSql = "INSERT INTO utilisateurs (nom, email, age, actif) VALUES (?, ?, ?, ?)";

            try (PreparedStatement preparedStatement = connection.prepareStatement(insertSql)) {
                preparedStatement.setString(1, "Marie Martin");
                preparedStatement.setString(2, "marie@exemple.com");
                preparedStatement.setInt(3, 28);
                preparedStatement.setBoolean(4, true);

                int rowsInserted = preparedStatement.executeUpdate();
                System.out.println(rowsInserted + " utilisateur(s) inséré(s)");
            }

        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

## Transactions

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;

public class JdbcTransactionExample {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/ma_base";
        String username = "root";
        String password = "password";

        Connection connection = null;

        try {
            connection = DriverManager.getConnection(url, username, password);

            // Désactiver l'auto-commit pour gérer manuellement la transaction
            connection.setAutoCommit(false);

            // Première opération: retrait d'un compte
            String updateAccount1 = "UPDATE comptes SET solde = solde - ? WHERE id = ?";
            try (PreparedStatement pstmt = connection.prepareStatement(updateAccount1)) {
                pstmt.setDouble(1, 1000.0);
                pstmt.setInt(2, 1);
                pstmt.executeUpdate();
            }

            // Deuxième opération: dépôt sur un autre compte
            String updateAccount2 = "UPDATE comptes SET solde = solde + ? WHERE id = ?";
            try (PreparedStatement pstmt = connection.prepareStatement(updateAccount2)) {
                pstmt.setDouble(1, 1000.0);
                pstmt.setInt(2, 2);
                pstmt.executeUpdate();
            }

            // Si tout s'est bien passé, valider la transaction
            connection.commit();
            System.out.println("Transaction validée avec succès!");

        } catch (SQLException e) {
            // En cas d'erreur, annuler la transaction
            if (connection != null) {
                try {
                    connection.rollback();
                    System.out.println("Transaction annulée!");
                } catch (SQLException ex) {
                    ex.printStackTrace();
                }
            }
            e.printStackTrace();
        } finally {
            // Rétablir l'auto-commit et fermer la connexion
            if (connection != null) {
                try {
                    connection.setAutoCommit(true);
                    connection.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

## Métadonnées

```java
import java.sql.Connection;
import java.sql.DatabaseMetaData;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.ResultSetMetaData;
import java.sql.SQLException;
import java.sql.Statement;

public class JdbcMetaDataExample {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/ma_base";
        String username = "root";
        String password = "password";

        try (Connection connection = DriverManager.getConnection(url, username, password)) {
            // Métadonnées de la base de données
            DatabaseMetaData dbMetaData = connection.getMetaData();

            System.out.println("Informations sur la base de données:");
            System.out.println("Nom du produit: " + dbMetaData.getDatabaseProductName());
            System.out.println("Version: " + dbMetaData.getDatabaseProductVersion());
            System.out.println("Driver: " + dbMetaData.getDriverName() + " " + dbMetaData.getDriverVersion());

            // Liste des tables
            System.out.println("\nListe des tables:");
            try (ResultSet tables = dbMetaData.getTables(null, null, "%", new String[]{"TABLE"})) {
                while (tables.next()) {
                    System.out.println(tables.getString("TABLE_NAME"));
                }
            }

            // Métadonnées d'une requête
            String sql = "SELECT id, nom, email, age FROM utilisateurs";
            try (
                Statement statement = connection.createStatement();
                ResultSet resultSet = statement.executeQuery(sql)
            ) {
                ResultSetMetaData rsMetaData = resultSet.getMetaData();

                System.out.println("\nInformations sur les colonnes:");
                for (int i = 1; i <= rsMetaData.getColumnCount(); i++) {
                    System.out.println("Colonne " + i + ":");
                    System.out.println("  Nom: " + rsMetaData.getColumnName(i));
                    System.out.println("  Type: " + rsMetaData.getColumnTypeName(i));
                    System.out.println("  Nullable: " + (rsMetaData.isNullable(i) == ResultSetMetaData.columnNullable));
                }
            }

        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

## Pool de connexions

L'utilisation d'un pool de connexions est recommandée pour les applications réelles. Voici un exemple avec HikariCP, une bibliothèque populaire pour la gestion de pool de connexions :

### Ajouter la dépendance HikariCP

```xml
<dependency>
    <groupId>com.zaxxer</groupId>
    <artifactId>HikariCP</artifactId>
    <version>5.0.1</version>
</dependency>
```

### Utilisation du pool de connexions

```java
import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class JdbcConnectionPoolExample {

    private static HikariDataSource dataSource;

    static {
        // Configuration du pool de connexions
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:mysql://localhost:3306/ma_base");
        config.setUsername("root");
        config.setPassword("password");
        config.setMaximumPoolSize(10);
        config.setMinimumIdle(5);
        config.setIdleTimeout(30000);
        config.setConnectionTimeout(30000);

        // Création du pool
        dataSource = new HikariDataSource(config);
    }

    public static Connection getConnection() throws SQLException {
        return dataSource.getConnection();
    }

    public static void closeDataSource() {
        if (dataSource != null) {
            dataSource.close();
        }
    }

    public static void main(String[] args) {
        try (
            Connection connection = getConnection();
            PreparedStatement statement = connection.prepareStatement("SELECT * FROM utilisateurs");
            ResultSet resultSet = statement.executeQuery()
        ) {
            while (resultSet.next()) {
                System.out.println(resultSet.getString("nom"));
            }
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            // Fermer le pool à la fin de l'application
            closeDataSource();
        }
    }
}
```

## Batch Processing (traitement par lots)

Le traitement par lots permet d'exécuter plusieurs requêtes SQL en une seule opération, ce qui améliore considérablement les performances pour les opérations en masse.

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;

public class JdbcBatchExample {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/ma_base";
        String username = "root";
        String password = "password";

        try (Connection connection = DriverManager.getConnection(url, username, password)) {
            // Désactiver l'auto-commit
            connection.setAutoCommit(false);

            String sql = "INSERT INTO utilisateurs (nom, email, age) VALUES (?, ?, ?)";

            try (PreparedStatement pstmt = connection.prepareStatement(sql)) {
                // Ajouter plusieurs opérations au batch
                for (int i = 0; i < 1000; i++) {
                    pstmt.setString(1, "Utilisateur " + i);
                    pstmt.setString(2, "user" + i + "@exemple.com");
                    pstmt.setInt(3, 20 + (i % 50));

                    // Ajouter au batch
                    pstmt.addBatch();

                    // Exécuter le batch tous les 100 enregistrements pour éviter de surcharger la mémoire
                    if (i % 100 == 0) {
                        int[] results = pstmt.executeBatch();
                        System.out.println("Lot exécuté: " + results.length + " opérations");
                    }
                }

                // Exécuter le batch final
                int[] results = pstmt.executeBatch();
                System.out.println("Lot final exécuté: " + results.length + " opérations");

                // Valider la transaction
                connection.commit();

            } catch (SQLException e) {
                // Annuler la transaction en cas d'erreur
                connection.rollback();
                throw e;
            } finally {
                // Rétablir l'auto-commit
                connection.setAutoCommit(true);
            }

        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

## Utilisation de DAO (Data Access Object)

Le pattern DAO permet de séparer la logique d'accès aux données du reste de l'application.

```java
// Classe modèle
public class Utilisateur {
    private int id;
    private String nom;
    private String email;
    private int age;

    // Constructeurs, getters et setters
    public Utilisateur() {}

    public Utilisateur(int id, String nom, String email, int age) {
        this.id = id;
        this.nom = nom;
        this.email = email;
        this.age = age;
    }

    // Getters et setters
    public int getId() { return id; }
    public void setId(int id) { this.id = id; }

    public String getNom() { return nom; }
    public void setNom(String nom) { this.nom = nom; }

    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }

    public int getAge() { return age; }
    public void setAge(int age) { this.age = age; }

    @Override
    public String toString() {
        return "Utilisateur{id=" + id + ", nom='" + nom + "', email='" + email + "', age=" + age + '}';
    }
}

// Interface DAO
public interface UtilisateurDAO {
    Utilisateur findById(int id) throws SQLException;
    List<Utilisateur> findAll() throws SQLException;
    void insert(Utilisateur utilisateur) throws SQLException;
    void update(Utilisateur utilisateur) throws SQLException;
    void delete(int id) throws SQLException;
}

// Implémentation DAO avec JDBC
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.ArrayList;
import java.util.List;

public class UtilisateurDAOImpl implements UtilisateurDAO {

    private Connection connection;

    public UtilisateurDAOImpl(Connection connection) {
        this.connection = connection;
    }

    @Override
    public Utilisateur findById(int id) throws SQLException {
        String sql = "SELECT id, nom, email, age FROM utilisateurs WHERE id = ?";

        try (PreparedStatement pstmt = connection.prepareStatement(sql)) {
            pstmt.setInt(1, id);

            try (ResultSet rs = pstmt.executeQuery()) {
                if (rs.next()) {
                    return extractUtilisateurFromResultSet(rs);
                }
            }
        }

        return null; // Utilisateur non trouvé
    }

    @Override
    public List<Utilisateur> findAll() throws SQLException {
        List<Utilisateur> utilisateurs = new ArrayList<>();
        String sql = "SELECT id, nom, email, age FROM utilisateurs";

        try (
            Statement stmt = connection.createStatement();
            ResultSet rs = stmt.executeQuery(sql)
        ) {
            while (rs.next()) {
                utilisateurs.add(extractUtilisateurFromResultSet(rs));
            }
        }

        return utilisateurs;
    }

    @Override
    public void insert(Utilisateur utilisateur) throws SQLException {
        String sql = "INSERT INTO utilisateurs (nom, email, age) VALUES (?, ?, ?)";

        try (PreparedStatement pstmt = connection.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS)) {
            pstmt.setString(1, utilisateur.getNom());
            pstmt.setString(2, utilisateur.getEmail());
            pstmt.setInt(3, utilisateur.getAge());

            int affectedRows = pstmt.executeUpdate();

            if (affectedRows == 0) {
                throw new SQLException("La création de l'utilisateur a échoué, aucune ligne affectée.");
            }

            try (ResultSet generatedKeys = pstmt.getGeneratedKeys()) {
                if (generatedKeys.next()) {
                    utilisateur.setId(generatedKeys.getInt(1));
                } else {
                    throw new SQLException("La création de l'utilisateur a échoué, aucun ID obtenu.");
                }
            }
        }
    }

    @Override
    public void update(Utilisateur utilisateur) throws SQLException {
        String sql = "UPDATE utilisateurs SET nom = ?, email = ?, age = ? WHERE id = ?";

        try (PreparedStatement pstmt = connection.prepareStatement(sql)) {
            pstmt.setString(1, utilisateur.getNom());
            pstmt.setString(2, utilisateur.getEmail());
            pstmt.setInt(3, utilisateur.getAge());
            pstmt.setInt(4, utilisateur.getId());

            pstmt.executeUpdate();
        }
    }

    @Override
    public void delete(int id) throws SQLException {
        String sql = "DELETE FROM utilisateurs WHERE id = ?";

        try (PreparedStatement pstmt = connection.prepareStatement(sql)) {
            pstmt.setInt(1, id);
            pstmt.executeUpdate();
        }
    }

    private Utilisateur extractUtilisateurFromResultSet(ResultSet rs) throws SQLException {
        Utilisateur utilisateur = new Utilisateur();
        utilisateur.setId(rs.getInt("id"));
        utilisateur.setNom(rs.getString("nom"));
        utilisateur.setEmail(rs.getString("email"));
        utilisateur.setAge(rs.getInt("age"));
        return utilisateur;
    }
}

// Utilisation du DAO
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.util.List;

public class DAOExample {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/ma_base";
        String username = "root";
        String password = "password";

        try (Connection connection = DriverManager.getConnection(url, username, password)) {
            // Créer une instance du DAO
            UtilisateurDAO utilisateurDAO = new UtilisateurDAOImpl(connection);

            // Créer un nouvel utilisateur
            Utilisateur nouvelUtilisateur = new Utilisateur();
            nouvelUtilisateur.setNom("Pierre Durand");
            nouvelUtilisateur.setEmail("pierre@exemple.com");
            nouvelUtilisateur.setAge(35);

            // Insérer l'utilisateur
            utilisateurDAO.insert(nouvelUtilisateur);
            System.out.println("Utilisateur créé avec ID: " + nouvelUtilisateur.getId());

            // Récupérer l'utilisateur par ID
            Utilisateur utilisateur = utilisateurDAO.findById(nouvelUtilisateur.getId());
            System.out.println("Utilisateur récupéré: " + utilisateur);

            // Mettre à jour l'utilisateur
            utilisateur.setEmail("pierre.durand@exemple.com");
            utilisateurDAO.update(utilisateur);
            System.out.println("Utilisateur mis à jour");

            // Récupérer tous les utilisateurs
            List<Utilisateur> utilisateurs = utilisateurDAO.findAll();
            System.out.println("Liste des utilisateurs:");
            for (Utilisateur u : utilisateurs) {
                System.out.println(u);
            }

            // Supprimer l'utilisateur
            utilisateurDAO.delete(nouvelUtilisateur.getId());
            System.out.println("Utilisateur supprimé");

        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

## Bonnes pratiques JDBC

1. **Toujours fermer les ressources** : Utilisez try-with-resources pour garantir que les connexions, statements et resultsets sont fermés.

2. **Utiliser des requêtes préparées** : Protégez-vous contre les injections SQL en utilisant des PreparedStatements.

3. **Gérer les transactions** : Utilisez des transactions pour garantir l'intégrité des données lors de plusieurs opérations liées.

4. **Utiliser un pool de connexions** : Améliorez les performances en réutilisant les connexions à la base de données.

5. **Séparer la logique d'accès aux données** : Utilisez le pattern DAO pour isoler le code d'accès à la base de données.

6. **Gérer correctement les exceptions** : Capturez et traitez les SQLException de manière appropriée.

7. **Utiliser le traitement par lots** : Pour les opérations en masse, utilisez le batch processing.

8. **Éviter de récupérer plus de données que nécessaire** : Sélectionnez uniquement les colonnes dont vous avez besoin.

9. **Indexer correctement les tables** : Assurez-vous que vos requêtes utilisent des index pour de meilleures performances.

10. **Tester avec une base de données en mémoire** : Utilisez H2 ou HSQLDB pour les tests unitaires.
