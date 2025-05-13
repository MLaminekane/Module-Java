# Préparation à Spring Boot

Cette section couvre les concepts fondamentaux nécessaires pour bien comprendre et utiliser Spring Boot. Avant de plonger dans Spring Boot, il est important de maîtriser les principes de base du framework Spring.

## Inversion de Contrôle (IoC) et Injection de Dépendances (DI)

L'Inversion de Contrôle est un principe fondamental de Spring qui transfère le contrôle de la création et de la gestion des objets au conteneur Spring.

### Principe de l'IoC

```java
// Sans IoC - Couplage fort
public class UserService {
    private UserRepository userRepository = new UserRepositoryImpl();

    // Méthodes du service...
}

// Avec IoC - Couplage faible
public class UserService {
    private UserRepository userRepository;

    // Injection par constructeur
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    // Méthodes du service...
}
```

### Types d'injection de dépendances

```java
// 1. Injection par constructeur (recommandée)
public class UserService {
    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}

// 2. Injection par setter
public class UserService {
    private UserRepository userRepository;

    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}

// 3. Injection par champ (déconseillée)
public class UserService {
    @Autowired
    private UserRepository userRepository;
}
```

## Annotations Spring essentielles

Spring utilise des annotations pour configurer les composants et définir leur comportement.

### Annotations de composants

```java
// Définit un composant générique
@Component
public class MyComponent {
    // ...
}

// Définit un contrôleur (couche présentation)
@Controller
public class UserController {
    // ...
}

// Définit un service (couche métier)
@Service
public class UserService {
    // ...
}

// Définit un repository (couche d'accès aux données)
@Repository
public class UserRepository {
    // ...
}
```

### Annotations d'injection

```java
public class UserService {
    private final UserRepository userRepository;

    // Injection automatique par constructeur
    @Autowired
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    // Injection optionnelle
    @Autowired(required = false)
    public void setEmailService(EmailService emailService) {
        this.emailService = emailService;
    }

    // Qualifier pour distinguer plusieurs beans du même type
    @Autowired
    public void setNotificationService(@Qualifier("smsNotification") NotificationService notificationService) {
        this.notificationService = notificationService;
    }
}
```

### Annotations de configuration

```java
// Classe de configuration
@Configuration
public class AppConfig {

    // Définition d'un bean
    @Bean
    public UserRepository userRepository() {
        return new UserRepositoryImpl();
    }

    // Bean avec dépendances
    @Bean
    public UserService userService(UserRepository userRepository) {
        return new UserService(userRepository);
    }

    // Bean conditionnel
    @Bean
    @ConditionalOnProperty(name = "app.cache.enabled", havingValue = "true")
    public CacheManager cacheManager() {
        return new SimpleCacheManager();
    }
}
```

## Profils Spring

Les profils permettent de définir des beans et des configurations spécifiques à certains environnements.

```java
// Configuration spécifique à un profil
@Configuration
@Profile("development")
public class DevConfig {
    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.H2)
            .build();
    }
}

@Configuration
@Profile("production")
public class ProdConfig {
    @Bean
    public DataSource dataSource() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setUrl("jdbc:mysql://production-server:3306/myapp");
        dataSource.setUsername("prod_user");
        dataSource.setPassword("prod_password");
        return dataSource;
    }
}
```

Activation des profils:

```java
// Dans le code
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(MyApplication.class);
        app.setAdditionalProfiles("development");
        app.run(args);
    }
}

// Ou via application.properties
spring.profiles.active=development
```

## Gestion des propriétés de configuration

Spring Boot propose plusieurs façons de gérer les propriétés de configuration.

### Fichier application.properties/application.yml

```properties
# application.properties
server.port=8080
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=password
app.feature.enabled=true
```

```yaml
# application.yml
server:
  port: 8080
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: root
    password: password
app:
  feature:
    enabled: true
```

### Accès aux propriétés

