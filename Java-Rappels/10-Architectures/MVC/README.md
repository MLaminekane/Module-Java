# Architecture MVC (Model-View-Controller)

L'architecture MVC est un pattern de conception qui divise une application en trois composants principaux, chacun responsable d'aspects spécifiques de l'application.

## Principes fondamentaux

### Model (Modèle)

- Représente les données et la logique métier de l'application
- Indépendant de l'interface utilisateur
- Notifie les vues des changements d'état (souvent via le pattern Observer)

### View (Vue)

- Responsable de l'affichage des données à l'utilisateur
- Reçoit les données du modèle et les présente
- Transmet les actions de l'utilisateur au contrôleur

### Controller (Contrôleur)

- Interprète les actions de l'utilisateur
- Met à jour le modèle en conséquence
- Sélectionne la vue appropriée à afficher

## Flux de données dans MVC

1. L'utilisateur interagit avec la vue (ex: clic sur un bouton)
2. La vue transmet l'action au contrôleur
3. Le contrôleur traite l'action et met à jour le modèle
4. Le modèle notifie la vue des changements
5. La vue se met à jour avec les nouvelles données du modèle

## Implémentation en Java

### Exemple simple de MVC

#### Model

```java
import java.util.ArrayList;
import java.util.List;

public class UserModel {
    private String username;
    private String email;
    private List<ModelObserver> observers = new ArrayList<>();

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
        notifyObservers();
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
        notifyObservers();
    }

    // Pattern Observer pour notifier les vues
    public void addObserver(ModelObserver observer) {
        observers.add(observer);
    }

    public void removeObserver(ModelObserver observer) {
        observers.remove(observer);
    }

    private void notifyObservers() {
        for (ModelObserver observer : observers) {
            observer.update();
        }
    }

    // Interface pour le pattern Observer
    public interface ModelObserver {
        void update();
    }
}
```

#### View

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionListener;

public class UserView extends JFrame implements UserModel.ModelObserver {
    private UserModel model;
    private JTextField usernameField;
    private JTextField emailField;
    private JButton saveButton;

    public UserView(UserModel model) {
        this.model = model;
        model.addObserver(this);

        // Configuration de la fenêtre
        setTitle("User Profile");
        setSize(300, 200);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        // Création des composants
        JPanel panel = new JPanel(new GridLayout(3, 2, 10, 10));
        panel.setBorder(BorderFactory.createEmptyBorder(10, 10, 10, 10));

        panel.add(new JLabel("Username:"));
        usernameField = new JTextField(model.getUsername());
        panel.add(usernameField);

        panel.add(new JLabel("Email:"));
        emailField = new JTextField(model.getEmail());
        panel.add(emailField);

        saveButton = new JButton("Save");
        panel.add(saveButton);

        add(panel);
    }

    public void setSaveButtonListener(ActionListener listener) {
        saveButton.addActionListener(listener);
    }

    public String getUsername() {
        return usernameField.getText();
    }

    public String getEmail() {
        return emailField.getText();
    }

    @Override
    public void update() {
        usernameField.setText(model.getUsername());
        emailField.setText(model.getEmail());
    }
}
```

#### Controller

```java
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;

public class UserController {
    private UserModel model;
    private UserView view;

    public UserController(UserModel model, UserView view) {
        this.model = model;
        this.view = view;

        // Ajouter un écouteur d'événements pour le bouton Save
        view.setSaveButtonListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                saveUser();
            }
        });
    }

    private void saveUser() {
        // Récupérer les données de la vue
        String username = view.getUsername();
        String email = view.getEmail();

        // Mettre à jour le modèle
        model.setUsername(username);
        model.setEmail(email);

        // Logique supplémentaire (ex: validation, persistance)
        System.out.println("User saved: " + username + " (" + email + ")");
    }
}
```

#### Application principale

```java
public class MVCApplication {
    public static void main(String[] args) {
        // Créer le modèle
        UserModel model = new UserModel();
        model.setUsername("john_doe");
        model.setEmail("john@example.com");

        // Créer la vue
        UserView view = new UserView(model);

        // Créer le contrôleur
        UserController controller = new UserController(model, view);

        // Afficher la vue
        view.setVisible(true);
    }
}
```

## MVC dans les frameworks Java

### Spring MVC

Spring MVC est un framework web basé sur le pattern MVC. Voici un exemple simplifié :

#### Model

```java
public class User {
    private String username;
    private String email;

    // Getters et setters
}
```

#### Controller

```java
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

@Controller
@RequestMapping("/users")
public class UserController {

    @GetMapping("/{id}")
    public String getUser(@PathVariable Long id, Model model) {
        // Récupérer l'utilisateur depuis la base de données
        User user = userService.findById(id);

        // Ajouter l'utilisateur au modèle
        model.addAttribute("user", user);

        // Retourner le nom de la vue
        return "user-details";
    }

    @PostMapping
    public String saveUser(@ModelAttribute User user) {
        // Sauvegarder l'utilisateur
        userService.save(user);

        // Rediriger vers la liste des utilisateurs
        return "redirect:/users";
    }
}
```

#### View (Thymeleaf)

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
  <head>
    <title>User Details</title>
  </head>
  <body>
    <h1>User Details</h1>
    <div>
      <p>
        <strong>Username:</strong>
        <span th:text="${user.username}">username</span>
      </p>
      <p><strong>Email:</strong> <span th:text="${user.email}">email</span></p>
    </div>

    <h2>Edit User</h2>
    <form th:action="@{/users}" method="post">
      <div>
        <label for="username">Username:</label>
        <input
          type="text"
          id="username"
          name="username"
          th:value="${user.username}"
        />
      </div>
      <div>
        <label for="email">Email:</label>
        <input type="email" id="email" name="email" th:value="${user.email}" />
      </div>
      <button type="submit">Save</button>
    </form>
  </body>
</html>
```

