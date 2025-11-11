Benchmark de performances – Web Services REST

Projet individuel – Réalisé par Amine Bahri – Novembre 2025 – Encadré par Pr. LACHGAR

 Objectif du projet

Ce projet a pour but de comparer plusieurs implémentations REST autour d’un même domaine métier (Category 1–N Item), en utilisant une base de données PostgreSQL commune.
L’objectif est de mesurer et d’analyser l’impact du choix de la stack REST sur les performances globales à travers plusieurs indicateurs :

Latence (p50, p95, p99)

Débit maximal (requêtes par seconde)

Taux d’erreur

Consommation CPU / RAM / Garbage Collector / threads

Surcharge logicielle entre contrôleurs manuels et exposition automatique via Spring Data REST

⚙️ Variantes implémentées

Variante A : JAX-RS (Jersey) + JPA/Hibernate

Variante C : Spring Boot + @RestController + Spring MVC + JPA/Hibernate

Variante D : Spring Boot + Spring Data REST (exposition automatique des repositories en HAL)

Les trois versions partagent :

La même base de données restbench

Le même volume de données (2 000 catégories, 100 000 items)

Le même pool de connexions HikariCP (max=20, min=10)

Aucun cache activé (ni query cache, ni second-level cache, ni HTTP cache)

 Modèle de données

Deux entités principales, reliées bidirectionnellement :

Category

Item

Jeu de données :

2 000 catégories (codes de CAT00001 à CAT02000)

100 000 items (≈ 50 items par catégorie)

Payloads de test : 1 KB et 5 KB (utilisation du champ description)

Endpoints communs :

GET /categories?page=&size=
GET /categories/{id}
GET /categories/{id}/items
POST /categories
PUT /categories/{id}
DELETE /categories/{id}

GET /items?page=&size=
GET /items/{id}
GET /items?categoryId=...&page=&size=
POST /items
PUT /items/{id}
DELETE /items/{id}


La Variante D expose automatiquement les relations via HAL (_links, _embedded) sans nécessiter de contrôleur manuel.

 Environnement de test
Élément	Spécification
Machine	AMD Ryzen 5 5600X (6 cœurs / 12 threads), 16 Go DDR4
Système	Windows 11 Pro 64 bits
Java	17 (Eclipse Temurin)
Base de données	PostgreSQL 16.4-alpine
Containers	Docker Desktop 4.35
Monitoring	Prometheus + Grafana + InfluxDB 2
Outil de charge	JMeter 5.6.3 + Backend Listener InfluxDB
JVM	-Xms512m -Xmx2g -XX:+UseG1GC
Pool HikariCP	max=20, min=10, timeout=30s
📈 Scénarios JMeter exécutés

READ-heavy (avec relations) → 50 → 200 threads

JOIN-filter ciblé (70 % de requêtes items?categoryId=…) → 60 → 120 threads

MIXED (lectures + écritures simultanées, payload 1 KB) → 50 → 100 threads

HEAVY-body (requêtes avec payload 5 KB) → 30 → 60 threads

Résultats obtenus (Novembre 2025)

La Variante D – Spring Data REST s’est révélée la plus performante sur l’ensemble des tests :

Critère	Résultat
Débit global	+3 % par rapport à C, +7 % par rapport à A
Latence p95	−6 % vs C, −13 % vs A
Consommation CPU moyenne	28 %
Heap moyen	720 Mo
Taux d’erreur	0 % sur tous les scénarios
Productivité	Aucune ligne de code supplémentaire pour les relations HAL
Structure du dépôt

variant-a-jersey/ → implémentation JAX-RS

variant-c-restcontroller/ → implémentation Spring MVC

variant-d-data-rest/ → implémentation Spring Data REST

data/ → fichiers CSV (2 000 catégories, 100 000 items, payloads 1 KB / 5 KB)

jmeter/ → fichiers .jmx prêts à l’emploi

grafana/ → tableaux de bord JVM et JMeter

docker-compose.yml → base PostgreSQL + monitoring

init.sql → création du schéma et des index

Rapport_Performances_REST_LACHGAR.pdf → rapport de synthèse (7 pages, tableaux T0 → T7)

Conclusion

La solution Spring Data REST (Variante D) s’impose comme la plus efficace et la plus légère.
Elle offre à la fois de meilleures performances brutes, une consommation mémoire réduite, et une simplicité de développement remarquable grâce à l’exposition automatique du modèle HAL.

C’est une approche particulièrement adaptée lorsque productivité, maintenabilité et performance doivent coexister harmonieusement.
