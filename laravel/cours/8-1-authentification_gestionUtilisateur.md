

# CHAPITRE 8 — VERSION PROFESSEUR

## Authentification et gestion des utilisateurs (approche pédagogique manuelle)

---

## 1. Intention pédagogique du chapitre

Ce chapitre n’a **pas pour objectif** de produire une authentification “idéale” ou “industrielle”.

L’objectif est de permettre aux étudiants de :

* comprendre **ce que fait réellement Laravel quand on parle d’auth**
* voir **les liens entre session, middleware et base de données**
* comprendre **avant d’automatiser**
* éviter l’effet “Breeze = magie noire”

Ce chapitre est donc **volontairement manuel**, progressif et explicite.

---

## 2. Pourquoi ne PAS commencer par Laravel Breeze

Si Breeze est introduit trop tôt :

* les étudiants copient sans comprendre
* ils ne savent pas :

  * d’où vient `auth()`
  * pourquoi `middleware('auth')` fonctionne
  * comment Laravel stocke l’utilisateur connecté
* les erreurs deviennent incompréhensibles

Ce chapitre crée **le socle mental nécessaire**.

---

## 3. Ce que Laravel fournit déjà (à rappeler clairement)

Laravel fournit **nativement** :

* le middleware `auth`
* la gestion de session
* le hashage des mots de passe
* le helper `auth()`

👉 **Aucune réinvention n’est faite ici**

Le chapitre montre **comment utiliser l’existant**, pas comment le recréer.

---

## 4. Structure logique retenue (et pourquoi)

### 4.1 Séparation des responsabilités

| Action              | Responsable    |
| ------------------- | -------------- |
| Inscription patient | Public         |
| Création médecin    | Administrateur |
| Connexion           | Système        |
| Autorisation        | Middleware     |
| Décision d’accès    | Rôle           |

Cette séparation évite :

* la confusion des rôles
* les failles de sécurité
* les “if” dans les vues

---

## 5. Inscription des patients : choix pédagogiques

### 5.1 Pourquoi autoriser l’inscription publique des patients

* logique métier réaliste
* simple à comprendre
* pas de conflit de rôle

### 5.2 Pourquoi le rôle est forcé à `patient`

```php
'role' => 'patient'
```

Objectif pédagogique :

* empêcher toute manipulation
* montrer que **le rôle est une décision serveur**
* préparer la notion d’autorisation

---

## 6. Création des médecins par l’admin : justification

Les médecins :

* ont un rôle sensible
* ont accès à des données médicales
* ne doivent pas être auto-créés

Pédagogiquement :

* on introduit la notion de **hiérarchie**
* on justifie le middleware `role:admin`
* on prépare les cas réels (SIH, ERP, etc.)

---

## 7. Middleware `auth` : ce qu’il faut vérifier en priorité

### 7.1 Fonctionnement réel

Le middleware `auth` vérifie :

```php
auth()->check()
```

S’il retourne `false` :

* Laravel redirige vers la route `login`

### 7.2 Erreur fréquente attendue

```txt
Route [login] not defined
```

➡️ **Erreur pédagogique attendue**, pas un bug.

À expliquer ainsi :

> Laravel sait protéger, mais il ne sait pas encore où rediriger.

---

## 8. Middleware `role` : points de vigilance

### 8.1 Pré-requis indispensable

Le middleware `role` **ne peut fonctionner que si** :

* `auth()` fonctionne
* `auth()->user()` retourne un utilisateur
* le champ `role` existe en base

### 8.2 Erreur fréquente des étudiants

```php
auth()->user()->role
```

alors que l’utilisateur n’est pas connecté.

Résultat :

* erreur 500
* confusion

👉 rappeler que **autorisation ≠ authentification**

---

## 9. Ordre STRICT d’activation des fonctionnalités

Cet ordre ne doit **pas être inversé** :

1. Données (users, rôles)
2. Authentification (login)
3. Middleware `auth`
4. Middleware `role`
5. Accès conditionnel aux vues

Tout autre ordre crée des bugs difficiles à expliquer.

---

## 10. Validation technique du chapitre (checklist professeur)

Avant d’aller au chapitre suivant, vérifier :

