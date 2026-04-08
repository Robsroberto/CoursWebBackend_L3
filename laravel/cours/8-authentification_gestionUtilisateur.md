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


# CHAPITRE 8: Authentification et gestion des utilisateurs


## Par Robert DIASSÉ &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
 <img src="lara.webp" alt="ISI" width="50px">



## 8.1 Objectif du chapitre

Dans ce chapitre, nous allons mettre en place **une authentification simple**, sans package externe, afin de :

* comprendre comment fonctionne l’authentification dans Laravel,
* permettre à un **patient de s’inscrire seul**,
* permettre à un **administrateur de créer des médecins**,
* utiliser le middleware `auth` déjà fourni par Laravel,
* préparer la mise en place future de Laravel Breeze.

À la fin de ce chapitre :

* un utilisateur peut se connecter,
* l’accès aux pages est protégé,
* les rôles sont fonctionnels,
* les dashboards dépendent du rôle.

---

## 8.2 Rappel : le modèle User

Le projet utilise déjà la table `users` avec le champ `role`.

Champs principaux :

* id
* name
* email
* password
* role (admin | medecin | patient)

Le modèle `User` existe déjà et étend `Authenticatable`.

---

## 8.3 Principe général de l’authentification Laravel

Laravel gère l’authentification via :

* la session,
* le helper `auth()`.

Fonctions essentielles :

* `auth()->attempt()` : tente une connexion
* `auth()->check()` : vérifie si l’utilisateur est connecté
* `auth()->user()` : retourne l’utilisateur connecté
* `auth()->logout()` : déconnecte l’utilisateur

Le middleware `auth` repose uniquement sur cela.

---

## 8.4 Création du contrôleur d’authentification

Créer un contrôleur :

```bash
php artisan make:controller AuthController
```

Emplacement :

```
app/Http/Controllers/AuthController.php
```

---

## 8.5 Inscription d’un patient (publique)

### 8.5.1 Route d’inscription

Dans `routes/web.php` :

```php
use App\Http\Controllers\AuthController;

Route::get('/register', [AuthController::class, 'showRegister']);
Route::post('/register', [AuthController::class, 'register']);
```

---

### 8.5.2 Méthode d’affichage du formulaire

```php
public function showRegister()
{
    return view('auth.register');
}
```

---

### 8.5.3 Méthode d’inscription

```php
use App\Models\User;
use Illuminate\Support\Facades\Hash;
use Illuminate\Http\Request;

public function register(Request $request)
{
    $validated = $request->validate([
        'name' => 'required|string|max:255',
        'email' => 'required|email|unique:users',
        'password' => 'required|min:6|confirmed',
    ]);

    User::create([
        'name' => $validated['name'],
        'email' => $validated['email'],
        'password' => Hash::make($validated['password']),
        'role' => 'patient',
    ]);

    return redirect('/login')->with('success', 'Compte créé avec succès.');
}
```

Explication :

* le rôle est forcé à `patient`,
* le mot de passe est chiffré,
* l’utilisateur ne choisit pas son rôle.

---

## 8.6 Connexion des utilisateurs

### 8.6.1 Routes de connexion

```php
Route::get('/login', [AuthController::class, 'showLogin'])->name('login');
Route::post('/login', [AuthController::class, 'login']);
Route::post('/logout', [AuthController::class, 'logout'])->middleware('auth');
```

---

### 8.6.2 Affichage du formulaire de connexion

```php
public function showLogin()
{
    return view('auth.login');
}
```

---

### 8.6.3 Traitement de la connexion

```php
use Illuminate\Support\Facades\Auth;

public function login(Request $request)
{
    $credentials = $request->validate([
        'email' => 'required|email',
        'password' => 'required',
    ]);

    if (Auth::attempt($credentials)) {
        $request->session()->regenerate();
        return redirect('/dashboard');
    }

    return back()->withErrors([
        'email' => 'Identifiants incorrects.',
    ]);
}
```

---

### 8.6.4 Déconnexion

```php
public function logout(Request $request)
{
    Auth::logout();
    $request->session()->invalidate();
    $request->session()->regenerateToken();

    return redirect('/login');
}
```

---

## 8.7 Tableau de bord selon le rôle

Dans `DashboardController` :

```php
public function index()
{
    $user = auth()->user();

    if ($user->role === 'admin') {
        return view('dashboard.admin');
    }

    if ($user->role === 'medecin') {
        return view('dashboard.medecin');
    }

    return view('dashboard.patient');
}
```

Route protégée :

```php
Route::get('/dashboard', [DashboardController::class, 'index'])
    ->middleware('auth')
    ->name('dashboard');
```

---

## 8.8 Création des médecins par l’administrateur

Les médecins **ne s’inscrivent pas eux-mêmes**.

### 8.8.1 Route admin

```php
Route::middleware(['auth', 'role:admin'])->group(function () {
    Route::get('/admin/users/create', [AdminUserController::class, 'create']);
    Route::post('/admin/users/store', [AdminUserController::class, 'store']);
});
```

---

### 8.8.2 Création du médecin

```php
public function store(Request $request)
{
    $validated = $request->validate([
        'name' => 'required',
        'email' => 'required|email|unique:users',
        'password' => 'required|min:6',
    ]);

    User::create([
        'name' => $validated['name'],
        'email' => $validated['email'],
        'password' => Hash::make($validated['password']),
        'role' => 'medecin',
    ]);

    return redirect('/admin/users')->with('success', 'Médecin créé.');
}
```

---

## 8.9 Protection des routes par rôle

Les routes du projet fil rouge sont protégées ainsi :

```php
Route::middleware(['auth', 'role:patient'])->group(function () {
    // routes patient
});

Route::middleware(['auth', 'role:medecin'])->group(function () {
    // routes médecin
});

Route::middleware(['auth', 'role:admin'])->group(function () {
    // routes admin
});
```

Le middleware `role` vérifie simplement :

* utilisateur connecté
* rôle correspondant

---

## 8.10 Pourquoi cette étape est volontairement manuelle

Cette authentification :

* n’est pas la plus rapide,
* n’est pas la plus sécurisée,
* mais elle est **la plus pédagogique**.

Elle permet de comprendre :

* ce que fait Laravel Breeze,
* ce que fait réellement le middleware `auth`,
* comment les rôles sont gérés.

---

## 8.11 Transition vers Laravel Breeze

Dans le prochain chapitre :

* nous installerons Laravel Breeze,
* nous comparerons avec cette version manuelle,
* nous verrons ce que Breeze automatise
---

À ce stade :

* inscription patient fonctionnelle
* création médecin par admin
* connexion / déconnexion
* middleware `auth` actif
* middleware `role` opérationnel
* redirection par rôle fonctionnelle

Le projet est maintenant **un vrai backend utilisable**.
