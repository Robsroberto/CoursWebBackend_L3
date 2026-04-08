---
marp: false
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
<img src="../isi.png" alt="ISI" width="100px">


# Travaux pratiques – Projets collaboratifs Backend (Laravel / PHP)


## Par Robert DIASSÉ &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
 <img src="lara.webp" alt="ISI" width="50px">



## Organisation générale

La classe est divisée en **5 groupes**.
Chaque groupe réalise **un projet différent**, distinct du **projet fil rouge** (réservation médicale).

Ces projets font partie de l’**évaluation continue**.

---

## Règles obligatoires (non négociables)

1. **Un dépôt GitHub par groupe**
2. **Travail collaboratif obligatoire**
3. Chaque membre du groupe doit :

   * avoir des **commits visibles et réguliers**,
   * travailler sur **une partie identifiable** du projet.
4. **Absence de commits individuels = pénalité importante**
5. Le projet doit être **fonctionnel**, même avec une interface simple.

---

## Exigences techniques communes à tous les projets

Chaque projet doit obligatoirement contenir :

### 1. Authentification

* Inscription
* Connexion
* Déconnexion

### 2. Gestion des rôles

* Visiteur (non connecté)
* Utilisateur connecté
* Administrateur

### 3. Accès différencié

* Visiteur : consultation uniquement
* Utilisateur : CRUD partiel
* Administrateur : CRUD complet

### 4. CRUD complet

* Create
* Read
* Update
* Delete

### 5. Base de données relationnelle

* Migrations claires
* Relations cohérentes entre les entités

---

## Les 5 projets proposés (hors projet fil rouge)

### Groupe 1 – Gestion de bibliothèque

* Livres
* Catégories
* Emprunts
* Administrateur gère les livres et catégories
* Utilisateur emprunte et rend des livres

---

### Groupe 2 – Gestion de restaurant

* Menus
* Plats
* Commandes
* Administrateur gère les menus
* Utilisateur passe des commandes

---

### Groupe 3 – Gestion de formations

* Formations
* Formateurs
* Inscriptions
* Administrateur crée les formations
* Utilisateur s’inscrit

---

### Groupe 4 – Gestion de réservation hôtelière

* Chambres
* Réservations
* Clients
* Administrateur gère les chambres
* Client réserve une chambre

---

### Groupe 5 – Gestion de stock / magasin

* Produits
* Catégories
* Entrées / sorties de stock
* Administrateur gère le stock
* Utilisateur consulte l’état du stock

---

## Répartition du travail (exigée)

Chaque groupe doit **définir clairement les responsabilités** :

* Authentification et rôles
* Modèles et migrations
* CRUD principal
* Tests et corrections
* Interface (si effectif suffisant)

Cette répartition doit être **visible dans l’historique GitHub**.

---

## Exigences GitHub

Le dépôt doit contenir :

* un `README.md` avec :

  * description du projet,
  * membres du groupe,
  * rôle de chaque membre.
* des commits réguliers et explicites :

  * `add product crud`
  * `implement auth roles`
  * `fix validation error`

---

## Critères de notation (projet)

| Critère                           | Points  |
| --------------------------------- | ------- |
| Fonctionnalités demandées         | /8      |
| Structure et organisation du code | /4      |
| Gestion des rôles et accès        | /3      |
| Qualité des commits GitHub        | /3      |
| Travail collaboratif réel         | /2      |
| **Total**                         | **/20** |

⚠️ Un étudiant **sans implication visible sur GitHub** ne pourra pas bénéficier pleinement de la note du groupe.

---

## Rappel important

* Le **projet fil rouge** (réservation médicale) est **traité séparément**.
* Ces projets servent à :

  * pratiquer,
  * comprendre Laravel et la POO,
  * apprendre le travail collaboratif.
* GitHub est **obligatoire**, pas optionnel.
