# Architecture Microservices en Java

L'architecture microservices est une approche de développement logiciel qui structure une application comme un ensemble de services faiblement couplés, déployables indépendamment et organisés autour des capacités métier.

## Principes fondamentaux

1. **Services indépendants** : Chaque microservice est une unité autonome qui peut être développée, déployée et mise à l'échelle indépendamment.

2. **Organisation autour des capacités métier** : Les services sont définis en fonction des domaines métier plutôt que des couches techniques.

3. **Décentralisation** : Chaque service peut utiliser sa propre technologie, sa propre base de données et son propre modèle de données.

4. **Communication par API** : Les services communiquent entre eux via des API bien définies, généralement REST ou messagerie asynchrone.

5. **Résilience** : Les services sont conçus pour être résistants aux pannes, avec des mécanismes comme le circuit breaker et le fallback.

6. **Évolutivité** : Les services peuvent être mis à l'échelle individuellement selon les besoins.

## Technologies Java pour les microservices

### Frameworks de microservices

#### Spring Boot et Spring Cloud

Spring Boot est l'un des frameworks les plus populaires pour développer des microservices en Java.

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class UserServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }
}
```

Spring Cloud fournit des outils pour les systèmes distribués :

- **Eureka** : Service discovery
- **Ribbon** : Load balancing côté client
- **Feign** : Client REST déclaratif
- **Hystrix** : Circuit breaker
- **Zuul/Gateway** : API Gateway
- **Config Server** : Configuration centralisée

#### Quarkus

Quarkus est un framework Java conçu pour les microservices et les environnements cloud-native.

```java
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;

@Path("/hello")
public class GreetingResource {

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    public String hello() {
        return "Hello RESTEasy";
    }
}
```

#### Micronaut

Micronaut est un framework moderne qui offre un démarrage rapide et une faible empreinte mémoire.

```java
import io.micronaut.http.annotation.Controller;
import io.micronaut.http.annotation.Get;
import io.micronaut.http.HttpStatus;

@Controller("/hello")
public class HelloController {

    @Get
    public HttpStatus index() {
        return HttpStatus.OK;
    }
}
```

### Communication entre services

#### REST avec RestTemplate (Spring)

```java
import org.springframework.web.client.RestTemplate;
import org.springframework.stereotype.Service;

@Service
public class ProductClient {
    private final RestTemplate restTemplate;
    private final String productServiceUrl = "http://product-service/products";

    public ProductClient(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    public Product getProduct(Long id) {
        return restTemplate.getForObject(productServiceUrl + "/" + id, Product.class);
    }
}
```

#### Feign Client (Spring Cloud)

```java
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@FeignClient(name = "product-service")
public interface ProductClient {

    @GetMapping("/products/{id}")
    Product getProduct(@PathVariable("id") Long id);
}
```

#### Messagerie avec Kafka

```java
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Service;

@Service
public class OrderService {
    private final KafkaTemplate<String, OrderEvent> kafkaTemplate;

