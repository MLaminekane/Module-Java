# JPA - Java Persistence API

JPA est une spécification Java qui définit une interface de programmation pour la gestion de la persistance et du mapping objet-relationnel. Elle simplifie considérablement le développement d'applications Java qui interagissent avec des bases de données par rapport à JDBC.

## Configuration et mise en place

### 1. Ajouter les dépendances JPA

Dans un projet Maven, ajoutez les dépendances suivantes :

```xml
<!-- API JPA -->
<dependency>
    <groupId>jakarta.persistence</groupId>
    <artifactId>jakarta.persistence-api</artifactId>
    <version>3.1.0</version>
</dependency>

<!-- Hibernate (implémentation de JPA) -->
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>6.2.7.Final</version>
</dependency>

<!-- Driver de base de données (exemple avec MySQL) -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.33</version>
</dependency>
```

### 2. Configuration de JPA

#### a. Fichier persistence.xml

JPA nécessite un fichier de configuration `persistence.xml` qui définit les unités de persistance. Ce fichier doit être placé dans le dossier `META-INF` du classpath.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="https://jakarta.ee/xml/ns/persistence"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="https://jakarta.ee/xml/ns/persistence https://jakarta.ee/xml/ns/persistence/persistence_3_0.xsd"
             version="3.0">

    <persistence-unit name="monUniteDePersistance" transaction-type="RESOURCE_LOCAL">
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>

        <!-- Entités gérées -->
        <class>com.example.model.Utilisateur</class>
        <class>com.example.model.Adresse</class>

        <properties>
            <!-- Propriétés de connexion à la base de données -->
            <property name="jakarta.persistence.jdbc.driver" value="com.mysql.cj.jdbc.Driver"/>
            <property name="jakarta.persistence.jdbc.url" value="jdbc:mysql://localhost:3306/ma_base"/>
            <property name="jakarta.persistence.jdbc.user" value="root"/>
            <property name="jakarta.persistence.jdbc.password" value="password"/>

            <!-- Propriétés Hibernate -->
            <property name="hibernate.dialect" value="org.hibernate.dialect.MySQLDialect"/>
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>

            <!-- Options pour la création/mise à jour automatique du schéma -->
            <property name="hibernate.hbm2ddl.auto" value="update"/>
        </properties>
    </persistence-unit>
</persistence>
```

#### b. Configuration programmatique

Il est également possible de configurer JPA par programmation :

```java
import jakarta.persistence.EntityManagerFactory;
import jakarta.persistence.Persistence;
import java.util.HashMap;
import java.util.Map;

public class JpaConfig {
    public static EntityManagerFactory createEntityManagerFactory() {
        Map<String, String> properties = new HashMap<>();

        // Propriétés de connexion
        properties.put("jakarta.persistence.jdbc.driver", "com.mysql.cj.jdbc.Driver");
        properties.put("jakarta.persistence.jdbc.url", "jdbc:mysql://localhost:3306/ma_base");
        properties.put("jakarta.persistence.jdbc.user", "root");
        properties.put("jakarta.persistence.jdbc.password", "password");

        // Propriétés Hibernate
        properties.put("hibernate.dialect", "org.hibernate.dialect.MySQLDialect");
        properties.put("hibernate.show_sql", "true");
        properties.put("hibernate.hbm2ddl.auto", "update");

        return Persistence.createEntityManagerFactory("monUniteDePersistance", properties);
    }
}
```

## Création d'entités

### 1. Définition d'une entité simple

```java
import jakarta.persistence.*;
import java.util.Date;

@Entity
@Table(name = "utilisateurs")
public class Utilisateur {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "nom", length = 100, nullable = false)
    private String nom;

    @Column(name = "email", unique = true)
    private String email;

    @Column(name = "age")
    private Integer age;

    @Temporal(TemporalType.TIMESTAMP)
    @Column(name = "date_inscription")
    private Date dateInscription;

    @Enumerated(EnumType.STRING)
    @Column(name = "statut")
    private UserStatus statut;

    @Transient
    private String champTemporaire;

    // Enumération pour le statut de l'utilisateur
    public enum UserStatus {
        ACTIF, INACTIF, BLOQUE
    }

    // Constructeur par défaut (requis par JPA)
    public Utilisateur() {
    }

    // Constructeur avec paramètres
    public Utilisateur(String nom, String email, Integer age) {
        this.nom = nom;
        this.email = email;
        this.age = age;
        this.dateInscription = new Date();
        this.statut = UserStatus.ACTIF;
    }

    // Getters et setters
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getNom() {
        return nom;
    }

    public void setNom(String nom) {
        this.nom = nom;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Date getDateInscription() {
        return dateInscription;
    }

    public void setDateInscription(Date dateInscription) {
        this.dateInscription = dateInscription;
    }

    public UserStatus getStatut() {
        return statut;
    }

    public void setStatut(UserStatus statut) {
        this.statut = statut;
    }

    @Override
    public String toString() {
        return "Utilisateur{" +
                "id=" + id +
                ", nom='" + nom + '\'' +
                ", email='" + email + '\'' +
                ", age=" + age +
                ", statut=" + statut +
                '}';
    }
}
```

### 2. Relations entre entités

#### a. Relation One-to-One

```java
@Entity
@Table(name = "utilisateurs")
public class Utilisateur {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    // Autres attributs...

    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "adresse_id", referencedColumnName = "id")
    private Adresse adresse;

    // Getters et setters
}

