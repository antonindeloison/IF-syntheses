# Synthèse : Ruby on Rails (comparé à Spring Boot)

## 1. Ce que Rails fait mieux que Spring Boot

### 1.1. On code beaucoup moins de "plomberie"
* **En Spring Boot** : Pour juste ajouter une dépense, on doit créer 5 ou 6 fichiers : l'entité (`@Entity`), l'interface de repository, le DTO, le mapper, le service et le contrôleur. C'est lourd et répétitif.
* **En Rails** : Tout marche par convention (*Convention over Configuration*). Le framework sait tout seul quelle table correspond à quel modèle, et comment faire le lien entre un portefeuille et ses dépenses (`belongs_to :wallet`). En une seule ligne de commande, la base du code est prête et on peut attaquer directement le besoin métier.

### 1.2. L'accès à la base de données est limpide
* **En Spring Boot** : Hibernate / JPA est une usine à gaz. On tombe vite sur des erreurs pénibles : chargements tardifs qui plantent (`LazyInitializationException`), boucles infinies quand on transforme les données en JSON, ou requêtes SQL cachées qui ralentissent tout.
* **En Rails (ActiveRecord)** : C'est ultra naturel à lire et à écrire. Écrire `@wallet.expenses.order(created_at: :desc)` fait exactement ce qu'on attend. En plus, la gestion des changements de base de données (les migrations) est incluse de base, sans devoir installer d'autres outils comme Flyway ou Liquibase.

### 1.3. Pas besoin de créer un deuxième projet en React / Vue
* **En Spring Boot** : Faire des pages web avec Thymeleaf est assez pénible. Du coup, en entreprise, on fait souvent une API en Spring et un site à part en React ou Angular. Résultat : deux projets à gérer, deux déploiements, et deux fois plus de travail.
* **En Rails** : Tout est dans le même projet. Avec les vues simples (ERB) et les outils modernes de Rails (Hotwire/Turbo), on a une page web fluide et réactive sans avoir besoin de coder du JavaScript lourd.

### 1.4. La sécurité est active par défaut
* **En Spring Boot** : Mettre en place la sécurité passe par *Spring Security*, qui est réputé pour être complexe à paramétrer (chaîne de filtres, gestion des tokens, CORS).
* **En Rails** : C'est sécurisé dès le départ. La protection contre les failles courantes comme le CSRF (falsification de requête) ou le XSS (injection de scripts dans les vues) est active sans rien toucher. De plus, le système de **Strong Parameters** (`params.require(:expense).permit(...)`) oblige à lister précisément ce que l'utilisateur a le droit d'envoyer, ce qui évite qu'un utilisateur malveillant modifie des données sensibles en douce.

### 1.5. Une vraie culture du test (TDD)
* **En Spring Boot** : Lancer un test d'intégration avec `@SpringBootTest` prend facilement 15 à 30 secondes parce qu'il faut charger tout le contexte de l'application. Ça donne vite envie de sauter l'étape des tests en cours de dev.
* **En Rails** : Les fichiers de test sont créés automatiquement avec chaque modèle ou contrôleur. La commande `rails test` démarre en 2 secondes chrono. C'est beaucoup plus facile et motivant d'adopter le réflexe TDD (écrire le test qui échoue avant d'écrire le code).

---

## 2. Les avantages que garde Spring Boot sur Rails

### 2.1. Les erreurs sont vues avant de lancer le code (le typage)
* **En Spring Boot (Java)** : Si on fait une faute de frappe dans un nom de méthode ou qu'on passe un mauvais type de variable, le compilateur refuse de démarrer l'application. On sait tout de suite où est l'erreur.
* **En Rails (Ruby)** : Il n'y a pas d'étape de compilation. Si on se trompe, l'erreur n'apparaît que quand l'utilisateur clique sur le bouton en production (l'erreur classique `undefined method ... for nil`). Pour éviter ça, on est obligé d'écrire des tests automatisés sur absolument tout.

