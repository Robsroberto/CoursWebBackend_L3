---
marp: true
size: 4:3
style: |
    h3, p {
    font-size: 20px;
    }
    li {
    font-size:20px
    }
headingDivider: 1
header: 
paginate: 
footer: Licence 3 Informatique &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ISI (Institut Supérieur D'Informatique) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RD
---
# Cours 1 — Introduction aux Technologies Backend

<img src="../isi.png" alt="ISI" width="100px">

## Par Robert DIASSÉ &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
 <img src="java.webp" alt="ISI" width="50px">

---


## 1. Introduction générale

### Qu’est-ce qu’une application web ?

Une application web moderne est généralement composée de deux grandes parties :

1. **Front-end**

   * Ce que l’utilisateur voit et manipule
   * Interfaces, boutons, formulaires, animations
   * Technologies : HTML, CSS, JavaScript, Frameworks (React, Vue, Angular)

2. **Back-end**

   * Ce qui fonctionne “derrière” l’interface
   * Traitements, logique métier, gestion des utilisateurs, enregistrement des données
   * Technologies : PHP, Python, Java, JavaScript côté serveur, etc.
---

**But du backend** :
→ **Recevoir des requêtes**
→ **Traiter ces requêtes**
→ **Accéder aux données**
→ **Renvoyer des réponses structurées**


## 2. À quoi sert le backend ?

Le backend permet à une application d’être :

* dynamique
* sécurisée
* persistante (les données restent même après fermeture du navigateur)
* fiable (règles métier appliquées correctement)
* multi-utilisateurs (espaces personnels, accès différents)
---

### Exemple concret

Sur un site de réservation :

* le front affiche le formulaire
* le backend vérifie si :

  * l’heure est disponible
  * le client est authentifié
  * le paiement est validé
  * la réservation peut être enregistrée
* le backend enregistre dans la base de données
* le backend renvoie une confirmation au front


## 3. Qu’est-ce qu’une API ?

**API = Application Programming Interface**

Une API est **un ensemble de règles** permettant à une application de communiquer avec une autre.

---

### Exemple 

Quand tu vas sur :

```
https://example.com/api/users
```

Tu demandes à l’application :

→ **Donne-moi la liste des utilisateurs**.

L’API répond généralement en **JSON** :

```json
[
  { "id": 1, "nom": "Diop", "email": "diop@example.com" },
  { "id": 2, "nom": "Fall", "email": "fall@example.com" }
]
```

---

## 4. Types d’API (sans entrer encore dans les frameworks)

Il existe plusieurs familles d’API :

## 4.1 API REST

### Définition  :

Une API REST expose des **ressources** accessibles via des **URL** et manipulées via les **méthodes HTTP**.

Exemple :

* GET `/api/services`
* POST `/api/users`
* DELETE `/api/reservations/10`

On y reviendra plus en détail dans le Cours 2.

---

## 4.2 API SOAP

* Format XML
* Plus ancienne
* Très utilisée dans les banques, ERP, systèmes lourds
* Plus stricte, plus verbeuse

## 4.3 API GraphQL

* Une alternative moderne à REST
* Le client peut demander précisément les données dont il a besoin
* Utile pour des applications front très dynamiques
* De plus en plus utilisé dans les startups
---

## 4.4 API WebSocket

* Communication en temps réel
* Pour du chat, des notifications, des dashboards en direct

**Conclusion pour eux :**
Dans ce cours, **nous utiliserons principalement API REST**, car c’est :

* le plus simple à apprendre
* le plus utilisé dans les entreprises
* idéal pour le backend web classique

---
## 5. Ressources, Endpoints et Méthodes HTTP

### 5.1 Ressource

Une ressource représente un élément manipulable :

* User
* Produit
* Article
* Réservation

### 5.2 Endpoint

Une URL qui donne accès à une ressource.

Exemple :
`/api/services`
`/api/reservations/12`

---
### 5.3 Méthodes HTTP

Chaque méthode a une signification précise :

| Méthode | Usage                              |
| ------- | ---------------------------------- |
| GET     | Lire une ressource                 |
| POST    | Créer une ressource                |
| PUT     | Remplacer totalement une ressource |
| PATCH   | Modifier partiellement             |
| DELETE  | Supprimer                          |

---

## 6. Comment fonctionne une requête HTTP ?

Une requête contient généralement :

* une **méthode** : GET, POST, PUT…
* une **URL** : `/api/users/3`
* des **headers** : métadonnées (type, token, etc.)
* un **body** : données envoyées (JSON)

Une **réponse** contient :

* un **status code** (200, 201, 400, 404, 500…)
* un **body** JSON
* des headers

### Exemple 

---
**Requête :**

```
POST /api/users
Content-Type: application/json

{
  "nom": "Diop",
  "email": "diop@example.com"
}
```

**Réponse :**

```
201 Created
{
  "id": 5,
  "nom": "Diop",
  "email": "diop@example.com"
}
```

---

## 7. Architecture Backend moderne : les couches essentielles

Voici une architecture universelle que l’on retrouve dans **tous les frameworks backend** :

### **1. Routes**

* Définissent quelle URL déclenche quelle action.

### **2. Controllers**

* Reçoivent la requête.
* Appellent les services.
* Renvoient la réponse.

### **3. Services / Logique métier**

---
* Règles de l’application :

  * puits de disponibilité
  * prix minimum
  * droits utilisateurs
  * calculs divers

### **4. Models / ORM**

* Interface entre le code et la base de données.

### **5. Base de données**

* PostgreSQL, MySQL, SQLite, etc.

### Exemple  de flux

1. Le front envoie : **POST /api/reservations**
2. La route détecte l’URL
3. Le controller appelle le service “Créer une réservation”

---
4. Le service vérifie les règles
5. Le modèle sauvegarde dans la base
6. Le controller renvoie la réponse


## 8. Les bases de données

Nous distinguons deux grandes familles :

## 8.1 Bases de données SQL

* Données structurées
* Tables, colonnes, relations
* Moteurs : PostgreSQL, MySQL, MariaDB, SQL Server
* Très utilisées dans l’administration, les entreprises, les startups

---
## 8.2 Bases NoSQL

* Données non structurées ou semi-structurées
* Documents, collections
* Exemples : MongoDB, Firebase
* Très utilisées dans les applications temps réel ou mobiles



## 9. Qu’est-ce qu’un ORM ?

**ORM = Object Relational Mapping**
→ C’est un outil qui permet de travailler avec la base de données **en utilisant des objets** plutôt que du SQL brut.

### Exemple (conceptuel)

Sans ORM :

```sql
SELECT * FROM users WHERE id = 5;
```
---
Avec ORM :

```php
User::find(5);     // Laravel
```

ou

```python
User.objects.get(id=5)    // Django
```

L’ORM traduit le code en SQL pour nous.


## 10. Authentification et Autorisation

### Authentification

→ Vérifie l’identité (connexion, mot de passe, token)

### Autorisation

→ Vérifie les permissions (admin / user normal)

---
Exemples de contrôles :

* Un utilisateur ne peut voir que ses propres réservations
* Un admin peut gérer les services
* Les mots de passe doivent être chiffrés
* Certaines pages sont privées

Nous aborderons les différentes stratégies plus tard (Sessions, Tokens, JWT).


## 11. Sécurité Backend : pourquoi c’est essentiel ?

Le backend doit garantir :

* la confidentialité
* l’intégrité des données
* la non-altération
* la résistance aux attaques

---
Principales menaces :

* Injection SQL
* Vol de token
* Mot de passe faible
* Attaques XSS / CSRF
* Bruteforce

Les frameworks modernes (Laravel, Django, Express + middlewares) intègrent déjà des protections, mais il faut comprendre ce qu’on fait.


## 12. Technologies Backend : panorama pour étudiants

### 12.1 PHP / Laravel

* Très utilisé dans les sites web d’entreprises, institutions, ONG
* Idéal pour comprendre la structure MVC
* Très simple pour commencer

---
### 12.2 JavaScript / Node.js / Express

* Backend moderne pour les applications rapides
* Excellent pour le temps réel
* Légèrement plus bas niveau

### 12.3 Python / Django REST Framework

* Très structuré
* Utilisé dans l’IA, la data science, la fintech
* Idéal pour apprendre une architecture professionnelle

### 12.4 Autres technos (culture générale) :

* Java/Spring
* Go/Fiber
* Ruby on Rails
* .NET Core

L’objectif n’est pas de tout apprendre, mais de **comprendre l’écosystème**.

---

## EXERCICES DE FIN DE COURS

Voici des exercices qui permettent de valider la compréhension immédiate.


## Exercice 1 — Identifier les couches

Pour chaque action, indiquer :
**Route**, **Controller**, **Service**, **Model**, **Base de données**.

1. Vérifier si l’heure de réservation est libre.
2. Chercher la liste des services.
3. Renvo­yer un message d’erreur “mot de passe incorrect”.
4. Enregistrer une nouvelle réservation.

---

## Exercice 2 — Décrire une API simple

Définir pour un “système d’articles” :

* une ressource
* 4 endpoints REST
* la méthode utilisée

---

## Exercice 3 — Trouver la différence

Entre :

* Authentification
* Autorisation

Avec un exemple simple.

---

## Exercice 4 — Classer les technologies

Classer :
Laravel, Django, Node/Express, SQL, MongoDB
dans :

* langages
* frameworks backend
* moteurs de base de données