@Entity
@Table(name = "adresses")
public class Adresse {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "rue")
    private String rue;

    @Column(name = "ville")
    private String ville;

    @Column(name = "code_postal")
    private String codePostal;

    @OneToOne(mappedBy = "adresse")
    private Utilisateur utilisateur;

    // Getters et setters
}
```

#### b. Relation One-to-Many / Many-to-One

```java
@Entity
@Table(name = "departements")
public class Departement {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "nom")
    private String nom;

    @OneToMany(mappedBy = "departement", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Employe> employes = new ArrayList<>();

    // Méthodes utilitaires pour gérer la relation bidirectionnelle
    public void addEmploye(Employe employe) {
        employes.add(employe);
        employe.setDepartement(this);
    }

    public void removeEmploye(Employe employe) {
        employes.remove(employe);
        employe.setDepartement(null);
    }

    // Getters et setters
}

@Entity
@Table(name = "employes")
public class Employe {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "nom")
    private String nom;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "departement_id")
    private Departement departement;

    // Getters et setters
}
```

#### c. Relation Many-to-Many

```java
@Entity
@Table(name = "etudiants")
public class Etudiant {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "nom")
    private String nom;

    @ManyToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
    @JoinTable(
        name = "inscriptions",
        joinColumns = @JoinColumn(name = "etudiant_id"),
        inverseJoinColumns = @JoinColumn(name = "cours_id")
    )
    private Set<Cours> cours = new HashSet<>();

    // Méthodes utilitaires
    public void addCours(Cours cours) {
        this.cours.add(cours);
        cours.getEtudiants().add(this);
    }

    public void removeCours(Cours cours) {
        this.cours.remove(cours);
        cours.getEtudiants().remove(this);
    }

    // Getters et setters
}

@Entity
@Table(name = "cours")
public class Cours {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "titre")
    private String titre;

    @ManyToMany(mappedBy = "cours")
    private Set<Etudiant> etudiants = new HashSet<>();

    // Getters et setters
}
```

## Opérations CRUD avec JPA

### 1. Création de l'EntityManager

```java
import jakarta.persistence.EntityManager;
import jakarta.persistence.EntityManagerFactory;
import jakarta.persistence.Persistence;

public class JpaUtil {
    private static final EntityManagerFactory emf;

    static {
        try {
            emf = Persistence.createEntityManagerFactory("monUniteDePersistance");
        } catch (Throwable ex) {
            System.err.println("Initial EntityManagerFactory creation failed: " + ex);
            throw new ExceptionInInitializerError(ex);
        }
    }

    public static EntityManager getEntityManager() {
        return emf.createEntityManager();
    }

    public static void close() {
        emf.close();
    }
}
```

### 2. Opérations CRUD de base

```java
import jakarta.persistence.EntityManager;
import jakarta.persistence.EntityTransaction;

public class UtilisateurService {

    // Créer un utilisateur
    public Utilisateur create(Utilisateur utilisateur) {
        EntityManager em = JpaUtil.getEntityManager();
        EntityTransaction tx = null;

        try {
            tx = em.getTransaction();
            tx.begin();

            em.persist(utilisateur);

            tx.commit();
            return utilisateur;
        } catch (Exception e) {
            if (tx != null && tx.isActive()) {
                tx.rollback();
            }
            e.printStackTrace();
            throw e;
        } finally {
            em.close();
        }
    }

    // Lire un utilisateur par ID
    public Utilisateur findById(Long id) {
        EntityManager em = JpaUtil.getEntityManager();

        try {
            return em.find(Utilisateur.class, id);
        } finally {
            em.close();
        }
    }

