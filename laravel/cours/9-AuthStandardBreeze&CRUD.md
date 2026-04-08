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

# **Chapitre 9 – Authentification standardisée et CRUD Administrateur**

## Par Robert DIASSÉ &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
 <img src="lara.webp" alt="ISI" width="50px">




## 9.1 Objectif général du chapitre

À ce stade du projet, l’application dispose déjà :

* d’une base de données fonctionnelle ;
* de modèles Eloquent (`User`, `Service`, `Reservation`) ;
* de contrôleurs opérationnels ;
* de routes de test validées ;
* d’un template frontend (SB Admin 2) correctement intégré.

Ce chapitre a pour objectif de :

* mettre en place une **authentification complète et fiable** ;
* sécuriser l’accès aux fonctionnalités selon le rôle ;
* développer un **CRUD Administrateur réel** ;
* peupler la base de données pour faciliter les tests ;
* obtenir une application **fonctionnelle de bout en bout**, proche d’un projet professionnel.

---

## Rappel du fonctionnement actuel (avant Breeze)

Avant l’introduction de Breeze :

* l’authentification a été implémentée manuellement ;
* les contrôleurs gèrent déjà les rôles (`admin`, `medecin`, `patient`) ;
* les routes protégées utilisent le middleware `auth`.

Ce travail **n’est pas supprimé**.
Il sert de base de compréhension pour la suite.

> L’objectif n’est pas de recommencer, mais de **standardiser et sécuriser**.

---


## 9.2 Avant installation : sécuriser l’existant (important)

### 9.2.1 Vérifier les routes actuelles

Afficher la liste complète des routes :

```bash
php artisan route:list
```

Ce que tu dois repérer :

* tes routes de test (ex : `/test-role-admin`, `/dashboard/admin`, etc.)
* tes routes métiers (services, réservations…)
* vérifier si tu as déjà une route `/login` ou `/register` personnalisée
  (si oui, elle entrera en conflit)

### 9.2.2 Sauvegarde des routes de test

Si tu avais des routes de test temporaires dans `web.php`, tu peux les garder mais les regrouper proprement dans un bloc :

```php
Route::prefix('test')->group(function () {
    Route::get('/role-admin', function () {
        return "OK ADMIN";
    })->middleware(['auth', 'role:admin']);
});
```

But :

* éviter les conflits avec `/login`, `/register`, `/dashboard` que Breeze va introduire.

---

## 9.3 Installation de Breeze

### 9.3.1 Installer le package Breeze

Dans le terminal à la racine du projet :

```bash
composer require laravel/breeze --dev
```

### 9.3.2 Générer Breeze (version Blade)

On choisit Blade on utilises un template avec vues Blade.

```bash
php artisan breeze:install blade
```

Cette commande :

* ajoute des contrôleurs d’auth,
* ajoute des routes d’auth,
* ajoute des vues `resources/views/auth` et `resources/views/profile`,
* ajoute des composants Blade `resources/views/components`,
* ajoute des `FormRequest` (dans `app/Http/Requests`) pour la validation.

### 9.3.3 Migration (à faire même si aucune nouvelle table Breeze)

Pourquoi on lance `migrate` :

* s’assurer que toutes les tables nécessaires existent (users, sessions, password_resets),
* éviter les erreurs si on n’avait pas encore migré proprement.

Commande :

```bash
php artisan migrate
```

Si tout est déjà migré, Laravel affichera simplement que rien n’a été fait.

### 9.3.4 Installation des assets front (si Breeze te le demande)

Selon la config Breeze/Laravel 12, tu auras besoin de lancer les assets (Vite) :

```bash
npm install
npm run dev
```

Remarque :

* Même si tu utilises SB Admin, Breeze apporte parfois des assets pour ses pages.
* En environnement cours, `npm run dev` suffit.

---

## 9.4 Comprendre ce qui a changé (sans encore détailler Breeze)

Après l’installation, tu verras :

* `app/Http/Controllers/Auth/*` (authentification)
* `ProfileController.php` (gestion profil)
* `app/Http/Requests/*` (validation)
* `resources/views/auth/*` (login/register)
* `resources/views/profile/*` (profil)
* `resources/views/components/*` (composants Blade)

On n’explique pas chaque fichier ici en profondeur :
ce sera dans l’annexe Breeze.
Mais on doit savoir **où agir** pour adapter au projet fil rouge.

---

## 9.5 Adapter Breeze à notre projet fil rouge (rôles + dashboards)

### 9.5.1 Problème : Breeze redirige vers une route “dashboard” générique

