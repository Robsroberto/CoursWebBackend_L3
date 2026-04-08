

# **CHAPITRE COMPLET : API REST AVEC LARAVEL 12**

## **Table des matières**

1. Comprendre une API REST
2. Structure et configuration
3. Authentification (Sanctum)
4. Créer les endpoints API
5. Resources API
6. Validation et erreurs
7. Pagination et filtrage
8. Tester l'API (Postman)
9. Consommer l'API
10. Déploiement

---

# **PARTIE 1 : CONCEPTS FONDAMENTAUX**

## **1.1 Qu'est-ce qu'une API REST ?**

Une API REST est une interface qui permet aux clients de communiquer avec votre serveur **sans interface graphique**.

### **Comparaison Web vs API**

```
WEB CLASSIQUE:
Navigateur → /services → Laravel → Retourne HTML → Affiche page

API REST:
Client (Mobile/Frontend) → /api/services → Laravel → Retourne JSON → Affiche données
```

### **Les 4 piliers REST**

$$\text{REST} = \text{Verbes HTTP} + \text{URLs} + \text{Statuts} + \text{JSON}$$

| Verbe | Action | Exemple |
|-------|--------|---------|
| **GET** | Lire | `GET /api/services` |
| **POST** | Créer | `POST /api/services` |
| **PUT/PATCH** | Modifier | `PUT /api/services/1` |
| **DELETE** | Supprimer | `DELETE /api/services/1` |

### **Les codes HTTP importants**

```
✅ 200 OK          → La requête a réussi
✅ 201 Created     → La ressource a été créée
❌ 400 Bad Request → Données invalides
❌ 401 Unauthorized→ Pas d'authentification
❌ 403 Forbidden   → Accès refusé
❌ 404 Not Found   → Ressource inexistante
❌ 422 Validation  → Erreur de validation
❌ 500 Server      → Erreur serveur
```

### **Format JSON standard**

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "titre": "Consultation",
      "prix": 15000,
      "medecin": {
        "name": "Dr. Jean"
      }
    }
  ],
  "message": "Opération réussie"
}
```

---

# **PARTIE 2 : INSTALLATION ET CONFIGURATION**

## **2.1 Installer Sanctum (authentification API)**

Sanctum est l'outil officiel de Laravel pour sécuriser les APIs.

```bash
php artisan install:api
```

Cela va :
- Créer la migration `personal_access_tokens`
- Ajouter les fichiers de configuration

Exécutez la migration :

```bash
php artisan migrate
```

## **2.2 Configurer le modèle User**

Modifiez `app/Models/User.php` :

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Relations\HasMany;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Sanctum\HasApiTokens; // ← IMPORTANT

class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable; // ← Ajouter HasApiTokens

    // ... reste du code
}
```

## **2.3 Comprendre les routes Web vs API**

**routes/web.php** → Retourne HTML (Blade)
```php
Route::get('/services', [ServiceController::class, 'index']); // HTML
```

**routes/api.php** → Retourne JSON
```php
Route::get('/services', [ServiceApiController::class, 'index']); // JSON
```

Les routes API sont automatiquement préfixées par `/api/`

```
Route définie dans api.php : /services
URL réelle : /api/services
```

---

# **PARTIE 3 : CRÉER LES CONTRÔLEURS API**

## **3.1 Créer le contrôleur API Services**

```bash
php artisan make:controller Api/V1/ServiceApiController
```

Pourquoi V1 ? Pour **versioning** (gérer les futures mises à jour sans casser les clients)

### **Contrôleur complet**

**app/Http/Controllers/Api/V1/ServiceApiController.php**

```php
<?php

namespace App\Http\Controllers\Api\V1;

use App\Http\Controllers\Controller;
use App\Models\Service;
use Illuminate\Http\Request;
use Illuminate\Http\JsonResponse;

class ServiceApiController extends Controller
{
    /**
     * GET /api/v1/services
     * 
     * Retourner tous les services
     */
    public function index(Request $request): JsonResponse
    {
        try {
            // Récupérer les services actifs
            $services = Service::where('statut', 'actif')
                ->with('medecin') // eager loading pour éviter N+1
                ->paginate(15);

            return response()->json([
                'success' => true,
                'message' => 'Services récupérés avec succès',
                'data' => $services,
            ], 200);

        } catch (\Exception $e) {
            return response()->json([
                'success' => false,
                'message' => 'Erreur lors de la récupération des services',
                'error' => $e->getMessage(),
            ], 500);
        }
    }

    /**
     * POST /api/v1/services
     * 
     * Créer un nouveau service (admin uniquement)
     */
    public function store(Request $request): JsonResponse
    {
        // Valider les données
        $validated = $request->validate([
            'titre' => 'required|string|max:255',
            'description' => 'nullable|string',
            'prix' => 'required|numeric|min:0',
            'duree' => 'required|integer|min:15',
            'medecin_id' => 'nullable|exists:users,id',
            'statut' => 'required|in:actif,inactif',
        ]);

        try {
            $service = Service::create($validated);

            return response()->json([
                'success' => true,
                'message' => 'Service créé avec succès',
                'data' => $service,
            ], 201); // 201 = Created

        } catch (\Exception $e) {
            return response()->json([
                'success' => false,
                'message' => 'Erreur lors de la création',
                'error' => $e->getMessage(),
            ], 500);
        }
    }

    /**
     * GET /api/v1/services/{service}
     * 
     * Afficher un service spécifique
     */
    public function show(Service $service): JsonResponse
    {
        try {
            $service->load('medecin', 'reservations');

            return response()->json([
                'success' => true,
                'message' => 'Service trouvé',
                'data' => $service,
            ], 200);

        } catch (\Exception $e) {
            return response()->json([
                'success' => false,
                'message' => 'Service non trouvé',
                'error' => $e->getMessage(),
            ], 404);
        }
    }

    /**
     * PUT /api/v1/services/{service}
     * 
     * Mettre à jour un service (admin uniquement)
     */
    public function update(Request $request, Service $service): JsonResponse
    {
        $validated = $request->validate([
            'titre' => 'required|string|max:255',
            'description' => 'nullable|string',
            'prix' => 'required|numeric|min:0',
            'duree' => 'required|integer|min:15',
            'medecin_id' => 'nullable|exists:users,id',
            'statut' => 'required|in:actif,inactif',
        ]);

        try {
            $service->update($validated);

            return response()->json([
                'success' => true,
                'message' => 'Service mis à jour avec succès',
                'data' => $service,
            ], 200);

        } catch (\Exception $e) {
            return response()->json([
                'success' => false,
                'message' => 'Erreur lors de la mise à jour',
                'error' => $e->getMessage(),
            ], 500);
        }
    }

    /**
     * DELETE /api/v1/services/{service}
     * 
     * Supprimer un service (admin uniquement)
     */
    public function destroy(Service $service): JsonResponse
    {
        try {
            $service->delete();

            return response()->json([
                'success' => true,
                'message' => 'Service supprimé avec succès',
            ], 200);

        } catch (\Exception $e) {
            return response()->json([
                'success' => false,
                'message' => 'Erreur lors de la suppression',
                'error' => $e->getMessage(),
            ], 500);
        }
    }
}
```

## **3.2 Créer le contrôleur API Authentification**

```bash
php artisan make:controller Api/V1/AuthApiController
```

**app/Http/Controllers/Api/V1/AuthApiController.php**

```php
<?php

namespace App\Http\Controllers\Api\V1;

use App\Http\Controllers\Controller;
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Http\JsonResponse;
use Illuminate\Support\Facades\Hash;

class AuthApiController extends Controller
{
    /**
     * POST /api/v1/auth/login
     * 
     * Connexion et génération de token
     */
    public function login(Request $request): JsonResponse
    {
        // Valider les données
        $validated = $request->validate([
            'email' => 'required|email',
            'password' => 'required|min:6',
        ]);

        // Chercher l'utilisateur par email
        $user = User::where('email', $validated['email'])->first();

        // Vérifier si l'utilisateur existe et le mot de passe est correct
        if (!$user || !Hash::check($validated['password'], $user->password)) {
            return response()->json([
                'success' => false,
                'message' => 'Identifiants incorrects',
            ], 401); // 401 = Unauthorized
        }

        // Créer un token
        $token = $user->createToken('api-token')->plainTextToken;

        return response()->json([
            'success' => true,
            'message' => 'Connexion réussie',
            'data' => [
                'user' => [
                    'id' => $user->id,
                    'name' => $user->name,
                    'email' => $user->email,
                    'role' => $user->role,
                ],
                'token' => $token,
            ],
        ], 200);
    }

    /**
     * POST /api/v1/auth/logout
     * 
     * Déconnexion et suppression du token
     */
    public function logout(Request $request): JsonResponse
    {
        // Supprimer le token actuel
        $request->user()->currentAccessToken()->delete();

        return response()->json([
            'success' => true,
            'message' => 'Déconnexion réussie',
        ], 200);
    }

    /**
     * GET /api/v1/auth/me
     * 
     * Récupérer les infos de l'utilisateur connecté
     */
    public function me(Request $request): JsonResponse
    {
        return response()->json([
            'success' => true,
            'data' => $request->user(),
        ], 200);
    }

    /**
     * POST /api/v1/auth/register
     * 
     * Inscription d'un nouveau patient
     */
    public function register(Request $request): JsonResponse
    {
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:users',
            'password' => 'required|min:6|confirmed',
        ]);

        try {
            // Créer l'utilisateur
            $user = User::create([
                'name' => $validated['name'],
                'email' => $validated['email'],
                'password' => Hash::make($validated['password']),
                'role' => 'patient', // Les nouveaux inscrits sont patients
            ]);

            // Créer un token
            $token = $user->createToken('api-token')->plainTextToken;

            return response()->json([
                'success' => true,
                'message' => 'Inscription réussie',
                'data' => [
                    'user' => $user,
                    'token' => $token,
                ],
            ], 201);

        } catch (\Exception $e) {
            return response()->json([
                'success' => false,
                'message' => 'Erreur lors de l\'inscription',
                'error' => $e->getMessage(),
            ], 500);
        }
    }
}
```