    public OrderService(KafkaTemplate<String, OrderEvent> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    public void createOrder(Order order) {
        // Logique de création de commande

        // Publier un événement
        OrderEvent event = new OrderEvent(order.getId(), "ORDER_CREATED", order);
        kafkaTemplate.send("order-events", event);
    }

    @KafkaListener(topics = "payment-events")
    public void handlePaymentEvent(PaymentEvent event) {
        if ("PAYMENT_COMPLETED".equals(event.getType())) {
            // Mettre à jour le statut de la commande
        }
    }
}
```

### API Gateway

#### Spring Cloud Gateway

```java
import org.springframework.cloud.gateway.route.RouteLocator;
import org.springframework.cloud.gateway.route.builder.RouteLocatorBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class GatewayConfig {

    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
            .route("user-service", r -> r.path("/users/**")
                .uri("lb://user-service"))
            .route("product-service", r -> r.path("/products/**")
                .uri("lb://product-service"))
            .route("order-service", r -> r.path("/orders/**")
                .uri("lb://order-service"))
            .build();
    }
}
```

### Service Discovery

#### Eureka (Spring Cloud Netflix)

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class DiscoveryServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(DiscoveryServiceApplication.class, args);
    }
}
```

Configuration client (application.yml) :

```yaml
spring:
  application:
    name: user-service

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```

### Circuit Breaker

#### Resilience4j

```java
import io.github.resilience4j.circuitbreaker.annotation.CircuitBreaker;
import org.springframework.stereotype.Service;

@Service
public class ProductService {

    private final ProductClient productClient;

    public ProductService(ProductClient productClient) {
        this.productClient = productClient;
    }

    @CircuitBreaker(name = "productService", fallbackMethod = "getDefaultProduct")
    public Product getProduct(Long id) {
        return productClient.getProduct(id);
    }

    public Product getDefaultProduct(Long id, Exception e) {
        return new Product(id, "Default Product", "This is a fallback product", 0.0);
    }
}
```

## Exemple d'architecture microservices e-commerce

### Structure des services

```
e-commerce/
├── discovery-service/       # Service discovery (Eureka)
├── config-service/          # Configuration centralisée
├── gateway-service/         # API Gateway
├── auth-service/            # Authentification et autorisation
├── user-service/            # Gestion des utilisateurs
├── product-service/         # Catalogue de produits
├── inventory-service/       # Gestion des stocks
├── order-service/           # Traitement des commandes
├── payment-service/         # Traitement des paiements
└── notification-service/    # Notifications (emails, SMS)
```

### Exemple de service produit

```java
// ProductController.java
import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController
@RequestMapping("/products")
public class ProductController {

    private final ProductService productService;

    public ProductController(ProductService productService) {
        this.productService = productService;
    }

    @GetMapping
    public List<Product> getAllProducts() {
        return productService.findAll();
    }

    @GetMapping("/{id}")
    public Product getProduct(@PathVariable Long id) {
        return productService.findById(id);
    }

    @PostMapping
    public Product createProduct(@RequestBody Product product) {
        return productService.save(product);
    }

    @PutMapping("/{id}")
    public Product updateProduct(@PathVariable Long id, @RequestBody Product product) {
        product.setId(id);
        return productService.update(product);
    }

    @DeleteMapping("/{id}")
    public void deleteProduct(@PathVariable Long id) {
        productService.deleteById(id);
    }
}

// Product.java
import javax.persistence.*;

@Entity
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String description;
    private Double price;

    // Getters, setters, constructors
}

// ProductService.java
import org.springframework.stereotype.Service;
import java.util.List;

@Service
public class ProductService {

    private final ProductRepository productRepository;

    public ProductService(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }

    public List<Product> findAll() {
        return productRepository.findAll();
    }

    public Product findById(Long id) {
        return productRepository.findById(id)
            .orElseThrow(() -> new ProductNotFoundException("Product not found with id: " + id));
    }

    public Product save(Product product) {
        return productRepository.save(product);
    }

    public Product update(Product product) {
        if (!productRepository.existsById(product.getId())) {
            throw new ProductNotFoundException("Product not found with id: " + product.getId());
        }
        return productRepository.save(product);
    }

    public void deleteById(Long id) {
        productRepository.deleteById(id);
    }
}

// ProductRepository.java
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface ProductRepository extends JpaRepository<Product, Long> {
}
```

### Communication entre services (exemple)

```java
// OrderService.java
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class OrderService {

    private final OrderRepository orderRepository;
    private final ProductClient productClient;
    private final InventoryClient inventoryClient;
    private final KafkaTemplate<String, OrderEvent> kafkaTemplate;

    public OrderService(OrderRepository orderRepository,
                        ProductClient productClient,
                        InventoryClient inventoryClient,
                        KafkaTemplate<String, OrderEvent> kafkaTemplate) {
        this.orderRepository = orderRepository;
        this.productClient = productClient;
        this.inventoryClient = inventoryClient;
        this.kafkaTemplate = kafkaTemplate;
    }

    @Transactional
    public Order createOrder(OrderRequest orderRequest) {
        // Vérifier la disponibilité des produits
        for (OrderItem item : orderRequest.getItems()) {
            Product product = productClient.getProduct(item.getProductId());
            boolean isAvailable = inventoryClient.checkAvailability(item.getProductId(), item.getQuantity());

            if (!isAvailable) {
                throw new InsufficientInventoryException("Product " + product.getName() + " is not available in the requested quantity");
            }
        }

        // Créer la commande
        Order order = new Order();
        order.setUserId(orderRequest.getUserId());
        order.setStatus(OrderStatus.PENDING);
        order.setItems(orderRequest.getItems());
        order.setTotalAmount(calculateTotalAmount(orderRequest.getItems()));

        Order savedOrder = orderRepository.save(order);

        // Réserver les produits dans l'inventaire
        for (OrderItem item : orderRequest.getItems()) {
            inventoryClient.reserveProduct(item.getProductId(), item.getQuantity(), savedOrder.getId());
        }

        // Publier un événement
        OrderEvent event = new OrderEvent(savedOrder.getId(), "ORDER_CREATED", savedOrder);
        kafkaTemplate.send("order-events", event);

        return savedOrder;
    }

    private double calculateTotalAmount(List<OrderItem> items) {
        return items.stream()
            .mapToDouble(item -> {
                Product product = productClient.getProduct(item.getProductId());
                return product.getPrice() * item.getQuantity();
            })
            .sum();
    }
}
```

## Avantages et défis des microservices

### Avantages

1. **Déploiement indépendant** : Chaque service peut être déployé séparément, ce qui permet des mises à jour plus fréquentes et plus sûres.

2. **Scalabilité ciblée** : Les services peuvent être mis à l'échelle individuellement selon les besoins, optimisant ainsi l'utilisation des ressources.

3. **Diversité technologique** : Différents services peuvent utiliser différentes technologies, permettant d'utiliser l'outil le plus adapté à chaque cas d'utilisation.

4. **Équipes autonomes** : Les équipes peuvent travailler sur des services distincts sans interférer les unes avec les autres.

5. **Résilience** : La défaillance d'un service n'entraîne pas nécessairement la défaillance de l'ensemble du système.

### Défis

1. **Complexité opérationnelle** : La gestion de nombreux services déployés indépendamment nécessite des outils et des processus sophistiqués.

2. **Transactions distribuées** : Maintenir la cohérence des données entre les services est complexe.

3. **Latence réseau** : La communication entre services ajoute de la latence.

4. **Observabilité** : Suivre les problèmes à travers plusieurs services nécessite des outils de surveillance et de traçage avancés.

5. **Coûts de développement** : Le développement initial peut être plus coûteux en raison de la complexité accrue.

## Bonnes pratiques

1. **Définir les limites des services** : Utiliser le Domain-Driven Design (DDD) pour définir les limites des services en fonction des contextes métier.

2. **API versioning** : Mettre en place un système de versionnement des API pour gérer les évolutions sans casser la compatibilité.

3. **Automatisation** : Automatiser les tests, le déploiement et la surveillance pour gérer efficacement de nombreux services.

4. **Conception pour la résilience** : Implémenter des patterns comme le circuit breaker, le retry et le fallback.

5. **Monitoring et traçage** : Mettre en place des outils comme Prometheus, Grafana et Jaeger pour surveiller et diagnostiquer les problèmes.

6. **Documentation des API** : Documenter clairement les API avec des outils comme Swagger/OpenAPI.

7. **Gestion des données** : Préférer le pattern "Database per Service" et utiliser des événements pour la synchronisation des données.

## Outils et technologies complémentaires

### Conteneurisation et orchestration

- **Docker** : Pour la conteneurisation des services
- **Kubernetes** : Pour l'orchestration des conteneurs
- **Helm** : Pour le packaging et le déploiement des applications Kubernetes

### Observabilité

- **Prometheus** : Pour la collecte de métriques
- **Grafana** : Pour la visualisation des métriques
- **ELK Stack** (Elasticsearch, Logstash, Kibana) : Pour la gestion des logs
- **Jaeger/Zipkin** : Pour le traçage distribué

### CI/CD

- **Jenkins** : Pour l'intégration et le déploiement continus
- **GitLab CI** : Pour l'intégration et le déploiement continus
- **GitHub Actions** : Pour l'automatisation des workflows

### API Management

- **Kong** : API Gateway open source
- **Apigee** : Solution complète d'API management
- **Swagger/OpenAPI** : Pour la documentation des API
