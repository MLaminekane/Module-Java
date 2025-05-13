# Architecture Hexagonale (Ports et Adaptateurs)

L'architecture hexagonale, également connue sous le nom d'architecture en oignon ou ports et adaptateurs, est un modèle architectural qui vise à créer des applications dont la logique métier est indépendante des technologies externes.

## Principes fondamentaux

### Domaine métier au centre

- Le cœur de l'application contient la logique métier pure
- Indépendant des frameworks, bases de données, UI et autres préoccupations techniques
- Exprime directement le modèle du domaine et les règles métier

### Ports

- Interfaces définissant comment la logique métier interagit avec l'extérieur
- **Ports primaires (entrants)** : interfaces utilisées par les acteurs externes pour interagir avec l'application
- **Ports secondaires (sortants)** : interfaces que l'application utilise pour interagir avec des services externes

### Adaptateurs

- Implémentations concrètes des ports
- **Adaptateurs primaires (entrants)** : convertissent les requêtes externes en appels vers la logique métier
- **Adaptateurs secondaires (sortants)** : convertissent les appels de la logique métier en interactions avec des services externes

## Structure typique d'une application hexagonale

```
application/
├── domain/                  # Cœur métier
│   ├── model/               # Entités et objets de valeur
│   └── service/             # Services métier
├── port/                    # Définition des interfaces
│   ├── primary/             # Ports entrants (API)
│   └── secondary/           # Ports sortants (SPI)
└── adapter/                 # Implémentations
    ├── primary/             # Adaptateurs entrants (REST, CLI, etc.)
    └── secondary/           # Adaptateurs sortants (DB, API externes, etc.)
```

## Implémentation en Java

### Domaine

```java
// Entité du domaine
public class Product {
    private final String id;
    private String name;
    private BigDecimal price;
    private int stock;

    public Product(String id, String name, BigDecimal price, int stock) {
        this.id = id;
        this.name = name;
        this.price = price;
        this.stock = stock;
    }

    // Méthode métier
    public boolean canBeOrdered(int quantity) {
        return stock >= quantity;
    }

    public void decreaseStock(int quantity) {
        if (!canBeOrdered(quantity)) {
            throw new IllegalArgumentException("Not enough stock available");
        }
        this.stock -= quantity;
    }

    // Getters
    public String getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public BigDecimal getPrice() {
        return price;
    }

    public int getStock() {
        return stock;
    }
}

// Service du domaine
public class OrderService {
    private final ProductRepository productRepository;
    private final OrderRepository orderRepository;

    public OrderService(ProductRepository productRepository, OrderRepository orderRepository) {
        this.productRepository = productRepository;
        this.orderRepository = orderRepository;
    }

    public Order createOrder(String productId, int quantity, String customerId) {
        Product product = productRepository.findById(productId)
            .orElseThrow(() -> new ProductNotFoundException(productId));

        if (!product.canBeOrdered(quantity)) {
            throw new InsufficientStockException(productId);
        }

        product.decreaseStock(quantity);
        productRepository.save(product);

        Order order = new Order(
            UUID.randomUUID().toString(),
            customerId,
            productId,
            quantity,
            product.getPrice().multiply(BigDecimal.valueOf(quantity)),
            LocalDateTime.now()
        );

        return orderRepository.save(order);
    }
}
```

### Ports (interfaces)

```java
// Port primaire (API)
public interface OrderUseCase {
    Order createOrder(String productId, int quantity, String customerId);
    List<Order> getOrdersByCustomerId(String customerId);
    Optional<Order> getOrderById(String orderId);
}

// Ports secondaires (SPI)
public interface ProductRepository {
    Optional<Product> findById(String id);
    List<Product> findAll();
    Product save(Product product);
}

public interface OrderRepository {
    Order save(Order order);
    Optional<Order> findById(String id);
    List<Order> findByCustomerId(String customerId);
}
```

### Adaptateurs

```java
// Adaptateur primaire (REST)
@RestController
@RequestMapping("/orders")
public class OrderController {
    private final OrderUseCase orderUseCase;

    public OrderController(OrderUseCase orderUseCase) {
        this.orderUseCase = orderUseCase;
    }

    @PostMapping
    public ResponseEntity<OrderResponse> createOrder(@RequestBody CreateOrderRequest request) {
        Order order = orderUseCase.createOrder(
            request.getProductId(),
            request.getQuantity(),
            request.getCustomerId()
        );

        OrderResponse response = mapToResponse(order);
        return ResponseEntity.status(HttpStatus.CREATED).body(response);
    }

    @GetMapping("/{id}")
    public ResponseEntity<OrderResponse> getOrder(@PathVariable String id) {
        return orderUseCase.getOrderById(id)
            .map(order -> ResponseEntity.ok(mapToResponse(order)))
            .orElse(ResponseEntity.notFound().build());
    }

    private OrderResponse mapToResponse(Order order) {
        return new OrderResponse(
            order.getId(),
            order.getCustomerId(),
            order.getProductId(),
            order.getQuantity(),
            order.getTotalPrice(),
            order.getCreatedAt()
        );
    }
}

// Adaptateur secondaire (JPA)
@Repository
public class JpaProductRepository implements ProductRepository {
    private final SpringDataProductRepository repository;

    public JpaProductRepository(SpringDataProductRepository repository) {
        this.repository = repository;
    }

    @Override
    public Optional<Product> findById(String id) {
        return repository.findById(id).map(this::mapToDomain);
    }

    @Override
    public List<Product> findAll() {
        return repository.findAll().stream()
            .map(this::mapToDomain)
            .collect(Collectors.toList());
    }

    @Override
    public Product save(Product product) {
        ProductEntity entity = mapToEntity(product);
        ProductEntity savedEntity = repository.save(entity);
        return mapToDomain(savedEntity);
    }

    private Product mapToDomain(ProductEntity entity) {
        return new Product(
            entity.getId(),
            entity.getName(),
            entity.getPrice(),
            entity.getStock()
        );
    }

    private ProductEntity mapToEntity(Product product) {
        ProductEntity entity = new ProductEntity();
        entity.setId(product.getId());
        entity.setName(product.getName());
        entity.setPrice(product.getPrice());
        entity.setStock(product.getStock());
        return entity;
    }
}

// Entité JPA
@Entity
@Table(name = "products")
public class ProductEntity {
    @Id
    private String id;
    private String name;
    private BigDecimal price;
    private int stock;

    // Getters et setters
}

// Interface Spring Data
public interface SpringDataProductRepository extends JpaRepository<ProductEntity, String> {
}
```