## **3.3 Créer le contrôleur API Réservations**

```bash
php artisan make:controller Api/V1/ReservationApiController
```

**app/Http/Controllers/Api/V1/ReservationApiController.php**

```php
<?php

namespace App\Http\Controllers\Api\V1;

use App\Http\Controllers\Controller;
use App\Models\Reservation;
use Illuminate\Http\Request;
use Illuminate\Http\JsonResponse;

class ReservationApiController extends Controller
{
    /**
     * GET /api/v1/reservations
     * 
     * Lister les réservations (avec filtrage)
     */
    public function index(Request $request): JsonResponse
    {
        try {
            $query = Reservation::with(['service', 'patient']);

            // Filtrer par patient si paramètre fourni
            if ($request->has('patient_id')) {
                $query->where('patient_id', $request->patient_id);
            }

            // Filtrer par service si paramètre fourni
            if ($request->has('service_id')) {
                $query->where('service_id', $request->service_id);
            }

            // Filtrer par statut
            if ($request->has('statut')) {
                $query->where('statut', $request->statut);
            }

            $perPage = $request->get('per_page', 15);
            $reservations = $query->paginate($perPage);

            return response()->json([
                'success' => true,
                'message' => 'Réservations récupérées',
                'data' => $reservations,
            ], 200);

        } catch (\Exception $e) {
            return response()->json([
                'success' => false,
                'message' => 'Erreur lors de la récupération',
                'error' => $e->getMessage(),
            ], 500);
        }
    }

    /**
     * POST /api/v1/reservations
     * 
     * Créer une réservation (patient)
     */
    public function store(Request $request): JsonResponse
    {
        $validated = $request->validate([
            'service_id' => 'required|exists:services,id',
            'date_reservation' => 'required|date|after_or_equal:today',
            'heure_reservation' => 'required|date_format:H:i',
            'commentaire' => 'nullable|string',
        ]);

        try {
            $validated['user_id'] = $request->user()->id;
            $validated['statut'] = 'en_attente';

            $reservation = Reservation::create($validated);
            $reservation->load('service', 'patient');

            return response()->json([
                'success' => true,
                'message' => 'Réservation créée avec succès',
                'data' => $reservation,
            ], 201);

        } catch (\Exception $e) {
            return response()->json([
                'success' => false,
                'message' => 'Erreur lors de la création',
                'error' => $e->getMessage(),
            ], 500);
        }
    }

    /**
     * GET /api/v1/reservations/{reservation}
     * 
     * Afficher une réservation
     */
    public function show(Reservation $reservation): JsonResponse
    {
        try {
            $reservation->load('service', 'patient');

            return response()->json([
                'success' => true,
                'message' => 'Réservation trouvée',
                'data' => $reservation,
            ], 200);

        } catch (\Exception $e) {
            return response()->json([
                'success' => false,
                'message' => 'Réservation non trouvée',
                'error' => $e->getMessage(),
            ], 404);
        }
    }

    /**
     * PUT /api/v1/reservations/{reservation}
     * 
     * Mettre à jour une réservation
     */
    public function update(Request $request, Reservation $reservation): JsonResponse
    {
        // Vérifier que c'est le patient qui fait la demande
        if ($reservation->patient_id !== $request->user()->id && !$request->user()->isAdmin()) {
            return response()->json([
                'success' => false,
                'message' => 'Action non autorisée',
            ], 403);
        }

        $validated = $request->validate([
            'date_reservation' => 'required|date',
            'heure_reservation' => 'required|date_format:H:i',
            'commentaire' => 'nullable|string',
        ]);

        try {
            $reservation->update($validated);

            return response()->json([
                'success' => true,
                'message' => 'Réservation mise à jour',
                'data' => $reservation,
            ], 200);

        } catch (\Exception $e) {
            return response()->json([
                'success' => false,
                'message' => 'Erreur lors de la mise à jour',
                'error' => $e->getMessage(),
            ], 500);
        }
    }

    /**
     * DELETE /api/v1/reservations/{reservation}
     * 
     * Annuler une réservation
     */
    public function destroy(Request $request, Reservation $reservation): JsonResponse
    {
        // Vérifier les permissions
        if ($reservation->patient_id !== $request->user()->id && !$request->user()->isAdmin()) {
            return response()->json([
                'success' => false,
                'message' => 'Action non autorisée',
            ], 403);
        }

        // Vérifier que la réservation peut être annulée
        if (!$reservation->peutEtreAnnulee()) {
            return response()->json([
                'success' => false,
                'message' => 'Cette réservation ne peut pas être annulée',
            ], 422);
        }

        try {
            $reservation->update(['statut' => 'annulee']);

            return response()->json([
                'success' => true,
                'message' => 'Réservation annulée',
            ], 200);

        } catch (\Exception $e) {
            return response()->json([
                'success' => false,
                'message' => 'Erreur lors de l\'annulation',
                'error' => $e->getMessage(),
            ], 500);
        }
    }
}
```

---

# **PARTIE 4 : DÉFINIR LES ROUTES API**

## **4.1 Structure des routes**

Créez **routes/api.php** avec une structure versionnée :

**routes/api.php**

```php
<?php

use App\Http\Controllers\Api\V1\AuthApiController;
use App\Http\Controllers\Api\V1\ServiceApiController;
use App\Http\Controllers\Api\V1\ReservationApiController;
use Illuminate\Support\Facades\Route;

/**
 * ============================================
 * API V1 - PUBLIQUE (pas d'authentification)
 * ============================================
 */
Route::prefix('v1')->group(function () {
    
    // Test de l'API
    Route::get('/ping', function () {
        return response()->json([
            'success' => true,
            'message' => 'API V1 fonctionne correctement',
            'timestamp' => now(),
        ]);
    });

    /**
     * Routes AUTHENTIFICATION (publiques)
     */
    Route::post('/auth/login', [AuthApiController::class, 'login']);
    Route::post('/auth/register', [AuthApiController::class, 'register']);

    /**
     * Routes SERVICES (publiques)
     */
    Route::get('/services', [ServiceApiController::class, 'index']);
    Route::get('/services/{service}', [ServiceApiController::class, 'show']);

    /**
     * ============================================
     * API V1 - PROTÉGÉES (authentification requise)
     * ============================================
     */
    Route::middleware('auth:sanctum')->group(function () {

        /**
         * Routes AUTHENTIFICATION (protégées)
         */
        Route::post('/auth/logout', [AuthApiController::class, 'logout']);
        Route::get('/auth/me', [AuthApiController::class, 'me']);

        /**
         * Routes RÉSERVATIONS (tous les utilisateurs)
         */
        Route::apiResource('reservations', ReservationApiController::class);

        /**
         * Routes SERVICES (admin seulement)
         */
        Route::middleware('role:admin')->group(function () {
            Route::post('/services', [ServiceApiController::class, 'store']);
            Route::put('/services/{service}', [ServiceApiController::class, 'update']);
            Route::delete('/services/{service}', [ServiceApiController::class, 'destroy']);
        });
    });
});
```

### **Explication de la structure**

```
/api/v1/                        → Base de l'API v1
├─ /ping                        → Test d'accès
├─ /auth/login                  → Connexion (public)
├─ /auth/register               → Inscription (public)
├─ /services                    → Lister services (public)
├─ /services/{id}               → Détail service (public)
└─ [Protégé par auth:sanctum]
   ├─ /auth/logout              → Déconnexion
   ├─ /auth/me                  → Infos utilisateur
   ├─ /reservations             → CRUD réservations
   ├─ /reservations/{id}
   └─ [Admin seulement]
      ├─ POST /services         → Créer service
      ├─ PUT /services/{id}     → Modifier service
      └─ DELETE /services/{id}  → Supprimer service
```

## **4.2 Comprendre apiResource vs resource**

```php
// Dans une route WEB (retourne des vues HTML)
Route::resource('services', ServiceController::class);
// Génère: create, store, edit, update, show, index, destroy

// Dans une route API (retourne du JSON)
Route::apiResource('reservations', ReservationApiController::class);
// Génère SEULEMENT: store, show, update, destroy, index
// (pas de create/edit car pas de HTML)
```

---

# **PARTIE 5 : CRÉER LES RESOURCES API**

## **5.1 Pourquoi les Resources ?**

Les Resources contrôlent exactement **quels champs** sont retournés dans la réponse JSON.

```bash
php artisan make:resource ServiceResource
php artisan make:resource ReservationResource
php artisan make:resource UserResource
```

## **5.2 Créer les Resources**

**app/Http/Resources/ServiceResource.php**

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class ServiceResource extends JsonResource
{
    /**
     * Transform the resource into an array.
     */
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'titre' => $this->titre,
            'description' => $this->description,
            'prix' => $this->prix,
            'prix_formate' => $this->prixFormate(),
            'duree' => $this->duree,
            'statut' => $this->statut,
            'medecin' => new UserResource($this->whenLoaded('medecin')),
            'nombre_reservations' => $this->reservations_count ?? $this->reservations()->count(),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }
}
```

**app/Http/Resources/ReservationResource.php**

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class ReservationResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'patient' => new UserResource($this->whenLoaded('patient')),
            'service' => new ServiceResource($this->whenLoaded('service')),
            'date_reservation' => $this->date_reservation->format('Y-m-d'),
            'heure_reservation' => $this->heure_reservation,
            'datetime' => $this->date_reservation->format('Y-m-d') . ' ' . $this->heure_reservation,
            'statut' => $this->statut,
            'statut_label' => $this->getStatutLabel(),
            'commentaire' => $this->commentaire,
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }
}
```

