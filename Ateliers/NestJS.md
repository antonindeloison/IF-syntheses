# Synthèse : NestJS (comparé à Spring Boot)

## 1. Qu'est-ce que NestJS et comment ça tourne ?

### 1.1. C'est quoi exactement ? (Le principe des poupées russes)
* **Un framework 100 % Back-end** : NestJS sert uniquement à fabriquer des API (REST, GraphQL, WebSockets) en TypeScript. Il ne faut pas le confondre avec **Next.js**, qui sert à faire du Front-end en React.
* **Comment c'est empilé sous le capot** :
  1. **Node.js** : C'est le moteur qui exécute le JavaScript/TypeScript sur le serveur.
  2. **Express.js** : C'est le serveur HTTP de base qui reçoit les requêtes brutes.
  3. **NestJS** : C'est la couche posée par-dessus pour forcer à ranger son code proprement avec des Modules, des Contrôleurs, des Services et des DTOs, exactement comme en Angular ou en Spring.

### 1.2. Le modèle « un seul fil » (Mono-thread asynchrone)
Contrairement à Spring Boot qui crée un fil d'exécution (*thread*) par utilisateur, NestJS tourne sur un **seul fil principal** (l'Event Loop) :
* **En asynchrone (`async/await`)** : Quand une requête attend une réponse de PostgreSQL ou de Prisma, le fil ne reste pas bloqué les bras croisés. Il va immédiatement traiter les requêtes d'autres utilisateurs (c'est ce qui permet d'encaisser facilement des gros volumes d'appels I/O, comme 200 requêtes/seconde chez un drive Intermarché).
* **Le gros danger du synchrone** : Si on lance un gros calcul lourd ou une boucle bloquante directement sur ce fil, **tout le serveur gèle** et plus aucun utilisateur ne reçoit de réponse.

---

## 2. NestJS vs Spring Boot

| Critère | NestJS (TypeScript) | Spring Boot (Java) |
| :--- | :--- | :--- |
| **Structure du code** | Modules, Contrôleurs, Services, DTOs (décorateurs `@`) | Packages, Contrôleurs, Services, DTOs (annotations `@`) |
| **Façon de tourner** | Mono-thread asynchrone (boucle d'événements) | Multi-thread (un fil d'exécution par requête) |
| **Gros point fort** | Rapide à dev, léger, 100 % TypeScript (on partage les types avec le Front) | Puissance de calcul brute sur plusieurs cœurs (CPU), stabilité industrielle |
| **Gros point faible** | Goulot d'étranglement dès qu'il y a du calcul CPU lourd | Lourd au démarrage, beaucoup de fichiers et de boilerplate |
| **Terrain de jeu idéal** | Startups, applis mobiles et web temps réel qui font surtout transiter de la donnée | Banques, assurances, gros calculs financiers et systèmes critiques |

### 2.1. Ce que NestJS apporte par rapport à Spring Boot
* **On utilise le même langage partout** : Une équipe peut faire du TypeScript sur le Front (React/Vue) et sur le Back (NestJS). On peut même partager les interfaces de données entre les deux sans devoir tout réécrire.
* **Moins d'attente au quotidien** : Lancer l'appli ou exécuter un test prend quelques secondes, là où compiler et démarrer un gros projet Spring Boot avec tous ses beans prend vite 30 à 45 secondes.

### 2.2. Ce que Spring Boot garde comme avantages
* **Le calcul lourd en parallèle** : Quand il faut faire tourner des algorithmes gourmands en calcul, la JVM exploite naturellement tous les cœurs du processeur sans risque de bloquer les autres requêtes.
* **Un écosystème où tout existe déjà** : En Spring, la sécurité, les batches ou la gestion des microservices sont intégrés depuis plus de 15 ans. Avec NestJS, on dépend souvent de packages npm tiers maintenus par la communauté.

---

## 3. Montée en charge (« Think HUGE ») et architecture

### 3.1. Le piège du fil unique
Démarrer un projet avec NestJS va très vite. Mais si l'application devient énorme (par exemple *« Les Bons Comptes »* avec 1 million d'utilisateurs et des groupes géants), faire tous les calculs d'équilibrage de dettes sur le fil unique de Node.js va finir par saturer le serveur.