* [ ] Un patient peut s’inscrire
* [ ] Un patient peut se connecter
* [ ] Un admin peut créer un médecin
* [ ] Un médecin peut se connecter
* [ ] `/dashboard` redirige selon le rôle
* [ ] Les routes protégées refusent l’accès non autorisé
* [ ] Le middleware `auth` bloque correctement
* [ ] Le middleware `role` fonctionne quand auth est actif

---

## 11. Pourquoi ne PAS tester les POST avec Postman à ce stade

Décision pédagogique assumée :

* routes web
* CSRF actif
* Postman non adapté sans configuration avancée

👉 On attend :

* formulaires Blade complets
* ou routes API dédiées

Cela évite :

* 419 incompris
* frustration inutile
* perte de temps

---

## 12. Transition propre vers Laravel Breeze

À ce stade, les étudiants savent :

* ce qu’est un login
* ce que fait `auth()`
* pourquoi un middleware bloque
* pourquoi un rôle est côté serveur

Quand Breeze sera introduit :

* ils reconnaîtront chaque élément
* ils comprendront ce qui est automatisé
* ils pourront debugger Breeze

---

## 13. Positionnement du chapitre dans le semestre

Ce chapitre est :

* central
* structurant
* déterminant pour la suite

Il conditionne :

* la sécurité
* la compréhension des API
* la compréhension du frontend futur

---

## 14. Décision finale

Ce chapitre est **validé pédagogiquement et techniquement**
Il peut être donné **sans risque de confusion**, à condition de :

* respecter l’ordre
* suspendre Postman pour les POST
* garder Breeze pour après




Parfait, on va **verrouiller proprement le Chapitre 8** avant d’aller plus loin 👍
Je te donne **exactement les modifications HTML/Blade à faire**, **sans ambiguïté**, **directement exploitables**, et **cohérentes avec SB Admin 2** et le projet fil rouge.

Je sépare en **3 parties claires** :

1. Formulaire **Login**
2. Formulaire **Register (Patient)**
3. **Listing des services (Admin – CRUD)**

Tout est présenté **côté étudiant**, mais suffisamment rigoureux pour toi.

---

# 1. Modifications du formulaire LOGIN

📁 Fichier concerné (Breeze Blade)

```
resources/views/auth/login.blade.php
```

## 1.1 Objectifs pédagogiques

* Comprendre les champs nécessaires à l’authentification
* Voir le lien entre formulaire HTML et `Auth::attempt()`
* Intégrer le formulaire dans le layout SB Admin

---

## 1.2 Structure recommandée (SB Admin intégré)

### Étape 1 : étendre le layout

En haut du fichier :

```blade
@extends('layouts.app')
```

---

### Étape 2 : contenu principal

```blade
@section('content')
<div class="container">
    <div class="row justify-content-center">
        <div class="col-md-6">
            <div class="card shadow">
                <div class="card-header">
                    <h4>Connexion</h4>
                </div>

                <div class="card-body">
                    <form method="POST" action="{{ route('login') }}">
                        @csrf
```

---

### Étape 3 : champ email

```blade
<div class="mb-3">
    <label>Email</label>
    <input type="email"
           name="email"
           class="form-control"
           value="{{ old('email') }}"
           required>
</div>
```

---

### Étape 4 : champ mot de passe

```blade
<div class="mb-3">
    <label>Mot de passe</label>
    <input type="password"
           name="password"
           class="form-control"
           required>
</div>
```

---

### Étape 5 : bouton de soumission

```blade
<div class="mb-3">
    <button type="submit" class="btn btn-primary w-100">
        Se connecter
    </button>
</div>
```

---

### Étape 6 : lien vers inscription

```blade
<div class="text-center">
    <a href="{{ route('register') }}">
        Créer un compte patient
    </a>
</div>
```

---

### Étape 7 : fermeture

```blade
                    </form>
                </div>
            </div>
        </div>
    </div>
</div>
@endsection
```

---

## 1.3 Points pédagogiques à expliquer

* `@csrf` : protection contre attaques CSRF
* `name="email"` et `name="password"` doivent correspondre au contrôleur
* `route('login')` est générée par Breeze

---

# 2. Modifications du formulaire REGISTER (Patient)

📁 Fichier concerné

```
resources/views/auth/register.blade.php
```

---

## 2.1 Principe métier

* L’inscription est **publique**
* Le rôle est **forcé à `patient` côté backend**
* Aucun champ `role` dans le formulaire