    // Mettre à jour un utilisateur
    public Utilisateur update(Utilisateur utilisateur) {
        EntityManager em = JpaUtil.getEntityManager();
        EntityTransaction tx = null;

        try {
            tx = em.getTransaction();
            tx.begin();

            Utilisateur utilisateurMaj = em.merge(utilisateur);

            tx.commit();
            return utilisateurMaj;
        } catch (Exception e) {
            if (tx != null && tx.isActive()) {
                tx.rollback();
            }
            e.printStackTrace();
            throw e;
        } finally {
            em.close();
        }
    }

    // Supprimer un utilisateur
    public void delete(Long id) {
        EntityManager em = JpaUtil.getEntityManager();
        EntityTransaction tx = null;

        try {
            tx = em.getTransaction();
            tx.begin();

            Utilisateur utilisateur = em.find(Utilisateur.class, id);
            if (utilisateur != null) {
                em.remove(utilisateur);
            }

            tx.commit();
        } catch (Exception e) {
            if (tx != null && tx.isActive()) {
                tx.rollback();
            }
            e.printStackTrace();
            throw e;
        } finally {
            em.close();
        }
    }
}
```

### 3. Exemple d'utilisation

```java
public class Main {
    public static void main(String[] args) {
        UtilisateurService service = new UtilisateurService();

        try {
            // Création d'un utilisateur
            Utilisateur utilisateur = new Utilisateur("Jean Dupont", "jean@exemple.com", 30);
            service.create(utilisateur);
            System.out.println("Utilisateur créé: " + utilisateur);

            // Récupération d'un utilisateur
            Utilisateur utilisateurRecup = service.findById(utilisateur.getId());
            System.out.println("Utilisateur récupéré: " + utilisateurRecup);

            // Mise à jour d'un utilisateur
            utilisateurRecup.setEmail("jean.dupont@exemple.com");
            utilisateurRecup = service.update(utilisateurRecup);
            System.out.println("Utilisateur mis à jour: " + utilisateurRecup);

            // Suppression d'un utilisateur
            service.delete(utilisateurRecup.getId());
            System.out.println("Utilisateur supprimé");

        } finally {
            JpaUtil.close();
        }
    }
}
```

## Requêtes avec JPQL

JPQL (Java Persistence Query Language) est un langage de requête orienté objet similaire à SQL mais qui opère sur des entités plutôt que sur des tables.

### 1. Requêtes simples

```java
import jakarta.persistence.EntityManager;
import jakarta.persistence.Query;
import jakarta.persistence.TypedQuery;

public class UtilisateurQueryService {

    // Trouver tous les utilisateurs
    public List<Utilisateur> findAll() {
        EntityManager em = JpaUtil.getEntityManager();

        try {
            TypedQuery<Utilisateur> query = em.createQuery(
                "SELECT u FROM Utilisateur u", Utilisateur.class);
            return query.getResultList();
        } finally {
            em.close();
        }
    }

    // Trouver par nom
    public List<Utilisateur> findByName(String nom) {
        EntityManager em = JpaUtil.getEntityManager();

        try {
            TypedQuery<Utilisateur> query = em.createQuery(
                "SELECT u FROM Utilisateur u WHERE u.nom LIKE :nom", Utilisateur.class);
            query.setParameter("nom", "%" + nom + "%");
            return query.getResultList();
        } finally {
            em.close();
        }
    }

    // Trouver par âge minimum
    public List<Utilisateur> findByMinimumAge(int age) {
        EntityManager em = JpaUtil.getEntityManager();

        try {
            TypedQuery<Utilisateur> query = em.createQuery(
                "SELECT u FROM Utilisateur u WHERE u.age >= :age ORDER BY u.age", Utilisateur.class);
            query.setParameter("age", age);
            return query.getResultList();
        } finally {
            em.close();
        }
    }

    // Compter le nombre d'utilisateurs
    public long countAll() {
        EntityManager em = JpaUtil.getEntityManager();

        try {
            TypedQuery<Long> query = em.createQuery(
                "SELECT COUNT(u) FROM Utilisateur u", Long.class);
            return query.getSingleResult();
        } finally {
            em.close();
        }
    }

    // Mettre à jour en masse
    public int updateUserStatus(Utilisateur.UserStatus fromStatus, Utilisateur.UserStatus toStatus) {
        EntityManager em = JpaUtil.getEntityManager();
        EntityTransaction tx = null;

        try {
            tx = em.getTransaction();
            tx.begin();

            Query query = em.createQuery(
                "UPDATE Utilisateur u SET u.statut = :toStatus WHERE u.statut = :fromStatus");
            query.setParameter("toStatus", toStatus);
            query.setParameter("fromStatus", fromStatus);

            int updatedCount = query.executeUpdate();

            tx.commit();
            return updatedCount;
        } catch (Exception e) {
            if (tx != null && tx.isActive()) {
                tx.rollback();
            }
            e.printStackTrace();
            throw e;
        } finally {
            em.close();
        }
    }

