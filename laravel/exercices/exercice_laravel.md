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

# Série d’exercices – Consolidation
## Par Robert DIASSÉ &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
 <img src="lara.webp" alt="ISI" width="50px">



*(Laravel – Authentification, rôles, CRUD, Blade, Eloquent)*

---
<!-- 
## Exercice 1 – Comprendre le flux Laravel 


### Travail demandé

Pour l’affichage de la liste des services :

1. Identifier :

   * la route utilisée,
   * le contrôleur appelé,
   * la méthode exécutée,
   * le modèle utilisé,
   * la vue affichée.
2. Expliquer (par écrit) le rôle de chaque fichier.
3. Expliquer comment les données arrivent dans la vue.

### Résultat attendu

Un schéma ou un texte clair décrivant :

```
URL → Route → Controller → Model → Controller → Vue
```

--- -->

## Exercice 1 – Affichage des services

### Objectif

Maîtriser l’affichage avec Blade et Eloquent.

### Travail demandé

1. Afficher la liste de tous les services.
2. Pour chaque service, afficher :

   * le titre,
   * le prix,
   * le nom du médecin associé.
3. Si aucun médecin n’est associé, afficher un texte par défaut.

### Contraintes

* Utiliser une boucle Blade.
* Utiliser une relation Eloquent.
* Aucun SQL manuel.

---

## Exercice 2 – Page de détail d’un service

### Objectif

Comprendre le **Route Model Binding**.

### Travail demandé

1. Créer une page permettant d’afficher :

   * le détail d’un service,
   * le médecin responsable,
   * la durée et le prix.
2. Ajouter un lien « Voir » depuis la liste des services.

### Contraintes

* Le contrôleur doit recevoir un objet `Service`.
* Aucun `find()` manuel dans la vue.

---

## Exercice 3 – Bouton Modifier (edit / update)

### Objectif

Comprendre la modification d’une ressource.

### Travail demandé

1. Ajouter un bouton « Modifier » pour chaque service.
2. Préremplir le formulaire avec les données existantes.
3. Mettre à jour les informations en base.

### Contraintes

* Utiliser `@method('PUT')`.
* Utiliser `$service->update()`.
* Rediriger vers la liste après modification.

---

## Exercice 4 – Bouton Supprimer (delete)

### Objectif

Comprendre la suppression sécurisée.

### Travail demandé

1. Ajouter un bouton « Supprimer ».
2. Afficher une confirmation avant suppression.
3. Supprimer le service de la base de données.

### Contraintes

* Utiliser un formulaire (pas un lien).
* Utiliser `@csrf` et `@method('DELETE')`.
* Utiliser Eloquent (`delete()`).

---

## Exercice 5 – Gestion des rôles et accès

### Objectif

Fixer la notion d’**autorisation**.

### Travail demandé

1. Vérifier que :

   * seul l’admin peut accéder au CRUD des services.
2. Tester l’accès :

   * avec un patient,
   * avec un médecin.
3. Observer et expliquer le comportement de Laravel.

### Question à répondre

> Pourquoi un middleware est préférable à un `if` dans le contrôleur ?

---

## Exercice 6 – CRUD Médecin (administrateur)

### Objectif

Renforcer la gestion des utilisateurs.

### Travail demandé

1. Créer une page listant les médecins.
2. Permettre à l’admin de :

   * créer un médecin,
   * supprimer un médecin.
3. Empêcher la suppression d’un médecin lié à des services.

### Contraintes

* Le médecin est un `User` avec le rôle `medecin`.
* Vérifier les relations avant suppression.

---

## Exercice 7 – Relations Eloquent (obligatoire)

### Objectif

S’assurer que les relations sont comprises.

### Travail demandé

1. Afficher :

   * les services d’un médecin,
   * le médecin d’un service.
2. Expliquer quelle relation est utilisée dans chaque cas.

### Question

> Pourquoi Laravel n’a pas besoin de requête SQL ici ?

---

## Exercice 8 – Données de test (factories)

### Objectif

Savoir tester une application proprement.

### Travail demandé

1. Créer :

   * un admin,
   * 2 médecins,
   * 5 services,
   * 5 patients.
2. Vérifier que l’application fonctionne sans saisie manuelle.

### Contraintes

* Utiliser des factories.
* Utiliser un seeder principal.

---

## Exercice 9 – Auto-évaluation (à rendre)

Chaque étudiant doit répondre  :

1. Quelle est la différence entre :

   * Blade,
   * Controller,
   * Model ?
2. Pourquoi utiliser Eloquent ?
3. À quoi sert `@csrf` ?
4. Pourquoi HTML ne supporte pas DELETE ?
5. Quel est le rôle exact du middleware `auth` ?

