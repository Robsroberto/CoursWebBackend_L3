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

# CHAPITRE 6 — LES CONTRÔLEURS DANS LARAVEL

## Par Robert DIASSÉ &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
 <img src="lara.webp" alt="ISI" width="50px">



---


## Application : Système de Réservation de Services Médicaux


## 1. Introduction

Dans une application Laravel, les contrôleurs jouent un rôle central.
Ils permettent de séparer correctement :

* la logique du traitement d’une requête,
* l’accès aux données (via les modèles),
* et la préparation des informations destinées aux vues.

Dans le cadre du projet fil rouge, les contrôleurs permettront :

* d’afficher les services médicaux disponibles,
* de créer et gérer les réservations,
* de gérer les services et les utilisateurs pour l’administrateur,
* de gérer les rendez-vous et services assignés au médecin.

Chaque contrôleur représente un ensemble cohérent de fonctionnalités, et chaque méthode correspond à une action précise.

---

## 2. Rappel sur la structure MVC dans Laravel

Pour situer le rôle du contrôleur :

* Une **route** reçoit une requête HTTP.
* Elle l’envoie au **contrôleur** correspondant.
* Le contrôleur utilise les **modèles** pour interagir avec la base.
* Enfin, il renvoie une **vue** avec les données nécessaires.

Exemple de route :

```php
Route::get('/services', [ServiceController::class, 'index']);
```

Cela signifie :

* L’URL `/services` appelle la méthode `index()`
* du contrôleur `ServiceController`.

Ce chapitre détaille ces contrôleurs.

---

## 3. Les différents contrôleurs utilisés dans le projet

Le projet fil rouge nécessite plusieurs contrôleurs correspondant aux rôles définis dans le cahier des charges.

### 3.1 Contrôleurs publics

* ServiceController
  Pour afficher la liste des services et leurs détails.

### 3.2 Contrôleurs utilisateur (authentifiés)

* DashboardController
  Pour orienter l’utilisateur selon son rôle (patient, médecin, administrateur).

### 3.3 Contrôleurs Patient

* ReservationController
  Pour créer, consulter et annuler ses réservations.

### 3.4 Contrôleurs Médecin

* MedecinController
  Pour consulter ses services et les réservations qui lui sont assignées.

### 3.5 Contrôleurs Administrateur

* AdminServiceController
* AdminReservationController
* AdminUserController

Chacun gère une partie de l'application pour la supervision globale.

---

## 4. ServiceController

Ce contrôleur gère l’affichage public des services.

Créer le contrôleur :

```bash
php artisan make:controller ServiceController
```

### 4.1 Méthode index() : afficher tous les services actifs

```php
public function index()
{
    $services = Service::where('statut', 'actif')
                ->with('medecin')
                ->get();

    return view('services.index', compact('services'));
}
```

Explication :

* `where('statut', 'actif')` sélectionne uniquement les services disponibles.
* `with('medecin')` charge les données du médecin associé.
* `get()` récupère tous les enregistrements trouvés.
* `compact('services')` envoie les données à la vue.

### 4.2 Méthode show() : afficher les détails d’un service

```php
public function show($id)
{
    $service = Service::with('medecin')->findOrFail($id);

    return view('services.show', compact('service'));
}
```

`findOrFail()` renvoie une erreur 404 si l'id n'existe pas.

---

## 5. DashboardController

Ce contrôleur dirige chaque utilisateur authentifié vers un tableau de bord adapté à son rôle.

Créer :

```bash
php artisan make:controller DashboardController
```

### 5.1 Méthode index()

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

Chaque utilisateur voit un espace différent.

---

## 6. ReservationController (Patient)

Créer :

```bash
php artisan make:controller ReservationController
```

### 6.1 create() : afficher le formulaire

```php
public function create($service_id)
{
    $service = Service::findOrFail($service_id);

    return view('reservations.create', compact('service'));
}
```

### 6.2 store() : enregistrer une réservation

```php
public function store(Request $request)
{
    $validated = $request->validate([
        'service_id' => 'required|exists:services,id',
        'date_reservation' => 'required|date|after_or_equal:today',
        'heure_reservation' => 'required',
        'commentaire' => 'nullable|string',
    ]);

    $validated['user_id'] = auth()->id();
    $validated['statut'] = 'en_attente';

    Reservation::create($validated);

    return redirect('/mes-reservations')
           ->with('success', 'Réservation enregistrée.');
}
```

Points essentiels :

* La validation empêche les entrées incorrectes.
* L'utilisateur connecté est associé automatiquement.
* Le statut de départ est "en attente".

### 6.3 myReservations() : afficher les réservations du patient

```php
public function myReservations()
{
    $reservations = Reservation::with('service')
                    ->where('user_id', auth()->id())
                    ->get();

    return view('reservations.patient_index', compact('reservations'));
}
```

### 6.4 cancel() : annuler

```php
public function cancel($id)
{
    $reservation = Reservation::findOrFail($id);

    if ($reservation->user_id != auth()->id()) {
        abort(403);
    }

    if ($reservation->statut !== 'en_attente') {
        return back()->with('error', 'Impossible d’annuler cette réservation.');
    }

    $reservation->update(['statut' => 'annulee']);

    return back()->with('success', 'Réservation annulée.');
}
```

Deux contrôles importants :

* la réservation doit appartenir à l'utilisateur,
* la réservation doit être encore annulable.

---

## 7. MedecinController

Ce contrôleur gère les fonctions propres au médecin.

Créer :

```bash
php artisan make:controller MedecinController
```

### 7.1 services() : afficher ses services

```php
public function services()
{
    $services = Service::where('medecin_id', auth()->id())->get();

    return view('medecin.services', compact('services'));
}
```

### 7.2 reservations() : afficher les réservations liées à ses services

```php
public function reservations()
{
    $reservations = Reservation::with(['service','user'])
                    ->whereHas('service', function ($q) {
                        $q->where('medecin_id', auth()->id());
                    })
                    ->get();

    return view('medecin.reservations', compact('reservations'));
}
```

### 7.3 updateStatus() : changer le statut d’une réservation

```php
public function updateStatus(Request $request, $id)
{
    $validated = $request->validate([
        'statut' => 'required|in:confirmée,annulée,effectuée',
    ]);

    $reservation = Reservation::findOrFail($id);

    if ($reservation->service->medecin_id != auth()->id()) {
        abort(403);
    }

    $reservation->update($validated);

    return back()->with('success', 'Statut mis à jour.');
}
```

---

## 8. Les contrôleurs Administrateur

L’administrateur gère :

* les services,
* les utilisateurs,
* toutes les réservations.

Chaque domaine a son contrôleur dédié :

* AdminServiceController
* AdminReservationController
* AdminUserController

Les méthodes suivent le même modèle que les contrôleurs précédents, mais avec une portée plus large.

Exemple pour la gestion des services :

```php
public function index()
{
    $services = Service::with('medecin')->get();

    return view('admin.services.index', compact('services'));
}
```

Chaque méthode effectue :

* récupération des données,
* validation,
* mise à jour,
* redirection avec un message de confirmation.

---

## 9. Résumé du chapitre

À ce stade, tu dois être capable de :

* lire et comprendre la structure des contrôleurs dans Laravel,
* créer une logique métier cohérente,
* connecter les routes aux contrôleurs,
* valider les données reçues,
* utiliser les modèles Eloquent pour manipuler la base,
* séparer les responsabilités selon les rôles (patient, médecin, admin).


---