### 3.2. Comment tenir la charge en pratique ?
* **Multiplier les instances (Scaling horizontal)** : Comme un processus Node n'utilise qu'un seul cœur de processeur, on lance plusieurs conteneurs Docker de NestJS en parallèle derrière un Load Balancer pour utiliser toute la machine.
* **Mettre du cache (Redis)** : Pour ne pas recalculer l'état des comptes à chaque fois qu'un utilisateur rafraîchit sa page.
* **Déporter les gros calculs (BullMQ)** : Envoyer les calculs lourds dans une file d'attente en tâche de fond, ou même confier ce calcul précis à un petit microservice dédié en Go ou en Java.

### 3.3. Éviter la dette technique : DDD et Hexagonale
* **Découper par métier (DDD)** : Ranger le code dans des modules NestJS bien séparés (`expenses/`, `wallets/`, `auth/`) plutôt que de mélanger toutes les tables ensemble.
* **Isoler le métier (Architecture Hexagonale)** : Écrire l'algorithme qui calcule qui doit combien à qui en **TypeScript pur**, sans toucher à NestJS ni à Prisma. Si un jour le calcul devient trop lourd pour Node.js, on peut l'extraire et le réécrire dans un autre langage sans devoir jeter tout le reste du projet.

---

## 4. Ce que la séance sur « Les Bons Comptes » nous a montré

La présentation et la démo en live ont mis en avant plusieurs réflexes très concrets :

### 4.1. Le parcours propre d'une requête HTTP
Pour ajouter une dépense (`POST /wallets/:id/expenses`), la requête passe par des étapes bien découpées :
1. **Le Guard JWT** (*Qui es-tu ?*) : Vérifie le jeton. Pas de jeton valide = direct une erreur **401 Unauthorized**.
2. **Le ValidationPipe + DTO** (*Données valides ?*) : Vérifie que le montant est positif et que le titre n'est pas vide via les décorateurs. Données foireuses = direct une erreur **400 Bad Request** avant même d'arriver au contrôleur.
3. **Le Contrôleur** : Récupère les paramètres et passe la main au service.
4. **Le Service métier** (*As-tu le droit ?*) : Vérifie que l'utilisateur est bien membre du groupe et que le portefeuille n'est pas fermé. Si le portefeuille est clôturé, il renvoie une erreur **409 Conflict** (le vrai code HTTP pour un conflit d'état, pas un 400 ni un 403).
5. **Prisma** : Enregistre la dépense en base dans PostgreSQL.

### 4.2. Les bonnes pratiques vues en démo
* **Les centimes entiers plutôt que les décimaux** : Pour éviter les bugs d'arrondi avec les nombres à virgule (les `float` en informatique), tous les montants sont stockés en centimes entiers (ex. 20,50 € devient `2050`).
* **La gestion des tokens** : L'access token (court) reste en mémoire, et le refresh token (long) est placé dans un cookie `HttpOnly`. Le JavaScript du navigateur ne peut pas le lire, ce qui protège contre le vol de session.
* **La double clôture évitée** : Utiliser des transactions SQL atomiques avec Prisma pour qu'on ne puisse jamais clôturer deux fois le même portefeuille en même temps.

---

## 5. L'impact de l'IA générative (IAg) sur les deux technos

* **Sur NestJS** : L'IA s'en sort très bien car le code est découpé de manière très prévisible (un fichier pour le module, un pour le service, un pour le contrôleur, un pour le DTO). Le gros point fort par rapport à du JavaScript pur ou Ruby, c'est que **TypeScript sert de premier compilateur** : si l'IA invente une méthode ou se plante sur un type, l'erreur est soulignée en rouge tout de suite dans l'éditeur.
* **Face à Spring Boot** : Avec Spring, l'IA doit générer des tonnes de boilerplate (DTO, mappers, interfaces de repository). Avec NestJS, l'IA génère les décorateurs de validation (`class-validator`) et les schémas Prisma beaucoup plus vite et avec moins de lignes.

---

## 6. En résumé : quand choisir quoi ?

* **On part sur NestJS** quand on veut développer vite une **API moderne et bien rangée**, qu'on a une équipe déjà à l'aise avec TypeScript/JavaScript, et que l'application fait surtout transiter des données (applis web, mobiles, temps réel) sans faire fondre le processeur avec des calculs mathématiques lourds.
* **On reste sur Spring Boot** quand on bosse dans un **grand groupe (banque, assurance)** avec des dizaines de développeurs, qu'on a besoin d'une solidité éprouvée depuis 15 ans et de la puissance du vrai multi-threading pour des calculs critiques en parallèle.