    // Supprimer en masse
    public int deleteInactiveUsers() {
        EntityManager em = JpaUtil.getEntityManager();
        EntityTransaction tx = null;

        try {
            tx = em.getTransaction();
            tx.begin();

            Query query = em.createQuery(
                "DELETE FROM Utilisateur u WHERE u.statut = :statut");
            query.setParameter("statut", Utilisateur.UserStatus.INACTIF);

            int deletedCount = query.executeUpdate();

            tx.commit();
            return deletedCount;
        } catch (Exception e) {
            if (tx != null && tx.isActive()) {
                tx.rollback();
            }
            e.printStackTrace();
            throw e;
        } finally {
            em.close();
        }
    }
}
```

### 2. Requêtes avec des associations

```java
// Récupérer des entités liées
public List<Departement> findDepartmentsWithEmployees() {
    EntityManager em = JpaUtil.getEntityManager();

    try {
        TypedQuery<Departement> query = em.createQuery(
            "SELECT DISTINCT d FROM Departement d JOIN FETCH d.employes", Departement.class);
        return query.getResultList();
    } finally {
        em.close();
    }
}

// Requête avec jointure mais sans FETCH (lazy loading)
public List<Employe> findEmployeesByDepartment(String nomDepartement) {
    EntityManager em = JpaUtil.getEntityManager();

    try {
        TypedQuery<Employe> query = em.createQuery(
            "SELECT e FROM Employe e JOIN e.departement d WHERE d.nom = :nom", Employe.class);
        query.setParameter("nom", nomDepartement);
        return query.getResultList();
    } finally {
        em.close();
    }
}
```

### 3. Named Queries

```java
@Entity
@Table(name = "utilisateurs")
@NamedQueries({
    @NamedQuery(
        name = "Utilisateur.findAll",
        query = "SELECT u FROM Utilisateur u"
    ),
    @NamedQuery(
        name = "Utilisateur.findByEmail",
        query = "SELECT u FROM Utilisateur u WHERE u.email = :email"
    ),
    @NamedQuery(
        name = "Utilisateur.countByStatus",
        query = "SELECT COUNT(u) FROM Utilisateur u WHERE u.statut = :statut"
    )
})
public class Utilisateur {
    // Définition de la classe comme avant
}

// Utilisation
public Utilisateur findByEmail(String email) {
    EntityManager em = JpaUtil.getEntityManager();

    try {
        TypedQuery<Utilisateur> query = em.createNamedQuery("Utilisateur.findByEmail", Utilisateur.class);
        query.setParameter("email", email);
        return query.getSingleResult();
    } catch (NoResultException e) {
        return null;
    } finally {
        em.close();
    }
}
```

### 4. Critères API (alternative au JPQL)

```java
import jakarta.persistence.EntityManager;
import jakarta.persistence.criteria.CriteriaBuilder;
import jakarta.persistence.criteria.CriteriaQuery;
import jakarta.persistence.criteria.Predicate;
import jakarta.persistence.criteria.Root;

public List<Utilisateur> findByNameAndStatus(String nom, Utilisateur.UserStatus statut) {
    EntityManager em = JpaUtil.getEntityManager();

    try {
        CriteriaBuilder cb = em.getCriteriaBuilder();
        CriteriaQuery<Utilisateur> cq = cb.createQuery(Utilisateur.class);
        Root<Utilisateur> root = cq.from(Utilisateur.class);

        Predicate nomPredicate = cb.like(root.get("nom"), "%" + nom + "%");
        Predicate statusPredicate = cb.equal(root.get("statut"), statut);

        cq.where(cb.and(nomPredicate, statusPredicate));
        cq.orderBy(cb.asc(root.get("nom")));

        TypedQuery<Utilisateur> query = em.createQuery(cq);
        return query.getResultList();
    } finally {
        em.close();
    }
}
```

## Gestion des transactions

### 1. Gestion manuelle des transactions

```java
import jakarta.persistence.EntityManager;
import jakarta.persistence.EntityTransaction;