```java
// Injection de propriétés individuelles
@Component
public class AppConfig {
    @Value("${server.port}")
    private int serverPort;

    @Value("${app.feature.enabled:false}")
    private boolean featureEnabled;
}

// Injection d'un groupe de propriétés
@Component
@ConfigurationProperties(prefix = "app")
public class AppProperties {
    private String name;
    private Feature feature = new Feature();

    public static class Feature {
        private boolean enabled;

        // getters et setters
    }

    // getters et setters
}
```

## AOP (Programmation Orientée Aspect)

L'AOP permet de séparer les préoccupations transversales (logging, sécurité, transactions) du code métier.

```java
@Aspect
@Component
public class LoggingAspect {
    private static final Logger logger = LoggerFactory.getLogger(LoggingAspect.class);

    // Exécution avant toutes les méthodes des services
    @Before("execution(* com.example.service.*.*(..))")
    public void logBeforeMethod(JoinPoint joinPoint) {
        logger.info("Exécution de la méthode: " + joinPoint.getSignature().getName());
    }

    // Exécution autour des méthodes annotées avec @LogExecutionTime
    @Around("@annotation(LogExecutionTime)")
    public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        Object result = joinPoint.proceed();
        long end = System.currentTimeMillis();
        logger.info("Méthode: " + joinPoint.getSignature().getName() + " exécutée en " + (end - start) + "ms");
        return result;
    }
}

// Annotation personnalisée
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface LogExecutionTime {
}
```

## Gestion des transactions

Spring facilite la gestion des transactions avec des annotations.

```java
@Service
public class UserService {
    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    // Transaction qui sera rollback en cas d'exception
    @Transactional
    public void createUser(User user) {
        userRepository.save(user);
        // Si une exception est lancée ici, la transaction sera annulée
    }

    // Transaction en lecture seule
    @Transactional(readOnly = true)
    public User getUserById(Long id) {
        return userRepository.findById(id).orElse(null);
    }

    // Transaction avec isolation spécifique
    @Transactional(isolation = Isolation.SERIALIZABLE)
    public void updateUserBalance(Long userId, BigDecimal amount) {
        // ...
    }

    // Transaction qui ne sera rollback que pour certaines exceptions
    @Transactional(rollbackFor = CustomException.class, noRollbackFor = IgnoredException.class)
    public void processUserData(User user) {
        // ...
    }
}
```

## Spring MVC - Bases

Spring MVC est le framework web de Spring pour développer des applications web et des API REST.

```java
@Controller
@RequestMapping("/users")
public class UserController {
    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    // Afficher un formulaire
    @GetMapping("/new")
    public String showForm(Model model) {
        model.addAttribute("user", new User());
        return "user-form"; // Nom de la vue à afficher
    }

    // Traiter la soumission d'un formulaire
    @PostMapping
    public String createUser(@Valid @ModelAttribute("user") User user, BindingResult result) {
        if (result.hasErrors()) {
            return "user-form";
        }
        userService.createUser(user);
        return "redirect:/users";
    }

    // API REST
    @GetMapping("/{id}")
    @ResponseBody
    public User getUser(@PathVariable Long id) {
        return userService.getUserById(id);
    }

    // Gestion des exceptions
    @ExceptionHandler(UserNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public String handleUserNotFound(UserNotFoundException ex, Model model) {
        model.addAttribute("message", ex.getMessage());
        return "error/not-found";
    }
}
```

## Validation des données

Spring intègre la validation des données via Bean Validation (JSR-380).

```java
public class User {
    @NotNull
    @Size(min = 2, max = 50)
    private String name;

    @NotBlank
    @Email
    private String email;

    @Min(18)
    private int age;

    @Pattern(regexp = "^\\d{10}$")
    private String phoneNumber;

    // getters et setters
}

@RestController
@RequestMapping("/api/users")
public class UserApiController {
    @PostMapping
    public ResponseEntity<User> createUser(@Valid @RequestBody User user) {
        // Si la validation échoue, une exception MethodArgumentNotValidException est lancée
        // et Spring renvoie automatiquement une réponse 400 Bad Request
        User savedUser = userService.save(user);
        return ResponseEntity.status(HttpStatus.CREATED).body(savedUser);
    }
}
```

## Tests avec Spring

Spring Boot facilite l'écriture de tests unitaires et d'intégration.

