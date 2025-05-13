# Maven - Gestion de dépendances et build

## Introduction à Maven

Maven est un outil de gestion de projets Java qui permet de gérer les dépendances, la documentation, les rapports, la construction et le déploiement des applications.

### Avantages de Maven

- Gestion automatisée des dépendances
- Structure de projet standardisée
- Cycle de vie de build unifié
- Possibilité d'étendre via des plugins
- Intégration avec les IDE majeurs (IntelliJ IDEA, Eclipse, NetBeans)

## Structure d'un projet Maven

```
mon-projet/
├── pom.xml                  # Fichier de configuration Maven
├── src/
│   ├── main/
│   │   ├── java/            # Code source Java
│   │   ├── resources/       # Ressources (fichiers de config, etc.)
│   │   └── webapp/          # Ressources web (pour applications web)
│   └── test/
│       ├── java/            # Tests unitaires
│       └── resources/       # Ressources pour les tests
└── target/                  # Répertoire de sortie (généré)
```

## Le fichier POM (Project Object Model)

Le fichier `pom.xml` est le cœur d'un projet Maven. Il contient toutes les informations nécessaires pour construire le projet.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <!-- Informations de base du projet -->
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>mon-projet</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>Mon Projet</name>
    <description>Description de mon projet</description>

    <!-- Propriétés -->
    <properties>
        <java.version>17</java.version>
        <maven.compiler.source>${java.version}</maven.compiler.source>
        <maven.compiler.target>${java.version}</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>

        <!-- Versions des dépendances -->
        <junit.version>5.9.2</junit.version>
        <log4j.version>2.20.0</log4j.version>
    </properties>

    <!-- Dépendances -->
    <dependencies>
        <!-- JUnit pour les tests -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-engine</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>

        <!-- Log4j pour la journalisation -->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>${log4j.version}</version>
        </dependency>
    </dependencies>

    <!-- Configuration de build -->
    <build>
        <plugins>
            <!-- Plugin Maven Compiler -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.11.0</version>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                </configuration>
            </plugin>

            <!-- Plugin Maven Surefire pour les tests -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.1.2</version>
            </plugin>

            <!-- Plugin pour créer un JAR exécutable -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>3.3.0</version>
                <configuration>
                    <archive>
                        <manifest>
                            <addClasspath>true</addClasspath>
                            <mainClass>com.example.MainClass</mainClass>
                        </manifest>
                    </archive>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

## Coordonnées Maven

Les coordonnées Maven identifient de manière unique un projet :

- **groupId** : Identifiant du groupe (souvent le nom de domaine inversé)
- **artifactId** : Nom du projet
- **version** : Version du projet
- **packaging** : Type de package (jar, war, ear, pom)

## Portées (Scopes) des dépendances

- **compile** (défaut) : Disponible pendant la compilation et l'exécution
- **provided** : Fourni par l'environnement d'exécution (ex: servlet-api)
- **runtime** : Nécessaire uniquement à l'exécution, pas à la compilation
- **test** : Nécessaire uniquement pour les tests
- **system** : Similaire à provided, mais il faut spécifier le chemin du JAR
- **import** : Utilisé avec `<dependencyManagement>` pour importer des dépendances

## Cycle de vie Maven

Maven définit plusieurs cycles de vie, chacun composé de phases :

### Cycle de vie par défaut (default)

1. **validate** : Valide le projet
2. **compile** : Compile le code source
3. **test** : Exécute les tests
4. **package** : Empaquette le code compilé (ex: JAR)
5. **verify** : Vérifie le package
6. **install** : Installe le package dans le dépôt local
7. **deploy** : Copie le package dans le dépôt distant

### Commandes Maven courantes

```bash
# Compiler le projet
mvn compile

# Exécuter les tests
mvn test

# Créer le package (JAR, WAR, etc.)
mvn package

# Installer dans le dépôt local
mvn install

# Nettoyer le répertoire target
mvn clean

# Nettoyer et installer
mvn clean install

# Sauter les tests
mvn install -DskipTests

# Générer la documentation
mvn javadoc:javadoc

# Créer un nouveau projet à partir d'un archétype
mvn archetype:generate -DgroupId=com.example -DartifactId=mon-projet -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

## Dépôts Maven

Maven utilise trois types de dépôts pour stocker les artefacts :

1. **Dépôt local** : `~/.m2/repository` sur votre machine
2. **Dépôt central** : https://repo.maven.apache.org/maven2/
3. **Dépôts distants** : Dépôts personnalisés (ex: Nexus, Artifactory)

Configuration des dépôts dans le POM :

```xml
<repositories>
    <repository>
        <id>central</id>
        <name>Maven Central</name>
        <url>https://repo.maven.apache.org/maven2</url>
    </repository>
    <repository>
        <id>company-repo</id>
        <name>Company Repository</name>
        <url>https://repo.company.com/maven</url>
    </repository>