### JavaFX MVC

JavaFX facilite l'implémentation du pattern MVC pour les applications desktop :

#### Model

```java
import javafx.beans.property.SimpleStringProperty;
import javafx.beans.property.StringProperty;

public class UserModel {
    private final StringProperty username = new SimpleStringProperty(this, "username", "");
    private final StringProperty email = new SimpleStringProperty(this, "email", "");

    public StringProperty usernameProperty() {
        return username;
    }

    public String getUsername() {
        return username.get();
    }

    public void setUsername(String username) {
        this.username.set(username);
    }

    public StringProperty emailProperty() {
        return email;
    }

    public String getEmail() {
        return email.get();
    }

    public void setEmail(String email) {
        this.email.set(email);
    }
}
```

#### View

```java
import javafx.geometry.Insets;
import javafx.scene.Scene;
import javafx.scene.control.Button;
import javafx.scene.control.Label;
import javafx.scene.control.TextField;
import javafx.scene.layout.GridPane;
import javafx.stage.Stage;

public class UserView {
    private UserModel model;
    private TextField usernameField;
    private TextField emailField;
    private Button saveButton;

    public UserView(UserModel model, Stage stage) {
        this.model = model;

        // Création de la grille
        GridPane grid = new GridPane();
        grid.setPadding(new Insets(10, 10, 10, 10));
        grid.setVgap(5);
        grid.setHgap(5);

        // Création des composants
        grid.add(new Label("Username:"), 0, 0);
        usernameField = new TextField();
        grid.add(usernameField, 1, 0);

        grid.add(new Label("Email:"), 0, 1);
        emailField = new TextField();
        grid.add(emailField, 1, 1);

        saveButton = new Button("Save");
        grid.add(saveButton, 1, 2);

        // Binding bidirectionnel avec le modèle
        usernameField.textProperty().bindBidirectional(model.usernameProperty());
        emailField.textProperty().bindBidirectional(model.emailProperty());

        // Configuration de la scène
        Scene scene = new Scene(grid, 300, 150);
        stage.setTitle("User Profile");
        stage.setScene(scene);
    }

    public Button getSaveButton() {
        return saveButton;
    }
}
```

#### Controller

```java
public class UserController {
    private UserModel model;
    private UserView view;

    public UserController(UserModel model, UserView view) {
        this.model = model;
        this.view = view;

        // Ajouter un écouteur d'événements pour le bouton Save
        view.getSaveButton().setOnAction(event -> saveUser());
    }

    private void saveUser() {
        // Les données sont déjà dans le modèle grâce au binding bidirectionnel

        // Logique supplémentaire (ex: validation, persistance)
        System.out.println("User saved: " + model.getUsername() + " (" + model.getEmail() + ")");
    }
}
```

#### Application principale

```java
import javafx.application.Application;
import javafx.stage.Stage;

public class MVCApplication extends Application {
    @Override
    public void start(Stage primaryStage) {
        // Créer le modèle
        UserModel model = new UserModel();
        model.setUsername("john_doe");
        model.setEmail("john@example.com");

        // Créer la vue
        UserView view = new UserView(model, primaryStage);

        // Créer le contrôleur
        UserController controller = new UserController(model, view);

        // Afficher la vue
        primaryStage.show();
    }

    public static void main(String[] args) {
        launch(args);
    }
}
```

## Avantages et inconvénients de MVC

### Avantages

- **Séparation des préoccupations** : Chaque composant a une responsabilité spécifique
- **Réutilisabilité** : Les modèles peuvent être réutilisés avec différentes vues
- **Facilité de maintenance** : Les modifications d'un composant ont un impact minimal sur les autres
- **Développement parallèle** : Différentes équipes peuvent travailler sur différents composants
- **Testabilité** : Les composants peuvent être testés indépendamment

### Inconvénients

- **Complexité** : Pour les petites applications, MVC peut être trop complexe
- **Courbe d'apprentissage** : Peut être difficile à comprendre pour les débutants
- **Overhead** : Nécessite plus de code que des approches plus simples
- **Couplage potentiel** : Si mal implémenté, peut conduire à un couplage fort entre les composants

## Variantes de MVC

### MVP (Model-View-Presenter)

- La vue est plus passive que dans MVC
- Le présentateur interagit directement avec la vue
- Utilisé dans Android et certaines applications desktop

### MVVM (Model-View-ViewModel)

- Utilise le binding de données bidirectionnel
- Le ViewModel expose les données du modèle dans un format adapté à la vue
- Populaire avec JavaFX, Android et les frameworks web modernes

## Bonnes pratiques MVC

1. **Garder la vue simple** : La vue ne doit contenir que la logique d'affichage
2. **Éviter la logique métier dans le contrôleur** : Le contrôleur doit déléguer la logique métier au modèle
3. **Utiliser des interfaces** : Définir des interfaces pour les composants MVC pour faciliter les tests
4. **Observer le modèle** : Utiliser le pattern Observer pour notifier les vues des changements du modèle
5. **Découpler les composants** : Minimiser les dépendances entre les composants
6. **Utiliser des DTOs** : Utiliser des objets de transfert de données pour éviter d'exposer directement le modèle
