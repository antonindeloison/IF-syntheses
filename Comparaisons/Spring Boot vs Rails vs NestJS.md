# Comparatif transversal : Spring Boot vs Ruby on Rails vs NestJS

Auteur : Antonin DELOISON
Groupe 2

Ce document met en perspective les trois technologies étudiées au début du module **Infrastructures & Frameworks (IF)** :
* **Java / Spring Boot** : Notre socle commun et base de référence d'entreprise.
* **Ruby on Rails** : Le framework Fullstack orienté productivité maximale (Séance n°1).
* **NestJS** : L'architecture Back-end modulaire en TypeScript (Séance n°2).

---

## 1. Tableau comparatif synthétique

| Dimension | Ruby on Rails | NestJS | Spring Boot |
| :--- | :--- | :--- | :--- |
| **Langage sous-jacent** | **Ruby** | **TypeScript** (sur Node.js) | **Java** (sur JVM) |
| **Spécificités du langage** | Dynamique fort, orienté objet pur, métaprogrammation et *duck typing* | Typage statique structurel sur JS, transpilé, asynchronisme natif (`async/await`) | Typage statique strict et nominal, orienté objet strict, compilé en bytecode |
| **Place de l'Injection de Dépendances (DI)** | **Quasiment absente** (inutile grâce à la métaprogrammation du langage) | **Centrale (`@Injectable`)** (indispensable pour structurer Node et tester avec des mocks) | **Vitale et historique (`@Autowired`, constructeur)** (fondement absolu de l'architecture) |
| **Avantage clé du langage** | **Concision et expressivité** (le code se lit comme de l'anglais, zéro boilerplate) | **Unification Front + Back** (même langage partout, types partagés avec React/Vue) | **Robustesse industrielle & CPU** (la JVM excelle en calcul, le compilateur interdit l'erreur) |
| **Périmètre applicatif** | **Fullstack tout-en-un** (Back + Vues avec Hotwire/ERB) | **Back-end pur (API)** (nécessite un Front React/Vue séparé) | **Back-end pur** (APIs REST/GraphQL ; Thymeleaf peu utilisé) |
| **Philosophie** | *Convention over Configuration* (magie, concision, peu de code) | Architecture modulaire explicite inspirée d'Angular et Spring | Architecture en couches rigide (séparation stricte par packages) |
| **Modèle d'exécution** | Multi-processus / Threads avec verrou global (GVL) | **Mono-thread asynchrone** (boucle d'événements / Event Loop) | **Multi-thread préemptif** (un fil d'exécution par requête) |
| **Accès aux données (ORM)** | **ActiveRecord** (l'objet métier représente directement la ligne SQL) | **Prisma** (schéma déclaratif `schema.prisma` et client 100 % typé) | **Hibernate / JPA** (Data Mapper lourd, entités et repositories) |
| **Gestion des erreurs** | Au *runtime* (nécessite une couverture de tests irréprochable) | À la sauvegarde / compilation (TypeScript détecte les incohérences) | À la compilation (le compilateur Java interdit l'exécution) |
| **Vitesse de démarrage / dev** | Rapide (rechargement à chaud automatique) | Ultra-rapide (quelques secondes avec Node.js / ts-node) | Lente (chargement complet du conteneur d'inversion de contrôle) |
| **Calculs lourds (CPU)** | Moyen (consommation mémoire élevée par conteneur) | Faible (le fil unique gèle si on lui confie de gros calculs bloquants) | **Excellente** (la JVM tire parti de tous les cœurs du processeur) |
| **Contrainte « Think HUGE »** | Scaling horizontal de conteneurs + calculs délégués au SQL | Scaling horizontal + mise en cache Redis + workers asynchrones (BullMQ) | Scaling vertical (multi-threading natif) et horizontal (microservices) |
| **Terrain de jeu privilégié** | Startups, SaaS, MVPs rapides livrés par de petites équipes | Applications web/mobiles temps réel avec Front TypeScript | Systèmes d'entreprise critiques (banques, assurances, logistique) |

---

## 2. Vélocité et architecture : Monolithe intégré vs API découplée

### Ruby on Rails : Le champion du Time-to-Market
* **Un seul projet pour tout faire** : Rails permet de développer une application interactive complète sans écrire une seule ligne de JavaScript lourd grâce à **Hotwire (Turbo + Stimulus)**. On ne gère ni les CORS, ni les tokens d'API pour le frontend, ni deux pipelines CI/CD séparés.
* **Le revers de la médaille** : L'absence de typage et la magie des conventions peuvent dérouter lors de gros refactorings si l'équipe manque de discipline.

### NestJS : L'unification autour de TypeScript
* **Le compromis moderne** : Contrairement à Rails, NestJS ne gère pas le Front-end. Il oblige à développer une application cliente séparée (React, Vue, mobile).
* **Le gros avantage** : Toute l'équipe parle le même langage (**TypeScript**). On peut partager les DTOs et les types d'interfaces directement entre le Front et le Back, évitant ainsi les désynchronisations de modèles de données.

### Spring Boot : L'usine d'ingénierie d'entreprise
* **La lourdeur initiale** : Pour un simple ajout de dépense, Spring Boot exige de créer une `@Entity`, un `JpaRepository`, un `DTO`, un `Mapper`, un `Service` et un `Controller`. 
* **L'avantage à long terme** : Cette séparation stricte par couches facilite la maintenance sur des projets gérés par des dizaines de développeurs avec un fort turnover.

---

## 3. Spécificités des langages et la question de l'Injection de Dépendances (DI)

L'injection de dépendances (DI) est souvent présentée comme une obligation d'architecture. En réalité, son utilité dépend directement de la nature du langage de programmation.

### 3.1. À quoi sert vraiment la DI ?
Le principe de la DI est simple : **on n'écrit jamais `new` dans le code métier**. Au lieu qu'un service instancie lui-même ses dépendances (`new PrismaService()`), le framework crée l'objet et le lui passe par le constructeur.
* **Force n°1 : La testabilité** : C'est l'argument majeur. Si un service crée lui-même sa connexion à PostgreSQL, on ne peut pas le tester sans monter une vraie base de données. Avec la DI, on lui injecte un faux service en mémoire (un *mock*), et le test s'exécute en 5 millisecondes.
* **Force n°2 : La gestion des Singletons** : Si 50 contrôleurs ont besoin d'accéder à la base, le framework injecte la même instance unique partagée au lieu d'ouvrir 50 connexions réseau concurrentes.

### 3.2. Pourquoi la DI est vitale en Java, utile en TypeScript, mais inutile en Ruby ?
* **En Java (Spring Boot) : Vital**
  * Java est un langage compilé, statique et nominal très strict. Sans interfaces et sans conteneur d'inversion de contrôle (IoC), il est techniquement impossible de substituer une classe par un mock au moment des tests. La DI est le poumon de Spring depuis son premier jour.
* **En TypeScript (NestJS) : Très utile pour cadrer Node.js**
  * Dans le monde JavaScript/Node classique (comme sur Express), l'absence de cadre mène souvent à du code anarchique avec des `import` sauvages de fichiers un peu partout. NestJS utilise la DI (`@Injectable()`) pour imposer la rigueur et la modularité des architectures d'entreprise au monde JavaScript.
* **En Ruby (Rails) : Inutile et absent**
  * Rails n'utilise quasiment jamais d'injection de dépendances. Pourquoi ? Parce que Ruby est un langage dynamique avec du **duck typing** et de la **métaprogrammation**. En Ruby, pendant un test unitaire, on peut intercepter et remplacer n'importe quelle méthode à chaud à la volée (`allow(Expense).to receive(:sum)...`). Les classes s'appellent directement (`Wallet.find(...)`) sans avoir besoin de passer par un conteneur IoC lourd.

### 3.3. Quel langage a l'avantage ?
* **Avantage TypeScript** pour **l'écosystème web moderne** : C'est le seul langage capable de faire tourner le Front-end (dans le navigateur) et le Back-end (sur Node), tout en apportant la sécurité du typage statique sans la lourdeur de Java.
* **Avantage Java** pour **la puissance et la stabilité** : La machine virtuelle (JVM) reste intouchable sur le calcul lourd en parallèle, et le compilateur Java est le meilleur allié pour maintenir un code critique pendant 15 ans.
* **Avantage Ruby** pour **l'expressivité et le plaisir de dev (*Developer Happiness*)** : Le code s'écrit deux fois plus vite, se lit comme de l'anglais et élimine toute la plomberie technique.

---

## 4. Modèle d'exécution et concurrence (« Think HUGE »)

La façon dont ces trois moteurs traitent la montée en charge illustre trois visions différentes du Web :

```text
[ Spring Boot : Multi-thread JVM ]
Requête 1 ────► Thread 1 (CPU / BDD) ──────┐
Requête 2 ────► Thread 2 (CPU / BDD) ──────┤──► Utilise tous les cœurs CPU
Requête 3 ────► Thread 3 (CPU / BDD) ──────┘

[ NestJS : Mono-thread asynchrone ]
Requêtes ───► [ Fil unique Event Loop ] ──┬──► Délègue l'attente BDD (async)
(I/O fluides, mais si calcul lourd ───────┘     => Gèle le serveur si bloquant !)

[ Ruby on Rails : Multi-processus ]
Requête 1 ──► Instance 1 (Processus Puma) ──┐
Requête 2 ──► Instance 2 (Processus Puma) ──┴──► Consomme plus de mémoire RAM
```

* **NestJS (Event Loop)** : C'est le roi des entrées/sorties (I/O). Tant qu'on fait de l'asynchrone (`async/await`) en attendant la base de données, un seul fil d'exécution peut traiter des milliers de requêtes par seconde avec une consommation de mémoire minimale. En revanche, pour la contrainte *Think HUGE*, tout calcul d'algorithme lourd (ex: calcul d'équilibrage des dettes) doit être déporté dans des files de tâches asynchrones (BullMQ) sous peine de geler l'application.
* **Spring Boot (Multi-thread)** : La JVM alloue un thread par requête et optimise le calcul brut en parallèle. Elle ne craint pas les gros calculs mathématiques synchrones.
* **Rails (Multi-processus)** : Pour monter en charge, Rails scale horizontalement en lançant plusieurs processus en parallèle. Il oblige le développeur à optimiser ses requêtes SQL (ex. calculer le total avec `sum(:amount)` en base au lieu de charger les données en mémoire vive).

---

## 5. Accès aux données et persistance : Trois approches de l'ORM

1. **ActiveRecord (Rails)** :
   * La table et l'objet métier ne font qu'un. La syntaxe est la plus lisible et intuitive du marché (`@wallet.expenses.sum(:amount)`).
   * Les migrations de schéma sont gérées nativement en Ruby (`db/migrate`), sans besoin d'outil externe.
2. **Prisma (NestJS)** :
   * Le schéma unique `schema.prisma` sert de contrat de vérité et génère un client TypeScript entièrement typé.
   * Si on modifie le nom d'une colonne, le compilateur TypeScript indique immédiatement toutes les erreurs dans l'application.
   * Les transactions conditionnelles (`$transaction`) permettent de gérer la concurrence facilement (comme éviter de clôturer un portefeuille deux fois).
3. **Hibernate / Spring Data JPA (Spring Boot)** :
   * Implémentation du pattern *Data Mapper*. C'est le plus rigide et le plus complexe à maîtriser (gestion des sessions, lazy loading qui plante à l'affichage, requêtes N+1 cachées).
   * En revanche, il offre un niveau de contrôle très fin sur les caches de niveau 1 et 2 pour les bases de données d'entreprise massives.

---

## 6. Typage et Sécurité : Prévention vs Détection

* **La détection à la compilation (Spring Boot & NestJS)** :
  * Java et TypeScript vérifient les types avant l'exécution.
  * Dans NestJS, la combinaison des **DTOs** et du **`ValidationPipe`** intercepte les requêtes invalides dès l'entrée (erreur `400 Bad Request`) sans polluer la logique métier du service.
* **La sécurité par défaut (Ruby on Rails)** :
  * Même sans compilation, Rails est « sécurisé par défaut » : protection CSRF automatique, échappement HTML contre les failles XSS, et **Strong Parameters** obligatoires (`params.require(...).permit(...)`) pour interdire l'injection de champs sensibles (comme falsifier l'auteur d'une dépense).

---

## 7. L'impact de l'IA générative (IAg) sur les trois technologies

| Technologie | Comportement avec les LLMs (ChatGPT / Claude) | Rôle du compilateur face aux hallucinations |
| :--- | :--- | :--- |
| **Ruby on Rails** | Code très court et propre grâce aux conventions strictes. | **Aucun filet** : si l'IA hallucine une méthode, l'erreur n'apparaît qu'au clic utilisateur en production. Piège classique : elle ressort souvent du code de vieilles versions (Rails 5 ou 6). |
| **NestJS** | Génération très rapide de toute la structure (modules, contrôleurs, décorateurs DTO, schéma Prisma). | **Filet immédiat** : TypeScript souligne en rouge la moindre propriété ou méthode inventée par l'IA dès l'enregistrement du fichier. |
| **Spring Boot** | L'IA doit générer des volumes de code très importants (classes, interfaces, configurations, mappers). | **Filet absolu** : le compilateur Java refuse strictement tout code erroné ou type incompatible généré par l'IA. |

---

## 8. Guide de décision : Quelle technologie pour quel projet ?

* **Choisir Ruby on Rails quand** :
  * L'objectif numéro un est le **Time-to-Market** : sortir un MVP ou un produit complet (front + back) très rapidement.
  * L'équipe est réduite et veut éviter la double gestion d'un projet API et d'une application React séparée.
  * Le projet est une application de gestion métier classique (CRUD, formulaires, dashboards).
* **Choisir NestJS quand** :
  * Le système doit exposer une **API moderne, modulaire et hautement typée** pour alimenter plusieurs clients (application mobile Flutter/React Native, interface web Next.js/React).
  * L'équipe souhaite capitaliser sur une compétence unique : **TypeScript de bout en bout**.
  * L'application gère beaucoup de trafic temps réel (WebSockets) ou des flux d'entrées/sorties sans calculs CPU bloquants.
* **Choisir Spring Boot quand** :
  * Le projet s'intègre dans le **Système d'Information d'une grande entreprise** (banque, assurance, secteur public) avec de fortes contraintes de gouvernance.
  * L'application doit exécuter des **calculs intensifs en parallèle** sur de gros volumes de données.
  * La pérennité à 10 ou 15 ans et la facilité de recrutement en ESN sont les critères prioritaires pour la DSI.