public void processBusinessLogic() {
    EntityManager em = JpaUtil.getEntityManager();
    EntityTransaction tx = null;

    try {
        tx = em.getTransaction();
        tx.begin();

        // Opérations sur la base de données
        Utilisateur utilisateur = new Utilisateur("Nouveau", "nouveau@exemple.com", 25);
        em.persist(utilisateur);

        // Autres opérations...
        Departement departement = em.find(Departement.class, 1L);
        Employe employe = new Employe();
        employe.setNom("Nouvel Employé");
        employe.setDepartement(departement);
        em.persist(employe);

        tx.commit();
    } catch (Exception e) {
        if (tx != null && tx.isActive()) {
            tx.rollback();
        }
        e.printStackTrace();
        throw e;
    } finally {
        em.close();
    }
}
```

### 2. Gestion des erreurs et rollback

```java
public void transfertBudget(Long fromDeptId, Long toDeptId, double montant) {
    EntityManager em = JpaUtil.getEntityManager();
    EntityTransaction tx = null;

    try {
        tx = em.getTransaction();
        tx.begin();

        // Récupérer les départements
        Departement fromDept = em.find(Departement.class, fromDeptId);
        Departement toDept = em.find(Departement.class, toDeptId);

        if (fromDept == null || toDept == null) {
            throw new IllegalArgumentException("Département(s) non trouvé(s)");
        }

        // Vérifier le budget disponible
        if (fromDept.getBudget() < montant) {
            throw new BusinessException("Budget insuffisant pour le transfert");
        }

        // Effectuer le transfert
        fromDept.setBudget(fromDept.getBudget() - montant);
        toDept.setBudget(toDept.getBudget() + montant);

        // Sauvegarder les modifications
        em.merge(fromDept);
        em.merge(toDept);

        // Si une exception est levée avant cette ligne, un rollback sera automatiquement effectué
        tx.commit();
    } catch (Exception e) {
        if (tx != null && tx.isActive()) {
            tx.rollback();
        }
        e.printStackTrace();
        throw e;
    } finally {
        em.close();
    }
}
```

## Optimisation des performances avec JPA

### 1. Stratégies de chargement (Lazy vs Eager)

```java
@Entity
public class Departement {
    // ...

    // Chargement paresseux (LAZY) - par défaut pour les collections
    @OneToMany(mappedBy = "departement", fetch = FetchType.LAZY)
    private List<Employe> employes = new ArrayList<>();

    // Chargement immédiat (EAGER) - utile pour les données toujours nécessaires
    @OneToOne(fetch = FetchType.EAGER)
    private Manager manager;
}
```

### 2. Mise en cache

```java
@Entity
@Cacheable
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class ConfigurationGlobale {
    @Id
    private String cle;

    private String valeur;

    // Getters et setters
}
```

### 3. Batch Processing

```java
public void importerUtilisateurs(List<Utilisateur> utilisateurs) {
    EntityManager em = JpaUtil.getEntityManager();
    EntityTransaction tx = null;

    try {
        tx = em.getTransaction();
        tx.begin();

        int batchSize = 50;
        for (int i = 0; i < utilisateurs.size(); i++) {
            Utilisateur utilisateur = utilisateurs.get(i);
            em.persist(utilisateur);

            // Flush et clear périodiquement pour éviter de saturer la mémoire
            if (i % batchSize == 0 && i > 0) {
                em.flush();
                em.clear();
            }
        }

        tx.commit();
    } catch (Exception e) {
        if (tx != null && tx.isActive()) {
            tx.rollback();
        }
        e.printStackTrace();
        throw e;
    } finally {
        em.close();
    }
}
```

## Bonnes pratiques JPA

1. **Utiliser des transactions explicites** : Envelopper toutes les opérations d'écriture dans des transactions explicites.

2. **Privilégier les relations lazy** : Utiliser le chargement paresseux (LAZY) pour les collections et les entités non systématiquement nécessaires.

3. **Join fetch pour éviter le problème N+1** : Utiliser JOIN FETCH pour charger les entités liées en une seule requête lorsque c'est nécessaire.

4. **Fermer l'EntityManager** : Toujours fermer l'EntityManager après utilisation pour libérer les ressources.

5. **Éviter les relations bidirectionnelles inutiles** : N'utiliser des relations bidirectionnelles que si nécessaire pour réduire la complexité.

6. **Utiliser des requêtes nommées** : Favoriser les requêtes nommées pour améliorer la lisibilité et la réutilisabilité.

7. **Choisir les bonnes stratégies de cascade** : Utiliser les cascades avec parcimonie, en comprenant bien leur impact.

8. **Éviter de trop charger le contexte de persistance** : Vider périodiquement le contexte de persistance lors du traitement de grands volumes de données.

9. **Utiliser des DTOs pour les vues** : Créer des objets de transfert de données (DTO) pour éviter d'exposer directement les entités JPA.

10. **Optimiser les mises à jour en masse** : Utiliser des requêtes JPQL de mise à jour/suppression pour les opérations en masse au lieu de charger et de modifier chaque entité individuellement.