</repositories>
```

## Profils Maven

Les profils permettent d'adapter la configuration selon l'environnement :

```xml
<profiles>
    <profile>
        <id>dev</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <db.url>jdbc:h2:mem:dev</db.url>
        </properties>
    </profile>
    <profile>
        <id>prod</id>
        <properties>
            <db.url>jdbc:mysql://production-db:3306/app</db.url>
        </properties>
    </profile>
</profiles>
```

Activation d'un profil :

```bash
mvn install -Pprod
```

## Gestion des dépendances

### Dépendances transitives

Maven gère automatiquement les dépendances transitives (dépendances des dépendances).

### Exclusions

Pour exclure une dépendance transitive :

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>5.3.27</version>
    <exclusions>
        <exclusion>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

### Gestion des versions

Pour centraliser la gestion des versions de dépendances :

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-framework-bom</artifactId>
            <version>5.3.27</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <!-- Pas besoin de spécifier la version, elle est définie dans le BOM -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
    </dependency>
</dependencies>
```

## Plugins Maven courants

### Maven Compiler Plugin

Configure le compilateur Java :

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.11.0</version>
    <configuration>
        <source>${java.version}</source>
        <target>${java.version}</target>
    </configuration>
</plugin>
```

### Maven Surefire Plugin

Exécute les tests unitaires :

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>3.1.2</version>
</plugin>
```

### Maven Shade Plugin

Crée un JAR uber (avec toutes les dépendances) :

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>3.4.1</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <transformers>
                    <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                        <mainClass>com.example.MainClass</mainClass>
                    </transformer>
                </transformers>
            </configuration>
        </execution>
    </executions>
</plugin>
```

### Spring Boot Maven Plugin

Pour les projets Spring Boot :

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <version>3.1.2</version>
</plugin>
```

## Projets multi-modules

Maven permet de gérer des projets composés de plusieurs modules :

```xml
<!-- POM parent -->
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>parent-project</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>

    <modules>
        <module>module-common</module>
        <module>module-service</module>
        <module>module-web</module>
    </modules>

    <!-- Configuration commune à tous les modules -->
    <properties>
        <java.version>17</java.version>
    </properties>

    <dependencyManagement>
        <!-- ... -->
    </dependencyManagement>

    <build>
        <!-- ... -->
    </build>
</project>
```

Dans chaque module :

```xml
<!-- POM d'un module -->
<project>
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>parent-project</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>module-service</artifactId>

    <dependencies>
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>module-common</artifactId>
            <version>${project.version}</version>
        </dependency>
        <!-- Autres dépendances spécifiques au module -->
    </dependencies>
</project>
```

## Fichier settings.xml

Le fichier `settings.xml` permet de configurer Maven globalement :

- Emplacement local : `~/.m2/settings.xml`
- Emplacement global : `$MAVEN_HOME/conf/settings.xml`

Exemple de configuration :

```xml
<settings>
    <!-- Configuration du proxy -->
    <proxies>
        <proxy>
            <id>example-proxy</id>
            <active>true</active>
            <protocol>http</protocol>
            <host>proxy.example.com</host>
            <port>8080</port>
            <username>proxyuser</username>
            <password>proxypass</password>
            <nonProxyHosts>localhost|*.example.com</nonProxyHosts>
        </proxy>
    </proxies>

    <!-- Serveurs pour l'authentification -->
    <servers>
        <server>
            <id>company-repo</id>
            <username>user</username>
            <password>password</password>
        </server>
    </servers>

    <!-- Miroirs de dépôts -->
    <mirrors>
        <mirror>
            <id>nexus</id>
            <mirrorOf>central</mirrorOf>
            <url>https://nexus.example.com/repository/maven-public/</url>
        </mirror>
    </mirrors>

    <!-- Profils globaux -->
    <profiles>
        <profile>
            <id>company-defaults</id>
            <repositories>
                <repository>
                    <id>company-repo</id>
                    <url>https://repo.company.com/maven</url>
                </repository>
            </repositories>
        </profile>
    </profiles>

    <!-- Profils actifs par défaut -->
    <activeProfiles>
        <activeProfile>company-defaults</activeProfile>
    </activeProfiles>
</settings>
```