Par défaut, Breeze crée souvent :

* une route `/dashboard`
* et redirige vers `/dashboard` après login

Or ton projet a :

* `/dashboard/admin`
* `/dashboard/medecin`
* `/dashboard/patient`

Donc on doit forcer la redirection selon le rôle.

---

### 9.5.2 Où se règle la redirection après login (Laravel 12 + Breeze)

Dans Breeze, la connexion passe par :

📁 `app/Http/Controllers/Auth/AuthenticatedSessionController.php`

Cherche la méthode `store(...)`.

Tu verras à la fin quelque chose comme :

```php
return redirect()->intended(RouteServiceProvider::HOME);
```

ou parfois :

```php
return redirect()->intended(route('dashboard', absolute: false));
```

On va remplacer cette redirection par une redirection vers **notre dashboard**.

#### Code à mettre (adaptation rôles)

```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

public function store(LoginRequest $request)
{
    $request->authenticate();

    $request->session()->regenerate();

    $user = Auth::user();

    if ($user->role === 'admin') {
        return redirect('/dashboard/admin');
    }

    if ($user->role === 'medecin') {
        return redirect('/dashboard/medecin');
    }

    return redirect('/dashboard/patient');
}
```

Explications :

* `authenticate()` fait le travail de `Auth::attempt(...)`
* `session()->regenerate()` sécurise la session (anti session fixation)
* on récupère le user connecté (`Auth::user()`)
* on redirige selon `role`

---

## 9.6 Adapter l’inscription Breeze : tout nouvel inscrit = patient

### 9.6.1 Problème

Par défaut, Breeze crée un user avec :

* `name`
* `email`
* `password`

Mais pas ton champ `role`.

### 9.6.2 Où se fait l’inscription

📁 `app/Http/Controllers/Auth/RegisteredUserController.php`

Dans `store(...)`, repère :

```php
User::create([
   'name' => $request->name,
   'email' => $request->email,
   'password' => Hash::make($request->password),
]);
```

Remplace par :

```php
User::create([
   'name' => $request->name,
   'email' => $request->email,
   'password' => Hash::make($request->password),
   'role' => 'patient',
]);
```

Explication :

* l’inscription publique ne doit jamais permettre de choisir un rôle
* sinon n’importe qui peut se créer admin

---

## 9.7 Adapter `/profile` avec ton middleware role

### 9.7.1 Ce que Breeze fournit

Breeze ajoute un module profil accessible via une route `profile.*` protégée par `auth`.

Exemple typique dans `routes/web.php` :

```php
Route::middleware('auth')->group(function () {
    Route::get('/profile', [ProfileController::class, 'edit'])->name('profile.edit');
    Route::patch('/profile', [ProfileController::class, 'update'])->name('profile.update');
    Route::delete('/profile', [ProfileController::class, 'destroy'])->name('profile.destroy');
});
```

### 9.7.2 Notre besoin

Dans notre projet, on veut :

* profil accessible à tout utilisateur connecté
* donc `auth` suffit
* pas besoin de `role:*` ici (car admin/medecin/patient ont tous un profil)

Donc :
✅ on garde `auth` seulement
✅ pas de role sur profile

---

## 9.8 Routes : éviter les conflits et rester cohérent

Après Breeze, refais :

```bash
php artisan route:list
```

Points à vérifier :

* routes `login`, `register`, `logout` existent
* tes dashboards existent toujours (`/dashboard/admin`, etc.)
* tes routes test existent toujours
* profile existe (`/profile`)

---

## 9.9 Validation de l’étape (tests obligatoires)

### Test 1 : accéder à /login

* doit afficher la page login

### Test 2 : inscription patient

* créer un compte via `/register`
* vérifier en base que `role = patient`

### Test 3 : login patient

* login avec ce compte
* doit rediriger vers `/dashboard/patient`

### Test 4 : login admin

* login admin existant
* doit rediriger vers `/dashboard/admin`

### Test 5 : profile

* une fois connecté, ouvrir `/profile`
* doit s’afficher

---

## 9.10 Erreurs fréquentes et corrections rapides

### Erreur : `Route [login] not defined`

Cause :

* routes Breeze non chargées ou cache route
  Solution :

```bash
php artisan optimize:clear
php artisan route:list
```

### Erreur : redirection vers /dashboard au lieu de /dashboard/admin

Cause :

* la redirection n’a pas été remplacée dans `AuthenticatedSessionController`
  Solution :
* appliquer la section 9.5.2

---

## 9.11 Exercice – Adaptation des pages Breeze au template