**app/Http/Resources/UserResource.php**

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'role' => $this->role,
            'created_at' => $this->created_at,
        ];
    }
}
```

## **5.3 Utiliser les Resources dans les contrôleurs**

Modifiez **app/Http/Controllers/Api/V1/ServiceApiController.php** :

```php
<?php

namespace App\Http\Controllers\Api\V1;

use App\Http\Controllers\Controller;
use App\Http\Resources\ServiceResource;
use App\Models\Service;
use Illuminate\Http\Request;
use Illuminate\Http\JsonResponse;

class ServiceApiController extends Controller
{
    public function index(Request $request): JsonResponse
    {
        try {
            $services = Service::where('statut', 'actif')
                ->with('medecin')
                ->withCount('reservations')
                ->paginate(15);

            return response()->json([
                'success' => true,
                'message' => 'Services récupérés avec succès',
                'data' => ServiceResource::collection($services),
            ], 200);

        } catch (\Exception $e) {
            return response()->json([
                'success' => false,
                'message' => 'Erreur lors de la récupération',
                'error' => $e->getMessage(),
            ], 500);
        }
    }

    public function show(Service $service): JsonResponse
    {
        return response()->json([
            'success' => true,
            'data' => new ServiceResource($service->load('medecin')),
        ], 200);
    }

    public function store(Request $request): JsonResponse
    {
        $validated = $request->validate([
            'titre' => 'required|string|max:255',
            'description' => 'nullable|string',
            'prix' => 'required|numeric|min:0',
            'duree' => 'required|integer|min:15',
            'medecin_id' => 'nullable|exists:users,id',
            'statut' => 'required|in:actif,inactif',
        ]);

        try {
            $service = Service::create($validated);

            return response()->json([
                'success' => true,
                'message' => 'Service créé',
                'data' => new ServiceResource($service),
            ], 201);

        } catch (\Exception $e) {
            return response()->json([
                'success' => false,
                'error' => $e->getMessage(),
            ], 500);
        }
    }

    // ... autres méthodes
}
```

---

# **PARTIE 6 : VALIDATION DES ERREURS API**

## **6.1 Gérer les erreurs de validation**

Créez une classe d'exception personnalisée :

```bash
php artisan make:exception ApiException
```

**app/Exceptions/ApiException.php**

```php
<?php

namespace App\Exceptions;

use Exception;
use Illuminate\Http\JsonResponse;

class ApiException extends Exception
{
    protected $statusCode;
    protected $errors;

    public function __construct(
        string $message = 'Erreur API',
        int $statusCode = 400,
        array $errors = [],
        int $code = 0
    ) {
        parent::__construct($message, $code);
        $this->statusCode = $statusCode;
        $this->errors = $errors;
    }

    public function render(): JsonResponse
    {
        return response()->json([
            'success' => false,
            'message' => $this->message,
            'errors' => $this->errors,
        ], $this->statusCode);
    }
}
```

## **6.2 Gérer les erreurs globales**

Modifiez **app/Exceptions/Handler.php** :

```php
<?php

namespace App\Exceptions;

use Illuminate\Foundation\Exceptions\Handler as ExceptionHandler;
use Illuminate\Http\JsonResponse;
use Throwable;

class Handler extends ExceptionHandler
{
    public function register(): void
    {
        $this->reportable(function (Throwable $e) {
            //
        });
    }

    public function render($request, Throwable $exception)
    {
        // Si c'est une requête API, retourner JSON
        if ($request->is('api/*')) {
            
            // Erreur de validation
            if ($exception instanceof \Illuminate\Validation\ValidationException) {
                return response()->json([
                    'success' => false,
                    'message' => 'Erreur de validation',
                    'errors' => $exception->errors(),
                ], 422);
            }

            // Ressource non trouvée
            if ($exception instanceof \Illuminate\Database\Eloquent\ModelNotFoundException) {
                return response()->json([
                    'success' => false,
                    'message' => 'Ressource non trouvée',
                ], 404);
            }

            // Non autorisé
            if ($exception instanceof \Illuminate\Auth\AuthenticationException) {
                return response()->json([
                    'success' => false,
                    'message' => 'Authentification requise',
                ], 401);
            }

            // Accès refusé
            if ($exception instanceof \Illuminate\Auth\Access\AuthorizationException) {
                return response()->json([
                    'success' => false,
                    'message' => 'Accès refusé',
                ], 403);
            }

            // Erreur générique
            return response()->json([
                'success' => false,
                'message' => $exception->getMessage() ?: 'Erreur serveur',
            ], 500);
        }

        return parent::render($request, $exception);
    }
}
```

---

# **PARTIE 7 : PAGINATION ET FILTRAGE**

## **7.1 Ajouter la pagination**

La pagination est automatique avec `paginate()` :

```php
// Dans le contrôleur
$services = Service::paginate(15); // 15 par page

// La réponse inclut automatiquement:
// - data: les éléments
// - current_page
// - last_page
// - total
// - per_page
```

## **7.2 Utiliser la pagination**

```bash
# Première page (défaut)
GET /api/v1/services

# Deuxième page
GET /api/v1/services?page=2

# 20 éléments par page
GET /api/v1/services?per_page=20

# Combiner
GET /api/v1/services?page=2&per_page=20
```

## **7.3 Ajouter des filtres**

Modifiez le contrôleur Services :

```php
public function index(Request $request): JsonResponse
{
    try {
        $query = Service::where('statut', 'actif')
            ->with('medecin');

        // Filtre: par médecin
        if ($request->has('medecin_id')) {
            $query->where('medecin_id', $request->medecin_id);
        }

        // Filtre: par prix minimum
        if ($request->has('prix_min')) {
            $query->where('prix', '>=', $request->prix_min);
        }

        // Filtre: par prix maximum
        if ($request->has('prix_max')) {
            $query->where('prix', '<=', $request->prix_max);
        }

        // Filtre: recherche par titre
        if ($request->has('search')) {
            $query->where('titre', 'like', '%' . $request->search . '%');
        }

        // Tri
        $sortBy = $request->get('sort_by', 'created_at');
        $sortOrder = $request->get('sort_order', 'desc');
        $query->orderBy($sortBy, $sortOrder);

        $perPage = $request->get('per_page', 15);
        $services = $query->paginate($perPage);

        return response()->json([
            'success' => true,
            'data' => ServiceResource::collection($services),
        ], 200);

    } catch (\Exception $e) {
        return response()->json([
            'success' => false,
            'error' => $e->getMessage(),
        ], 500);
    }
}
```

## **7.4 Exemples de filtrage**

```bash
# Filtrer par médecin
GET /api/v1/services?medecin_id=2

# Filtrer par prix minimum
GET /api/v1/services?prix_min=10000

# Filtrer par prix minimum ET maximum
GET /api/v1/services?prix_min=10000&prix_max=50000

# Recherche
GET /api/v1/services?search=consultation

# Tri
GET /api/v1/services?sort_by=prix&sort_order=asc