---

## 2.2 Structure du formulaire

### Étape 1 : layout

```blade
@extends('layouts.app')
```

---

### Étape 2 : formulaire

```blade
@section('content')
<div class="container">
    <div class="row justify-content-center">
        <div class="col-md-6">
            <div class="card shadow">
                <div class="card-header">
                    <h4>Inscription Patient</h4>
                </div>

                <div class="card-body">
                    <form method="POST" action="{{ route('register') }}">
                        @csrf
```

---

### Étape 3 : nom

```blade
<div class="mb-3">
    <label>Nom complet</label>
    <input type="text"
           name="name"
           class="form-control"
           value="{{ old('name') }}"
           required>
</div>
```

---

### Étape 4 : email

```blade
<div class="mb-3">
    <label>Email</label>
    <input type="email"
           name="email"
           class="form-control"
           value="{{ old('email') }}"
           required>
</div>
```

---

### Étape 5 : mot de passe

```blade
<div class="mb-3">
    <label>Mot de passe</label>
    <input type="password"
           name="password"
           class="form-control"
           required>
</div>
```

---

### Étape 6 : confirmation

```blade
<div class="mb-3">
    <label>Confirmation</label>
    <input type="password"
           name="password_confirmation"
           class="form-control"
           required>
</div>
```

---

### Étape 7 : bouton

```blade
<div class="mb-3">
    <button type="submit" class="btn btn-success w-100">
        S'inscrire
    </button>
</div>
```

---

### Étape 8 : fermeture

```blade
                    </form>
                </div>
            </div>
        </div>
    </div>
</div>
@endsection
```

---

## 2.3 Points pédagogiques essentiels

* `password_confirmation` est utilisé par Laravel automatiquement
* Le rôle n’est **jamais** fourni par le client
* Toute logique métier est côté contrôleur

---

# 3. Listing des SERVICES (CRUD Admin)

📁 Fichier

```
resources/views/admin/services/index.blade.php
```

---

## 3.1 Objectif

* Visualiser les services
* Accéder à modifier / supprimer
* Comprendre le lien CRUD ↔ vues ↔ routes

---

## 3.2 Structure SB Admin 2

```blade
@extends('layouts.admin')

@section('content')
<div class="container-fluid">
```

---

## 3.3 Bouton d’ajout

```blade
<div class="mb-3">
    <a href="{{ route('services.create') }}" class="btn btn-primary">
        Ajouter un service
    </a>
</div>
```

---

## 3.4 Tableau des services

```blade
<table class="table table-bordered">
    <thead>
        <tr>
            <th>Nom</th>
            <th>Prix</th>
            <th>Durée (min)</th>
            <th>Actions</th>
        </tr>
    </thead>

    <tbody>
        @foreach ($services as $service)
        <tr>
            <td>{{ $service->nom }}</td>
            <td>{{ $service->prix }}</td>
            <td>{{ $service->duree }}</td>
            <td>
```

---

## 3.5 Boutons Modifier / Supprimer

```blade
<a href="{{ route('services.edit', $service->id) }}"
   class="btn btn-warning btn-sm">
    Modifier
</a>

<form action="{{ route('services.destroy', $service->id) }}"
      method="POST"
      style="display:inline;">
    @csrf
    @method('DELETE')
    <button type="submit"
            class="btn btn-danger btn-sm"
            onclick="return confirm('Confirmer la suppression ?')">
        Supprimer
    </button>
</form>
```

---

### Fermeture

```blade
            </td>
        </tr>
        @endforeach
    </tbody>
</table>
</div>
@endsection
```

---

## 3.6 Points pédagogiques à expliquer

* `@method('DELETE')` simule une requête DELETE
* `route('services.destroy')` vient de `Route::resource`
* La vue ne fait **aucune logique métier**

---

# 4. État du projet après ces modifications

À ce stade :

* Login fonctionnel (Breeze)
* Register patient fonctionnel
* CRUD admin visible et utilisable
* Intégration SB Admin cohérente
* Base prête pour factories & seeders

---

## Prochaine étape logique

👉 Ajouter les **factories + seeders**
👉 Finaliser le **CRUD admin complet**
👉 Valider l’application comme **backend fonctionnel**

Dis-moi :

* **« c’est bon, on continue »**
* ou **« ajuste tel formulaire »**