### Objectif

Adapter les pages fournies par Breeze (`login`, `register`, `profile`) au template frontend déjà intégré.

### Travail demandé

1. Identifier les fichiers Breeze :

   ```
   resources/views/auth/
   ```
2. Adapter uniquement :

   * la structure HTML,
   * les classes CSS,
   * le layout utilisé (`@extends` ou composants).
3. **Aucune modification de logique PHP** n’est attendue.

### Pages concernées

* login
* register
* profile

### Important

* Breeze gère déjà :

  * l’authentification,
  * la validation,
  * les redirections.
* Le travail ici est **strictement visuel**.

---

## 9.12 Modèle Service – Rappel

Dans le projet fil rouge, un **service médical** :

* a un médecin responsable,
* peut recevoir plusieurs réservations.

### Modèle `Service`

**Emplacement**

```
app/Models/Service.php
```

### Attributs métier

* titre
* description
* prix
* duree
* statut
* medecin_id

### Code

```php
class Service extends Model
{
    protected $fillable = [
        'titre',
        'description',
        'prix',
        'duree',
        'statut',
        'medecin_id'
    ];

    public function reservations()
    {
        return $this->hasMany(Reservation::class);
    }

    public function medecin()
    {
        return $this->belongsTo(User::class, 'medecin_id');
    }
}
```

---

## 9.13 CRUD Admin – Gestion des médecins

### Règles

* Seul l’admin peut créer / supprimer un médecin.
* Un médecin est un `User` avec `role = medecin`.
* Un médecin peut être affecté à plusieurs services.

---

### Routes (admin uniquement)

```php
Route::middleware(['auth', 'role:admin'])->group(function () {
    Route::resource('medecins', MedecinController::class);
});
```

---

### Contrôleur `MedecinController` (extraits essentiels)

**Emplacement**

```
app/Http/Controllers/MedecinController.php
```

#### Création

```php
public function store(Request $request)
{
    User::create([
        'name' => $request->name,
        'email' => $request->email,
        'password' => bcrypt('password'),
        'role' => 'medecin',
    ]);

    return redirect()->route('medecins.index');
}
```

#### Suppression

```php
public function destroy(User $medecin)
{
    $medecin->delete();
    return redirect()->route('medecins.index');
}
```

---

### Vue – Exemple liste des médecins

```php
@foreach($medecins as $medecin)
    {{ $medecin->name }}
    <form method="POST" action="{{ route('medecins.destroy', $medecin) }}">
        @csrf @method('DELETE')
        <button>Supprimer</button>
    </form>
@endforeach
```

---

## 9.14 CRUD Admin – Création d’un service avec affectation médecin

### Principe

Lors de la création d’un service :

* l’admin choisit le médecin responsable via une liste déroulante.

---

### ServiceController – Préparation du formulaire

```php
public function create()
{
    $medecins = User::where('role', 'medecin')->get();
    return view('services.create', compact('medecins'));
}
```

---

### Vue – Formulaire service (extrait)

``` php
<select name="medecin_id">
    @foreach($medecins as $medecin)
        <option value="{{ $medecin->id }}">
            {{ $medecin->name }}
        </option>
    @endforeach
</select>
```

---

### Enregistrement (ServiceController)

```php
Service::create($request->only([
    'titre',
    'description',
    'prix',
    'duree',
    'statut',
    'medecin_id'
]));
```

---

## 9.15 Suppression d’un médecin et cohérence métier

### Règle métier

Avant de supprimer un médecin :

* vérifier s’il a des services associés,
* ou supprimer / réaffecter ses services.

### Exemple simple

```php
if ($medecin->services()->count() > 0) {
    return back()->with('error', 'Médecin lié à des services');
}
```

---

## 9.16 Vérifications attendues

* Un admin peut :

  * créer un médecin,
  * supprimer un médecin,
  * créer un service,
  * affecter un service à un médecin.
* Un médecin ne peut :
  * ni créer un autre médecin,
  * ni créer un service.
* Un patient :
  * ne voit que les services disponibles.

---

## 9.17  Données de test avec Factories et Seeders

À ce stade du projet, plusieurs fonctionnalités dépendent de données existantes :

* authentification (users),
* rôles (admin, médecin, patient),
* services médicaux,
* réservations.

Créer toutes ces données **à la main** serait :

* long,
* source d’erreurs,
* peu réaliste.

> Laravel fournit deux outils complémentaires :

* **Factories** : génèrent automatiquement des données fictives,
* **Seeders** : organisent l’insertion de ces données.