# Combiner filtres et pagination
GET /api/v1/services?search=consultation&prix_min=5000&page=2&per_page=10
```

---

# **PARTIE 8 : TESTER L'API AVEC POSTMAN**

## **8.1 Installer Postman**

Téléchargez Postman : https://www.postman.com/downloads/

## **8.2 Structure d'une requête API**

Tous les tests suivent ce pattern :

```
┌─ Méthode HTTP (GET, POST, PUT, DELETE)
├─ URL (http://localhost:8000/api/v1/services)
├─ Headers (Authorization: Bearer TOKEN)
├─ Body (JSON pour POST/PUT)
└─ Réponse (JSON)
```

## **8.3 Workflow complet de test**

### **Étape 1 : Tester /ping**

```
GET http://localhost:8000/api/v1/ping

Réponse :
{
  "success": true,
  "message": "API V1 fonctionne correctement",
  "timestamp": "2026-02-27T22:30:00Z"
}
```

### **Étape 2 : Inscription**

```
POST http://localhost:8000/api/v1/auth/register
Content-Type: application/json

{
  "name": "Jean Dupont",
  "email": "jean@example.com",
  "password": "password123",
  "password_confirmation": "password123"
}

Réponse (201) :
{
  "success": true,
  "message": "Inscription réussie",
  "data": {
    "user": {
      "id": 15,
      "name": "Jean Dupont",
      "email": "jean@example.com",
      "role": "patient"
    },
    "token": "1|abc123def456..."
  }
}
```

### **Étape 3 : Connexion**

```
POST http://localhost:8000/api/v1/auth/login
Content-Type: application/json

{
  "email": "admin@demo.com",
  "password": "password"
}

Réponse (200) :
{
  "success": true,
  "message": "Connexion réussie",
  "data": {
    "user": {
      "id": 1,
      "name": "Admin Demo",
      "email": "admin@demo.com",
      "role": "admin"
    },
    "token": "2|xyz789abc..."
  }
}

👉 COPIER LE TOKEN pour l'utiliser dans les autres requêtes
```

### **Étape 4 : Configurer l'authentification dans Postman**

1. Créez une collection "Réservation API"
2. Onglet "Authorization"
3. Type: `Bearer Token`
4. Token: `{le token reçu}`

Ou ajoutez le header manuellement :

```
Authorization: Bearer 2|xyz789abc...
```

### **Étape 5 : Lister les services**

```
GET http://localhost:8000/api/v1/services

Réponse (200) :
{
  "success": true,
  "message": "Services récupérés avec succès",
  "data": {
    "data": [
      {
        "id": 1,
        "titre": "Consultation générale",
        "description": "Consultation de base",
        "prix": 15000,
        "prix_formate": "15 000 F",
        "duree": 30,
        "statut": "actif",
        "medecin": {
          "id": 2,
          "name": "Dr. Jean",
          "email": "jean@example.com",
          "role": "medecin"
        },
        "nombre_reservations": 5,
        "created_at": "2026-02-27T10:00:00Z",
        "updated_at": "2026-02-27T10:00:00Z"
      }
    ],
    "current_page": 1,
    "last_page": 2,
    "total": 25,
    "per_page": 15
  }
}
```

### **Étape 6 : Créer une réservation**

```
POST http://localhost:8000/api/v1/reservations
Content-Type: application/json
Authorization: Bearer {TOKEN}

{
  "service_id": 1,
  "date_reservation": "2026-03-15",
  "heure_reservation": "10:30",
  "commentaire": "Je souffre de migraines"
}

Réponse (201) :
{
  "success": true,
  "message": "Réservation créée avec succès",
  "data": {
    "id": 1,
    "patient": {
      "id": 15,
      "name": "Jean Dupont",
      "email": "jean@example.com",
      "role": "patient"
    },
    "service": {
      "id": 1,
      "titre": "Consultation générale",
      ...
    },
    "date_reservation": "2026-03-15",
    "heure_reservation": "10:30",
    "datetime": "2026-03-15 10:30",
    "statut": "en_attente",
    "statut_label": "En attente",
    "commentaire": "Je souffre de migraines"
  }
}
```

### **Étape 7 : Récupérer mes réservations**

```
GET http://localhost:8000/api/v1/reservations?patient_id=15
Authorization: Bearer {TOKEN}

Réponse :
{
  "success": true,
  "message": "Réservations récupérées",
  "data": {
    "data": [...],
    "current_page": 1,
    "total": 2
  }
}
```

### **Étape 8 : Mettre à jour une réservation**

```
PUT http://localhost:8000/api/v1/reservations/1
Content-Type: application/json
Authorization: Bearer {TOKEN}

{
  "date_reservation": "2026-03-20",
  "heure_reservation": "14:00",
  "commentaire": "Finalement une autre date"
}

Réponse (200) :
{
  "success": true,
  "message": "Réservation mise à jour",
  "data": { ... }
}
```

### **Étape 9 : Annuler une réservation**

```
DELETE http://localhost:8000/api/v1/reservations/1
Authorization: Bearer {TOKEN}

Réponse (200) :
{
  "success": true,
  "message": "Réservation annulée"
}
```

### **Étape 10 : Créer un service (admin)**

```
POST http://localhost:8000/api/v1/services
Content-Type: application/json
Authorization: Bearer {ADMIN_TOKEN}

{
  "titre": "Consultation cardiologie",
  "description": "Consultation avec ECG",
  "prix": 25000,
  "duree": 45,
  "medecin_id": 2,
  "statut": "actif"
}

Réponse (201) :
{
  "success": true,
  "message": "Service créé",
  "data": { ... }
}
```

## **8.4 Tableau de tous les endpoints**

| Méthode | Endpoint | Public | Auth | Admin | Description |
|---------|----------|--------|------|-------|-------------|
| GET | `/api/v1/ping` | ✅ | ❌ | ❌ | Test API |
| POST | `/api/v1/auth/login` | ✅ | ❌ | ❌ | Connexion |
| POST | `/api/v1/auth/register` | ✅ | ❌ | ❌ | Inscription |
| POST | `/api/v1/auth/logout` | ❌ | ✅ | ❌ | Déconnexion |
| GET | `/api/v1/auth/me` | ❌ | ✅ | ❌ | Infos utilisateur |
| GET | `/api/v1/services` | ✅ | ❌ | ❌ | Lister services |
| GET | `/api/v1/services/{id}` | ✅ | ❌ | ❌ | Détail service |
| POST | `/api/v1/services` | ❌ | ❌ | ✅ | Créer service |
| PUT | `/api/v1/services/{id}` | ❌ | ❌ | ✅ | Modifier service |
| DELETE | `/api/v1/services/{id}` | ❌ | ❌ | ✅ | Supprimer service |
| GET | `/api/v1/reservations` | ❌ | ✅ | ✅ | Lister réservations |
| GET | `/api/v1/reservations/{id}` | ❌ | ✅ | ✅ | Détail réservation |
| POST | `/api/v1/reservations` | ❌ | ✅ | ✅ | Créer réservation |
| PUT | `/api/v1/reservations/{id}` | ❌ | ✅ | ✅ | Modifier réservation |
| DELETE | `/api/v1/reservations/{id}` | ❌ | ✅ | ✅ | Annuler réservation |

---

# **PARTIE 9 : CONSOMMER L'API CÔTÉ FRONTEND**

## **9.1 Créer un service API en JavaScript/Vue.js**

Créez **resources/js/services/api.js** :

```javascript
/**
 * Service API pour communiquer avec le backend Laravel
 */

const API_BASE_URL = 'http://localhost:8000/api/v1';

class ApiService {
    constructor() {
        this.token = localStorage.getItem('api_token');
        this.user = localStorage.getItem('user') ? 
            JSON.parse(localStorage.getItem('user')) : null;
    }

    /**
     * Effectuer une requête API générique
     */
    async request(method, endpoint, data = null) {
        const headers = {
            'Content-Type': 'application/json',
            'Accept': 'application/json',
        };

        // Ajouter le token d'authentification si disponible
        if (this.token) {
            headers['Authorization'] = `Bearer ${this.token}`;
        }

        const config = {
            method,
            headers,
        };

        if (data) {
            config.body = JSON.stringify(data);
        }

        try {
            const response = await fetch(`${API_BASE_URL}${endpoint}`, config);
            const result = await response.json();

            // Si non authentifié, supprimer le token
            if (response.status === 401) {
                this.logout();
            }

            return {
                status: response.status,
                ...result,
            };

        } catch (error) {
            return {
                success: false,
                message: 'Erreur réseau',
                error: error.message,
            };
        }
    }

    /**
     * =====================================
     * AUTHENTIFICATION
     * =====================================
     */

    async login(email, password) {
        const response = await this.request('POST', '/auth/login', {
            email,
            password,
        });

        if (response.success) {
            // Sauvegarder le token et l'utilisateur
            this.token = response.data.token;
            this.user = response.data.user;
            localStorage.setItem('api_token', this.token);
            localStorage.setItem('user', JSON.stringify(this.user));
        }

        return response;
    }

    async register(name, email, password, passwordConfirmation) {
        return await this.request('POST', '/auth/register', {
            name,
            email,
            password,
            password_confirmation: passwordConfirmation,
        });
    }

    async logout() {
        await this.request('POST', '/auth/logout');
        this.token = null;
        this.user = null;
        localStorage.removeItem('api_token');
        localStorage.removeItem('user');
    }

    async getMe() {
        return await this.request('GET', '/auth/me');
    }

    /**
     * =====================================
     * SERVICES
     * =====================================
     */

    async getServices(page = 1, perPage = 15, filters = {}) {
        let endpoint = `/services?page=${page}&per_page=${perPage}`;

        // Ajouter les filtres à l'URL
        if (filters.medecin_id) {
            endpoint += `&medecin_id=${filters.medecin_id}`;
        }
        if (filters.prix_min) {
            endpoint += `&prix_min=${filters.prix_min}`;
        }
        if (filters.prix_max) {
            endpoint += `&prix_max=${filters.prix_max}`;
        }
        if (filters.search) {
            endpoint += `&search=${filters.search}`;
        }

        return await this.request('GET', endpoint);
    }

    async getService(id) {
        return await this.request('GET', `/services/${id}`);
    }

    async createService(data) {
        return await this.request('POST', '/services', data);
    }

    async updateService(id, data) {
        return await this.request('PUT', `/services/${id}`, data);
    }

    async deleteService(id) {
        return await this.request('DELETE', `/services/${id}`);
    }

    /**
     * =====================================
     * RÉSERVATIONS
     * =====================================
     */

    async getReservations(page = 1, filters = {}) {
        let endpoint = `/reservations?page=${page}`;

        if (filters.patient_id) {
            endpoint += `&patient_id=${filters.patient_id}`;
        }
        if (filters.service_id) {
            endpoint += `&service_id=${filters.service_id}`;
        }
        if (filters.statut) {
            endpoint += `&statut=${filters.statut}`;
        }

        return await this.request('GET', endpoint);
    }

    async getReservation(id) {
        return await this.request('GET', `/reservations/${id}`);
    }

    async createReservation(data) {
        return await this.request('POST', '/reservations', data);
    }

    async updateReservation(id, data) {
        return await this.request('PUT', `/reservations/${id}`, data);
    }

    async cancelReservation(id) {
        return await this.request('DELETE', `/reservations/${id}`);
    }

    /**
     * Helpers
     */
    isAuthenticated() {
        return !!this.token;
    }

    isAdmin() {
        return this.user && this.user.role === 'admin';
    }

    isMedecin() {
        return this.user && this.user.role === 'medecin';
    }

    isPatient() {
        return this.user && this.user.role === 'patient';
    }
}

export default new ApiService();
```

## **9.2 Utiliser le service API avec Vue.js**

Exemple de composant Vue :

```vue
<!-- resources/js/components/ServicesList.vue -->

<template>
    <div class="services-container">
        <h1>Services Médicaux</h1>

        <!-- Barre de recherche et filtres -->
        <div class="filters mb-4">
            <input 
                v-model="search" 
                type="text" 
                placeholder="Rechercher..."
                @input="fetchServices(1)"
                class="form-control mb-2"
            >
            <input 
                v-model.number="prix_min" 
                type="number" 
                placeholder="Prix min"
                @change="fetchServices(1)"
                class="form-control mb-2"
            >
        </div>

        <!-- Liste des services -->
        <div v-if="loading" class="alert alert-info">
            Chargement...
        </div>

        <div v-else-if="services.length === 0" class="alert alert-warning">
            Aucun service trouvé
        </div>

        <div v-else class="row">
            <div v-for="service in services" :key="service.id" class="col-md-4 mb-3">
                <div class="card">
                    <div class="card-body">
                        <h5 class="card-title">{{ service.titre }}</h5>
                        <p class="card-text">{{ service.description }}</p>
                        <div class="d-flex justify-content-between">
                            <span class="badge bg-primary">{{ service.duree }} min</span>
                            <strong>{{ service.prix_formate }}</strong>
                        </div>
                        <button 
                            v-if="isAuthenticated && isPatient"
                            @click="goToReservation(service.id)"
                            class="btn btn-sm btn-primary mt-2 w-100"
                        >
                            Réserver
                        </button>
                    </div>
                </div>
            </div>
        </div>

        <!-- Pagination -->
        <nav v-if="totalPages > 1">
            <ul class="pagination">
                <li class="page-item" :class="{ disabled: currentPage === 1 }">
                    <button @click="fetchServices(1)" class="page-link">1</button>
                </li>
                <li class="page-item" :class="{ disabled: currentPage === totalPages }">
                    <button @click="fetchServices(totalPages)" class="page-link">{{ totalPages }}</button>
                </li>
            </ul>
        </nav>
    </div>
</template>

<script>
import apiService from '@/services/api';

export default {
    name: 'ServicesList',
    data() {
        return {
            services: [],
            loading: false,
            currentPage: 1,
            totalPages: 1,
            search: '',
            prix_min: null,
        };
    },
    computed: {
        isAuthenticated() {
            return apiService.isAuthenticated();
        },
        isPatient() {
            return apiService.isPatient();
        },
    },
    methods: {
        async fetchServices(page = 1) {
            this.loading = true;

            try {
                const response = await apiService.getServices(page, 15, {
                    search: this.search,
                    prix_min: this.prix_min,
                });

                if (response.success) {
                    this.services = response.data.data;
                    this.currentPage = response.data.current_page;
                    this.totalPages = response.data.last_page;
                } else {
                    console.error('Erreur:', response.message);
                }
            } catch (error) {
                console.error('Erreur réseau:', error);
            } finally {
                this.loading = false;
            }
        },
        goToReservation(serviceId) {
            this.$router.push(`/reservation/${serviceId}`);
        },
    },
    mounted() {
        this.fetchServices();
    },
};
</script>
```

## **9.3 Composant de connexion**

```vue
<!-- resources/js/components/LoginForm.vue -->

<template>
    <div class="login-form">
        <form @submit.prevent="handleLogin">
            <div class="form-group mb-3">
                <label>Email</label>
                <input 
                    v-model="email" 
                    type="email" 
                    class="form-control"
                    required
                >
            </div>

            <div class="form-group mb-3">
                <label>Mot de passe</label>
                <input 
                    v-model="password" 
                    type="password" 
                    class="form-control"
                    required
                >
            </div>

            <button type="submit" class="btn btn-primary w-100" :disabled="loading">
                {{ loading ? 'Connexion...' : 'Se connecter' }}
            </button>
        </form>

        <div v-if="error" class="alert alert-danger mt-3">
            {{ error }}
        </div>
    </div>
</template>

<script>
import apiService from '@/services/api';

export default {
    name: 'LoginForm',
    data() {
        return {
            email: '',
            password: '',
            loading: false,
            error: null,
        };
    },
    methods: {
        async handleLogin() {
            this.loading = true;
            this.error = null;

            try {
                const response = await apiService.login(this.email, this.password);

                if (response.success) {
                    // Rediriger vers le dashboard
                    this.$router.push('/dashboard');
                } else {
                    this.error = response.message || 'Erreur de connexion';
                }
            } catch (error) {
                this.error = 'Erreur réseau: ' + error.message;
            } finally {
                this.loading = false;
            }
        },
    },
};
</script>
```

---

# **PARTIE 10 : TESTER AVEC CURL (Terminal)**

Pour les étudiants qui veulent tester sans Postman :

```bash
# Test /ping
curl http://localhost:8000/api/v1/ping

# Inscription
curl -X POST http://localhost:8000/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Jean",
    "email": "jean@test.com",
    "password": "password123",
    "password_confirmation": "password123"
  }'