### Configuration (câblage des composants)

```java
@Configuration
public class ApplicationConfig {

    @Bean
    public ProductRepository productRepository(SpringDataProductRepository springDataRepository) {
        return new JpaProductRepository(springDataRepository);
    }

    @Bean
    public OrderRepository orderRepository(SpringDataOrderRepository springDataRepository) {
        return new JpaOrderRepository(springDataRepository);
    }

    @Bean
    public OrderUseCase orderUseCase(ProductRepository productRepository, OrderRepository orderRepository) {
        return new OrderServiceImpl(productRepository, orderRepository);
    }
}

// Implémentation du port primaire
public class OrderServiceImpl implements OrderUseCase {
    private final OrderService orderService;
    private final OrderRepository orderRepository;

    public OrderServiceImpl(ProductRepository productRepository, OrderRepository orderRepository) {
        this.orderService = new OrderService(productRepository, orderRepository);
        this.orderRepository = orderRepository;
    }

    @Override
    public Order createOrder(String productId, int quantity, String customerId) {
        return orderService.createOrder(productId, quantity, customerId);
    }

    @Override
    public List<Order> getOrdersByCustomerId(String customerId) {
        return orderRepository.findByCustomerId(customerId);
    }

    @Override
    public Optional<Order> getOrderById(String orderId) {
        return orderRepository.findById(orderId);
    }
}
```

## Avantages de l'architecture hexagonale

1. **Indépendance du domaine** : La logique métier est isolée des détails techniques et peut évoluer indépendamment.

2. **Testabilité** : Le domaine peut être testé sans dépendances externes, en utilisant des adaptateurs simulés (mocks).

3. **Flexibilité** : Les adaptateurs peuvent être remplacés sans modifier le cœur de l'application (ex: changer de base de données).

4. **Maintenabilité** : La séparation claire des responsabilités facilite la compréhension et la maintenance du code.

5. **Évolution progressive** : Possibilité de commencer avec des adaptateurs simples et de les remplacer au fur et à mesure.

## Défis et considérations

1. **Complexité initiale** : Structure plus élaborée qui peut sembler excessive pour de petites applications.

2. **Surcharge de code** : Nécessite plus d'interfaces et de classes que des approches plus directes.

3. **Mapping des données** : Conversion fréquente entre les modèles du domaine et les modèles des adaptateurs.

4. **Courbe d'apprentissage** : Concept qui peut être difficile à saisir pour les développeurs habitués à d'autres architectures.

## Comparaison avec d'autres architectures

### Architecture hexagonale vs MVC

- MVC est centré sur l'interface utilisateur, tandis que l'architecture hexagonale est centrée sur le domaine.
- Dans MVC, le modèle est souvent couplé à la persistance, alors que dans l'architecture hexagonale, le domaine est totalement indépendant.
- MVC est plus simple à mettre en œuvre, mais offre moins de flexibilité pour les systèmes complexes.

### Architecture hexagonale vs Microservices

- Les microservices concernent la décomposition d'un système en services indépendants, tandis que l'architecture hexagonale concerne la structure interne d'une application.
- Les deux approches peuvent être combinées : chaque microservice peut suivre l'architecture hexagonale en interne.
- Les microservices ajoutent une complexité opérationnelle que l'architecture hexagonale n'aborde pas directement.

## Cas d'utilisation idéaux

L'architecture hexagonale est particulièrement adaptée pour :

1. **Applications à logique métier complexe** : Systèmes où les règles métier sont nombreuses et évolutives.

2. **Systèmes à longue durée de vie** : Applications qui doivent être maintenues pendant plusieurs années.

3. **Applications nécessitant une grande adaptabilité technique** : Systèmes qui pourraient avoir besoin de changer de base de données, de framework ou d'interface utilisateur.

4. **Développement piloté par les tests (TDD)** : La séparation claire facilite l'écriture de tests unitaires et d'intégration.

5. **Projets avec plusieurs équipes** : La séparation des préoccupations permet à différentes équipes de travailler sur différentes parties du système.

## Bonnes pratiques

1. **Garder le domaine pur** : Éviter d'introduire des dépendances techniques dans le domaine.

2. **Utiliser l'injection de dépendances** : Pour connecter les adaptateurs aux ports.

3. **Définir des interfaces claires** : Les ports doivent avoir une responsabilité unique et bien définie.

4. **Éviter les fuites de détails d'implémentation** : Les adaptateurs ne doivent pas exposer leurs détails internes.

5. **Utiliser des objets de transfert de données (DTO)** : Pour isoler le modèle du domaine des formats de données externes.