---

## 9.18 Principe général

* **Les factories décrivent comment fabriquer un objet**
* **Les seeders décident combien en créer et quand**

Schéma logique :

```
Factory  → définit la structure des données
Seeder   → lance la création en base
```

---

## 9.19 Factory – Utilisateurs (admin, médecin, patient)

### Objectif

Pouvoir disposer rapidement de comptes :

* administrateur,
* médecins,
* patients.

---

### 9.19.1 Création de la factory User

Commande :

```bash
php artisan make:factory UserFactory
```

---

### 9.19.2 Contenu essentiel de la factory

**Fichier**

```
database/factories/UserFactory.php
```

Extrait adapté au projet fil rouge :

```php
public function definition()
{
    return [
        'name' => fake()->name(),
        'email' => fake()->unique()->safeEmail(),
        'password' => bcrypt('password'),
        'role' => 'patient',
    ];
}
```

---

### 9.19.3 États spécifiques (admin / médecin)

Toujours dans `UserFactory` :

```php
public function admin()
{
    return $this->state(fn () => ['role' => 'admin']);
}

public function medecin()
{
    return $this->state(fn () => ['role' => 'medecin']);
}
```

---

## 9.20 Factory – Services médicaux

### Objectif

Créer automatiquement des services affectés à des médecins.

---

### 9.20.1 Création de la factory

```bash
php artisan make:factory ServiceFactory
```

---

### 9.20.2 Contenu essentiel

**Fichier**

```
database/factories/ServiceFactory.php
```

```php
public function definition()
{
    return [
        'titre' => fake()->words(2, true),
        'description' => fake()->sentence(),
        'prix' => fake()->numberBetween(5000, 50000),
        'duree' => fake()->numberBetween(15, 90),
        'statut' => 'actif',
        'medecin_id' => null,
    ];
}
```

Le `medecin_id` sera assigné dans le seeder.

---

## 9.21 Factory – Réservations

### Objectif

Générer des réservations réalistes pour tester :

* relations,
* dashboards,
* listing.

---

### 9.21.1 Création

```bash
php artisan make:factory ReservationFactory
```

---

### 9.21.2 Contenu minimal

```php
public function definition()
{
    return [
        'date_reservation' => fake()->dateTimeBetween('+1 day', '+1 month'),
        'heure_reservation' => fake()->time(),
        'statut' => 'en_attente',
        'commentaire' => fake()->sentence(),
    ];
}
```

---

## 9.22 Seeder principal – Organisation des données

### Objectif

Créer :

* 1 admin,
* plusieurs médecins,
* plusieurs patients,
* des services liés aux médecins,
* des réservations liées aux patients et services.

---

### 9.22.1 Modification du `DatabaseSeeder`

**Fichier**

```
database/seeders/DatabaseSeeder.php
```

---

### 9.22.2 Code structuré

```php
public function run()
{
    $admin = User::factory()->admin()->create([
        'email' => 'admin@demo.com',
    ]);

    $medecins = User::factory()
        ->medecin()
        ->count(3)
        ->create();

    $patients = User::factory()
        ->count(5)
        ->create();

    $medecins->each(function ($medecin) {
        Service::factory()
            ->count(2)
            ->create([
                'medecin_id' => $medecin->id,
            ]);
    });

    $patients->each(function ($patient) {
        Reservation::factory()
            ->count(2)
            ->create([
                'user_id' => $patient->id,
                'service_id' => Service::inRandomOrder()->first()->id,
            ]);
    });
}
```

---

## 9.23 Lancer les données de test

Commande à exécuter :

```bash
php artisan migrate:fresh --seed
```

Cette commande :

* recrée la base,
* insère toutes les données de test.

---

## 9.24 Comptes de test disponibles

Après le seeding :

* **Admin**

  * email : `admin@demo.com`
  * mot de passe : `password`

* **Médecins**

  * générés automatiquement

* **Patients**

  * générés automatiquement

---

## 9.25 Vérifications à effectuer

1. Connexion admin → accès CRUD services & médecins
2. Connexion médecin → voir ses services
3. Connexion patient → voir services & réservations
4. Relations fonctionnelles :

   * service → médecin
   * réservation → service
   * réservation → patient

---

## 9.26 Bilan du Chapitre 9

À la fin du chapitre :

* Authentification fonctionnelle (Breeze)
* Autorisation par rôle
* CRUD admin complet :
  * médecins
  * services
* Données réalistes pour tester l’application
* Base solide pour la suite (API, REST, permissions fines)