### 2.2. La vitesse brute et la charge ("Think HUGE")
* **En Spring Boot** : Java et sa machine virtuelle (la JVM) sont ultra rapides pour faire tourner de gros calculs et gérer des milliers de requêtes en même temps grâce au vrai multi-threading.
* **En Rails** : Ruby est un langage interprété plus lent et plus gourmand en mémoire vive. Pour encaisser de la grosse charge, on doit multiplier les conteneurs et bien optimiser son code pour laisser la base de données faire le travail lourd.

### 2.3. Le marché du travail
* **En Spring Boot** : C'est la techno la plus demandée dans les grandes entreprises, les banques et les ESN en France. N'importe quelle équipe trouve facilement des développeurs Java.
* **En Rails** : C'est beaucoup plus rare. C'est utilisé surtout dans des startups ou certaines boîtes tech (comme Doctolib ou GitHub), mais c'est bien plus dur de recruter des gens qui connaissent bien Ruby.

---

## 3. L'impact de l'IA générative (IAg) sur les deux technos

* **Rails avec l'IA** : Comme le code Ruby est très concis et que les conventions de Rails sont ultra standardisées, les outils d'IA (ChatGPT, Claude, etc.) sortent des bouts de code très propres et fonctionnels en très peu de lignes. Le piège principal vient des **mélanges de versions** : l'IA a tendance à ressortir de vieilles recettes de Rails 5 ou 6 (comme proposer la gem Devise ou Webpacker) alors que Rails 8 intègre désormais ses propres solutions (comme l'authentification native ou Solid Queue).
* **Spring Boot avec l'IA** : L'IA doit générer beaucoup plus de lignes de code pour faire la même chose (les DTOs, mappers, etc.). Par contre, **le compilateur Java sert de garde-fou immédiat** : si l'IA hallucine une méthode qui n'existe pas ou fait une erreur de type, le compilateur bloque tout de suite. En Rails, une hallucination de l'IA ne se voit qu'à l'exécution.

---

## 4. Ce que le TP Splitly nous a appris (mise en pratique)

Pendant le TP, on a pu voir concrètement comment Rails fonctionne sur la gestion des dépenses :

1. **Lier un modèle à une autre table sans se compliquer la vie** : Quand on a dû relier une dépense à un créateur et à un payeur, il a suffi d'écrire `belongs_to :creator, class_name: "User"`. Rails comprend immédiatement qu'il doit pointer vers la table des utilisateurs, sans avoir à créer une classe intermédiaire.
2. **Le piège du calcul en mémoire vs SQL** : Au début, on a calculé le total avec `@expenses.sum(&:amount)`. Ça fonctionne, mais Ruby doit charger toutes les dépenses une par une dans la mémoire du serveur. En basculant sur `@wallet.expenses.sum(:amount)`, Rails génère une seule requête SQL `SELECT SUM(...)` : c'est la base de données qui fait le calcul instantanément, ce qui est indispensable pour respecter la contrainte *Think HUGE*.
3. **La sécurité dans les contrôleurs** : Dans le formulaire d'ajout, on a volontairement exclu le créateur de la liste autorisée (`params.require(:expense).permit(:description, :amount, :payer_id)`). C'est le serveur qui assigne `Current.user` directement dans le contrôleur. Cela empêche n'importe qui de falsifier l'auteur de la dépense en modifiant la requête HTTP.

---

## 5. En résumé : quand choisir quoi ?

* **On choisit Ruby on Rails** quand on a une petite équipe et qu'on veut **sortir une application complète très rapidement**, sans s'embêter avec des configurations complexes ou un projet React séparé.
* **On choisit Spring Boot** quand on est dans une **grande entreprise avec beaucoup de développeurs**, où la sécurité du typage de Java évite les bêtises et où le projet doit s'intégrer dans un gros système avec plein de microservices.
