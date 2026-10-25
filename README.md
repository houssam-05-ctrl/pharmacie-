# Pharmacie — Mini-projet Spring Boot (TP1 + TP2)

Application de gestion de stock de pharmacie développée avec **Spring Boot 3**, **MySQL**, **Thymeleaf**, **JPA/Hibernate** et **Spring Security**, dans le cadre des TP Module "Spring Boot - MicroServices" (Prof. Youness IDRISSI KHAMLICHI, ENSA Fès).

Ce projet réunit les deux travaux pratiques :

* **TP1 — Gestion de stock de pharmacie (MySQL)** : entités JPA avec relations (`@OneToOne`, `@ManyToOne`, `@ManyToMany`), repositories `JpaRepository`, contrôleurs MVC (Thymeleaf) et contrôleurs REST (`/api/*`), tests Postman.
* **TP2 — Sécurisation (Spring Security)** : authentification et autorisation par rôle (`ADMIN`, `PHARMACIEN`, `CAISSIER`, `CLIENT`) avec pages dédiées (`login`, `home`, `admin`, `medicaments`, `access-denied`).

---

## Prérequis

* Java 17+
* Apache Maven 3.6+
* MySQL 8 (serveur local sur `localhost:3306`)

## Base de données

1. Démarrer MySQL.
2. Créer la base (ou laisser l'application la créer) :

```sql
CREATE DATABASE IF NOT EXISTS pharmacie_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

3. Adapter les identifiants dans `src/main/resources/application.properties` si nécessaire :

```properties
spring.datasource.username=root
spring.datasource.password=VOTRE_MOT_DE_PASSE
```

L'application démarre avec `spring.jpa.hibernate.ddl-auto=update` : les tables sont créées automatiquement. Le jeu de données TP1 (catégories, fournisseurs, détails produits, tags, produits) et les comptes de test TP2 sont insérés au démarrage par `InitData`.

## Lancement

```bash
mvn spring-boot:run
```

Puis ouvrir : http://localhost:8080

## Structure du projet

```
pharmacie/
├── pom.xml
├── src/main/java/com/pharmacie/
│   ├── PharmacieApplication.java
│   ├── config/            SecurityConfig (Spring Security)
│   ├── controller/        Contrôleurs MVC (LoginController, PharmacieController,
│   │                      MedicamentController, AdminController, CaisseController, ClientController)
│   ├── controller/rest/   Contrôleurs REST (Produit, Categorie, Fournisseur, Tag)
│   ├── model/             Entités JPA (Produit, Categorie, Fournisseur, DetailProduit, Tag, User, Role, ERole)
│   ├── repository/        Repositories JpaRepository
│   └── service/           CustomUserDetailsService, InitData (semencement des données)
└── src/main/resources/
    ├── application.properties
    ├── static/css/style.css
    └── templates/
        ├── login.html
        ├── home.html
        ├── medicaments.html
        ├── produit-form.html
        ├── produit-detail.html
        ├── admin.html
        ├── caisse.html
        ├── client.html
        └── access-denied.html
```

## Comptes de test (TP2)

| Utilisateur | Mot de passe | Rôle | Droits |
|-------------|--------------|------|--------|
| admin | admin123 | ADMIN | Tout (utilisateurs, médicaments, stocks) |
| pharma | pharma123 | PHARMACIEN | Consulter et modifier les stocks, ventes |
| caisse | caisse123 | CAISSIER | Enregistrer les ventes uniquement |
| client | client123 | CLIENT | Consultation lecture seule des médicaments |

## Règles d'accès (Spring Security)

* `/` et `/home` : publics
* `/login` : public
* `/admin/**` : `ROLE_ADMIN` uniquement (gestion des utilisateurs)
* `/medicaments/**` : `ADMIN` et `PHARMACIEN` (gestion des stocks)
* `/caisse/**` : `ADMIN`, `PHARMACIEN`, `CAISSIER`
* `/client/**` : tous les rôles connectés
* API REST `GET /api/**` : accessible à tous (consultation)
* API REST `POST/PUT/DELETE /api/**` : `ADMIN` uniquement

## Endpoints REST (tests Postman — TP1)

```
GET    /api/produits                Lister les produits
GET    /api/produits/{id}           Détail d'un produit
POST   /api/produits                Ajouter un produit
PUT    /api/produits/{id}           Modifier un produit
DELETE /api/produits/{id}           Supprimer un produit

GET    /api/categories              Liste des catégories
GET    /api/categories/{id}
POST   /api/categories
PUT    /api/categories/{id}
DELETE /api/categories/{id}

GET    /api/fournisseurs            Liste des fournisseurs
... (mêmes verbes que ci-dessus)

GET    /api/tags                    Liste des tags
... (mêmes verbes que ci-dessus)
```

Exemple de corps JSON `POST /api/produits` :

```json
{
  "nom": "Voltaren 50mg",
  "description": "Anti-inflammatoire pour douleurs musculaires et articulaires.",
  "prix": 58.90,
  "codeBarre": "VOLTA50-2025",
  "dateExpiration": "2026-12-15",
  "categorie": { "id": 5 },
  "fournisseur": { "id": 5 },
  "detailProduit": {
    "fabricant": "Novartis",
    "paysOrigine": "Suisse",
    "composition": "Diclofénac sodique 50 mg",
    "dosage": "50 mg",
    "indications": "Traitement des douleurs musculaires"
  },
  "tags": [ { "id": 1 }, { "id": 3 } ]
}
```

## Jeu de données (TP1 — inséré automatiquement)

* **Catégories** : Antibiotiques, Antalgiques, Vitamines, Antiseptiques
* **Fournisseurs** : PharmaMaroc, Medisup France, Sandoz, Cooper Pharma
* **Tags** : Sans ordonnance, Prescription obligatoire, En promotion, Stock limité, Importé
* **Produits** : Amoxicilline 500mg, Doliprane 500mg, Vitamine C 1000mg, Bétadine Solution 10%, Nurofen 400mg

## Notes

Cet environnement de sandbox ne disposait d'aucun JDK/Maven, le projet n'a donc pas pu être compilé localement ; il est conçu pour être ouvert et exécuté en local (Java 17 + Maven + MySQL).