# Connexion (et copier le token retourné)
curl -X POST http://localhost:8000/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "admin@demo.com",
    "password": "password"
  }'

# Lister les services
curl http://localhost:8000/api/v1/services

# Créer une réservation (remplacer TOKEN par le vrai token)
curl -X POST http://localhost:8000/api/v1/reservations \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -d '{
    "service_id": 1,
    "date_reservation": "2026-03-15",
    "heure_reservation": "10:30",
    "commentaire": "Test"
  }'
```

---

# **PARTIE 11 : BEST PRACTICES ET SÉCURITÉ**

## **11.1 Principes de sécurité API**

### **1. Validation TOUJOURS**

```php
// ✅ BON
$validated = $request->validate([
    'email' => 'required|email',
]);

// ❌ MAUVAIS
$email = $request->email; // Pas de validation
```

### **2. Authentification pour les données sensibles**

```php
// ✅ Routes protégées
Route::middleware('auth:sanctum')->group(function () {
    // Seuls les utilisateurs connectés
});

// ❌ Routes publiques
Route::get('/user-emails', ...); // DANGER!
```

### **3. Autorisation par rôle**

```php
// ✅ BON
if ($reservation->patient_id !== auth()->id()) {
    abort(403);
}

// ❌ MAUVAIS
// Permettre à n'importe qui de modifier n'importe quelle réservation
```

### **4. HTTPS en production**

```php
// Force HTTPS dans production
if (app()->environment('production')) {
    URL::forceScheme('https');
}
```

### **5. Rate limiting**

```php
// Limiter les tentatives de connexion
Route::post('/auth/login', [AuthApiController::class, 'login'])
    ->middleware('throttle:5,1'); // 5 tentatives par minute
```

## **11.2 Structure de réponse API**

La structure doit être **cohérente** :

```json
{
  "success": true/false,
  "message": "Description de l'action",
  "data": { /* Données */ },
  "errors": { /* Erreurs de validation */ }
}
```

---

# **RÉSUMÉ FINAL**

## **Checklist pour une API REST complète**

✅ Routes API versionnées (`/api/v1/`)
✅ Authentification avec Sanctum
✅ Contrôleurs API dédiés
✅ Resources pour formatter la réponse
✅ Validation des entrées
✅ Gestion des erreurs personnalisées
✅ Pagination et filtrage
✅ Middlewares de sécurité
✅ Tests avec Postman
✅ Documentation des endpoints

---

**Vous avez maintenant une API REST complète et fonctionnelle ! 🚀**

Les étudiants peuvent :
- Comprendre comment une API fonctionne
- Créer leurs propres endpoints
- Sécuriser leurs API
- Tester avec Postman
- Consommer l'API depuis un frontend

# **COMPLÉMENT PÉDAGOGIQUE : REST, Sanctum et Tests d'API**

Je vais expliquer ces concepts de manière détaillée et progressive.

---

# **PARTIE 1 : COMPRENDRE REST (Representational State Transfer)**

## **1.1 Qu'est-ce que REST ?**

REST est un **style architectural** pour concevoir des services web. C'est un ensemble de **principes** et **conventions**, pas une technologie.

### **L'acronyme REST décortiqué**

$$\text{REST} = \text{REpresentational} + \text{State} + \text{Transfer}$$

| Mot | Signification | Explication |
|-----|---------------|-------------|
| **Representational** | Représentation | Les données sont représentées au format JSON, XML, etc. |
| **State** | État | Chaque requête contient toutes les infos (pas de session) |
| **Transfer** | Transfert | On transfère l'état via HTTP |

### **Analogie simple : Commander au restaurant**

```
❌ NON REST :
Serveur : "Dis-moi ce que tu veux"
Toi : "Je voudrais un café"
Serveur : "OK, tu as un café"
Toi : "Maintenant du sucre"
Serveur : "Tu dois dire: je voudrais un café avec sucre"

✅ REST :
Toi : "Je voudrais 1 café avec sucre"
Serveur : "OK, voici ton café avec sucre"
(Chaque requête est complète, pas besoin du contexte précédent)
```

## **1.2 Les 6 contraintes REST**

### **Contrainte 1 : Client-Serveur**

```
Client (navigateur, app mobile)
    ↓ Requête HTTP
    ↓
Serveur (votre API Laravel)
    ↓ Réponse JSON
    ↓
Client
```

Le client et le serveur sont **indépendants** et communiquent uniquement via HTTP.

### **Contrainte 2 : Stateless (Sans état)**

Chaque requête doit contenir **toutes les informations nécessaires**. Le serveur ne doit rien "mémoriser" entre deux requêtes.

```php
// ❌ NON STATELESS
// Requête 1
POST /login
→ Serveur stocke "tu es connecté"

// Requête 2
GET /services
→ Serveur se souvient que tu es connecté

// ✅ STATELESS (REST)
// Requête 1
POST /login
→ Retourne un TOKEN