```java
// Test unitaire avec Mockito
@ExtendWith(MockitoExtension.class)
public class UserServiceTest {
    @Mock
    private UserRepository userRepository;

    @InjectMocks
    private UserService userService;

    @Test
    public void testGetUserById() {
        // Arrange
        User user = new User();
        user.setId(1L);
        user.setName("John");

        when(userRepository.findById(1L)).thenReturn(Optional.of(user));

        // Act
        User result = userService.getUserById(1L);

        // Assert
        assertNotNull(result);
        assertEquals("John", result.getName());
        verify(userRepository).findById(1L);
    }
}

// Test d'intégration avec Spring Boot
@SpringBootTest
public class UserControllerIntegrationTest {
    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Test
    public void testGetUser() throws Exception {
        User user = new User();
        user.setId(1L);
        user.setName("John");

        when(userService.getUserById(1L)).thenReturn(user);

        mockMvc.perform(get("/api/users/1"))
               .andExpect(status().isOk())
               .andExpect(jsonPath("$.name").value("John"));
    }
}
```

## Concepts avancés

### Spring Boot Actuator

Spring Boot Actuator fournit des endpoints pour surveiller et gérer votre application.

```java
// Configuration dans application.properties
management.endpoints.web.exposure.include=health,info,metrics
management.endpoint.health.show-details=always

// Personnalisation des informations
@Component
public class CustomHealthIndicator implements HealthIndicator {
    @Override
    public Health health() {
        // Vérification de l'état du système
        boolean systemUp = checkSystemHealth();

        if (systemUp) {
            return Health.up()
                         .withDetail("message", "Le système fonctionne correctement")
                         .build();
        } else {
            return Health.down()
                         .withDetail("message", "Problème détecté")
                         .build();
        }
    }
}
```

### Spring Data JPA

Spring Data JPA simplifie l'accès aux données avec JPA.

```java
// Entité JPA
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String email;

    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL)
    private List<Order> orders;

    // getters et setters
}

// Repository Spring Data JPA
public interface UserRepository extends JpaRepository<User, Long> {
    // Méthodes générées automatiquement: save, findById, findAll, delete...

    // Requêtes dérivées des noms de méthode
    List<User> findByNameContaining(String name);
    Optional<User> findByEmail(String email);

    // Requête JPQL personnalisée
    @Query("SELECT u FROM User u WHERE u.email LIKE %:domain%")
    List<User> findByEmailDomain(@Param("domain") String domain);

    // Requête native SQL
    @Query(value = "SELECT * FROM users WHERE active = true", nativeQuery = true)
    List<User> findAllActiveUsers();
}
```

## Préparation à Spring Boot

Spring Boot simplifie la configuration de Spring avec des configurations par défaut et l'auto-configuration.

### Structure d'un projet Spring Boot

```
src/
├── main/
│   ├── java/
│   │   └── com/
│   │       └── example/
│   │           └── myapp/
│   │               ├── MyApplication.java
│   │               ├── config/
│   │               ├── controller/
│   │               ├── model/
│   │               ├── repository/
│   │               └── service/
│   └── resources/
│       ├── application.properties
│       ├── static/
│       └── templates/
└── test/
    └── java/
        └── com/
            └── example/
                └── myapp/
                    └── ...
```

### Classe principale Spring Boot

```java
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

L'annotation `@SpringBootApplication` combine:

- `@Configuration`: Marque la classe comme source de définitions de beans
- `@EnableAutoConfiguration`: Active l'auto-configuration de Spring Boot
- `@ComponentScan`: Active la recherche de composants dans le package courant et ses sous-packages

### Starters Spring Boot

Les starters sont des dépendances qui regroupent des bibliothèques couramment utilisées:

```xml
<!-- Web (Spring MVC, Tomcat) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- JPA (Hibernate, Spring Data JPA) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<!-- Sécurité -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

<!-- Tests -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

## Ressources supplémentaires

- [Documentation officielle de Spring](https://spring.io/docs)
- [Guides Spring](https://spring.io/guides)
- [Tutoriels Spring Boot](https://www.baeldung.com/spring-boot)
- [Spring Boot Reference Documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/)
