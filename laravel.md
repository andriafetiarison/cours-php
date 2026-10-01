# Cours Laravel : construire une Todo List avec utilisateurs

Tu connais PHP, la POO, Composer, le MVC et les bases de la sécurité. Tu as même construit ton propre mini-framework. Il est temps de découvrir **ce que Laravel fait à ta place**, avec une vraie petite application : une **Todo List multi-utilisateurs**.

> **Version utilisée : Laravel 13** (série 13.x, sortie le 17 mars 2026, **PHP 8.3 minimum**, compatible PHP 8.3 à 8.5). Elle reçoit des corrections de bugs jusqu'au 3ᵉ trimestre 2027 et des correctifs de sécurité jusqu'en mars 2028.
>
> **Ton PHP est plus ancien (8.2) ?** Utilise **Laravel 12** (`composer create-project laravel/laravel:^12.0 todo-app`). Tout le code de ce cours fonctionne à l'identique.
>
> Vérifie ta version de PHP avec `php -v`. Si elle est trop ancienne, mets à jour XAMPP, ou utilise Laravel Herd / `php.new` (voir la page *Installation* de la documentation officielle).

**L'application finale permettra de :**

* s'inscrire, se connecter, se déconnecter ;
* créer, afficher, modifier, terminer et supprimer des tâches ;
* **ne voir que ses propres tâches** (chaque tâche appartient à l'utilisateur connecté).

**Méthode pour chaque étape :** explication courte → commandes → code → résultat → exercice → correction.

**Plan :**

1. Pourquoi Laravel
2. Installation
3. Structure du projet
4. Routes
5. Controllers
6. Views
7. Blade
8. Migrations
9. Modèles Eloquent
10. Relations simples
11. CRUD
12. Validation
13. Sessions
14. Authentification
15. Middleware
16. `.env`

À la fin : la **comparaison avec ton PHP MVC** (ce que Laravel prend en charge automatiquement), le récapitulatif des commandes Artisan, et la suite.

---

## Étape 1 : Pourquoi Laravel

### Explication

Dans les cours précédents, tu as écrit toi-même un routeur, des contrôleurs, un conteneur, un moteur de vues, de la validation, la protection CSRF, la gestion de session... C'était **le meilleur moyen de comprendre**. Mais pour un vrai projet, refaire tout cela est long, risqué (sécurité !) et difficile à maintenir en équipe.

**Laravel** est un framework PHP qui fournit tout ça, **déjà écrit, testé et sécurisé**, avec une syntaxe agréable. Il apporte aussi une **convention** : tous les projets Laravel ont la même structure, donc tu retrouves tes repères partout (et les autres développeurs aussi).

| Ce que tu as écrit à la main | Ce que Laravel te donne |
|---|---|
| `Router` avec expressions régulières | `routes/web.php` |
| `Container` avec réflexion | Service Container (injection automatique) |
| Vues PHP / Twig | **Blade** |
| `Task` + `TaskRepository` + SQL | **Eloquent** (ORM) |
| `schema.sql` | **Migrations** versionnées |
| `Task::validateTitle()` | `$request->validate([...])` |
| Classe `Csrf` | Protection CSRF automatique |
| `password_hash` + sessions à la main | Authentification intégrée |
| Aucun outil en ligne de commande | **Artisan** (générateurs, migrations, console) |

Laravel ne fait pas de magie : sous le capot, tu retrouveras tout ce que tu as déjà construit.

### Exercice

Cite cinq tâches techniques que tu as dû coder toi-même dans ton mini-framework MVC, et que Laravel fournit déjà.

### Correction

Par exemple : le **routage** (routeur + paramètres `{id}`), l'**injection de dépendances** (`Container`), les **vues** (`View` / Twig), l'accès aux **données** (`TaskRepository` avec PDO), la **validation**, la protection **CSRF**, les **sessions** sécurisées, le **hachage** des mots de passe, la lecture du **`.env`**.

---

## Étape 2 : Installation

### Explication

On installe d'abord l'**installateur Laravel** (une commande `laravel`), puis on crée le projet.

**Prérequis :** PHP 8.3+, Composer, les extensions PHP usuelles (XAMPP les fournit).

### Commandes

```bash
# 1. Installer l'installateur Laravel (une seule fois)
composer global require laravel/installer

# 2. Créer le projet
laravel new todo-app

# 3. Entrer dans le projet et lancer le serveur de développement
cd todo-app
php artisan serve
```

*(Si la commande `laravel` est introuvable, ajoute le dossier `bin` global de Composer à ton `PATH`, ou utilise `composer create-project laravel/laravel:^13.0 todo-app`.)*

L'installateur te pose quelques questions (elles peuvent légèrement évoluer selon la version). Réponds ainsi :

| Question | Réponse conseillée |
|---|---|
| Starter kit | **None** (on construit l'authentification nous-mêmes pour comprendre) |
| Framework de tests | Pest ou PHPUnit, au choix |
| Base de données | **SQLite** (rien à installer ; on passera à MySQL à l'étape 16) |
| Installer les dépendances npm | Peu importe : nous n'utilisons pas Node dans ce cours |

> **Si tu vois une erreur « could not find driver »**, active l'extension `pdo_sqlite` dans ton `php.ini`, ou choisis MySQL à l'installation.

### Code

Aucun code à écrire. Ouvre l'adresse affichée par `php artisan serve`.

### Résultat

```
   INFO  Server running on [http://127.0.0.1:8000].

  Press Ctrl+C to stop the server
```

`http://127.0.0.1:8000` affiche la **page d'accueil de Laravel**. Pour arrêter le serveur : `Ctrl + C`.

L'installateur a aussi créé le fichier `.env`, généré la clé de l'application, créé la base SQLite (`database/database.sqlite`) et exécuté les migrations par défaut.

### Exercice

Lance le serveur sur le port `8001`, puis affiche un résumé de ton application (version de Laravel, de PHP, environnement...).

### Correction

```bash
php artisan serve --port=8001
php artisan about
```

`php artisan about` affiche la version de Laravel, de PHP, l'environnement, le mode debug, la base de données, etc. C'est un excellent réflexe de diagnostic.

---

## Étape 3 : Structure du projet

### Explication

```
todo-app/
├── app/
│   ├── Http/
│   │   └── Controllers/      # les contrôleurs
│   ├── Models/               # les modèles Eloquent (User.php existe déjà)
│   └── Providers/
├── bootstrap/
│   └── app.php               # configuration de l'application (routes, middleware…)
├── config/                   # fichiers de configuration (database.php, session.php…)
├── database/
│   ├── migrations/           # structure de la base, versionnée
│   ├── factories/            # fabriques de données de test
│   └── seeders/
├── public/
│   └── index.php             # point d'entrée unique (le seul dossier accessible du web)
├── resources/
│   └── views/                # les vues Blade
├── routes/
│   └── web.php               # les routes de l'application
├── storage/                  # logs, cache, fichiers générés
├── tests/
├── vendor/                   # bibliothèques Composer
├── .env                      # configuration locale et secrets (jamais dans Git)
├── artisan                   # l'outil en ligne de commande
└── composer.json
```

**Correspondance avec ton projet MVC :**

| Ton mini-framework | Laravel |
|---|---|
| `public/index.php` | `public/index.php` |
| `config/routes.php` | `routes/web.php` |
| `src/Controller/` | `app/Http/Controllers/` |
| `src/Model/` | `app/Models/` |
| `views/` | `resources/views/` |
| `config/database.php` | `config/database.php` + `.env` |
| `database/schema.sql` | `database/migrations/` |
| `src/Core/` (Router, Container…) | `vendor/laravel/framework` (déjà écrit) |
| Namespace `App\` → `src/` | Namespace `App\` → `app/` |

### Commandes

```bash
php artisan list           # toutes les commandes Artisan disponibles
php artisan route:list     # toutes les routes de l'application
```

### Résultat

`php artisan route:list` affiche pour l'instant la page d'accueil et quelques routes ajoutées par Laravel (comme `/up`, qui sert de test de santé).

### Exercice

Dans quel dossier ou fichier mets-tu : (a) une nouvelle URL ; (b) le HTML d'une page ; (c) la structure d'une table ; (d) le mot de passe de la base de données ; (e) le fichier `logo.png` ?

### Correction

(a) `routes/web.php` · (b) `resources/views/` · (c) `database/migrations/` · (d) `.env` · (e) `public/` (seul dossier accessible depuis le navigateur).

---

## Étape 4 : Routes

### Explication

Une **route** associe une **méthode HTTP + une URL** à un morceau de code. Elles se déclarent dans `routes/web.php`. Les routes de ce fichier passent automatiquement par la **session** et la **protection CSRF** (nous y reviendrons).

### Commandes

```bash
php artisan route:list
```

### Code

Remplace le contenu de `routes/web.php` :

```php
<?php

use Illuminate\Support\Facades\Route;

// Route simple
Route::get('/', function () {
    return 'Bienvenue dans Todo App';
})->name('home');

// Paramètre d'URL : {name} est passé à la fonction
Route::get('/hello/{name}', function (string $name) {
    return "Bonjour $name !";
});

// Contrainte : {id} doit être un nombre, sinon 404
Route::get('/demo/{id}', function (int $id) {
    return "Identifiant : $id";
})->whereNumber('id');

// Redirection
Route::redirect('/accueil', '/');
```

### Résultat

* `/` → « Bienvenue dans Todo App »
* `/hello/Alice` → « Bonjour Alice ! »
* `/demo/42` → « Identifiant : 42 » ; `/demo/abc` → page 404
* `/accueil` → redirige vers `/`

`->name('home')` donne un **nom** à la route : on s'en servira avec `route('home')` pour générer des liens sans écrire d'URL en dur.

> Un `POST` envoyé sans jeton CSRF (avec `curl`, par exemple) renvoie **419 Page Expired** : c'est la protection CSRF qui travaille pour toi.

### Exercice

Ajoute :

1. une route `/about` nommée `about` qui affiche « À propos » ;
2. une route `/users/{id}` qui n'accepte que des nombres.

### Correction

```php
Route::get('/about', function () {
    return 'À propos';
})->name('about');

Route::get('/users/{id}', function (int $id) {
    return "Utilisateur n°$id";
})->whereNumber('id');
```

---

## Étape 5 : Controllers

### Explication

Comme dans ton MVC, on regroupe la logique des routes dans des **classes** : les contrôleurs. Artisan génère le squelette.

Laravel **injecte automatiquement** les objets dont tes méthodes ont besoin (comme `Request`), grâce à son conteneur : c'est le même mécanisme que ton `Container` avec autowiring.

### Commandes

```bash
php artisan make:controller TaskController
```

Crée `app/Http/Controllers/TaskController.php`.

### Code

`app/Http/Controllers/TaskController.php` :

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class TaskController extends Controller
{
    public function index(Request $request): string
    {
        $filter = $request->query('filter', 'all');

        return "Liste des tâches (filtre : $filter)";
    }
}
```

`routes/web.php` (garde la route `/` et ajoute) :

```php
use App\Http\Controllers\TaskController;

Route::get('/tasks', [TaskController::class, 'index'])->name('tasks.index');
```

### Résultat

* `/tasks` → « Liste des tâches (filtre : all) »
* `/tasks?filter=done` → « Liste des tâches (filtre : done) »

`[TaskController::class, 'index']` signifie « la méthode `index` de `TaskController` », comme dans ton routeur.

### Exercice

Crée un `PageController` avec une méthode `about()` et branche la route `/about` dessus (en remplaçant la fonction anonyme de l'étape 4).

### Correction

```bash
php artisan make:controller PageController
```

```php
// app/Http/Controllers/PageController.php
public function about(): string
{
    return 'À propos de Todo App';
}
```

```php
// routes/web.php
use App\Http\Controllers\PageController;

Route::get('/about', [PageController::class, 'about'])->name('about');
```

---

## Étape 6 : Views

### Explication

Une **vue** est un fichier dans `resources/views/`. Le helper `view('nom', $données)` la charge et lui transmet des variables. Les sous-dossiers s'écrivent avec un point : `view('tasks.index')` charge `resources/views/tasks/index.blade.php`.

### Commandes

Aucune commande Artisan : on crée simplement le fichier.

### Code

`resources/views/home.blade.php` :

```blade
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <title>Todo App</title>
</head>
<body>
    <h1>Bienvenue, {{ $name }} !</h1>
    <p>Voici ma première vue Laravel.</p>
</body>
</html>
```

`routes/web.php` : remplace la route `/` :

```php
Route::get('/', function () {
    return view('home', ['name' => 'Alice']);
})->name('home');
```

### Résultat

`/` affiche « Bienvenue, Alice ! ». `{{ $name }}` affiche la variable **en l'échappant** (protection XSS automatique, comme ton `e()`).

### Exercice

Crée une vue `about.blade.php` qui affiche l'année en cours, transmise par `PageController::about()`.

### Correction

```php
// PageController
public function about(): \Illuminate\View\View
{
    return view('about', ['year' => date('Y')]);
}
```

`resources/views/about.blade.php` :

```blade
<h1>À propos</h1>
<p>© {{ $year }} Todo App</p>
```

---

## Étape 7 : Blade

### Explication

**Blade** est le moteur de templates de Laravel (comme Twig). Il compile les vues en PHP puis les met en cache. L'essentiel :

| Syntaxe | Rôle |
|---|---|
| `{{ $var }}` | Affiche **en échappant** (sûr) |
| `{!! $html !!}` | Affiche sans échapper (**dangereux** : à éviter) |
| `@if` / `@else` / `@endif` | Conditions |
| `@foreach` / `@forelse` / `@empty` | Boucles |
| `@extends('layouts.app')` | Hérite d'un layout |
| `@section('content')` / `@yield('content')` | Bloc remplissable / emplacement du bloc |
| `@include('partial')` | Inclut une vue partielle |
| `@csrf` | Ajoute le jeton CSRF dans un formulaire |
| `@auth` / `@guest` | Contenu selon l'état de connexion |
| `{{ route('nom') }}` | URL d'une route nommée |

### Commandes

Aucune.

### Code

Ajoute d'abord une feuille de style simple. `public/css/app.css` :

```css
* { box-sizing: border-box; }
body { font-family: system-ui, sans-serif; margin: 0; background: #f4f5f7; color: #1f2937; }
header { background: #1f2937; padding: 12px 24px; }
nav { display: flex; align-items: center; gap: 16px; }
nav a, nav span { color: white; text-decoration: none; }
nav .spacer { flex: 1; }
nav form { margin: 0; }
nav button { background: transparent; border: 1px solid #9ca3af; }
main { max-width: 640px; margin: 32px auto; background: white; padding: 24px; border-radius: 8px; box-shadow: 0 1px 4px rgba(0, 0, 0, .1); }
ul.tasks { list-style: none; padding: 0; }
ul.tasks li { display: flex; align-items: center; gap: 8px; padding: 10px 0; border-bottom: 1px solid #eee; }
ul.tasks li .title { flex: 1; }
li.done .title { text-decoration: line-through; color: #9ca3af; }
form { margin: 0; }
label { display: block; margin-top: 12px; }
input[type="text"], input[type="email"], input[type="password"] { width: 100%; padding: 8px; margin: 6px 0; }
button, .button { background: #2563eb; color: white; border: 0; border-radius: 4px; padding: 6px 12px; cursor: pointer; text-decoration: none; font-size: 14px; display: inline-block; }
button.danger { background: #dc2626; }
.error { color: #b91c1c; font-size: 14px; }
.flash { background: #dcfce7; color: #166534; padding: 8px 12px; border-radius: 4px; }
```

*(Laravel utilise normalement Vite pour compiler le CSS et le JavaScript. Nous le laissons de côté pour rester concentrés sur PHP.)*

Le **layout** commun. `resources/views/layouts/app.blade.php` :

```blade
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>@yield('title', 'Todo') — {{ config('app.name') }}</title>
    <link rel="stylesheet" href="{{ asset('css/app.css') }}">
</head>
<body>
<header>
    <nav>
        <a href="{{ route('home') }}">Accueil</a>
        <a href="{{ route('tasks.index') }}">Mes tâches</a>
    </nav>
</header>
<main>
    @yield('content')
</main>
</body>
</html>
```

`resources/views/home.blade.php` (remplace son contenu) :

```blade
@extends('layouts.app')

@section('title', 'Accueil')

@section('content')
    <h1>Todo App</h1>
    <p>Une petite application de tâches construite avec Laravel {{ app()->version() }}.</p>
@endsection
```

`routes/web.php` : la route `/` devient

```php
Route::get('/', fn () => view('home'))->name('home');
```

Une liste de tâches **provisoire** (des données fictives). `TaskController::index()` :

```php
public function index(): \Illuminate\View\View
{
    $tasks = [
        (object) ['title' => 'Apprendre Laravel', 'done' => false],
        (object) ['title' => 'Installer Composer', 'done' => true],
    ];

    return view('tasks.index', compact('tasks'));
}
```

`resources/views/tasks/index.blade.php` :

```blade
@extends('layouts.app')

@section('title', 'Mes tâches')

@section('content')
    <h1>Mes tâches</h1>

    <ul class="tasks">
        @forelse ($tasks as $task)
            <li class="{{ $task->done ? 'done' : '' }}">
                <span class="title">{{ $task->title }}</span>
            </li>
        @empty
            <li>Aucune tâche pour l'instant.</li>
        @endforelse
    </ul>
@endsection
```

### Résultat

`/` affiche l'accueil avec le menu et la version de Laravel ; `/tasks` affiche les deux tâches fictives (la seconde est barrée). Le layout est écrit **une seule fois**.

`compact('tasks')` est un raccourci PHP pour `['tasks' => $tasks]`. `@forelse ... @empty` gère aussi le cas de la liste vide.

### Exercice

Crée une vue partielle `resources/views/partials/footer.blade.php` avec le pied de page « © année — Todo App » et inclus-la dans le layout.

### Correction

`resources/views/partials/footer.blade.php` :

```blade
<footer>
    <p>© {{ date('Y') }} — {{ config('app.name') }}</p>
</footer>
```

Dans `layouts/app.blade.php`, après `</main>` :

```blade
@include('partials.footer')
```

---

## Étape 8 : Migrations

### Explication

Une **migration** décrit une modification de la base de données **en PHP**. Elles sont **versionnées** (le nom contient la date), donc toute l'équipe peut reconstruire la même base avec une seule commande. C'est le remplaçant de ton `schema.sql`.

Laravel fournit déjà des migrations pour les tables `users`, `sessions`, etc. Nous ajoutons celle des **tâches**, liée à un utilisateur.

### Commandes

```bash
php artisan make:migration create_tasks_table   # crée le fichier de migration
php artisan migrate                             # exécute les migrations en attente
php artisan migrate:status                      # état des migrations
php artisan migrate:rollback                    # annule le dernier lot
php artisan migrate:fresh                       # supprime TOUT et recrée (développement uniquement !)
```

### Code

Ouvre le fichier créé dans `database/migrations/` (son nom commence par la date) :

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('tasks', function (Blueprint $table) {
            $table->id();
            $table->foreignId('user_id')->constrained()->cascadeOnDelete();
            $table->string('title');
            $table->boolean('done')->default(false);
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('tasks');
    }
};
```

Puis :

```bash
php artisan migrate
```

### Résultat

```
   INFO  Running migrations.

  2026_10_01_100000_create_tasks_table .................... 9.40ms DONE
```

`php artisan db:table tasks` affiche les colonnes de la table créée.

* `id()` : clé primaire auto-incrémentée.
* `foreignId('user_id')->constrained()` : colonne liée à `users.id` (Laravel devine la table à partir du nom). `cascadeOnDelete()` supprime les tâches d'un utilisateur supprimé.
* `timestamps()` : colonnes `created_at` et `updated_at`, remplies automatiquement.
* `up()` applique la migration, `down()` l'annule.

### Exercice

Crée une **seconde migration** qui ajoute une colonne `due_date` (date, facultative) à `tasks`. Exécute-la, puis annule-la avec `migrate:rollback` (on ne l'utilisera pas dans la suite).

### Correction

```bash
php artisan make:migration add_due_date_to_tasks_table --table=tasks
```

```php
public function up(): void
{
    Schema::table('tasks', function (Blueprint $table) {
        $table->date('due_date')->nullable()->after('title');
    });
}

public function down(): void
{
    Schema::table('tasks', function (Blueprint $table) {
        $table->dropColumn('due_date');
    });
}
```

```bash
php artisan migrate
php artisan migrate:rollback     # annule uniquement cette dernière migration
```

---

## Étape 9 : Modèles Eloquent

### Explication

**Eloquent** est l'ORM de Laravel : chaque table a un **modèle**, une classe PHP pour lire et écrire ses lignes, sans écrire de SQL. Un modèle `Task` correspond automatiquement à la table `tasks` (convention de nommage).

Contrairement à ton `Task` + `TaskRepository`, un modèle Eloquent **réunit les deux rôles** (le modèle d'une ligne *et* l'accès aux données) : on appelle cela le pattern *Active Record*.

### Commandes

```bash
php artisan make:model Task        # crée app/Models/Task.php
php artisan tinker                 # console interactive pour tester
```

Raccourcis utiles : `php artisan make:model Task -m` (avec la migration), `-c` (contrôleur), `-f` (fabrique), `-a` (tout).

### Code

`app/Models/Task.php` :

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Task extends Model
{
    // Seuls ces champs peuvent être remplis en masse (protection "mass assignment")
    protected $fillable = ['title', 'done'];

    protected function casts(): array
    {
        return [
            'done' => 'boolean',
        ];
    }
}
```

Teste dans Tinker (`php artisan tinker`) :

```php
use App\Models\{Task, User};

// Un utilisateur de test (la fabrique par défaut de Laravel)
$user = User::factory()->create(['name' => 'Alice', 'email' => 'alice@example.com']);

// Créer une tâche
$task = new Task(['title' => 'Apprendre Eloquent']);
$task->user_id = $user->id;
$task->save();

Task::all();                          // toutes les tâches
Task::find(1);                        // par identifiant
Task::where('done', false)->get();    // avec condition
$task->update(['done' => true]);      // modifier
$task->delete();                      // supprimer
```

### Résultat

`Task::all()` affiche une collection contenant ta tâche. Tu peux vérifier en base avec `php artisan db:table tasks`.

* `$fillable` liste les champs modifiables **en masse** (`new Task([...])`, `create([...])`, `update([...])`). `user_id` n'y figure **volontairement** : on ne veut pas qu'un formulaire puisse choisir à qui appartient une tâche (protection contre l'*assignation de masse*, comme au cours de sécurité).
* `casts()` convertit `done` en booléen PHP (sinon on récupèrerait `0`/`1`).
* Les requêtes sont **préparées automatiquement** : pas d'injection SQL.

### Exercice

Ajoute au modèle un **scope** `done()` qui ne retourne que les tâches terminées, puis compte-les dans Tinker avec `Task::done()->count()`.

### Correction

```php
// app/Models/Task.php
use Illuminate\Database\Eloquent\Builder;

public function scopeDone(Builder $query): Builder
{
    return $query->where('done', true);
}
```

```php
Task::done()->count();
```

Le préfixe `scope` est retiré à l'appel : `scopeDone` devient `done()`.

---

## Étape 10 : Relations simples

### Explication

Un utilisateur **a plusieurs** tâches ; une tâche **appartient à** un utilisateur. Eloquent traduit ça en deux méthodes :

* `User` → `hasMany(Task::class)` ;
* `Task` → `belongsTo(User::class)`.

La clé étrangère `user_id` est devinée grâce aux conventions de nommage.

### Commandes

Aucune : on édite les deux modèles.

### Code

`app/Models/User.php` : ajoute l'import et la méthode (garde le reste du fichier tel quel) :

```php
use Illuminate\Database\Eloquent\Relations\HasMany;

public function tasks(): HasMany
{
    return $this->hasMany(Task::class);
}
```

`app/Models/Task.php` : ajoute

```php
use Illuminate\Database\Eloquent\Relations\BelongsTo;

public function user(): BelongsTo
{
    return $this->belongsTo(User::class);
}
```

Dans Tinker :

```php
use App\Models\User;

$user = User::first();

$user->tasks()->create(['title' => 'Écrire mon premier modèle']);   // user_id est rempli tout seul
$user->tasks;                       // la collection des tâches de l'utilisateur
$user->tasks()->where('done', false)->count();

$task = $user->tasks()->first();
$task->user->name;                  // "Alice"
```

### Résultat

`$user->tasks()->create([...])` crée la tâche **déjà associée** à l'utilisateur : c'est l'astuce qui nous servira pour « chaque utilisateur ne voit que ses tâches ».

* `$user->tasks` (sans parenthèses) retourne le **résultat** (une collection) ; `$user->tasks()` (avec) retourne la **relation**, sur laquelle on peut continuer à construire une requête (`where`, `latest`...).
* Pour éviter le problème dit « N+1 » (une requête par tâche) quand tu affiches beaucoup de lignes liées, utilise le **chargement anticipé** : `Task::with('user')->get()`.

### Exercice

Ajoute à `User` une relation `pendingTasks()` qui ne retourne que les tâches **non terminées**.

### Correction

```php
public function pendingTasks(): HasMany
{
    return $this->hasMany(Task::class)->where('done', false);
}
```

```php
User::first()->pendingTasks;
```

---

## Étape 11 : CRUD

### Explication

**CRUD** : Create, Read, Update, Delete. Laravel propose une **route de ressource** qui déclare d'un coup les routes classiques et suit des conventions que tu connais déjà (`index`, `create`, `store`, `edit`, `update`, `destroy`).

On utilise aussi le **route model binding** : avec `/tasks/{task}/edit` et `edit(Task $task)`, Laravel va chercher **automatiquement** la tâche en base (et renvoie une 404 si elle n'existe pas). C'est ce que ton routeur ne faisait pas.

Comme l'authentification arrive à l'étape 14, nous utilisons un utilisateur **provisoire** (le premier de la base), isolé dans une seule méthode.

### Commandes

```bash
php artisan route:list --except-vendor
```

*(Pour générer d'un coup un contrôleur de ressource : `php artisan make:controller TaskController --resource --model=Task`. Ici, on remplace simplement le contenu du contrôleur existant.)*

### Code

`routes/web.php` : supprime l'ancienne route `/tasks` de l'étape 5 et ajoute :

```php
Route::patch('/tasks/{task}/toggle', [TaskController::class, 'toggle'])->name('tasks.toggle');
Route::resource('tasks', TaskController::class)->except('show');
```

`app/Http/Controllers/TaskController.php` :

```php
<?php

namespace App\Http\Controllers;

use App\Models\Task;
use App\Models\User;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\View\View;

class TaskController extends Controller
{
    public function index(): View
    {
        $tasks = $this->user()->tasks()->latest()->get();

        return view('tasks.index', compact('tasks'));
    }

    public function create(): View
    {
        return view('tasks.create');
    }

    public function store(Request $request): RedirectResponse
    {
        $this->user()->tasks()->create(['title' => $request->input('title')]);

        return redirect()->route('tasks.index');
    }

    public function edit(Task $task): View
    {
        $this->ensureOwner($task);

        return view('tasks.edit', compact('task'));
    }

    public function update(Request $request, Task $task): RedirectResponse
    {
        $this->ensureOwner($task);

        $task->update(['title' => $request->input('title')]);

        return redirect()->route('tasks.index');
    }

    public function toggle(Task $task): RedirectResponse
    {
        $this->ensureOwner($task);

        $task->update(['done' => ! $task->done]);

        return redirect()->route('tasks.index');
    }

    public function destroy(Task $task): RedirectResponse
    {
        $this->ensureOwner($task);

        $task->delete();

        return redirect()->route('tasks.index');
    }

    private function user(): User
    {
        // PROVISOIRE : pas encore d'authentification (étape 14)
        return User::firstOrFail();
    }

    private function ensureOwner(Task $task): void
    {
        // 403 si la tâche n'appartient pas à l'utilisateur courant
        abort_unless((int) $task->user_id === (int) $this->user()->id, 403);
    }
}
```

`resources/views/tasks/index.blade.php` :

```blade
@extends('layouts.app')

@section('title', 'Mes tâches')

@section('content')
    <h1>Mes tâches</h1>

    <p><a class="button" href="{{ route('tasks.create') }}">+ Nouvelle tâche</a></p>

    <ul class="tasks">
        @forelse ($tasks as $task)
            <li class="{{ $task->done ? 'done' : '' }}">
                <span class="title">{{ $task->title }}</span>

                <form method="POST" action="{{ route('tasks.toggle', $task) }}">
                    @csrf
                    @method('PATCH')
                    <button type="submit">{{ $task->done ? 'Rouvrir' : 'Terminer' }}</button>
                </form>

                <a class="button" href="{{ route('tasks.edit', $task) }}">Modifier</a>

                <form method="POST" action="{{ route('tasks.destroy', $task) }}"
                      onsubmit="return confirm('Supprimer cette tâche ?');">
                    @csrf
                    @method('DELETE')
                    <button type="submit" class="danger">Supprimer</button>
                </form>
            </li>
        @empty
            <li>Aucune tâche pour l'instant.</li>
        @endforelse
    </ul>
@endsection
```

`resources/views/tasks/_form.blade.php` (champs partagés) :

```blade
<label for="title">Titre</label>
<input type="text" id="title" name="title" value="{{ old('title', $task->title ?? '') }}" autofocus>
@error('title')
    <p class="error">{{ $message }}</p>
@enderror

<p>
    <button type="submit">Enregistrer</button>
    <a href="{{ route('tasks.index') }}">Annuler</a>
</p>
```

`resources/views/tasks/create.blade.php` :

```blade
@extends('layouts.app')

@section('title', 'Nouvelle tâche')

@section('content')
    <h1>Nouvelle tâche</h1>

    <form method="POST" action="{{ route('tasks.store') }}">
        @csrf
        @include('tasks._form')
    </form>
@endsection
```

`resources/views/tasks/edit.blade.php` :

```blade
@extends('layouts.app')

@section('title', 'Modifier la tâche')

@section('content')
    <h1>Modifier la tâche</h1>

    <form method="POST" action="{{ route('tasks.update', $task) }}">
        @csrf
        @method('PUT')
        @include('tasks._form')
    </form>
@endsection
```

### Résultat

Avec l'utilisateur Alice créé à l'étape 9, `/tasks` affiche ses tâches. Tu peux **ajouter**, **terminer/rouvrir**, **modifier** et **supprimer**. La liste, le formulaire et les liens fonctionnent.

Si tu envoies un titre **vide**, tu obtiens une **page d'erreur 500** (« NOT NULL constraint failed: tasks.title »). C'est normal : on n'a pas encore de validation (étape suivante).

**Ce que fait Laravel pour toi ici :**

* `Route::resource` génère les routes (vérifie avec `php artisan route:list`) ;
* `@csrf` insère le jeton et le middleware le vérifie : plus de classe `Csrf` à écrire ;
* `@method('PUT')` / `@method('DELETE')` : les navigateurs n'envoient que GET et POST, Laravel simule les autres méthodes avec un champ caché ;
* `Task $task` : récupération du modèle (route model binding) ;
* `latest()` : tri par `created_at` décroissant.

### Exercice

Dans `tasks/index.blade.php`, affiche sous le titre : « X tâche(s), dont Y terminée(s) », en utilisant les méthodes de la collection `$tasks`.

### Correction

```blade
<p>{{ $tasks->count() }} tâche(s), dont {{ $tasks->where('done', true)->count() }} terminée(s)</p>
```

`$tasks` est une **Collection** Laravel (pas un simple tableau) : elle offre `count()`, `where()`, `map()`, `filter()`, `sum()`...

---

## Étape 12 : Validation

### Explication

Ne jamais faire confiance aux données envoyées (cours de sécurité !). Laravel valide en **une instruction** : `$request->validate([...])`. Si une règle échoue, Laravel :

1. **redirige** automatiquement vers la page précédente ;
2. conserve les **erreurs** (accessibles avec `@error`) ;
3. conserve les **valeurs saisies** (accessibles avec `old()`).

Tu n'as plus rien à écrire pour ça, alors que dans ton MVC tu gérais le code 422, le réaffichage et les messages à la main.

### Commandes

```bash
php artisan make:request TaskRequest     # (pour l'exercice : une classe dédiée à la validation)
```

### Code

Dans `TaskController`, ajoute une méthode privée de validation :

```php
private function validated(Request $request): array
{
    return $request->validate(
        ['title' => ['required', 'string', 'max:150']],
        [
            'title.required' => 'Le titre est obligatoire.',
            'title.max' => 'Le titre ne doit pas dépasser :max caractères.',
        ],
    );
}
```

Puis dans `store()` et `update()`, utilise-la à la place de `$request->input('title')` :

```php
// store()
$this->user()->tasks()->create($this->validated($request));

// update()
$task->update($this->validated($request));
```

### Résultat

Un titre vide (ou de plus de 150 caractères) **ne plante plus** : tu retournes sur le formulaire avec le message sous le champ, et ce que tu avais saisi est conservé.

Règles les plus courantes :

| Règle | Signification |
|---|---|
| `required` | Obligatoire |
| `string`, `integer`, `boolean`, `date` | Type attendu |
| `min:3`, `max:150` | Longueur ou valeur minimale / maximale |
| `email` | Adresse email valide |
| `unique:users,email` | Valeur absente de la colonne `email` de la table `users` |
| `confirmed` | Le champ `xxx_confirmation` doit être identique |
| `in:a,b,c` | Valeur parmi une liste blanche |
| `nullable` | Peut être vide |

`validate()` **retourne uniquement les champs validés** : même si quelqu'un ajoute un champ `user_id` dans la requête, il n'ira pas plus loin (double protection avec `$fillable`).

Les messages par défaut de Laravel sont en anglais. Pour des messages français sur toute l'application, utilise un paquet de traductions communautaire (par exemple *laravel-lang*) et règle `APP_LOCALE=fr` dans `.env`.

### Exercice

Remplace `validated()` par un **Form Request** : une classe dédiée `TaskRequest` utilisée par `store()` et `update()`.

### Correction

`app/Http/Requests/TaskRequest.php` (généré par `make:request`) :

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class TaskRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;   // le squelette généré peut retourner false, ce qui provoque une erreur 403
    }

    public function rules(): array
    {
        return [
            'title' => ['required', 'string', 'max:150'],
        ];
    }

    public function messages(): array
    {
        return [
            'title.required' => 'Le titre est obligatoire.',
            'title.max' => 'Le titre ne doit pas dépasser :max caractères.',
        ];
    }
}
```

Dans le contrôleur :

```php
use App\Http\Requests\TaskRequest;

public function store(TaskRequest $request): RedirectResponse
{
    $this->user()->tasks()->create($request->validated());

    return redirect()->route('tasks.index');
}

public function update(TaskRequest $request, Task $task): RedirectResponse
{
    $this->ensureOwner($task);

    $task->update($request->validated());

    return redirect()->route('tasks.index');
}
```

Laravel valide **avant même** d'entrer dans la méthode du contrôleur. Dans la suite du cours, nous gardons la version `validated()` pour simplifier ; les deux sont équivalentes.

---

## Étape 13 : Sessions

### Explication

Une **session** conserve des informations entre deux requêtes. Laravel la gère pour toi : il crée le cookie de session (chiffré, `HttpOnly`, `SameSite=Lax` par défaut), stocke les données (dans la table `sessions` avec le réglage par défaut) et l'utilise pour la protection CSRF et la connexion.

Un usage très courant : les **messages flash**, des messages affichés **une seule fois** après une redirection (« Tâche créée. »).

### Commandes

Aucune. (La table `sessions` a été créée par les migrations par défaut. Le type de stockage se règle avec `SESSION_DRIVER` dans `.env`.)

### Code

Les méthodes de session les plus utiles :

```php
session()->put('theme', 'dark');       // enregistrer
session('theme');                      // lire (ou session()->get('theme', 'light'))
session()->has('theme');               // existe ?
session()->forget('theme');            // supprimer une clé
session()->flash('status', 'Fait !');  // disponible uniquement à la requête suivante
```

Ajoute les messages flash à l'application. Dans `TaskController`, enchaîne `->with('status', ...)` sur les redirections :

```php
// store()
return redirect()->route('tasks.index')->with('status', 'Tâche créée.');

// update()
return redirect()->route('tasks.index')->with('status', 'Tâche modifiée.');

// destroy()
return redirect()->route('tasks.index')->with('status', 'Tâche supprimée.');
```

Dans `layouts/app.blade.php`, juste avant `@yield('content')` :

```blade
@if (session('status'))
    <p class="flash">{{ session('status') }}</p>
@endif
```

### Résultat

Après un ajout, une modification ou une suppression, un bandeau vert s'affiche. **Il disparaît si tu recharges la page** : c'est ce qu'on appelle des données *flash*.

Pour tester la session « brute », ajoute temporairement dans `routes/web.php` :

```php
Route::get('/visites', function () {
    $visites = session()->increment('visites');

    return "Pages vues dans cette session : $visites";
});
```

Recharge `/visites` plusieurs fois : le compteur monte, car la session est conservée. Supprime cette route ensuite.

**Rappel :** `old('title')` (étapes 11 et 12) utilise aussi la session : Laravel y « flashe » les valeurs saisies quand la validation échoue.

### Exercice

Ajoute un message flash à `toggle()` : « Tâche terminée. » ou « Tâche rouverte. » selon le cas.

### Correction

```php
public function toggle(Task $task): RedirectResponse
{
    $this->ensureOwner($task);

    $task->update(['done' => ! $task->done]);

    return redirect()
        ->route('tasks.index')
        ->with('status', $task->done ? 'Tâche terminée.' : 'Tâche rouverte.');
}
```

---

## Étape 14 : Authentification

### Explication

Laravel fournit tout le nécessaire pour authentifier : le modèle `User`, le **hachage** des mots de passe, la **façade `Auth`**, la session et les **middlewares**. Pour comprendre, nous construisons **à la main** l'inscription, la connexion et la déconnexion avec ces briques.

> **En vrai projet**, tu utiliseras plutôt un **starter kit** officiel (choisi au moment de `laravel new`, basé sur *Laravel Fortify*), qui ajoute aussi la vérification d'email, la réinitialisation de mot de passe, la double authentification et une limitation de tentatives plus fine. Comprendre cette étape te permettra de les lire sans mystère.

Les briques :

| Besoin | Outil Laravel | Équivalent dans ton PHP |
|---|---|---|
| Hacher un mot de passe | `Hash::make($mdp)` | `password_hash()` |
| Vérifier + connecter | `Auth::attempt([...])` | `password_verify()` + `$_SESSION` |
| Renouveler la session | `$request->session()->regenerate()` | `session_regenerate_id(true)` |
| Utilisateur courant | `Auth::user()`, `$request->user()` | `$_SESSION['user_id']` + requête |
| Déconnecter | `Auth::logout()` + `invalidate()` | `logout()` maison |

### Commandes

```bash
php artisan make:controller Auth/RegisterController
php artisan make:controller Auth/LoginController
```

### Code

**1. Le contrôleur d'inscription.** `app/Http/Controllers/Auth/RegisterController.php` :

```php
<?php

namespace App\Http\Controllers\Auth;

use App\Http\Controllers\Controller;
use App\Models\User;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Hash;
use Illuminate\Validation\Rules\Password;
use Illuminate\View\View;

class RegisterController extends Controller
{
    public function create(): View
    {
        return view('auth.register');
    }

    public function store(Request $request): RedirectResponse
    {
        $data = $request->validate([
            'name' => ['required', 'string', 'max:255'],
            'email' => ['required', 'email', 'max:255', 'unique:users,email'],
            'password' => ['required', 'confirmed', Password::min(12)],
        ]);

        $user = User::create([
            'name' => $data['name'],
            'email' => $data['email'],
            'password' => Hash::make($data['password']),
        ]);

        Auth::login($user);
        $request->session()->regenerate();

        return redirect()->route('tasks.index');
    }
}
```

**2. Le contrôleur de connexion / déconnexion.** `app/Http/Controllers/Auth/LoginController.php` :

```php
<?php

namespace App\Http\Controllers\Auth;

use App\Http\Controllers\Controller;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Illuminate\View\View;

class LoginController extends Controller
{
    public function create(): View
    {
        return view('auth.login');
    }

    public function store(Request $request): RedirectResponse
    {
        $credentials = $request->validate([
            'email' => ['required', 'email'],
            'password' => ['required'],
        ]);

        if (! Auth::attempt($credentials, $request->boolean('remember'))) {
            return back()
                ->withErrors(['email' => 'Identifiants incorrects.'])
                ->onlyInput('email');
        }

        $request->session()->regenerate();   // protection contre la fixation de session

        return redirect()->intended(route('tasks.index'));
    }

    public function destroy(Request $request): RedirectResponse
    {
        Auth::logout();

        $request->session()->invalidate();
        $request->session()->regenerateToken();

        return redirect()->route('login');
    }
}
```

**3. Les routes** (`routes/web.php`, version finale complète) :

```php
<?php

use App\Http\Controllers\Auth\LoginController;
use App\Http\Controllers\Auth\RegisterController;
use App\Http\Controllers\TaskController;
use Illuminate\Support\Facades\Route;

Route::get('/', fn () => view('home'))->name('home');

// Réservé aux visiteurs non connectés
Route::middleware('guest')->group(function () {
    Route::get('/register', [RegisterController::class, 'create'])->name('register');
    Route::post('/register', [RegisterController::class, 'store']);

    Route::get('/login', [LoginController::class, 'create'])->name('login');
    Route::post('/login', [LoginController::class, 'store'])->middleware('throttle:5,1');
});

// Réservé aux utilisateurs connectés
Route::middleware('auth')->group(function () {
    Route::post('/logout', [LoginController::class, 'destroy'])->name('logout');

    Route::patch('/tasks/{task}/toggle', [TaskController::class, 'toggle'])->name('tasks.toggle');
    Route::resource('tasks', TaskController::class)->except('show');
});
```

**4. Les vues.** `resources/views/auth/register.blade.php` :

```blade
@extends('layouts.app')

@section('title', 'Inscription')

@section('content')
    <h1>Inscription</h1>

    <form method="POST" action="{{ route('register') }}">
        @csrf

        <label for="name">Nom</label>
        <input type="text" id="name" name="name" value="{{ old('name') }}" required autofocus>
        @error('name') <p class="error">{{ $message }}</p> @enderror

        <label for="email">Email</label>
        <input type="email" id="email" name="email" value="{{ old('email') }}" required>
        @error('email') <p class="error">{{ $message }}</p> @enderror

        <label for="password">Mot de passe (12 caractères minimum)</label>
        <input type="password" id="password" name="password" required>
        @error('password') <p class="error">{{ $message }}</p> @enderror

        <label for="password_confirmation">Confirmer le mot de passe</label>
        <input type="password" id="password_confirmation" name="password_confirmation" required>

        <p><button type="submit">Créer mon compte</button></p>
    </form>

    <p>Déjà inscrit ? <a href="{{ route('login') }}">Connexion</a></p>
@endsection
```

`resources/views/auth/login.blade.php` :

```blade
@extends('layouts.app')

@section('title', 'Connexion')

@section('content')
    <h1>Connexion</h1>

    <form method="POST" action="{{ route('login') }}">
        @csrf

        <label for="email">Email</label>
        <input type="email" id="email" name="email" value="{{ old('email') }}" required autofocus>
        @error('email') <p class="error">{{ $message }}</p> @enderror

        <label for="password">Mot de passe</label>
        <input type="password" id="password" name="password" required>

        <p><button type="submit">Se connecter</button></p>
    </form>

    <p>Pas encore de compte ? <a href="{{ route('register') }}">Inscription</a></p>
@endsection
```

Le **layout complet** (`layouts/app.blade.php`), avec le menu qui change selon l'état de connexion :

```blade
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>@yield('title', 'Todo') — {{ config('app.name') }}</title>
    <link rel="stylesheet" href="{{ asset('css/app.css') }}">
</head>
<body>
<header>
    <nav>
        <a href="{{ route('home') }}">Accueil</a>

        @auth
            <a href="{{ route('tasks.index') }}">Mes tâches</a>
            <span class="spacer"></span>
            <span>{{ auth()->user()->name }}</span>
            <form method="POST" action="{{ route('logout') }}">
                @csrf
                <button type="submit">Déconnexion</button>
            </form>
        @endauth

        @guest
            <span class="spacer"></span>
            <a href="{{ route('login') }}">Connexion</a>
            <a href="{{ route('register') }}">Inscription</a>
        @endguest
    </nav>
</header>
<main>
    @if (session('status'))
        <p class="flash">{{ session('status') }}</p>
    @endif

    @yield('content')
</main>
</body>
</html>
```

**5. On associe les tâches à l'utilisateur connecté.** Dans `TaskController`, **une seule méthode change** (la méthode provisoire de l'étape 11) :

```php
use Illuminate\Support\Facades\Auth;

private function user(): User
{
    return Auth::user();
}
```

Contrôleur complet, avec les messages flash et la validation (version finale) :

```php
<?php

namespace App\Http\Controllers;

use App\Models\Task;
use App\Models\User;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Illuminate\View\View;

class TaskController extends Controller
{
    public function index(): View
    {
        $tasks = $this->user()->tasks()->latest()->get();

        return view('tasks.index', compact('tasks'));
    }

    public function create(): View
    {
        return view('tasks.create');
    }

    public function store(Request $request): RedirectResponse
    {
        $this->user()->tasks()->create($this->validated($request));

        return redirect()->route('tasks.index')->with('status', 'Tâche créée.');
    }

    public function edit(Task $task): View
    {
        $this->ensureOwner($task);

        return view('tasks.edit', compact('task'));
    }

    public function update(Request $request, Task $task): RedirectResponse
    {
        $this->ensureOwner($task);

        $task->update($this->validated($request));

        return redirect()->route('tasks.index')->with('status', 'Tâche modifiée.');
    }

    public function toggle(Task $task): RedirectResponse
    {
        $this->ensureOwner($task);

        $task->update(['done' => ! $task->done]);

        return redirect()
            ->route('tasks.index')
            ->with('status', $task->done ? 'Tâche terminée.' : 'Tâche rouverte.');
    }

    public function destroy(Task $task): RedirectResponse
    {
        $this->ensureOwner($task);

        $task->delete();

        return redirect()->route('tasks.index')->with('status', 'Tâche supprimée.');
    }

    private function user(): User
    {
        return Auth::user();
    }

    private function ensureOwner(Task $task): void
    {
        abort_unless((int) $task->user_id === (int) $this->user()->id, 403);
    }

    private function validated(Request $request): array
    {
        return $request->validate(
            ['title' => ['required', 'string', 'max:150']],
            [
                'title.required' => 'Le titre est obligatoire.',
                'title.max' => 'Le titre ne doit pas dépasser :max caractères.',
            ],
        );
    }
}
```

### Résultat

1. `/tasks` sans être connecté → **redirection automatique vers `/login`** (middleware `auth`).
2. `/register` → crée un compte (mot de passe de 12 caractères minimum, confirmé), te connecte et t'envoie sur tes tâches.
3. **Déconnexion** (bouton du menu) → retour sur `/login`.
4. **Connexion** → tu es redirigé vers la page que tu voulais voir (`intended`), ou vers tes tâches.
5. Crée deux comptes : chacun **ne voit que ses tâches**. Essaie d'ouvrir `/tasks/ID_DE_L_AUTRE/edit` : **403 Interdit**.

**Ce que Laravel fait pour toi, ce que tu dois toujours faire toi-même :**

* Laravel hache avec **bcrypt** par défaut, vérifie le mot de passe, gère la session et le cookie « se souvenir de moi ».
* `Auth::attempt()` cherche l'utilisateur par `email` et vérifie le hash avec `password_verify` : tu n'écris plus de SQL de connexion.
* `redirect()->intended()` ramène l'utilisateur vers la page qu'il visait avant d'être renvoyé sur `/login`.
* `throttle:5,1` limite à 5 tentatives de connexion par minute.
* **L'autorisation reste ton travail** : savoir qu'un utilisateur est connecté ne signifie pas qu'il a le droit de modifier **cette** tâche. D'où `ensureOwner()` (un contrôle « propriétaire ou 403 »). Laravel propose des **Policies** pour structurer cela quand l'application grandit.
* Le modèle `User` généré par Laravel sait aussi hacher automatiquement un mot de passe (cast `hashed`) : nous le faisons ici explicitement pour que le mécanisme soit visible.

### Exercice

Ajoute une case **« Se souvenir de moi »** au formulaire de connexion (le contrôleur la gère déjà).

### Correction

Dans `auth/login.blade.php`, avant le bouton :

```blade
<label>
    <input type="checkbox" name="remember"> Se souvenir de moi
</label>
```

`$request->boolean('remember')` retourne `true` si la case est cochée, et `Auth::attempt($credentials, true)` crée alors un cookie de connexion longue durée.

---

## Étape 15 : Middleware

### Explication

Un **middleware** est une couche qui s'exécute **avant** (ou **après**) ton contrôleur, comme les filtres d'une chaîne : il peut laisser passer la requête, la modifier ou l'arrêter. Tu en utilises déjà depuis le début, sans le savoir :

| Middleware | Rôle |
|---|---|
| `auth` | Refuse les visiteurs non connectés (redirige vers `login`) |
| `guest` | Refuse les utilisateurs déjà connectés |
| `throttle:5,1` | Limite le nombre de requêtes |
| *(groupe `web`)* | Démarre la session, chiffre les cookies, **vérifie le jeton CSRF** |

Dans ton PHP MVC, ces contrôles étaient dispersés dans les contrôleurs. Ici, ils sont **déclarés** sur les routes.

Écrivons le nôtre : limiter chaque utilisateur à **20 tâches**.

### Commandes

```bash
php artisan make:middleware LimitTasks
```

### Code

`app/Http/Middleware/LimitTasks.php` :

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class LimitTasks
{
    private const MAX_TASKS = 20;

    public function handle(Request $request, Closure $next): Response
    {
        if ($request->user()->tasks()->count() >= self::MAX_TASKS) {
            return redirect()
                ->route('tasks.index')
                ->withErrors(['limit' => 'Tu as atteint la limite de ' . self::MAX_TASKS . ' tâches.']);
        }

        return $next($request);   // on laisse passer la requête vers le contrôleur
    }
}
```

On lui donne un **alias** dans `bootstrap/app.php` (c'est ici que Laravel 11+ configure les middlewares) :

```php
use App\Http\Middleware\LimitTasks;
use Illuminate\Foundation\Configuration\Middleware;

// ...
    ->withMiddleware(function (Middleware $middleware): void {
        $middleware->alias([
            'task.limit' => LimitTasks::class,
        ]);
    })
```

Dans `routes/web.php`, groupe `auth` : on déclare la création à part pour lui appliquer le middleware.

```php
Route::middleware('auth')->group(function () {
    Route::post('/logout', [LoginController::class, 'destroy'])->name('logout');

    Route::patch('/tasks/{task}/toggle', [TaskController::class, 'toggle'])->name('tasks.toggle');

    Route::post('/tasks', [TaskController::class, 'store'])
        ->middleware('task.limit')
        ->name('tasks.store');

    Route::resource('tasks', TaskController::class)->except(['show', 'store']);
});
```

Dans `tasks/index.blade.php`, sous le titre :

```blade
@error('limit')
    <p class="error">{{ $message }}</p>
@enderror
```

### Résultat

Pour tester sans créer 20 tâches, passe temporairement `MAX_TASKS` à `3` : à la quatrième création, tu es renvoyé sur la liste avec le message d'erreur, **sans que le contrôleur ne soit appelé**.

`$next($request)` est le point clé : appeler la suite de la chaîne (le contrôleur) ou retourner autre chose (une redirection, une erreur) pour **bloquer**. Le code **après** `$next(...)` s'exécute à la sortie, ce qui permet d'agir sur la réponse (journaliser, ajouter un en-tête...).

> **Dans Laravel 13**, il existe aussi l'attribut `#[Middleware('nom')]` pour déclarer un middleware directement sur un contrôleur ou une méthode.

### Exercice

Crée un middleware `LogTaskActions` qui écrit dans le journal, **après** chaque requête, l'identifiant de l'utilisateur, la méthode HTTP et l'URL. Applique-le à tout le groupe `auth`.

### Correction

```bash
php artisan make:middleware LogTaskActions
```

```php
use Illuminate\Support\Facades\Log;

public function handle(Request $request, Closure $next): Response
{
    $response = $next($request);   // d'abord le contrôleur…

    Log::info('Action utilisateur', [
        'user_id' => $request->user()?->id,
        'method' => $request->method(),
        'url' => $request->path(),
        'status' => $response->getStatusCode(),
    ]);

    return $response;              // …puis on journalise
}
```

```php
use App\Http\Middleware\LogTaskActions;

Route::middleware(['auth', LogTaskActions::class])->group(function () {
    // ...
});
```

*(On peut utiliser directement la classe, sans alias.)* Consulte `storage/logs/laravel.log` après quelques actions.

---

## Étape 16 : `.env`

### Explication

Le fichier **`.env`** contient la configuration **propre à chaque environnement** (ton poste, le serveur de production) et les **secrets**. Il ne va **jamais dans Git** (Laravel l'ignore déjà dans `.gitignore`) : c'est exactement ce que tu as vu au cours de sécurité, mais ici tout est **intégré**, sans bibliothèque à ajouter.

Le circuit :

```
.env  →  config/*.php (via env())  →  ton code (via config())
```

**Règle d'or :** `env('CLE')` ne s'utilise **que dans les fichiers de `config/`**. Partout ailleurs, on utilise `config('fichier.cle')`. Sinon, en production, après la mise en cache de la configuration, `env()` renverrait `null`.

### Commandes

```bash
php artisan key:generate     # génère APP_KEY (déjà fait à l'installation)
php artisan config:clear     # vide le cache de configuration
php artisan config:cache     # met la configuration en cache (production)
php artisan about            # affiche la configuration active
```

### Code

Extrait d'un `.env` (les valeurs exactes peuvent différer selon la version) :

```
APP_NAME="Todo App"
APP_ENV=local
APP_KEY=base64:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx=
APP_DEBUG=true
APP_URL=http://localhost:8000

APP_LOCALE=fr

DB_CONNECTION=sqlite

SESSION_DRIVER=database
SESSION_LIFETIME=120
```

Le fichier **`.env.example`** est, lui, **versionné** (sans valeurs secrètes) pour documenter les variables attendues.

**Passer à MySQL (XAMPP/WAMP)** : crée une base `todo_laravel` (phpMyAdmin), puis modifie le `.env` :

```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=todo_laravel
DB_USERNAME=root
DB_PASSWORD=
```

```bash
php artisan config:clear
php artisan migrate
```

*(Les tables sont recréées dans MySQL ; il faudra de nouveau créer un compte.)*

**Une configuration à toi :** `config/todo.php` :

```php
<?php

return [
    'max_tasks' => (int) env('TODO_MAX_TASKS', 20),
];
```

`.env` : `TODO_MAX_TASKS=20`. Dans le code : `config('todo.max_tasks')`.

### Résultat

Après avoir changé `DB_CONNECTION`, l'application utilise MySQL sans modifier une seule ligne de PHP : la configuration est **séparée du code**. `config('app.name')` (utilisé dans le layout) lit `APP_NAME`.

**Les réglages importants pour la production** (rappel du cours de sécurité) :

* `APP_ENV=production` et **`APP_DEBUG=false`** (jamais de pages d'erreur détaillées en ligne) ;
* un `.env` **différent** pour chaque environnement, avec ses propres mots de passe ;
* `composer install --no-dev --optimize-autoloader`, puis `php artisan config:cache`, `route:cache` et `view:cache` ;
* `SESSION_SECURE_COOKIE=true` si le site est en HTTPS (obligatoire en production) ;
* seul le dossier `public/` doit être accessible depuis le web.

### Exercice

1. Remplace la constante `MAX_TASKS` du middleware par la valeur `config('todo.max_tasks')`, configurable depuis le `.env`.
2. Teste avec `TODO_MAX_TASKS=2` dans le `.env`.

### Correction

`app/Http/Middleware/LimitTasks.php` :

```php
public function handle(Request $request, Closure $next): Response
{
    $max = config('todo.max_tasks');

    if ($request->user()->tasks()->count() >= $max) {
        return redirect()
            ->route('tasks.index')
            ->withErrors(['limit' => "Tu as atteint la limite de $max tâches."]);
    }

    return $next($request);
}
```

Dans `.env` : `TODO_MAX_TASKS=2`. Puis :

```bash
php artisan config:clear
```

La limite passe à 2 tâches, sans toucher au code.

---

## Ce que Laravel prend en charge à la place de ton PHP MVC

| Besoin | Ton mini-framework | Laravel |
|---|---|---|
| **Point d'entrée** | `public/index.php` crée le conteneur et le routeur | `public/index.php` → `bootstrap/app.php` (configure routes, middlewares, exceptions) |
| **Routage** | `Router` maison (expressions régulières, `{id}` numérique) | `Route::get/post/resource`, contraintes, noms, groupes, cache des routes, `route:list` |
| **Requête** | Classe `Request` (query, input) | `Illuminate\Http\Request` : JSON, fichiers, cookies, session, utilisateur, validation |
| **Injection de dépendances** | `Container` avec réflexion | Service Container (liaisons, singletons, providers) ; injection dans contrôleurs et middlewares |
| **Récupération d'une ressource** | `find($id)` + gestion du 404 à la main | **Route model binding** (`Task $task`, 404 automatique) |
| **Contrôleurs** | `Controller` de base avec `view()` et `redirect()` | Helpers globaux `view()`, `redirect()`, `back()`, `abort()` |
| **Modèles / données** | `Task` + `TaskRepository` + SQL PDO | **Eloquent** (Active Record), query builder, relations, casts, scopes |
| **Schéma de base** | `schema.sql` exécuté à la main | **Migrations** versionnées, `migrate`, `rollback`, seeders, fabriques |
| **Connexion à la base** | `Database::connect()` + `config/database.php` | `config/database.php` + `.env`, plusieurs connexions |
| **Vues** | Classe `View` + `e()` (puis Twig) | **Blade** compilé et mis en cache, échappement automatique, layouts, composants |
| **Validation** | `Task::validateTitle()` + code 422 + réaffichage à la main | `validate()`, 100+ règles, messages, Form Requests, **erreurs et `old()` automatiques** |
| **Messages flash** | Non prévus | `->with('status', ...)` + `session('status')` |
| **Sessions** | `startSecureSession()` écrite à la main | Sessions configurables (`SESSION_DRIVER`), cookie sécurisé par défaut |
| **CSRF** | Classe `Csrf` + vérification dans chaque contrôleur | Middleware automatique + `@csrf` (nommé `PreventRequestForgery` en Laravel 13) |
| **Authentification** | `password_hash`, `password_verify`, `$_SESSION` à la main | `Auth`, `Hash`, `Auth::attempt()`, `intended()`, « se souvenir de moi », middlewares `auth`/`guest` |
| **Middlewares** | N'existent pas (contrôles dans chaque action) | Pipeline, groupes, alias, `throttle` |
| **Configuration et secrets** | `.env` à gérer avec `vlucas/phpdotenv` | `.env` + `config()` intégrés, cache de configuration |
| **Outils en ligne de commande** | Aucun | **Artisan** : générateurs (`make:*`), migrations, console `tinker`, `route:list`, `about`… |
| **Erreurs et journaux** | Basique | Page d'erreur détaillée en debug, pages 403/404/419/500, journaux (`storage/logs`) |
| **Tests** | Non prévus | PHPUnit / Pest intégrés (`php artisan test`) |

**Ce que Laravel ne fait pas à ta place :** décider **qui a le droit de faire quoi** (l'autorisation, ici `ensureOwner()`), valider **toutes** tes entrées, concevoir ton architecture et ton interface. Tout ce que tu as appris en sécurité reste valable : Laravel te donne les bons outils (échappement Blade, requêtes préparées Eloquent, CSRF, hachage), mais il faut les utiliser, et ne pas les contourner (`{!! !!}`, `DB::raw()`, désactivation du CSRF...).

---

## Récapitulatif des commandes Artisan utilisées

| Commande | Rôle |
|---|---|
| `laravel new todo-app` | Crée un nouveau projet |
| `php artisan serve` | Lance le serveur de développement |
| `php artisan about` | Résumé de l'application et de l'environnement |
| `php artisan list` | Liste toutes les commandes |
| `php artisan route:list` | Liste les routes |
| `php artisan make:controller NomController` | Crée un contrôleur (`--resource` : les 7 méthodes CRUD) |
| `php artisan make:migration create_xxx_table` | Crée une migration |
| `php artisan migrate` | Exécute les migrations |
| `php artisan migrate:status` | Affiche l'état des migrations |
| `php artisan migrate:rollback` | Annule le dernier lot de migrations |
| `php artisan migrate:fresh` | Recrée toute la base (développement uniquement) |
| `php artisan db:table tasks` | Affiche la structure d'une table |
| `php artisan make:model Task` | Crée un modèle (`-m` migration, `-c` contrôleur, `-a` tout) |
| `php artisan tinker` | Console interactive pour tester le code |
| `php artisan make:request TaskRequest` | Crée un Form Request |
| `php artisan make:middleware LimitTasks` | Crée un middleware |
| `php artisan key:generate` | Génère `APP_KEY` |
| `php artisan config:clear` / `config:cache` | Vide / met en cache la configuration |

---

## Structure finale du projet (fichiers créés ou modifiés)

```
todo-app/
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── Auth/
│   │   │   │   ├── LoginController.php
│   │   │   │   └── RegisterController.php
│   │   │   ├── PageController.php
│   │   │   └── TaskController.php
│   │   ├── Middleware/
│   │   │   ├── LimitTasks.php
│   │   │   └── LogTaskActions.php
│   │   └── Requests/
│   │       └── TaskRequest.php          (exercice 12)
│   └── Models/
│       ├── Task.php
│       └── User.php                     (méthode tasks() ajoutée)
├── bootstrap/app.php                    (alias du middleware)
├── config/todo.php
├── database/migrations/xxxx_create_tasks_table.php
├── public/css/app.css
├── resources/views/
│   ├── layouts/app.blade.php
│   ├── partials/footer.blade.php
│   ├── auth/{login,register}.blade.php
│   ├── tasks/{index,create,edit,_form}.blade.php
│   └── home.blade.php
├── routes/web.php
└── .env
```

---

## Et maintenant ?

Tu as maintenant les fondations pour développer une vraie application Laravel. Dans cet ordre, je te conseille d'apprendre :

1. **Les Policies** (`php artisan make:policy`) : structurer les autorisations proprement, à la place de `ensureOwner()`.
2. **La pagination** : `->paginate(10)` et `{{ $tasks->links() }}` dès que les listes grandissent.
3. **Les fabriques et les seeders** : générer des données de test (`php artisan db:seed`).
4. **Les tests** : `php artisan test` avec Pest ou PHPUnit (tests de fonctionnalités sur tes routes).
5. **Les starter kits officiels** (React, Vue ou Livewire, basés sur Fortify) : authentification complète avec vérification d'email, réinitialisation de mot de passe et double authentification.
6. **Vite et Tailwind** pour les assets front-end, que nous avons volontairement laissés de côté.
7. **Les relations avancées** (plusieurs-à-plusieurs), les **files d'attente** (queues), les **emails** (mail) et le **stockage de fichiers** (n'oublie pas les règles du cours de sécurité sur les uploads).
8. **Le déploiement** (Laravel Cloud, Forge ou un serveur classique) avec la checklist de production de l'étape 16.

La documentation officielle (laravel.com/docs, choisis la version 13.x) est excellente : maintenant que tu comprends ce qui se passe sous le capot, elle sera beaucoup plus facile à lire.