// Requête 2
GET /services + Header: Authorization: Bearer TOKEN
→ Serveur vérifie le TOKEN (pas de mémoire)
```

### **Contrainte 3 : Cacheable (Mise en cache)**

Les réponses doivent être cachéables pour améliorer les performances.

```
GET /api/v1/services?page=1
→ Cache pendant 1 heure
→ Les prochaines requêtes ne vont pas au serveur
```

### **Contrainte 4 : Interface uniforme**

Tous les endpoints suivent le **même pattern**.

```
✅ BON (Uniforme)
GET    /api/v1/services          (lire tous)
GET    /api/v1/services/1        (lire un)
POST   /api/v1/services          (créer)
PUT    /api/v1/services/1        (modifier)
DELETE /api/v1/services/1        (supprimer)

❌ MAUVAIS (Pas uniforme)
GET    /api/get-services         (pourquoi "get"?)
GET    /api/service-detail/1     (pourquoi "detail"?)
POST   /api/add-service          (pourquoi "add"?)
PUT    /api/edit-service/1       (pourquoi "edit"?)
DELETE /api/remove-service/1     (pourquoi "remove"?)
```

### **Contrainte 5 : Système en couches**

L'architecture est divisée en **couches** :

```
Frontend (Vue, React)
    ↓ (n'interagit qu'avec API)
API (Laravel)
    ↓ (n'interagit qu'avec BD)
Base de données
```

Le frontend ne connaît pas la base de données. La BD ne connaît pas le frontend.

### **Contrainte 6 : Code à la demande (optionnel)**

Le serveur peut envoyer du code exécutable (JavaScript, etc.). **Rarement utilisé**.

---

## **1.3 Les verbes HTTP et leurs sémantiques**

### **GET - Récupérer une ressource**

- **Sûr** : Aucune modification
- **Idempotent** : Toujours le même résultat
- **Code retourné** : 200 OK

```bash
# Exemples
GET /api/v1/services              # Lister tous
GET /api/v1/services/1            # Récupérer un
GET /api/v1/services?page=2       # Avec paramètres
GET /api/v1/services?search=test  # Avec filtres
```

### **POST - Créer une ressource**

- **Non sûr** : Modifie les données
- **Non idempotent** : Plusieurs POST = plusieurs créations
- **Code retourné** : 201 Created

```bash
# Exemple
POST /api/v1/services
{
  "titre": "Consultation",
  "prix": 15000
}
→ Retour 201 + la ressource créée
→ Si on répète : crée DEUX fois
```

### **PUT - Remplacer UNE ressource ENTIÈREMENT**

- **Non sûr** : Modifie
- **Idempotent** : Remplacer 2 fois = même résultat
- **Code retourné** : 200 OK ou 204 No Content

```bash
# Remplacer COMPLÈTEMENT
PUT /api/v1/services/1
{
  "titre": "Nouvelle consultation",
  "prix": 20000,
  "duree": 45,
  "statut": "actif"
}
→ Remplace tout

# Si on met un champ à null, c'est volontaire
```

### **PATCH - Mettre à jour PARTIELLEMENT**

- **Non sûr** : Modifie
- **Idempotent** : Peut être répété
- **Code retourné** : 200 OK

```bash
# Mettre à jour SEULEMENT les champs fournis
PATCH /api/v1/services/1
{
  "titre": "Consultation modifiée"
}
→ Seul le titre change, le reste reste pareil
```

### **DELETE - Supprimer une ressource**

- **Non sûr** : Supprime
- **Idempotent** : Supprimer 2 fois = même résultat
- **Code retourné** : 204 No Content ou 200 OK

```bash
DELETE /api/v1/services/1
→ Supprimé

# Si on supprime à nouveau
DELETE /api/v1/services/1
→ Déjà supprimé (même résultat)
```

### **Tableau récapitulatif**

| Verbe | Action | Sûr | Idempotent | Code |
|-------|--------|-----|------------|------|
| **GET** | Lire | ✅ | ✅ | 200 |
| **POST** | Créer | ❌ | ❌ | 201 |
| **PUT** | Remplacer | ❌ | ✅ | 200 |
| **PATCH** | Modifier | ❌ | ✅ | 200 |
| **DELETE** | Supprimer | ❌ | ✅ | 204 |

---

## **1.4 Codes HTTP et leurs significations**

Les codes HTTP sont **essentiels** pour indiquer le résultat :

$$\text{Codes} = \text{Centaine qui indique la catégorie}$$

### **2xx - Succès**

```
200 OK              → Requête OK, réponse avec contenu
201 Created         → Ressource créée avec succès
204 No Content      → Succès, mais pas de contenu (DELETE)
```

### **3xx - Redirection**

```
301 Moved Permanently → Ressource déplacée définitivement
302 Found             → Redirection temporaire
```

### **4xx - Erreur Client**

```
400 Bad Request       → Données invalides
401 Unauthorized      → Pas authentifié (pas de token)
403 Forbidden         → Authentifié mais pas d'accès
404 Not Found         → Ressource inexistante
422 Unprocessable     → Erreur de validation
429 Too Many Requests → Trop de requêtes (rate limiting)
```

### **5xx - Erreur Serveur**

```
500 Internal Server   → Erreur du serveur
503 Service Unavailable → Serveur indisponible
```

### **Utilisation dans Laravel**

```php
// ✅ BON
return response()->json($data, 200);           // GET réussi
return response()->json($data, 201);           // POST réussi
return response()->json(['msg' => 'Créé'], 201); // Au lieu de 200

