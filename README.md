# Synthèses du Module IF (Infrastructures & Frameworks)

Ce dépôt regroupe les rapports individuels de synthèse et les analyses critiques des technologies web étudiées dans le cadre du module **Infrastructures & Frameworks (IF)** en **Master MIAGE** à l'**Université de Lille**.

Chaque synthèse évalue une technologie selon la grille critériée du module, en la comparant à notre base de référence commune (**Java / Spring Boot**).

---

## Informations

* **Module** : Infrastructures & Frameworks (IF)
* **Formation** : Master 2 MIAGE
* **Auteur** : Antonin Deloison (`antonin.deloison.etu@univ-lille.fr`)
* **Groupe** : Groupe 2

---

## Arborescence du projet

```text
IF-syntheses/
├── Ateliers/
│   ├── NestJS.md                                # Synthèse de la séance n°2 (NestJS vs Spring Boot)
│   └── Ruby on Rails.md                         # Synthèse de la séance n°1 (Ruby on Rails vs Spring Boot)
├── Comparaisons/
│   └── Spring Boot vs Rails vs NestJS.md        # Analyse comparative transversale des 3 technologies
└── README.md                                    # Présentation du dépôt, informations et structure
```

### Détail des dossiers et fichiers

* **`Ateliers/`** : Contient l'ensemble des synthèses rédigées à l'issue de chaque séance et atelier technique (Ruby on Rails, NestJS, Phoenix, Gin, Django, etc.).
  * **[`NestJS.md`](Ateliers/NestJS.md)** : Synthèse de la séance n°2 abordant l'architecture en poupées russes (Node, Express, NestJS), le modèle mono-thread asynchrone, la comparaison avec Spring Boot et Rails, la montée en charge ("Think HUGE"), les architectures DDD / Hexagonale, la sécurité des requêtes HTTP et l'impact de l'IAg.
  * **[`Ruby on Rails.md`](Ateliers/Ruby%20on%20Rails.md)** : Synthèse de la séance n°1 abordant la productivité, ActiveRecord, l'approche fullstack sans SPA (Hotwire/Turbo), les limites de typage/performance face à la JVM, l'usage de l'IA générative et le retour d'expérience du TP *Splitly*.
* **`Comparaisons/`** : Contient les analyses comparatives transversales et multidimensionnelles entre plusieurs frameworks du module.
  * **[`Spring Boot vs Rails vs NestJS.md`](Comparaisons/Spring%20Boot%20vs%20Rails%20vs%20NestJS.md)** : Comparatif complet à 3 colonnes croisant vélocité, modèle d'exécution (mono-thread vs multi-processus vs multi-thread), ORMs (ActiveRecord vs Prisma vs Hibernate), montée en charge ("Think HUGE"), impact de l'IAg et guide de choix par contexte d'entreprise.
* **`README.md`** : Ce fichier de documentation générale.