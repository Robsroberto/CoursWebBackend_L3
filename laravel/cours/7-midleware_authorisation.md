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


# CHAPITRE 7 :Routes, Middlewares, Rôles et Sécurisation de l’Application


## Par Robert DIASSÉ &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
 <img src="lara.webp" alt="ISI" width="50px">


---

## 1. Introduction

Jusqu’à présent, l’application fonctionne visuellement et structurellement :

* les pages existent
* les contrôleurs existent
* les vues sont prêtes
* les dashboards par rôle sont en place

Cependant, un problème majeur subsiste :

> N’importe quel utilisateur peut accéder à n’importe quelle page.

Par exemple :

* un patient peut accéder au dashboard administrateur
* un médecin peut voir la gestion des utilisateurs
* une personne non connectée peut accéder aux pages protégées

Ce chapitre a pour objectif de **résoudre ce problème proprement**, en introduisant :

* les routes bien organisées
* les méthodes HTTP
* les middlewares
* la gestion des rôles

---

## 2. Rappel fondamental : rôle des routes

Une **route** indique à Laravel :

> « Quand une requête arrive à telle URL, exécute telle action. »

Une route relie toujours :

* une URL
* une méthode HTTP
* une action (contrôleur ou fonction)

Exemple simple :

```php
Route::get('/login', function () {
    return view('auth.login');
});
```

Cela signifie :

* quand on tape `/login` dans le navigateur
* Laravel retourne la vue `auth.login`

---

## 3. Les méthodes HTTP : comprendre quand utiliser GET, POST, PUT, DELETE

### 3.1 Méthode GET

GET sert à :

* afficher une page
* afficher un formulaire
* afficher des données

Exemples dans le projet :

```php
Route::get('/services', [ServiceController::class, 'index']);
Route::get('/services/{id}', [ServiceController::class, 'show']);
```

Règle simple :

> GET = lire / afficher

---

### 3.2 Méthode POST

POST sert à :

* envoyer des données
* créer une ressource
* soumettre un formulaire

Exemple :

```php
Route::post('/reservations', [ReservationController::class, 'store']);
```

Règle simple :

> POST = créer / envoyer

---

### 3.3 Méthode PUT / PATCH

PUT ou PATCH sert à :

* modifier une donnée existante

Exemple :

```php
Route::put('/reservations/{id}', [MedecinController::class, 'updateStatus']);
```

Règle simple :

> PUT / PATCH = modifier

---

### 3.4 Méthode DELETE

DELETE sert à :

* supprimer une donnée

Exemple :

```php
Route::delete('/services/{id}', [AdminServiceController::class, 'destroy']);
```

Règle simple :

> DELETE = supprimer

---

## 4. Comment formater correctement les routes selon le besoin

### 4.1 Routes publiques

Les routes accessibles à tout le monde :

```php
Route::get('/', function () {
    return view('home');
});

Route::get('/services', [ServiceController::class, 'index']);
Route::get('/services/{id}', [ServiceController::class, 'show']);
```

---

### 4.2 Routes liées à un rôle

On ne mélange pas toutes les routes ensemble.
On les **regroupe par fonctionnalité** et par **rôle**.

Exemple : routes patient

```php
Route::get('/mes-reservations', [ReservationController::class, 'myReservations']);
Route::post('/reservations', [ReservationController::class, 'store']);
Route::post('/reservations/{id}/annuler', [ReservationController::class, 'cancel']);
```

---

### 4.3 Routes administrateur

```php
Route::get('/admin/services', [AdminServiceController::class, 'index']);
Route::get('/admin/users', [AdminUserController::class, 'index']);
Route::get('/admin/reservations', [AdminReservationController::class, 'index']);
```

À ce stade, **elles ne sont pas encore protégées**, c’est normal.

---

## 5. Problème actuel de sécurité

Actuellement :

* toutes les routes sont accessibles
* Laravel ne vérifie pas qui accède à quoi

Cela pose un problème réel de sécurité.

La solution n’est **pas** :

* de mettre des `if` partout
* de dupliquer le code

La solution est :

> utiliser des **middlewares**

---

## 6. Qu’est-ce qu’un middleware

Un middleware est un **filtre**.

Il se place **entre la requête et le contrôleur**.

Schéma logique :

```
Navigateur
   ↓
Route
   ↓
Middleware
   ↓
Contrôleur
   ↓
Vue
```

Le middleware peut :

* autoriser l’accès
* bloquer l’accès
* rediriger l’utilisateur

---

## 7. Authentification vs Autorisation

### 7.1 Authentification

Question :

> L’utilisateur est-il connecté ?

Laravel fournit le middleware :

```php
auth
```

---

### 7.2 Autorisation

Question :

> L’utilisateur a-t-il le droit d’accéder à cette page ?

C’est ici qu’interviennent les **rôles** :

* admin
* medecin
* patient

---

## 8. Middleware d’authentification (`auth`)

Pour protéger une route :

```php
Route::get('/dashboard/admin', function () {
    return view('dashboard.admin');
})->middleware('auth');
```

Résultat :

* utilisateur non connecté → redirigé vers login
* utilisateur connecté → accès autorisé

---

## 9. Création d’un middleware de rôle

Créer un middleware :

```bash
php artisan make:middleware RoleMiddleware
```

Fichier généré :

```
app/Http/Middleware/RoleMiddleware.php
```

---

### Contenu du middleware

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class RoleMiddleware
{
    public function handle(Request $request, Closure $next, $role)
    {
        if (!auth()->check()) {
            return redirect('/login');
        }

        if (auth()->user()->role !== $role) {
            abort(403);
        }

        return $next($request);
    }
}
```

---

## 10. Déclaration du middleware

Dans :

```
app/Http/Kernel.php
```

Ajouter :

```php
'role' => \App\Http\Middleware\RoleMiddleware::class,
```

---

## 11. Protection des routes par rôle

### Routes administrateur

```php
Route::middleware(['auth', 'role:admin'])->group(function () {

    Route::get('/dashboard/admin', function () {
        return view('dashboard.admin');
    });

    Route::get('/admin/services', [AdminServiceController::class, 'index']);
    Route::get('/admin/users', [AdminUserController::class, 'index']);
});
```

---

### Routes médecin

```php
Route::middleware(['auth', 'role:medecin'])->group(function () {

    Route::get('/dashboard/medecin', function () {
        return view('dashboard.medecin');
    });

});
```

---

### Routes patient

```php
Route::middleware(['auth', 'role:patient'])->group(function () {

    Route::get('/dashboard/patient', function () {
        return view('dashboard.patient');
    });

});
```

---

## 12. Redirection après connexion

Dans le contrôleur de login, après authentification :

```php
if (auth()->user()->role === 'admin') {
    return redirect('/dashboard/admin');
}

if (auth()->user()->role === 'medecin') {
    return redirect('/dashboard/medecin');
}

return redirect('/dashboard/patient');
```

---

## 13. Tests à effectuer

* essayer d’accéder à `/dashboard/admin` sans être connecté
* essayer avec un patient
* essayer avec un médecin
* vérifier les erreurs 403
* vérifier la redirection vers login

---

## 14. Erreurs courantes

* oublier `auth`
* oublier d’enregistrer le middleware
* se tromper de rôle (`medecin` ≠ `medicin`)
* mal grouper les routes
* mettre la sécurité dans le contrôleur

---

## 15. Résumé du chapitre

À la fin de ce chapitre, tu sais :

* structurer correctement des routes
* choisir la bonne méthode HTTP
* protéger des pages
* utiliser des middlewares
* gérer les rôles
* sécuriser une application Laravel