// Erreurs
return response()->json(['error' => 'Invalid'], 422); // Validation
return response()->json(['error' => 'Not found'], 404);
abort(401);  // Non authentifié
abort(403);  // Non autorisé
```

---

## **1.5 Structure des URLs REST**

### **Règle 1 : Utiliser des noms (pas des verbes)**

```
❌ MAUVAIS (verbes dans l'URL)
GET    /api/get-services
POST   /api/create-service
PUT    /api/update-service/1
DELETE /api/delete-service/1

✅ BON (noms, verbes dans la méthode HTTP)
GET    /api/services
POST   /api/services
PUT    /api/services/1
DELETE /api/services/1
```

**Pourquoi ?** Car HTTP a déjà les verbes. L'URL décrit la **ressource**, pas l'**action**.

### **Règle 2 : Utiliser le pluriel**

```
✅ /api/services           (collection)
✅ /api/services/1         (élément)

❌ /api/service            (confus)
❌ /api/service/1          (confus)
```

### **Règle 3 : Relations imbriquées**

```
✅ Ressources imbriquées (si logique)
GET /api/services/1/reservations      (réservations du service 1)
GET /api/users/5/reservations         (réservations de l'utilisateur 5)

❌ Trop imbriqué
GET /api/users/5/services/1/reservations/2/medecin/appointments
```

### **Règle 4 : Paramètres de requête pour les filtres**

```
✅ BON
GET /api/services?page=2&per_page=20&prix_min=5000

❌ MAUVAIS
GET /api/services/page/2/per-page/20/prix-min/5000
```

---

# **PARTIE 2 : SANCTUM - L'AUTHENTIFICATION API DE LARAVEL**

## **2.1 Qu'est-ce que Sanctum ?**

Sanctum est le **système d'authentification officiel de Laravel** pour les APIs.

### **Comparaison : Sessions vs Tokens**

```
SESSIONS (Web classique)
  ├─ Navigateur stocke un cookie
  ├─ Serveur se souvient (stateful)
  ├─ Valide pour les sites web
  └─ Pas bon pour les APIs (plusieurs appareils)

TOKENS (API avec Sanctum)
  ├─ Client stocke un token JWT/Bearer
  ├─ Serveur ne se souvient pas (stateless)
  ├─ Valide pour les APIs
  └─ Même client peut avoir plusieurs tokens
```

### **Flux d'authentification Sanctum**

```
1. Client envoie email + password
   POST /api/v1/auth/login
   
2. Serveur valide et crée un TOKEN
   $token = $user->createToken('api-token')->plainTextToken
   
3. Serveur retourne le TOKEN
   {
     "success": true,
     "token": "1|abc123xyz789..."
   }
   
4. Client stocke le TOKEN
   localStorage.setItem('api_token', token)
   
5. Client envoie le TOKEN pour chaque requête
   GET /api/v1/reservations
   Header: Authorization: Bearer 1|abc123xyz789...
   
6. Serveur valide le TOKEN
   middleware('auth:sanctum')
   
7. Si valide → accès autorisé
   Si invalide → erreur 401
```

## **2.2 Installation de Sanctum**

```bash
# Installation
php artisan install:api

# Migration
php artisan migrate

# Vérification : vérifiez la table personal_access_tokens
```

## **2.3 Configuration de Sanctum**

**config/sanctum.php** (créé automatiquement)

```php
<?php

return [
    // Domaines autorisés (CORS)
    'stateful' => explode(',', env('SANCTUM_STATEFUL_DOMAINS', 'localhost,127.0.0.1')),
    
    // Durée de vie du token (en minutes)
    'expiration' => 525600, // 1 an par défaut
    
    // Prefix du token
    'token_prefix' => env('SANCTUM_TOKEN_PREFIX', ''),
];
```

## **2.4 Comment Sanctum crée un TOKEN**

Dans le contrôleur d'authentification :

```php
<?php

namespace App\Http\Controllers\Api\V1;

use App\Models\User;

class AuthApiController
{
    public function login(Request $request)
    {
        $user = User::where('email', $request->email)->first();
        
        // Créer un token
        $token = $user->createToken('api-token')->plainTextToken;
        
        // Retourner le token
        return response()->json([
            'token' => $token,
        ]);
    }
}
```

### **Qu'est-ce qui se passe derrière ?**

```php
// $user->createToken('api-token')
// ↓ crée une ligne dans personal_access_tokens

// Table personal_access_tokens
id | tokenable_id | tokenable_type | name | token | abilities | last_used_at | created_at | updated_at
1  | 1            | App\Models\User| api  | abc.. | *         | NULL         | 2026-02-27 | 2026-02-27

// Le token est un HASH SHA-256 du token réel
// Token réel : 1|abc123xyz789...
// Stocké en BD : [hash du token]
```

## **2.5 Valider le token dans les requêtes**

Utiliser le middleware `auth:sanctum` :

```php
// routes/api.php

Route::middleware('auth:sanctum')->group(function () {
    Route::get('/auth/me', [AuthApiController::class, 'me']);
    Route::get('/reservations', [ReservationApiController::class, 'index']);
});
```

### **Comment ça marche ?**

```
1. Client envoie :
   GET /api/v1/reservations
   Header: Authorization: Bearer 1|abc123xyz789...

2. Middleware auth:sanctum reçoit le token
   ├─ Extrait "1|abc123xyz789..."
   ├─ Crée un hash du token
   └─ Cherche dans personal_access_tokens

3. Si trouvé :
   ├─ auth()->user() retourne l'utilisateur
   └─ Accès autorisé ✅

4. Si non trouvé :
   ├─ Erreur 401
   └─ Accès refusé ❌
```

## **2.6 Différents types de tokens Sanctum**

### **Type 1 : Token simple (API)**

```php
// Pour des API de longue durée
$token = $user->createToken('api-token')->plainTextToken;

// Pas d'expiration (à moins de spécifier)
```

### **Type 2 : Token avec capacités (permissions)**

```php
// Créer un token avec permissions spécifiques
$token = $user->createToken('limited-token', ['read', 'create'])->plainTextToken;

// Vérifier les permissions
if (auth()->user()->tokenCan('read')) {
    // L'utilisateur peut lire
}
```

### **Type 3 : Token temporaire**

```php
// Créer un token qui expire
$token = $user->createToken('temp-token')->plainTextToken;

// Configurer l'expiration dans config/sanctum.php
'expiration' => 60, // 60 minutes
```

## **2.7 Révoquer un token (déconnexion)**

```php
// Supprimer le token actuel
auth()->user()->currentAccessToken()->delete();

// Supprimer TOUS les tokens de l'utilisateur
auth()->user()->tokens()->delete();

// Supprimer un token spécifique
$user->tokens()->where('name', 'api-token')->delete();
```

---

# **PARTIE 3 : ALTERNATIVES À SANCTUM**

## **3.1 Comparaison des méthodes d'authentification API**

| Solution | Type | Complexité | Sécurité | Idéale pour |
|----------|------|-----------|---------|------------|
| **Sanctum** | Token simple | Facile | Bonne | SPAs, mobiles, Laravel classique |
| **Passport** | OAuth 2.0 | Complexe | Très bonne | APIs partagées, services externes |
| **JWT (manuellement)** | Token auto-signé | Moyenne | Bonne | Microservices, stateless total |
| **API Keys** | Clé simple | Très facile | Faible | Services publiques, rate limiting |
| **Sessions** | Cookie session | Facile | Bonne | Sites web classiques (pas APIs) |

## **3.2 JWT - Alternative populaire**

JWT (JSON Web Token) est une **alternative** à Sanctum.

### **Structure d'un JWT**

```
header.payload.signature
┌──────┬────────┬──────────────┐
│HEADER│PAYLOAD │ SIGNATURE    │
└──────┴────────┴──────────────┘
```

**Header** : Indique que c'est un JWT
```json
{
  "typ": "JWT",
  "alg": "HS256"
}
```

**Payload** : Les données
```json
{
  "user_id": 1,
  "email": "admin@demo.com",
  "iat": 1624000000,
  "exp": 1624003600
}
```

**Signature** : Preuve d'authenticité (créée avec clé secrète)

### **Exemple JWT complet**

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjoxLCJlbWFpbCI6ImFkbWluIiwiaWF0IjoxNjI0MDAwMDAwfQ.xyz789...
```

### **Avantages JWT**

✅ Pas de stockage en BD (stateless total)
✅ Peut être validé par n'importe quel serveur
✅ Idéal pour les microservices

### **Inconvénients JWT**

❌ Impossible de révoquer un token avant son expiration
❌ Peut être décodé (pas sûr si secret partagé)
❌ Plus lourd que Sanctum

### **Utiliser JWT avec Laravel**

Package : `tymon/jwt-auth`

```bash
composer require tymon/jwt-auth

php artisan jwt:secret
```

---

# **PARTIE 4 : TESTER LES APIS**

## **4.1 Postman - L'outil complet**

### **Installation**

1. Téléchargez : https://www.postman.com/downloads/
2. Installez
3. Créez un compte gratuit

### **Workflow Postman étape par étape**

#### **Étape 1 : Créer une collection**

Une collection regroupe vos requêtes API :

```
Clic droit → New Collection
→ Nommer : "Réservation API"
→ Create
```

#### **Étape 2 : Ajouter l'environnement**

Un environnement stocke les **variables** (URL de base, token, etc.)

```
Postman → Environments → Create Environment
→ Nommer : "Development"

Variables :
- base_url : http://localhost:8000
- api_version : v1
- token : (vide au début)
```

#### **Étape 3 : Tester /ping**

```
+ New Request
Méthode : GET
URL : {{base_url}}/api/{{api_version}}/ping

Send

Réponse :
{
  "success": true,
  "message": "API V1 fonctionne correctement"
}
```

#### **Étape 4 : Inscription et récupérer le token**

```
+ New Request
Méthode : POST
URL : {{base_url}}/api/{{api_version}}/auth/register

Body (JSON) :
{
  "name": "Test User",
  "email": "test@example.com",
  "password": "password123",
  "password_confirmation": "password123"
}

Send

Réponse :
{
  "success": true,
  "data": {
    "user": { ... },
    "token": "1|abc123xyz789..."
  }
}
```

#### **Étape 5 : Sauvegarder le token dans la variable d'environnement**

Après réception du token :

```
Onglet "Tests"

pm.environment.set("token", pm.response.json().data.token);
```

Cela **automatise** la sauvegarde du token !

#### **Étape 6 : Utiliser le token dans les requêtes suivantes**

```
+ New Request
Méthode : GET
URL : {{base_url}}/api/{{api_version}}/reservations

Onglet "Authorization"
Type : Bearer Token
Token : {{token}}

Send

✅ Maintenant authentifié !
```

### **Postman Test Script complet**

Vous pouvez **automatiser les tests** :

```javascript
// Onglet "Tests"

// Test 1 : Vérifier le code 200
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Test 2 : Vérifier que success est true
pm.test("Response is successful", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.success).to.be.true;
});

// Test 3 : Sauvegarder le token
pm.test("Token saved", function () {
    var jsonData = pm.response.json();
    pm.environment.set("token", jsonData.data.token);
});

// Test 4 : Vérifier la présence d'un champ
pm.test("Response contains required fields", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.data).to.have.property("token");
    pm.expect(jsonData.data.user).to.have.property("email");
});
```

---

## **4.2 Alternatives à Postman plus simples/récentes**

### **1️⃣ Thunder Client (VS Code)**

**Plus léger et plus simple que Postman**

Installation :
```
VS Code → Extensions
Rechercher : Thunder Client
Installer
```

Avantages :
✅ Directement dans VS Code
✅ Interface plus simple
✅ Plus léger
✅ Gratuit

Utilisation :
```
Clic sur Thunder Client
+ New Request
GET http://localhost:8000/api/v1/ping
Send
```

### **2️⃣ Insomnia**

Installation : https://insomnia.rest/

Avantages :
✅ Interface intuitive
✅ Bonne gestion des environnements
✅ Gratuit (version de base)

### **3️⃣ REST Client (VS Code)**

**L'option la plus simple**

Installation :
```
VS Code → Extensions
Rechercher : REST Client
Installer
```

Créer un fichier `api.http` :

```http
### Test ping
GET http://localhost:8000/api/v1/ping

### Connexion
POST http://localhost:8000/api/v1/auth/login
Content-Type: application/json

{
  "email": "admin@demo.com",
  "password": "password"
}

### Lister les services
GET http://localhost:8000/api/v1/services

### Créer une réservation
POST http://localhost:8000/api/v1/reservations
Content-Type: application/json
Authorization: Bearer 1|abc123xyz789...

{
  "service_id": 1,
  "date_reservation": "2026-03-15",
  "heure_reservation": "10:30"
}
```

Utilisation :
```
Clic sur "Send Request" au-dessus de chaque requête
La réponse s'affiche dans un panel à droite
```

**Avantages** :
✅ Super simple
✅ Directement dans l'éditeur
✅ Pas d'appli supplémentaire
✅ Parfait pour les étudiants

---

### **4️⃣ Bruno (Open Source)**

Installation : https://www.usebruno.com/

Avantages :
✅ Open source (gratuit)
✅ Stocke les requêtes en fichiers texte
✅ Versionnable avec Git
✅ Pas de compte requis

### **Tableau comparatif**

| Outil | Simplicité | Fonctionnalités | Prix | Idéal pour |
|-------|-----------|-----------------|------|-----------|
| **Postman** | Moyen | Très complètes | Gratuit/Payant | Professionnels |
| **Thunder Client** | Facile | Bonnes | Gratuit | Développeurs VS Code |
| **REST Client** | Très facile | Basiques | Gratuit | Étudiants/débutants |
| **Insomnia** | Facile | Bonnes | Gratuit | Développeurs |
| **Bruno** | Facile | Bonnes | Gratuit/Open-source | Équipes, Git |
| **cURL** | Difficile | Bonnes | Gratuit | Terminal/Scripts |

---

## **4.3 Tester avec cURL (Terminal/CMD)**

### **Pourquoi cURL ?**

✅ Aucune installation
✅ Fonctionne partout
✅ Parfait pour l'automatisation
✅ À connaître comme développeur

### **Syntaxe de base**

```bash
curl -X METHOD \
     -H "Header: value" \
     -d "data" \
     URL
```

### **Exemples complets**

#### **1. GET simple**

```bash
curl http://localhost:8000/api/v1/ping
```

#### **2. GET avec paramètres**

```bash
curl "http://localhost:8000/api/v1/services?page=1&per_page=10"
```

#### **3. POST (Connexion)**

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@demo.com","password":"password"}' \
  http://localhost:8000/api/v1/auth/login
```

**Résultat** : Le JSON est retourné avec le token

#### **4. POST avec Bearer Token**

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer 1|abc123xyz789..." \
  -d '{"service_id":1,"date_reservation":"2026-03-15","heure_reservation":"10:30"}' \
  http://localhost:8000/api/v1/reservations
```

#### **5. PUT (Modifier)**

```bash
curl -X PUT \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer 1|abc123xyz789..." \
  -d '{"titre":"Consultation modifiée"}' \
  http://localhost:8000/api/v1/services/1
```

#### **6. DELETE**

```bash
curl -X DELETE \
  -H "Authorization: Bearer 1|abc123xyz789..." \
  http://localhost:8000/api/v1/reservations/1
```

#### **7. Sauvegarder la réponse dans un fichier**

```bash
curl http://localhost:8000/api/v1/services > services.json
```

#### **8. Afficher les headers de réponse**

```bash
curl -i http://localhost:8000/api/v1/services
```

---

## **4.4 Script Bash pour tester l'API complète**

Créez un fichier `test-api.sh` :

```bash
#!/bin/bash

# Variables
API_URL="http://localhost:8000/api/v1"
TOKEN=""

# Couleurs pour l'affichage
GREEN='\033[0;32m'
RED='\033[0;31m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

echo -e "${BLUE}=== TEST API RÉSERVATION ===${NC}\n"

# Test 1 : Ping
echo -e "${BLUE}1. Test /ping${NC}"
curl -s "$API_URL/ping" | jq '.'
echo -e "\n"

# Test 2 : Connexion
echo -e "${BLUE}2. Connexion${NC}"
RESPONSE=$(curl -s -X POST "$API_URL/auth/login" \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@demo.com","password":"password"}')

TOKEN=$(echo $RESPONSE | jq -r '.data.token')
echo $RESPONSE | jq '.'
echo -e "${GREEN}Token: $TOKEN${NC}\n"

# Test 3 : Récupérer infos utilisateur
echo -e "${BLUE}3. Infos utilisateur (GET /auth/me)${NC}"
curl -s -H "Authorization: Bearer $TOKEN" "$API_URL/auth/me" | jq '.'
echo -e "\n"

# Test 4 : Lister les services
echo -e "${BLUE}4. Lister les services${NC}"
curl -s "$API_URL/services" | jq '.data[0:2]'
echo -e "\n"

# Test 5 : Créer une réservation
echo -e "${BLUE}5. Créer une réservation${NC}"
RESPONSE=$(curl -s -X POST "$API_URL/reservations" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "service_id":1,
    "date_reservation":"2026-03-20",
    "heure_reservation":"14:00",
    "commentaire":"Test"
  }')

RESERVATION_ID=$(echo $RESPONSE | jq -r '.data.id')
echo $RESPONSE | jq '.'
echo -e "${GREEN}Reservation ID: $RESERVATION_ID${NC}\n"

# Test 6 : Récupérer la réservation
echo -e "${BLUE}6. Récupérer la réservation${NC}"
curl -s -H "Authorization: Bearer $TOKEN" \
  "$API_URL/reservations/$RESERVATION_ID" | jq '.'
echo -e "\n"

# Test 7 : Modifier la réservation
echo -e "${BLUE}7. Modifier la réservation${NC}"
curl -s -X PUT "$API_URL/reservations/$RESERVATION_ID" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"heure_reservation":"16:00"}' | jq '.'
echo -e "\n"

# Test 8 : Déconnexion
echo -e "${BLUE}8. Déconnexion${NC}"
curl -s -X POST "$API_URL/auth/logout" \
  -H "Authorization: Bearer $TOKEN" | jq '.'
echo -e "\n"

echo -e "${GREEN}=== TESTS TERMINÉS ===${NC}"
```

Utilisation :

```bash
chmod +x test-api.sh
./test-api.sh
```

---

## **4.5 Tests unitaires avec Laravel**

Laravel a un framework de test intégré :

**tests/Feature/ApiTest.php**

```php
<?php

namespace Tests\Feature;

use App\Models\User;
use Tests\TestCase;

class ApiTest extends TestCase
{
    protected $token;
    protected $user;

    protected function setUp(): void
    {
        parent::setUp();
        
        // Créer un utilisateur pour les tests
        $this->user = User::factory()->create();
        
        // Obtenir un token
        $this->token = $this->user->createToken('test-token')->plainTextToken;
    }

    /**
     * Test /ping
     */
    public function test_ping()
    {
        $response = $this->getJson('/api/v1/ping');
        
        $response->assertStatus(200)
                 ->assertJson([
                     'success' => true,
                 ]);
    }

    /**
     * Test connexion
     */
    public function test_login()
    {
        $response = $this->postJson('/api/v1/auth/login', [
            'email' => 'admin@demo.com',
            'password' => 'password',
        ]);

        $response->assertStatus(200)
                 ->assertJsonStructure([
                     'success',
                     'data' => ['token', 'user'],
                 ]);
    }

    /**
     * Test lister les services
     */
    public function test_get_services()
    {
        $response = $this->getJson('/api/v1/services');
        
        $response->assertStatus(200)
                 ->assertJsonStructure([
                     'data' => [
                         '*' => ['id', 'titre', 'prix'],
                     ],
                 ]);
    }

    /**
     * Test créer une réservation
     */
    public function test_create_reservation()
    {
        $response = $this->withHeader('Authorization', "Bearer $this->token")
                         ->postJson('/api/v1/reservations', [
                             'service_id' => 1,
                             'date_reservation' => '2026-03-20',
                             'heure_reservation' => '14:00',
                         ]);

        $response->assertStatus(201)
                 ->assertJsonStructure([
                     'data' => ['id', 'service', 'patient'],
                 ]);
    }

    /**
     * Test non authentifié
     */
    public function test_unauthenticated_access()
    {
        $response = $this->getJson('/api/v1/reservations');
        
        $response->assertStatus(401);
    }
}
```

Exécuter les tests :

```bash
php artisan test

# Ou un test spécifique
php artisan test tests/Feature/ApiTest.php::test_ping

# Avec un rapport de couverture
php artisan test --coverage
```

---

# **PARTIE 5 : WORKFLOW COMPLET DE DÉVELOPPEMENT API**

## **5.1 Étapes recommandées**

### **Phase 1 : Conception**

```
1. Définir les ressources
   - Services
   - Réservations
   - Utilisateurs

2. Définir les endpoints
   GET    /services
   POST   /services
   PUT    /services/{id}
   DELETE /services/{id}
   ...

3. Documenter les données
   Service:
   - id
   - titre
   - prix
   - duree
   ...
```

### **Phase 2 : Implémentation**

```
1. Créer les modèles
2. Créer les migrations
3. Créer les contrôleurs API
4. Créer les routes
5. Ajouter la validation
6. Ajouter les Resources
```

### **Phase 3 : Tests**

```
1. Test /ping
2. Test authentification
3. Test GET (lister, détail)
4. Test POST (création)
5. Test PUT (modification)
6. Test DELETE (suppression)
7. Test erreurs (401, 403, 404, 422)
```

### **Phase 4 : Documentation**

```
Créer une documentation API (Swagger, OpenAPI, etc.)
```

---

## **5.2 Checklist d'une API REST bien faite**

```
✅ STRUCTURE
  ✓ Routes versionnées (/api/v1)
  ✓ Noms au pluriel (/services)
  ✓ Pas de verbes dans l'URL
  ✓ Paramètres de requête pour filtres

✅ HTTP
  ✓ Verbes corrects (GET, POST, PUT, DELETE)
  ✓ Codes statuts corrects (200, 201, 401, 404, 422)

✅ SÉCURITÉ
  ✓ Authentification (Sanctum)
  ✓ Autorisation (vérifier les rôles)
  ✓ Validation des données
  ✓ Rate limiting

✅ RÉPONSES
  ✓ Format JSON cohérent
  ✓ Structure uniforme (success, data, message)
  ✓ Resources pour contrôler les champs

✅ TESTS
  ✓ Tests unitaires
  ✓ Tests d'intégration
  ✓ Postman collection
```

---

# **RÉSUMÉ FINAL**

## **REST est un concept**

REST n'est pas une technologie, c'est un **style architectural** basé sur :
- **HTTP** comme protocole
- **URLs** pour les ressources
- **Verbes HTTP** pour les actions
- **Codes HTTP** pour les statuts
- **JSON** pour les données

## **Sanctum est l'implémentation de Laravel**

Sanctum fournit :
- Création de tokens
- Validation de tokens
- Gestion des permissions
- Révocation de tokens

## **Pour tester, vous avez plusieurs options**

| Situation | Outil recommandé |
|-----------|-----------------|
| Étudiants débutants | REST Client (VS Code) |
| Développement | Thunder Client ou Postman |
| Équipes | Bruno ou Postman |
| Automatisation | cURL ou scripts Bash |
| Tests | Laravel Testing Framework |

---

**Vous avez maintenant une compréhension complète de REST, Sanctum et comment tester les APIs ! 🚀**