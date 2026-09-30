# Cours PHP : comprendre le MVC en construisant un mini-framework

Dans ce cours, tu vas construire **à la main** un petit framework MVC, étape par étape, avec une application de **gestion de tâches**. À la fin, tu sauras ce que font réellement Laravel ou Symfony « sous le capot » : routeur, contrôleurs, modèles, vues, conteneur de dépendances, moteur de templates.

**Prérequis :** PHP 8.1+, Composer, les bases de la POO, PDO et MySQL (MySQL n'est utilisé qu'à partir de l'étape 9).

**Ce que tu vas construire :** une application qui permet de lister, voir, créer, modifier, terminer, supprimer et filtrer des tâches.

**Méthode pour chaque étape :** concept → fichiers → code → explication → test → exercice.

**Lancer l'application (à partir de l'étape 3) :** depuis le dossier du projet,

```bash
php -S localhost:8000 -t public public/index.php
```

Puis ouvre `http://localhost:8000`. (Avec XAMPP/WAMP, utilise quand même cette commande avec le PHP de XAMPP : tu profiteras de son MySQL et de phpMyAdmin. La configuration Apache est expliquée à l'étape 11.)

**Plan :**

1. Pourquoi séparer le code
2. Le principe MVC
3. Le routeur
4. Les contrôleurs
5. Les modèles
6. Les vues
7. Les paramètres d'URL
8. Les formulaires
9. Connexion à une base de données
10. Utilisation de classes (injection de dépendances)
11. Organisation des fichiers
12. Introduction à Twig

---

## Étape 1 : Pourquoi séparer le code

### Concept

Quand on débute, on met tout dans un seul fichier : connexion à la base, traitement du formulaire, requêtes SQL et HTML. Ça marche... jusqu'à ce que le projet grossisse.

Regarde ce fichier `tasks.php` (à **ne pas** reproduire, c'est un contre-exemple) :

```php
<?php
// 1. Connexion à la base (à recopier dans CHAQUE fichier)
$pdo = new PDO('mysql:host=localhost;dbname=mvc_taches;charset=utf8mb4', 'root', '');

// 2. Traitement du formulaire
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['title'])) {
    // 3. SQL directement dans la page, et vulnérable à l'injection SQL !
    $pdo->exec("INSERT INTO tasks (title) VALUES ('" . $_POST['title'] . "')");
    header('Location: tasks.php');
    exit;
}

$tasks = $pdo->query('SELECT * FROM tasks')->fetchAll(PDO::FETCH_ASSOC);
?>
<html>
<body>
    <h1>Tâches</h1>
    <form method="post"><input name="title"><button>Ajouter</button></form>
    <ul>
        <?php foreach ($tasks as $t): ?>
            <li><?= $t['title'] ?></li>  <!-- 4. Faille XSS : rien n'est échappé -->
        <?php endforeach; ?>
    </ul>
</body>
</html>
```

**Les problèmes :**

| Problème | Conséquence |
|---|---|
| Configuration, SQL, logique et HTML mélangés | Impossible de changer l'un sans risquer de casser l'autre |
| Connexion recopiée dans chaque page | Changer un mot de passe = modifier 20 fichiers |
| Une page = un fichier (`tasks.php`, `edit.php`...) | URL peu propres, code dupliqué |
| Un designer et un développeur travaillent sur le même fichier | Conflits permanents |
| Failles de sécurité faciles à oublier | Injection SQL, XSS |
| Difficile à tester et à faire évoluer | Le projet devient ingérable |

**La solution :** séparer le code par **responsabilité**. Chaque morceau fait une seule chose, et c'est exactement ce que propose le modèle **MVC**.

### Petit exercice

Dans `tasks.php` ci-dessus, liste les **quatre responsabilités** différentes qui sont mélangées.

### Correction

1. La **configuration** et la connexion à la base de données.
2. Le **traitement de la requête** (lire le formulaire, décider quoi faire, rediriger).
3. L'**accès aux données** (les requêtes SQL).
4. L'**affichage** (le HTML).

Les étapes suivantes donnent une place à chacune.

---

## Étape 2 : Le principe MVC

### Concept

**MVC** signifie **Modèle – Vue – Contrôleur** :

| Partie | Rôle | Exemple dans notre application |
|---|---|---|
| **Modèle** (Model) | Les **données** et les règles métier : lire/écrire en base, valider | `Task`, `TaskRepository` |
| **Vue** (View) | L'**affichage** : le HTML, sans logique métier | `views/tasks/index.php` |
| **Contrôleur** (Controller) | Le **chef d'orchestre** : reçoit la demande, appelle le modèle, choisit la vue | `TaskController` |

À cela s'ajoutent deux éléments que tous les frameworks possèdent :

* un **point d'entrée unique** (le *front controller*, `public/index.php`) par lequel **toutes** les requêtes passent ;
* un **routeur**, qui décide quel contrôleur appeler selon l'URL.

Voici le trajet d'une requête `GET /tasks/3` :

```
Navigateur ── GET /tasks/3 ──▶ public/index.php   (point d'entrée unique)
                                     │
                                     ▼
                                  Routeur   ── trouve : TaskController::show(3)
                                     │
                                     ▼
                                Contrôleur ──▶ Modèle (TaskRepository) ──▶ Base de données
                                     │                  │
                                     │◀── objet Task ───┘
                                     ▼
                                   Vue (HTML) ──▶ réponse renvoyée au navigateur
```

Règle d'or : **chaque couche ne fait que son travail**. Le contrôleur ne contient pas de SQL, la vue ne contient pas de logique métier, le modèle ne produit pas de HTML.

### Fichiers à créer

Crée le dossier du projet et son squelette :

```bash
mkdir taches-mvc
cd taches-mvc
mkdir -p config public src/Controller src/Core src/Model storage views/tasks views/errors database
```

*(PowerShell : `mkdir config, public, src/Controller, src/Core, src/Model, storage, views/tasks, views/errors, database`.)*

`composer.json` :

```json
{
    "name": "moi/taches-mvc",
    "description": "Mini-framework MVC pour apprendre",
    "type": "project",
    "require": {
        "php": ">=8.1"
    },
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```

```bash
composer dump-autoload
```

`public/index.php` (un simple test pour l'instant) :

```php
<?php

declare(strict_types=1);

require __DIR__ . '/../vendor/autoload.php';

echo 'Le MVC démarre !';
```

### Explication

* Seul le dossier **`public/`** est accessible depuis le navigateur. Le code (`src/`), les vues et la configuration restent **hors de portée** du public : c'est une protection de base.
* L'autoload PSR-4 (`App\` → `src/`) charge nos classes automatiquement, comme dans le cours Composer.

### Test

```bash
php -S localhost:8000 -t public public/index.php
```

`http://localhost:8000` affiche « Le MVC démarre ! ».

> **Dans Laravel :** la structure est la même : `public/index.php` est le point d'entrée unique, `app/` contient le code, `resources/views` les vues, `config/` la configuration.

### Petit exercice

Pour chaque action, dis si elle relève du **M**odèle, de la **V**ue ou du **C**ontrôleur :

1. vérifier que le titre d'une tâche n'est pas vide ;
2. afficher la liste des tâches sous forme de `<ul>` ;
3. lire `$_POST['title']` et décider de rediriger vers la liste ;
4. exécuter `INSERT INTO tasks ...` ;
5. colorer en gris les tâches terminées ;
6. choisir quelle page afficher après une erreur de validation.

### Correction

1. **Modèle** (règle métier)
2. **Vue**
3. **Contrôleur**
4. **Modèle** (accès aux données)
5. **Vue**
6. **Contrôleur**

---

## Étape 3 : Le routeur

### Concept

Avec un seul point d'entrée, `index.php` reçoit toutes les requêtes. Il faut donc un **routeur** : il associe une **méthode HTTP + une URL** à un morceau de code.

```
GET  /        → page d'accueil
GET  /tasks   → liste des tâches
POST /tasks   → création d'une tâche
```

Pour cela, on crée aussi un petit objet **`Request`** qui représente la requête entrante (méthode, chemin, paramètres) au lieu de manipuler partout `$_GET` et `$_POST`.

### Fichiers à créer

* `src/Core/Request.php`
* `src/Core/Router.php`
* `config/routes.php`
* `public/index.php` (modifié)

`src/Core/Request.php` :

```php
<?php

declare(strict_types=1);

namespace App\Core;

final class Request
{
    public function __construct(
        private string $method,
        private string $path,
        private array $query = [],
        private array $post = [],
    ) {
    }

    /** Construit la requête à partir des variables globales de PHP. */
    public static function fromGlobals(): self
    {
        $path = parse_url($_SERVER['REQUEST_URI'] ?? '/', PHP_URL_PATH) ?: '/';

        return new self(
            strtoupper($_SERVER['REQUEST_METHOD'] ?? 'GET'),
            $path,
            $_GET,
            $_POST,
        );
    }

    public function method(): string
    {
        return $this->method;
    }

    /** Chemin sans barre finale : "/tasks/" devient "/tasks". */
    public function path(): string
    {
        return $this->path === '/' ? '/' : rtrim($this->path, '/');
    }

    /** Paramètre de l'URL (?cle=valeur). */
    public function query(string $key, mixed $default = null): mixed
    {
        return $this->query[$key] ?? $default;
    }

    /** Donnée envoyée par un formulaire (POST). */
    public function input(string $key, mixed $default = null): mixed
    {
        return $this->post[$key] ?? $default;
    }
}
```

`src/Core/Router.php` :

```php
<?php

declare(strict_types=1);

namespace App\Core;

final class Router
{
    /** @var array<string, array<string, callable|array>> */
    private array $routes = [];

    public function get(string $path, callable|array $handler): void
    {
        $this->routes['GET'][$path] = $handler;
    }

    public function post(string $path, callable|array $handler): void
    {
        $this->routes['POST'][$path] = $handler;
    }

    public function dispatch(Request $request): void
    {
        $handler = $this->routes[$request->method()][$request->path()] ?? null;

        if ($handler === null) {
            http_response_code(404);
            echo '<h1>404</h1><p>Page introuvable.</p>';
            return;
        }

        $handler($request);
    }
}
```

`config/routes.php` :

```php
<?php

declare(strict_types=1);

use App\Core\Request;
use App\Core\Router;

return static function (Router $router): void {
    $router->get('/', function (Request $request): void {
        echo '<h1>Accueil</h1>';
    });

    $router->get('/about', function (Request $request): void {
        echo '<h1>À propos</h1>';
    });
};
```

`public/index.php` :

```php
<?php

declare(strict_types=1);

use App\Core\Request;
use App\Core\Router;

require __DIR__ . '/../vendor/autoload.php';

// Serveur intégré de PHP : laisser passer les fichiers statiques (CSS, images…)
if (PHP_SAPI === 'cli-server') {
    $file = __DIR__ . (string) parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);

    if (is_file($file)) {
        return false;
    }
}

$router = new Router();
(require __DIR__ . '/../config/routes.php')($router);

$router->dispatch(Request::fromGlobals());
```

### Explication

* `Router::get()` et `post()` **enregistrent** une route : un tableau `méthode → chemin → action`.
* `dispatch()` cherche la route qui correspond à la requête. Sinon : **404**. Sinon, il exécute l'action en lui passant la `Request`.
* `config/routes.php` **retourne une fonction** qui déclare toutes les routes : on garde ainsi la liste des URL de l'application au même endroit.
* Dans `index.php`, le bloc `PHP_SAPI === 'cli-server'` sert uniquement au serveur intégré : sans lui, `style.css` serait aussi envoyé au routeur. Avec Apache, c'est le `.htaccess` qui s'en charge (étape 11).
* Le dernier paramètre `Request::fromGlobals()` fait le lien entre le monde de PHP (`$_SERVER`, `$_GET`, `$_POST`) et notre code orienté objet.

### Test

Lance le serveur et visite :

* `http://localhost:8000/` → « Accueil » ;
* `http://localhost:8000/about` → « À propos » ;
* `http://localhost:8000/nimportequoi` → « 404 ».

Tu peux aussi regarder le code HTTP renvoyé : `curl -i http://localhost:8000/nimportequoi` affiche `HTTP/1.1 404 Not Found`.

> **Dans Laravel :** les routes se déclarent dans `routes/web.php`, avec `Route::get('/tasks', [TaskController::class, 'index']);`. Le principe est identique.

### Petit exercice

Ajoute une route **POST** `/ping` qui affiche `pong`. Vérifie qu'un `GET /ping` donne bien une 404.

### Correction

Dans `config/routes.php`, à l'intérieur de la fonction :

```php
$router->post('/ping', function (Request $request): void {
    echo 'pong';
});
```

Test (un navigateur ne sait envoyer un POST qu'avec un formulaire, donc on utilise `curl`) :

```bash
curl -X POST http://localhost:8000/ping     # pong
curl -i http://localhost:8000/ping          # 404 : cette route n'existe qu'en POST
```

---

## Étape 4 : Les contrôleurs

### Concept

Les closures dans `routes.php` deviennent vite ingérables. On regroupe les actions liées dans des **classes** : les **contrôleurs**. Chaque **action** est une méthode publique.

Le rôle d'un contrôleur :

1. lire la **requête** (paramètres, formulaire) ;
2. demander au **modèle** de travailler ;
3. choisir la **vue** à afficher ou **rediriger**.

Un bon contrôleur est **mince** : il orchestre, mais ne contient ni SQL ni HTML complexe.

### Fichiers à créer

* `src/Controller/HomeController.php`
* `src/Controller/TaskController.php`
* `config/routes.php` (modifié)
* `src/Core/Router.php` (modifié : appeler `[Classe, 'méthode']`)

`src/Controller/HomeController.php` :

```php
<?php

declare(strict_types=1);

namespace App\Controller;

use App\Core\Request;

final class HomeController
{
    public function index(Request $request): void
    {
        echo '<h1>Accueil</h1>';
        echo '<p><a href="/tasks">Voir les tâches</a></p>';
    }
}
```

`src/Controller/TaskController.php` :

```php
<?php

declare(strict_types=1);

namespace App\Controller;

use App\Core\Request;

final class TaskController
{
    public function index(Request $request): void
    {
        echo '<h1>Liste des tâches</h1>';
    }
}
```

`config/routes.php` :

```php
<?php

declare(strict_types=1);

use App\Controller\HomeController;
use App\Controller\TaskController;
use App\Core\Router;

return static function (Router $router): void {
    $router->get('/', [HomeController::class, 'index']);
    $router->get('/tasks', [TaskController::class, 'index']);
};
```

Dans `src/Core/Router.php`, remplace la dernière ligne de `dispatch()` (`$handler($request);`) par :

```php
        if (is_array($handler)) {
            [$class, $action] = $handler;
            (new $class())->$action($request);
            return;
        }

        $handler($request);
```

### Explication

* `[TaskController::class, 'index']` signifie « la méthode `index` de la classe `TaskController` ». `::class` retourne le nom complet de la classe (avec son namespace).
* Le routeur **instancie** le contrôleur (`new $class()`) puis appelle l'action avec la requête : `(new $class())->$action($request)`. C'est du PHP dynamique : `$class` et `$action` sont des variables contenant des noms.
* Les closures sont toujours acceptées (`$handler($request)`) : on peut en garder pour des tests rapides.

### Test

* `http://localhost:8000/` → accueil avec le lien ;
* `http://localhost:8000/tasks` → « Liste des tâches » ;
* `/about` n'existe plus : 404.

> **Dans Laravel :** `php artisan make:controller TaskController` génère la classe dans `app/Http/Controllers`. Le `Request` est injecté dans les méthodes du contrôleur de la même manière.

### Petit exercice

Ajoute une méthode `about()` dans `HomeController` et la route `GET /about` qui l'appelle.

### Correction

Dans `HomeController` :

```php
    public function about(Request $request): void
    {
        echo '<h1>À propos</h1><p>Mini-framework MVC pédagogique.</p>';
    }
```

Dans `config/routes.php` :

```php
    $router->get('/about', [HomeController::class, 'about']);
```

---

## Étape 5 : Les modèles

### Concept

Le **modèle** contient tout ce qui concerne les **données** : leur structure, leurs règles de validité et leur stockage. Il n'a **aucune idée** de HTML ni d'URL.

On le sépare en deux classes simples :

* **`Task`** : représente **une** tâche (titre, état, date) et ses règles ;
* **`TaskRepository`** : sait **lire et écrire** les tâches (ajouter, chercher, modifier, supprimer).

Pour l'instant, on stocke les tâches dans un **fichier JSON**. À l'étape 9, on passera à MySQL **sans toucher au contrôleur ni aux vues** : c'est tout l'intérêt de la séparation.

### Fichiers à créer

* `src/Model/Task.php`
* `src/Model/TaskRepository.php`
* `storage/tasks.json`
* `src/Controller/TaskController.php` (modifié)

`src/Model/Task.php` :

```php
<?php

declare(strict_types=1);

namespace App\Model;

final class Task
{
    public function __construct(
        public readonly int $id,
        public string $title,
        public bool $done = false,
        public readonly string $createdAt = '',
    ) {
    }

    public function toggle(): void
    {
        $this->done = !$this->done;
    }

    /** Retourne un message d'erreur, ou null si le titre est valide. */
    public static function validateTitle(string $title): ?string
    {
        $title = trim($title);

        if ($title === '') {
            return 'Le titre est obligatoire.';
        }

        if (mb_strlen($title) > 150) {
            return 'Le titre ne doit pas dépasser 150 caractères.';
        }

        return null;
    }
}
```

`storage/tasks.json` :

```json
[
    {"id": 1, "title": "Apprendre le MVC", "done": false, "created_at": "2026-09-30 09:00:00"},
    {"id": 2, "title": "Installer Composer", "done": true, "created_at": "2026-09-29 18:30:00"}
]
```

`src/Model/TaskRepository.php` :

```php
<?php

declare(strict_types=1);

namespace App\Model;

final class TaskRepository
{
    private string $file;

    public function __construct(?string $file = null)
    {
        $this->file = $file ?? dirname(__DIR__, 2) . '/storage/tasks.json';
    }

    /** @return Task[] */
    public function all(): array
    {
        return array_map(fn (array $row): Task => $this->hydrate($row), $this->read());
    }

    public function find(int $id): ?Task
    {
        foreach ($this->read() as $row) {
            if ($row['id'] === $id) {
                return $this->hydrate($row);
            }
        }

        return null;
    }

    public function create(string $title): Task
    {
        $rows = $this->read();
        $id = $rows === [] ? 1 : max(array_column($rows, 'id')) + 1;

        $row = [
            'id' => $id,
            'title' => trim($title),
            'done' => false,
            'created_at' => date('Y-m-d H:i:s'),
        ];

        $rows[] = $row;
        $this->write($rows);

        return $this->hydrate($row);
    }

    public function update(Task $task): void
    {
        $rows = $this->read();

        foreach ($rows as $index => $row) {
            if ($row['id'] === $task->id) {
                $rows[$index]['title'] = trim($task->title);
                $rows[$index]['done'] = $task->done;
            }
        }

        $this->write($rows);
    }

    public function delete(int $id): void
    {
        $rows = array_filter($this->read(), static fn (array $row): bool => $row['id'] !== $id);

        $this->write(array_values($rows));
    }

    private function read(): array
    {
        if (!is_file($this->file)) {
            return [];
        }

        return json_decode((string) file_get_contents($this->file), true) ?? [];
    }

    private function write(array $rows): void
    {
        file_put_contents(
            $this->file,
            json_encode($rows, JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE),
            LOCK_EX,
        );
    }

    /** Transforme une ligne de données brutes en objet Task. */
    private function hydrate(array $row): Task
    {
        return new Task(
            (int) $row['id'],
            (string) $row['title'],
            (bool) $row['done'],
            (string) ($row['created_at'] ?? ''),
        );
    }
}
```

`src/Controller/TaskController.php` (version temporaire) :

```php
<?php

declare(strict_types=1);

namespace App\Controller;

use App\Core\Request;
use App\Model\TaskRepository;

final class TaskController
{
    private TaskRepository $tasks;

    public function __construct()
    {
        $this->tasks = new TaskRepository();
    }

    public function index(Request $request): void
    {
        echo '<h1>Liste des tâches</h1><ul>';

        foreach ($this->tasks->all() as $task) {
            $mark = $task->done ? '✔' : '○';
            echo '<li>' . $mark . ' ' . htmlspecialchars($task->title) . '</li>';
        }

        echo '</ul>';
    }
}
```

### Explication

* `Task` a des **propriétés publiques** (simples à lire dans les vues) mais ses règles sont dans la classe : `toggle()` change l'état, `validateTitle()` valide un titre. C'est la logique métier.
* `TaskRepository` est le seul endroit qui sait que les données sont dans un JSON. Les méthodes publiques (`all`, `find`, `create`, `update`, `delete`) forment son « contrat ».
* `hydrate()` transforme une ligne brute (tableau) en objet `Task` : on manipule des **objets**, pas des tableaux anonymes.
* Le contrôleur utilise le dépôt, mais n'a **aucune idée** du format de stockage.
* Ce contrôleur génère du HTML avec `echo` : ça marche, mais c'est exactement le mélange qu'on voulait éviter. L'étape suivante règle ça.

### Test

Visite `http://localhost:8000/tasks` : les deux tâches apparaissent. Modifie `storage/tasks.json` à la main (ajoute une tâche, change un titre), recharge : la page suit.

> **Dans Laravel :** le modèle s'appelle `App\Models\Task` (Eloquent). Il combine les deux rôles : `Task::all()` et `Task::find($id)` jouent le rôle de notre `TaskRepository`. Ici, on les sépare pour que chaque classe ait une seule responsabilité.

### Petit exercice

Ajoute à `TaskRepository` une méthode `countDone(): int` qui retourne le nombre de tâches terminées, et affiche-le dans le contrôleur sous le titre (`Terminées : X`).

### Correction

Dans `TaskRepository` :

```php
    public function countDone(): int
    {
        return count(array_filter($this->all(), static fn (Task $task): bool => $task->done));
    }
```

Dans `TaskController::index()`, après le premier `echo` :

```php
        echo '<p>Terminées : ' . $this->tasks->countDone() . '</p>';
```

---

## Étape 6 : Les vues

### Concept

Une **vue** est un fichier de **HTML avec un peu de PHP** (`echo`, `if`, `foreach`). Elle reçoit des **données** du contrôleur et les affiche. Elle ne fait ni SQL ni calcul métier.

On ajoute trois éléments :

1. une classe **`View`** qui charge un fichier de vue, lui donne des variables et renvoie le HTML ;
2. un **layout** (gabarit commun : `<head>`, menu...) pour ne pas répéter le HTML de base ;
3. une classe de base **`Controller`** avec une méthode `view()` pour tous les contrôleurs.

### Fichiers à créer

* `src/helpers.php` (la fonction d'échappement `e()`)
* `composer.json` (modifié : charger `helpers.php`)
* `src/Core/View.php`
* `src/Core/Controller.php`
* `views/layout.php`, `views/home.php`, `views/tasks/index.php`
* `public/style.css`
* les deux contrôleurs (modifiés)

`src/helpers.php` :

```php
<?php

declare(strict_types=1);

/** Échappe une valeur pour l'afficher en HTML sans risque (protection XSS). */
function e(mixed $value): string
{
    return htmlspecialchars((string) $value, ENT_QUOTES, 'UTF-8');
}
```

Dans `composer.json`, complète la section `autoload` (la clé `files` charge un fichier à chaque exécution) :

```json
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        },
        "files": ["src/helpers.php"]
    }
```

```bash
composer dump-autoload
```

`src/Core/View.php` :

```php
<?php

declare(strict_types=1);

namespace App\Core;

use RuntimeException;

final class View
{
    public function __construct(private string $viewsPath)
    {
    }

    /** Affiche la vue dans le layout et retourne le HTML complet. */
    public function render(string $template, array $data = []): string
    {
        $content = $this->capture($template, $data);

        return $this->capture('layout', $data + ['content' => $content]);
    }

    private function capture(string $template, array $data): string
    {
        $file = $this->viewsPath . '/' . $template . '.php';

        if (!is_file($file)) {
            throw new RuntimeException("Vue introuvable : $template");
        }

        extract($data, EXTR_SKIP);   // crée une variable par clé : ['tasks' => …] devient $tasks

        ob_start();
        require $file;

        return (string) ob_get_clean();
    }
}
```

`src/Core/Controller.php` :

```php
<?php

declare(strict_types=1);

namespace App\Core;

abstract class Controller
{
    protected function view(string $template, array $data = [], int $status = 200): void
    {
        http_response_code($status);

        $view = new View(dirname(__DIR__, 2) . '/views');
        echo $view->render($template, $data);
    }

    protected function redirect(string $path): never
    {
        header('Location: ' . $path);
        exit;
    }
}
```

`views/layout.php` :

```php
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title><?= e($pageTitle ?? 'Tâches') ?> — Tâches MVC</title>
    <link rel="stylesheet" href="/style.css">
</head>
<body>
<header>
    <nav>
        <a href="/">Accueil</a>
        <a href="/tasks">Tâches</a>
    </nav>
</header>
<main>
    <?= $content ?>
</main>
</body>
</html>
```

`views/home.php` :

```php
<h1>Bienvenue</h1>
<p>Cette application de gestion de tâches est construite avec un mini-framework MVC écrit à la main.</p>
<p><a class="button" href="/tasks">Voir mes tâches</a></p>
```

`views/tasks/index.php` :

```php
<h1>Mes tâches</h1>

<?php if ($tasks === []): ?>
    <p>Aucune tâche pour l'instant.</p>
<?php else: ?>
    <ul class="tasks">
        <?php foreach ($tasks as $task): ?>
            <li class="<?= $task->done ? 'done' : '' ?>">
                <span><?= e($task->title) ?></span>
            </li>
        <?php endforeach; ?>
    </ul>
<?php endif; ?>
```

`public/style.css` :

```css
* { box-sizing: border-box; }
body { font-family: system-ui, sans-serif; margin: 0; background: #f4f5f7; color: #1f2937; }
header { background: #1f2937; padding: 12px 24px; }
header a { color: white; margin-right: 16px; text-decoration: none; }
main { max-width: 640px; margin: 32px auto; background: white; padding: 24px; border-radius: 8px; box-shadow: 0 1px 4px rgba(0, 0, 0, .1); }
ul.tasks { list-style: none; padding: 0; }
ul.tasks li { display: flex; align-items: center; gap: 8px; padding: 10px 0; border-bottom: 1px solid #eee; }
ul.tasks li > :first-child { flex: 1; }
li.done > :first-child { text-decoration: line-through; color: #9ca3af; }
form { margin: 0; }
input[type="text"] { width: 100%; padding: 8px; margin: 6px 0 12px; }
button, .button { background: #2563eb; color: white; border: 0; border-radius: 4px; padding: 6px 12px; cursor: pointer; text-decoration: none; font-size: 14px; display: inline-block; }
button.danger { background: #dc2626; }
.error { background: #fee2e2; color: #b91c1c; padding: 8px; border-radius: 4px; }
```

`src/Controller/HomeController.php` :

```php
<?php

declare(strict_types=1);

namespace App\Controller;

use App\Core\Controller;
use App\Core\Request;

final class HomeController extends Controller
{
    public function index(Request $request): void
    {
        $this->view('home', ['pageTitle' => 'Accueil']);
    }
}
```

*(Si tu as fait l'exercice de l'étape 4, tu peux aussi créer une vue `about.php` pour `about()`.)*

`src/Controller/TaskController.php` :

```php
<?php

declare(strict_types=1);

namespace App\Controller;

use App\Core\Controller;
use App\Core\Request;
use App\Model\TaskRepository;

final class TaskController extends Controller
{
    private TaskRepository $tasks;

    public function __construct()
    {
        $this->tasks = new TaskRepository();
    }

    public function index(Request $request): void
    {
        $this->view('tasks/index', [
            'pageTitle' => 'Mes tâches',
            'tasks' => $this->tasks->all(),
        ]);
    }
}
```

### Explication

* **`extract($data)`** transforme `['tasks' => $liste]` en variable `$tasks` disponible dans la vue. C'est ainsi que le contrôleur « donne » des données à la vue.
* **`ob_start()` / `ob_get_clean()`** capturent ce que la vue affiche, pour le récupérer sous forme de texte. On peut ainsi **insérer le contenu de la vue dans le layout** (`<?= $content ?>`).
* **`e()`** échappe tout ce qui vient de l'utilisateur : on écrit toujours `<?= e($task->title) ?>`. Sans cela, un titre contenant `<script>` s'exécuterait dans le navigateur (faille XSS).
* `$content` n'est **pas** échappé : c'est du HTML déjà généré par nos vues.
* `redirect()` est typée `never` : la fonction ne retourne jamais, car elle termine le script avec `exit`.
* Le contrôleur ne contient **plus aucun HTML** : il choisit la vue (`'tasks/index'`) et lui passe des données.

### Test

* `/` → page d'accueil avec le menu et le style ;
* `/tasks` → liste de tâches, la terminée est barrée.

Tente d'ajouter un titre `<b>gras</b>` dans `storage/tasks.json` : la vue affiche le texte littéralement (`<b>gras</b>`), sans l'interpréter grâce à `e()`.

> **Dans Laravel :** `return view('tasks.index', ['tasks' => $tasks]);` charge `resources/views/tasks/index.blade.php` dans un layout. Même principe, avec le moteur Blade (voir étape 12).

### Petit exercice

Dans `layout.php`, ajoute un pied de page avec l'année en cours (`date('Y')`). Dans `tasks/index.php`, affiche le nombre de tâches sous le titre.

### Correction

`views/layout.php`, après `</main>` :

```php
<footer>
    <p>© <?= date('Y') ?> — Tâches MVC</p>
</footer>
```

`views/tasks/index.php`, sous le `<h1>` :

```php
<p><?= count($tasks) ?> tâche(s)</p>
```

---

## Étape 7 : Les paramètres d'URL

### Concept

Il existe deux sortes de paramètres dans une URL :

| Type | Exemple | Usage | Récupération |
|---|---|---|---|
| **Segment de chemin** | `/tasks/3` | Identifier une ressource | Routeur (`{id}`) |
| **Query string** | `/tasks?filter=done` | Filtrer, trier, rechercher | `$request->query('filter')` |

Il faut que le routeur comprenne les routes à **paramètres** comme `/tasks/{id}` : il va les transformer en **expression régulière**, extraire la valeur et la passer à l'action.

### Fichiers à créer ou modifier

* `src/Core/Router.php` (réécrit)
* `src/Core/Controller.php` (ajout de `notFound()`)
* `views/errors/404.php`, `views/tasks/show.php`
* `views/tasks/index.php` (lien vers le détail, filtre)
* `src/Controller/TaskController.php` (ajout de `show()`, filtre dans `index()`)
* `config/routes.php` (nouvelle route)

`src/Core/Router.php` (remplace tout le fichier) :

```php
<?php

declare(strict_types=1);

namespace App\Core;

final class Router
{
    /** @var array<int, array{method: string, pattern: string, handler: callable|array}> */
    private array $routes = [];

    public function get(string $path, callable|array $handler): void
    {
        $this->add('GET', $path, $handler);
    }

    public function post(string $path, callable|array $handler): void
    {
        $this->add('POST', $path, $handler);
    }

    private function add(string $method, string $path, callable|array $handler): void
    {
        // "/tasks/{id}" devient "#^/tasks/(?P<id>\d+)$#"
        $pattern = preg_replace_callback(
            '#\{(\w+)\}#',
            static fn (array $m): string => '(?P<' . $m[1] . '>' . ($m[1] === 'id' ? '\d+' : '[^/]+') . ')',
            $path,
        );

        $this->routes[] = [
            'method' => $method,
            'pattern' => '#^' . $pattern . '$#',
            'handler' => $handler,
        ];
    }

    public function dispatch(Request $request): void
    {
        foreach ($this->routes as $route) {
            if ($route['method'] !== $request->method()) {
                continue;
            }

            if (preg_match($route['pattern'], $request->path(), $matches) !== 1) {
                continue;
            }

            // On ne garde que les groupes nommés ({id}, {slug}…), "123" devient l'entier 123
            $params = [];
            foreach ($matches as $key => $value) {
                if (is_string($key)) {
                    $params[$key] = ctype_digit($value) ? (int) $value : $value;
                }
            }

            $this->call($route['handler'], $request, $params);
            return;
        }

        http_response_code(404);
        echo '<h1>404</h1><p>Page introuvable.</p>';
    }

    private function call(callable|array $handler, Request $request, array $params): void
    {
        if (is_array($handler)) {
            [$class, $action] = $handler;
            (new $class())->$action($request, ...$params);
            return;
        }

        $handler($request, ...$params);
    }
}
```

Dans `src/Core/Controller.php`, ajoute :

```php
    protected function notFound(): void
    {
        $this->view('errors/404', ['pageTitle' => 'Page introuvable'], 404);
    }
```

`views/errors/404.php` :

```php
<h1>404</h1>
<p>Cette page n'existe pas.</p>
<p><a href="/tasks">← Retour aux tâches</a></p>
```

`views/tasks/show.php` :

```php
<h1><?= e($task->title) ?></h1>

<p>Statut : <?= $task->done ? 'terminée' : 'à faire' ?></p>
<p>Créée le <?= e($task->createdAt) ?></p>

<p><a href="/tasks">← Retour à la liste</a></p>
```

`config/routes.php` : ajoute

```php
    $router->get('/tasks/{id}', [TaskController::class, 'show']);
```

Dans `TaskController`, ajoute `use App\Model\Task;` en haut, remplace `index()` et ajoute `show()` :

```php
    public function index(Request $request): void
    {
        $tasks = $this->tasks->all();

        $filter = $request->query('filter');   // ex. /tasks?filter=done
        if ($filter === 'done') {
            $tasks = array_values(array_filter($tasks, static fn (Task $task): bool => $task->done));
        }

        $this->view('tasks/index', [
            'pageTitle' => 'Mes tâches',
            'tasks' => $tasks,
        ]);
    }

    public function show(Request $request, int $id): void
    {
        $task = $this->tasks->find($id);

        if ($task === null) {
            $this->notFound();
            return;
        }

        $this->view('tasks/show', [
            'pageTitle' => $task->title,
            'task' => $task,
        ]);
    }
```

Dans `views/tasks/index.php`, remplace la ligne du titre par un lien, et ajoute des liens de filtre sous le compteur :

```php
<p>
    <a href="/tasks">Toutes</a> |
    <a href="/tasks?filter=done">Terminées</a>
</p>
```

```php
                <a href="/tasks/<?= $task->id ?>"><?= e($task->title) ?></a>
```

*(à la place de `<span><?= e($task->title) ?></span>`)*

### Explication

* `preg_replace_callback` transforme `{id}` en un **groupe de capture nommé** : `(?P<id>\d+)`. Pour `{id}`, on n'accepte que des **chiffres** : `/tasks/abc` ne correspond pas et donne donc une 404 propre.
* `preg_match` teste l'URL ; `$matches['id']` contient la valeur. On la convertit en `int` pour la passer à `show(Request $request, int $id)`.
* `(new $class())->$action($request, ...$params)` : les **paramètres nommés** (`...['id' => 3]`) sont passés à l'action par leur nom. C'est ce qui permet d'écrire `show(Request $request, int $id)`.
* **Ordre des routes :** `{id}` n'accepte que des chiffres, donc `/tasks/create` (étape suivante) ne sera jamais confondu avec `/tasks/{id}`.
* `$request->query('filter')` lit `?filter=done`. Le contrôleur filtre les tâches avant de les donner à la vue.
* Quand la tâche n'existe pas, on affiche une **vraie 404** plutôt qu'une page cassée.

### Test

* `/tasks/1` → détail de la tâche 1 ;
* `/tasks/999` → page 404 (code HTTP 404) ;
* `/tasks/abc` → 404 ;
* `/tasks?filter=done` → uniquement les tâches terminées.

> **Dans Laravel :** `Route::get('/tasks/{id}', ...)` et `$request->query('filter')` fonctionnent exactement ainsi. Laravel peut même « injecter » directement le modèle (`show(Task $task)`), grâce à un mécanisme appelé *route model binding*.

### Petit exercice

Ajoute un filtre `?filter=todo` (tâches **à faire**) et le lien correspondant dans la vue.

### Correction

Dans `TaskController::index()` :

```php
        if ($filter === 'done') {
            $tasks = array_values(array_filter($tasks, static fn (Task $task): bool => $task->done));
        } elseif ($filter === 'todo') {
            $tasks = array_values(array_filter($tasks, static fn (Task $task): bool => !$task->done));
        }
```

Dans `views/tasks/index.php` :

```php
<p>
    <a href="/tasks">Toutes</a> |
    <a href="/tasks?filter=done">Terminées</a> |
    <a href="/tasks?filter=todo">À faire</a>
</p>
```

---

## Étape 8 : Les formulaires

### Concept

Un formulaire suit toujours le même cycle, appelé **Post/Redirect/Get** :

1. `GET /tasks/create` → le contrôleur **affiche** le formulaire ;
2. le navigateur envoie `POST /tasks` → le contrôleur **valide** les données ;
3. si c'est **invalide** : on réaffiche le formulaire avec l'erreur (code **422**) et la valeur saisie ;
4. si c'est **valide** : on enregistre puis on **redirige** vers la liste (ainsi, actualiser la page ne renvoie pas le formulaire une seconde fois).

Règle : toute action qui **modifie** des données se fait en **POST**, jamais en GET (un simple lien ou un robot ne doit pas pouvoir supprimer quelque chose).

### Fichiers à créer ou modifier

* `config/routes.php` (routes complètes)
* `views/tasks/form.php` (formulaire partagé création/modification)
* `views/tasks/index.php` (boutons terminer/supprimer)
* `src/Controller/TaskController.php` (version complète)

`config/routes.php` :

```php
<?php

declare(strict_types=1);

use App\Controller\HomeController;
use App\Controller\TaskController;
use App\Core\Router;

return static function (Router $router): void {
    $router->get('/', [HomeController::class, 'index']);

    $router->get('/tasks', [TaskController::class, 'index']);
    $router->get('/tasks/create', [TaskController::class, 'create']);
    $router->post('/tasks', [TaskController::class, 'store']);
    $router->get('/tasks/{id}', [TaskController::class, 'show']);
    $router->post('/tasks/{id}/toggle', [TaskController::class, 'toggle']);
    $router->post('/tasks/{id}/delete', [TaskController::class, 'delete']);
};
```

`views/tasks/form.php` :

```php
<h1><?= e($pageTitle) ?></h1>

<?php if ($error !== null): ?>
    <p class="error"><?= e($error) ?></p>
<?php endif; ?>

<form method="post" action="<?= e($action) ?>">
    <label for="title">Titre</label>
    <input type="text" id="title" name="title" value="<?= e($old) ?>" autofocus>

    <button type="submit">Enregistrer</button>
    <a href="/tasks">Annuler</a>
</form>
```

`views/tasks/index.php` :

```php
<h1>Mes tâches</h1>

<p><a class="button" href="/tasks/create">+ Nouvelle tâche</a></p>

<p><?= count($tasks) ?> tâche(s)</p>
<p>
    <a href="/tasks">Toutes</a> |
    <a href="/tasks?filter=done">Terminées</a> |
    <a href="/tasks?filter=todo">À faire</a>
</p>

<?php if ($tasks === []): ?>
    <p>Aucune tâche pour l'instant.</p>
<?php else: ?>
    <ul class="tasks">
        <?php foreach ($tasks as $task): ?>
            <li class="<?= $task->done ? 'done' : '' ?>">
                <a href="/tasks/<?= $task->id ?>"><?= e($task->title) ?></a>

                <form method="post" action="/tasks/<?= $task->id ?>/toggle">
                    <button type="submit"><?= $task->done ? 'Rouvrir' : 'Terminer' ?></button>
                </form>

                <form method="post" action="/tasks/<?= $task->id ?>/delete"
                      onsubmit="return confirm('Supprimer cette tâche ?');">
                    <button type="submit" class="danger">Supprimer</button>
                </form>
            </li>
        <?php endforeach; ?>
    </ul>
<?php endif; ?>
```

`src/Controller/TaskController.php` :

```php
<?php

declare(strict_types=1);

namespace App\Controller;

use App\Core\Controller;
use App\Core\Request;
use App\Model\Task;
use App\Model\TaskRepository;

final class TaskController extends Controller
{
    private TaskRepository $tasks;

    public function __construct()
    {
        $this->tasks = new TaskRepository();
    }

    public function index(Request $request): void
    {
        $tasks = $this->tasks->all();

        $filter = $request->query('filter');
        if ($filter === 'done') {
            $tasks = array_values(array_filter($tasks, static fn (Task $task): bool => $task->done));
        } elseif ($filter === 'todo') {
            $tasks = array_values(array_filter($tasks, static fn (Task $task): bool => !$task->done));
        }

        $this->view('tasks/index', [
            'pageTitle' => 'Mes tâches',
            'tasks' => $tasks,
        ]);
    }

    public function show(Request $request, int $id): void
    {
        $task = $this->tasks->find($id);

        if ($task === null) {
            $this->notFound();
            return;
        }

        $this->view('tasks/show', [
            'pageTitle' => $task->title,
            'task' => $task,
        ]);
    }

    public function create(Request $request): void
    {
        $this->renderForm('Nouvelle tâche', '/tasks');
    }

    public function store(Request $request): void
    {
        $title = trim((string) $request->input('title', ''));
        $error = Task::validateTitle($title);

        if ($error !== null) {
            $this->renderForm('Nouvelle tâche', '/tasks', $title, $error, 422);
            return;
        }

        $this->tasks->create($title);
        $this->redirect('/tasks');
    }

    public function toggle(Request $request, int $id): void
    {
        $task = $this->tasks->find($id);

        if ($task === null) {
            $this->notFound();
            return;
        }

        $task->toggle();
        $this->tasks->update($task);
        $this->redirect('/tasks');
    }

    public function delete(Request $request, int $id): void
    {
        $this->tasks->delete($id);
        $this->redirect('/tasks');
    }

    private function renderForm(
        string $pageTitle,
        string $action,
        string $old = '',
        ?string $error = null,
        int $status = 200,
    ): void {
        $this->view('tasks/form', [
            'pageTitle' => $pageTitle,
            'action' => $action,
            'old' => $old,
            'error' => $error,
        ], $status);
    }
}
```

### Explication

* **`create()`** affiche un formulaire vide ; **`store()`** traite l'envoi. Ce couple `create`/`store` est une convention très répandue (Laravel l'utilise aussi).
* La validation est **dans le modèle** (`Task::validateTitle()`) : le contrôleur se contente de la demander et de réagir.
* En cas d'erreur, on renvoie le **code 422** (« données non valides ») et on redonne à la vue le texte saisi (`$old`) pour que l'utilisateur ne retape rien.
* Après un succès, `redirect('/tasks')` : c'est le **Redirect** du cycle Post/Redirect/Get.
* `toggle()` charge la tâche, appelle `$task->toggle()` (règle métier dans le modèle), puis la sauvegarde.
* `renderForm()` évite de répéter le tableau de données : une petite méthode privée suffit.
* **Sécurité, ce que les vrais frameworks ajoutent :** un **jeton CSRF** dans chaque formulaire (il empêche un site malveillant de soumettre un formulaire à ta place). Laravel l'ajoute avec `@csrf`. Nous ne l'implémentons pas ici pour rester simples, mais tu dois le connaître.
* **Astuce des frameworks :** les navigateurs n'envoient que `GET` et `POST`. Pour simuler `PUT` ou `DELETE`, Laravel utilise un champ caché `_method`. Nous avons simplement choisi des URL explicites (`/tasks/3/delete`).

### Test

1. `/tasks` → clique sur **+ Nouvelle tâche**.
2. Envoie le formulaire **vide** → message d'erreur, code 422.
3. Saisis « Écrire mon premier contrôleur » → tu reviens à la liste, la tâche apparaît.
4. Clique sur **Terminer** : la tâche est barrée ; sur **Rouvrir** : elle redevient active.
5. Clique sur **Supprimer** → confirmation → la tâche disparaît.
6. Actualise la page après un ajout : **rien n'est renvoyé** deux fois, grâce à la redirection.

> **Dans Laravel :** `$request->validate(['title' => 'required|max:150'])` joue le rôle de notre validation, et `return redirect('/tasks');` celui de `redirect()`.

### Petit exercice

Ajoute la **modification** d'une tâche : `GET /tasks/{id}/edit` (formulaire prérempli) et `POST /tasks/{id}` (enregistrement), en réutilisant `tasks/form.php`. Ajoute un lien **Modifier** dans la liste.

### Correction

Routes (dans `config/routes.php`) :

```php
    $router->get('/tasks/{id}/edit', [TaskController::class, 'edit']);
    $router->post('/tasks/{id}', [TaskController::class, 'update']);
```

Actions (dans `TaskController`) :

```php
    public function edit(Request $request, int $id): void
    {
        $task = $this->tasks->find($id);

        if ($task === null) {
            $this->notFound();
            return;
        }

        $this->renderForm('Modifier la tâche', "/tasks/{$id}", $task->title);
    }

    public function update(Request $request, int $id): void
    {
        $task = $this->tasks->find($id);

        if ($task === null) {
            $this->notFound();
            return;
        }

        $title = trim((string) $request->input('title', ''));
        $error = Task::validateTitle($title);

        if ($error !== null) {
            $this->renderForm('Modifier la tâche', "/tasks/{$id}", $title, $error, 422);
            return;
        }

        $task->title = $title;
        $this->tasks->update($task);
        $this->redirect('/tasks');
    }
```

Lien dans `views/tasks/index.php`, juste avant le formulaire « terminer » :

```php
                <a class="button" href="/tasks/<?= $task->id ?>/edit">Modifier</a>
```

Le formulaire est **le même** pour la création et la modification : seuls le titre de la page, l'URL d'envoi et la valeur initiale changent.

---

## Étape 9 : Connexion à une base de données

### Concept

Le fichier JSON était pratique pour démarrer, mais une vraie application utilise une **base de données**. Grâce à la séparation MVC, **seul le modèle change** : `TaskRepository` parle maintenant à MySQL avec **PDO**, et les contrôleurs et les vues ne bougent presque pas.

C'est le vrai bénéfice du MVC : on remplace une couche sans casser les autres.

### Fichiers à créer ou modifier

* `database/schema.sql`
* `config/database.php`
* `src/Core/Database.php`
* `src/Model/TaskRepository.php` (réécrit avec PDO)
* `src/Controller/TaskController.php` (seul le constructeur change)

`database/schema.sql` :

```sql
CREATE DATABASE IF NOT EXISTS mvc_taches
    CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

USE mvc_taches;

CREATE TABLE IF NOT EXISTS tasks (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(150) NOT NULL,
    done TINYINT(1) NOT NULL DEFAULT 0,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO tasks (title, done) VALUES
    ('Apprendre le MVC', 0),
    ('Installer Composer', 1);
```

Exécute-le : avec **phpMyAdmin** (`http://localhost/phpmyadmin` → onglet **SQL** → colle le contenu → **Exécuter**) ou en terminal :

```bash
mysql -u root -p < database/schema.sql
```

`config/database.php` :

```php
<?php

declare(strict_types=1);

return [
    'host' => 'localhost',
    'name' => 'mvc_taches',
    'user' => 'root',
    'pass' => '',          // XAMPP/WAMP : mot de passe vide par défaut
    'charset' => 'utf8mb4',
];
```

`src/Core/Database.php` :

```php
<?php

declare(strict_types=1);

namespace App\Core;

use PDO;

final class Database
{
    public static function connect(array $config): PDO
    {
        $dsn = sprintf(
            'mysql:host=%s;dbname=%s;charset=%s',
            $config['host'],
            $config['name'],
            $config['charset'],
        );

        return new PDO($dsn, $config['user'], $config['pass'], [
            PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
            PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        ]);
    }
}
```

`src/Model/TaskRepository.php` (remplace tout le fichier) :

```php
<?php

declare(strict_types=1);

namespace App\Model;

use PDO;
use RuntimeException;

final class TaskRepository
{
    public function __construct(private PDO $pdo)
    {
    }

    /** @return Task[] */
    public function all(): array
    {
        $rows = $this->pdo->query('SELECT * FROM tasks ORDER BY id DESC')->fetchAll();

        return array_map($this->hydrate(...), $rows);
    }

    public function find(int $id): ?Task
    {
        $stmt = $this->pdo->prepare('SELECT * FROM tasks WHERE id = :id');
        $stmt->execute(['id' => $id]);
        $row = $stmt->fetch();

        return $row === false ? null : $this->hydrate($row);
    }

    public function create(string $title): Task
    {
        $stmt = $this->pdo->prepare('INSERT INTO tasks (title) VALUES (:title)');
        $stmt->execute(['title' => trim($title)]);

        return $this->find((int) $this->pdo->lastInsertId())
            ?? throw new RuntimeException('Tâche introuvable après insertion.');
    }

    public function update(Task $task): void
    {
        $stmt = $this->pdo->prepare('UPDATE tasks SET title = :title, done = :done WHERE id = :id');
        $stmt->execute([
            'title' => trim($task->title),
            'done' => (int) $task->done,
            'id' => $task->id,
        ]);
    }

    public function delete(int $id): void
    {
        $stmt = $this->pdo->prepare('DELETE FROM tasks WHERE id = :id');
        $stmt->execute(['id' => $id]);
    }

    /** Transforme une ligne SQL en objet Task. */
    private function hydrate(array $row): Task
    {
        return new Task(
            (int) $row['id'],
            (string) $row['title'],
            (bool) $row['done'],
            (string) $row['created_at'],
        );
    }
}
```

Dans `TaskController`, remplace le constructeur (et ajoute `use App\Core\Database;`) :

```php
    public function __construct()
    {
        $config = require dirname(__DIR__, 2) . '/config/database.php';
        $this->tasks = new TaskRepository(Database::connect($config));
    }
```

### Explication

* `Database::connect()` lit la configuration (`config/database.php`) et retourne un objet `PDO`. La configuration est à **un seul endroit**.
* `TaskRepository` reçoit le `PDO` par son **constructeur** et utilise **toujours des requêtes préparées** (`:id`, `:title`) : pas d'injection SQL.
* Les **méthodes publiques sont les mêmes** qu'avec le JSON (`all`, `find`, `create`, `update`, `delete`). C'est pour cela que les contrôleurs et les vues n'ont pas changé.
* `$this->hydrate(...)` est la syntaxe moderne de PHP 8.1 qui transforme une méthode en callable.
* `?? throw new ...` : en PHP 8, `throw` peut s'utiliser comme expression.
* **Point faible à retenir :** le contrôleur sait maintenant lire la configuration et construire la connexion. Il en sait trop. On règle ça à l'étape suivante.

### Test

Refais le scénario de l'étape 8 (ajouter, terminer, modifier, supprimer). Tout fonctionne comme avant, mais les données sont maintenant dans MySQL : vérifie-les dans phpMyAdmin (base `mvc_taches`, table `tasks`).

**Erreurs fréquentes :** `Access denied` (mauvais identifiants dans `config/database.php`), `Unknown database` (le script SQL n'a pas été exécuté), `could not find driver` (extension `pdo_mysql` désactivée : voir le cours PHP, chapitre 7).

> **Dans Laravel :** la connexion se configure dans `config/database.php` et le fichier `.env` (`DB_HOST`, `DB_DATABASE`...), et Eloquent construit les requêtes SQL à ta place, mais c'est toujours PDO en dessous.

### Petit exercice

Ajoute une recherche : `TaskRepository::search(string $term)` (avec `LIKE`) et un paramètre `?q=` dans `/tasks`, avec un petit formulaire de recherche dans la vue.

### Correction

Dans `TaskRepository` :

```php
    /** @return Task[] */
    public function search(string $term): array
    {
        $stmt = $this->pdo->prepare('SELECT * FROM tasks WHERE title LIKE :term ORDER BY id DESC');
        $stmt->execute(['term' => '%' . $term . '%']);

        return array_map($this->hydrate(...), $stmt->fetchAll());
    }
```

Dans `TaskController::index()`, remplace la première ligne :

```php
        $q = trim((string) $request->query('q', ''));
        $tasks = $q !== '' ? $this->tasks->search($q) : $this->tasks->all();
```

Et dans la vue `tasks/index.php`, sous le `<h1>` :

```php
<form method="get" action="/tasks">
    <input type="text" name="q" placeholder="Rechercher…" value="<?= e($_GET['q'] ?? '') ?>">
</form>
```

---

## Étape 10 : Utilisation de classes (injection de dépendances)

### Concept

À l'étape 9, `TaskController` **fabrique lui-même** ses dépendances (`Database::connect(...)`, `new TaskRepository(...)`). Problèmes :

* chaque contrôleur devra répéter ce code ;
* impossible de remplacer facilement une classe (pour des tests, par exemple) ;
* le contrôleur connaît des détails qui ne le concernent pas.

La solution, utilisée par tous les frameworks : l'**injection de dépendances**. Le contrôleur **déclare ce dont il a besoin** dans son constructeur, et c'est quelqu'un d'autre qui le lui fournit :

```php
public function __construct(private TaskRepository $tasks) {}
```

Ce « quelqu'un d'autre » s'appelle un **conteneur** (*container*). Le nôtre va lire les types des paramètres du constructeur avec la **réflexion** de PHP et construire automatiquement les objets nécessaires. C'est l'*autowiring*, le même principe que le « Service Container » de Laravel.

### Fichiers à créer ou modifier

* `src/Core/Container.php` (nouveau)
* `src/Core/Router.php` (le routeur utilise le conteneur)
* `public/index.php` (configure le conteneur)
* `src/Controller/TaskController.php` (constructeur simplifié, version complète ci-dessous)

`src/Core/Container.php` :

```php
<?php

declare(strict_types=1);

namespace App\Core;

use ReflectionClass;
use ReflectionNamedType;
use RuntimeException;

final class Container
{
    /** @var array<string, object> */
    private array $instances = [];

    /**
     * @param array<string, callable> $bindings Recettes manuelles : classe => fonction qui la fabrique
     */
    public function __construct(private array $bindings = [])
    {
    }

    public function get(string $class): object
    {
        // Déjà fabriqué ? On réutilise le même objet.
        if (isset($this->instances[$class])) {
            return $this->instances[$class];
        }

        // Une recette manuelle existe (ex. PDO) ?
        if (isset($this->bindings[$class])) {
            return $this->instances[$class] = ($this->bindings[$class])($this);
        }

        // Sinon : on lit le constructeur et on fabrique les dépendances une par une.
        $reflection = new ReflectionClass($class);
        $constructor = $reflection->getConstructor();

        if ($constructor === null) {
            return $this->instances[$class] = new $class();
        }

        $arguments = [];

        foreach ($constructor->getParameters() as $parameter) {
            $type = $parameter->getType();

            if ($type instanceof ReflectionNamedType && !$type->isBuiltin()) {
                $arguments[] = $this->get($type->getName());     // une classe : on la fabrique
            } elseif ($parameter->isDefaultValueAvailable()) {
                $arguments[] = $parameter->getDefaultValue();
            } else {
                throw new RuntimeException(
                    "Impossible de fournir le paramètre \${$parameter->getName()} de $class."
                );
            }
        }

        return $this->instances[$class] = $reflection->newInstanceArgs($arguments);
    }
}
```

Dans `src/Core/Router.php`, ajoute un constructeur au début de la classe :

```php
    public function __construct(private Container $container)
    {
    }
```

et dans `call()`, remplace `(new $class())->$action(...)` par :

```php
            $this->container->get($class)->$action($request, ...$params);
```

`public/index.php` (version finale) :

```php
<?php

declare(strict_types=1);

use App\Core\Container;
use App\Core\Database;
use App\Core\Request;
use App\Core\Router;

require __DIR__ . '/../vendor/autoload.php';

// Serveur intégré de PHP : laisser passer les fichiers statiques (CSS, images…)
if (PHP_SAPI === 'cli-server') {
    $file = __DIR__ . (string) parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);

    if (is_file($file)) {
        return false;
    }
}

// Le conteneur sait fabriquer seul la plupart des classes.
// On lui explique uniquement comment obtenir un PDO, qui a besoin de la configuration.
$container = new Container([
    PDO::class => static fn (): PDO => Database::connect(require __DIR__ . '/../config/database.php'),
]);

$router = new Router($container);
(require __DIR__ . '/../config/routes.php')($router);

$router->dispatch(Request::fromGlobals());
```

`src/Controller/TaskController.php` (version complète, avec la modification de l'exercice 8 et la recherche de l'exercice 9) :

```php
<?php

declare(strict_types=1);

namespace App\Controller;

use App\Core\Controller;
use App\Core\Request;
use App\Model\Task;
use App\Model\TaskRepository;

final class TaskController extends Controller
{
    public function __construct(private TaskRepository $tasks)
    {
    }

    public function index(Request $request): void
    {
        $q = trim((string) $request->query('q', ''));
        $tasks = $q !== '' ? $this->tasks->search($q) : $this->tasks->all();

        $filter = $request->query('filter');
        if ($filter === 'done') {
            $tasks = array_values(array_filter($tasks, static fn (Task $task): bool => $task->done));
        } elseif ($filter === 'todo') {
            $tasks = array_values(array_filter($tasks, static fn (Task $task): bool => !$task->done));
        }

        $this->view('tasks/index', [
            'pageTitle' => 'Mes tâches',
            'tasks' => $tasks,
        ]);
    }

    public function show(Request $request, int $id): void
    {
        $task = $this->tasks->find($id);

        if ($task === null) {
            $this->notFound();
            return;
        }

        $this->view('tasks/show', [
            'pageTitle' => $task->title,
            'task' => $task,
        ]);
    }

    public function create(Request $request): void
    {
        $this->renderForm('Nouvelle tâche', '/tasks');
    }

    public function store(Request $request): void
    {
        $title = trim((string) $request->input('title', ''));
        $error = Task::validateTitle($title);

        if ($error !== null) {
            $this->renderForm('Nouvelle tâche', '/tasks', $title, $error, 422);
            return;
        }

        $this->tasks->create($title);
        $this->redirect('/tasks');
    }

    public function edit(Request $request, int $id): void
    {
        $task = $this->tasks->find($id);

        if ($task === null) {
            $this->notFound();
            return;
        }

        $this->renderForm('Modifier la tâche', "/tasks/{$id}", $task->title);
    }

    public function update(Request $request, int $id): void
    {
        $task = $this->tasks->find($id);

        if ($task === null) {
            $this->notFound();
            return;
        }

        $title = trim((string) $request->input('title', ''));
        $error = Task::validateTitle($title);

        if ($error !== null) {
            $this->renderForm('Modifier la tâche', "/tasks/{$id}", $title, $error, 422);
            return;
        }

        $task->title = $title;
        $this->tasks->update($task);
        $this->redirect('/tasks');
    }

    public function toggle(Request $request, int $id): void
    {
        $task = $this->tasks->find($id);

        if ($task === null) {
            $this->notFound();
            return;
        }

        $task->toggle();
        $this->tasks->update($task);
        $this->redirect('/tasks');
    }

    public function delete(Request $request, int $id): void
    {
        $this->tasks->delete($id);
        $this->redirect('/tasks');
    }

    private function renderForm(
        string $pageTitle,
        string $action,
        string $old = '',
        ?string $error = null,
        int $status = 200,
    ): void {
        $this->view('tasks/form', [
            'pageTitle' => $pageTitle,
            'action' => $action,
            'old' => $old,
            'error' => $error,
        ], $status);
    }
}
```

### Explication

Quand le routeur doit appeler `TaskController`, voici ce que fait `$container->get(TaskController::class)` :

1. il lit le constructeur : `__construct(TaskRepository $tasks)` ;
2. `TaskRepository` est une classe : il appelle `get(TaskRepository::class)` ;
3. il lit **son** constructeur : `__construct(PDO $pdo)` ;
4. pour `PDO`, il existe une **recette manuelle** dans `index.php` : il l'applique ;
5. il assemble alors `PDO` → `TaskRepository` → `TaskController`.

Conséquences :

* le contrôleur **ne connaît plus** la configuration ni la façon de créer la connexion ;
* un nouveau contrôleur qui demande `TaskRepository` dans son constructeur le reçoit **automatiquement**, sans aucune ligne de câblage ;
* chaque objet n'est fabriqué **qu'une fois** (une seule connexion PDO par requête) ;
* pour tester, on pourrait injecter un faux dépôt.

**Les classes de notre framework et leur responsabilité unique :**

| Classe | Responsabilité |
|---|---|
| `Request` | Représenter la requête entrante |
| `Router` | Trouver l'action correspondant à la requête |
| `Container` | Fabriquer les objets avec leurs dépendances |
| `Controller` (base) | Outils communs aux contrôleurs : `view()`, `redirect()`, `notFound()` |
| `View` | Rendre une vue dans un layout |
| `Database` | Créer la connexion PDO |
| `Task` / `TaskRepository` | Les données (règles métier) / leur stockage |

### Test

Refais tout le scénario : liste, filtre, recherche, création, modification, terminer, supprimer, `/tasks/999`. Le comportement est **strictement identique** : on a amélioré la structure sans changer ce que voit l'utilisateur. C'est ça, une refactorisation réussie.

> **Dans Laravel :** le *Service Container* fait exactement cela : tu écris `public function __construct(private TaskRepository $tasks)` et Laravel injecte l'objet. Tout le framework repose sur ce mécanisme.

### Petit exercice

Crée un `StatsController` dont l'action `index` affiche le nombre total de tâches et le nombre de tâches terminées, sur l'URL `/stats`. Il doit recevoir `TaskRepository` par **injection**.

### Correction

`src/Controller/StatsController.php` :

```php
<?php

declare(strict_types=1);

namespace App\Controller;

use App\Core\Controller;
use App\Core\Request;
use App\Model\Task;
use App\Model\TaskRepository;

final class StatsController extends Controller
{
    public function __construct(private TaskRepository $tasks)
    {
    }

    public function index(Request $request): void
    {
        $all = $this->tasks->all();
        $done = count(array_filter($all, static fn (Task $task): bool => $task->done));

        $this->view('stats', [
            'pageTitle' => 'Statistiques',
            'total' => count($all),
            'done' => $done,
        ]);
    }
}
```

`views/stats.php` :

```php
<h1>Statistiques</h1>
<p>Total : <strong><?= $total ?></strong> tâche(s)</p>
<p>Terminées : <strong><?= $done ?></strong></p>
<p><a href="/tasks">← Retour aux tâches</a></p>
```

Route :

```php
    $router->get('/stats', [StatsController::class, 'index']);
```

N'oublie pas `use App\Controller\StatsController;` dans `routes.php`. Aucune ligne n'a été ajoutée pour construire `StatsController` : le conteneur s'en charge.

---

## Étape 11 : Organisation des fichiers

### Concept

Une bonne organisation répond à une question : « **où dois-je mettre ce code ?** ». Voici l'arborescence finale de notre application :

```
taches-mvc/
├── composer.json
├── composer.lock
├── .gitignore
├── config/
│   ├── database.php          # paramètres de la base de données
│   └── routes.php            # toutes les URL de l'application
├── database/
│   └── schema.sql            # structure de la base
├── public/                   # SEUL dossier accessible depuis le web
│   ├── .htaccess             # redirige tout vers index.php (Apache)
│   ├── index.php             # point d'entrée unique
│   └── style.css
├── src/
│   ├── Controller/
│   │   ├── HomeController.php
│   │   ├── StatsController.php
│   │   └── TaskController.php
│   ├── Core/                 # notre mini-framework
│   │   ├── Container.php
│   │   ├── Controller.php
│   │   ├── Database.php
│   │   ├── Request.php
│   │   ├── Router.php
│   │   └── View.php
│   ├── Model/
│   │   ├── Task.php
│   │   └── TaskRepository.php
│   └── helpers.php
├── storage/                  # fichiers générés (logs, cache…)
├── vendor/                   # bibliothèques Composer
└── views/
    ├── layout.php
    ├── home.php
    ├── stats.php
    ├── errors/404.php
    └── tasks/
        ├── form.php
        ├── index.php
        └── show.php
```

**Où mettre quoi ?**

| Tu veux ajouter… | Tu le mets dans… |
|---|---|
| Une nouvelle URL | `config/routes.php` |
| La logique d'une page (recevoir, décider) | `src/Controller/` |
| Une règle métier, un accès à la base | `src/Model/` |
| Du HTML | `views/` |
| Un mot de passe, une option de config | `config/` |
| Une image, un CSS, un JS | `public/` |
| Du code technique partagé (routeur, vue…) | `src/Core/` |
| Un script SQL | `database/` |

**Conventions à respecter :**

* **un namespace = un dossier**, une classe par fichier, le fichier porte le nom de la classe ;
* un contrôleur s'appelle `XxxController` et ses méthodes sont des **verbes** (`index`, `show`, `create`, `store`, `edit`, `update`, `delete`) ;
* les vues sont rangées **par ressource** (`views/tasks/…`) ;
* **aucune logique** dans les vues, **aucun HTML** dans les contrôleurs, **aucun SQL** hors des modèles.

### Fichiers à créer

`.gitignore` :

```
/vendor/
/storage/*.json
.env
```

*(Un mot de passe de base de données ne doit jamais être envoyé dans un dépôt public. Les vrais projets utilisent un fichier `.env` ignoré par Git, que l'on lit avec une bibliothèque comme `vlucas/phpdotenv`.)*

`public/.htaccess` (pour **Apache**, sans le serveur intégré) :

```apache
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^ index.php [QSA,L]
```

Il dit à Apache : « si le fichier demandé n'existe pas, envoie la requête à `index.php` ». C'est lui qui joue le rôle du bloc `cli-server` de `index.php`.

**Utiliser XAMPP/WAMP avec des URL propres :** l'application suppose qu'elle est servie **à la racine** du site (`/tasks`, pas `/taches-mvc/public/tasks`). Le plus propre est de créer un **hôte virtuel** dont la racine est le dossier `public/` (exemple pour XAMPP, dans `httpd-vhosts.conf`) :

```apache
<VirtualHost *:80>
    ServerName taches.local
    DocumentRoot "C:/xampp/htdocs/taches-mvc/public"

    <Directory "C:/xampp/htdocs/taches-mvc/public">
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

Ajoute ensuite `127.0.0.1 taches.local` au fichier `hosts` de ton système, redémarre Apache et ouvre `http://taches.local`. Sinon, reste simplement sur `php -S`.

### Explication

* **Seul `public/` est exposé** : un visiteur ne peut pas lire `config/database.php` ni `src/`, même en devinant l'URL.
* `src/Core/` est la frontière entre **ton application** (`Controller`, `Model`, `views`) et le **mécanisme** qui la fait tourner. Dans un vrai projet, ce `Core` est remplacé par un framework installé dans `vendor/`.
* Cette organisation est très proche de celle de Laravel : ce que tu viens d'apprendre s'y applique directement.

### Test

Ajoute une nouvelle page `/contact` **sans réfléchir** à l'emplacement des fichiers : route dans `config/routes.php`, méthode dans `HomeController`, vue `views/contact.php`. Si tu sais où mettre chaque chose, l'organisation est comprise.

### Petit exercice

Dans quel dossier ou fichier placerais-tu :

1. une classe `Validator` réutilisée par plusieurs modèles ;
2. le mot de passe MySQL ;
3. le fichier `logo.png` ;
4. la règle « un titre ne dépasse pas 150 caractères » ;
5. la requête `SELECT * FROM tasks` ;
6. l'URL `/tasks/export` ?

### Correction

1. `src/Core/` si c'est un outil technique du framework, ou `src/Model/` si elle porte sur des règles métier.
2. `config/database.php` (ou un fichier `.env` non versionné).
3. `public/`.
4. Dans le **modèle** : `Task::validateTitle()`.
5. Dans le **dépôt** : `TaskRepository`.
6. `config/routes.php` pour l'URL, et une méthode `export()` dans `TaskController`.

---

## Étape 12 : Introduction à Twig

### Concept

Nos vues en PHP fonctionnent, mais elles ont des défauts :

* il faut **penser à écrire `e()`** partout : un seul oubli = une faille XSS ;
* la syntaxe (`<?php foreach (...): ?>`) est lourde ;
* rien n'empêche un développeur de mettre de la **logique métier** ou du SQL dans une vue.

Un **moteur de templates** règle ces trois problèmes. **Twig** (utilisé par Symfony, et très proche de **Blade** de Laravel) offre :

* l'**échappement automatique** : `{{ variable }}` est toujours protégé ;
* une syntaxe courte : `{% for %}`, `{% if %}` ;
* l'**héritage de templates** : un layout avec des blocs que les pages remplissent ;
* des **filtres** (`|upper`, `|date`, `|length`) ;
* une vue **limitée** volontairement : on ne peut pas y exécuter n'importe quel PHP.

Bonne nouvelle : **seule la classe `View` et les fichiers de vue changent**. Les contrôleurs, le routeur et les modèles ne bougent pas.

### Fichiers à créer ou modifier

```bash
composer require twig/twig
```

* `src/Core/View.php` (réécrit avec Twig)
* les vues : `.php` → `.html.twig`

`src/Core/View.php` :

```php
<?php

declare(strict_types=1);

namespace App\Core;

use Twig\Environment;
use Twig\Loader\FilesystemLoader;

final class View
{
    private Environment $twig;

    public function __construct(string $viewsPath)
    {
        $this->twig = new Environment(new FilesystemLoader($viewsPath), [
            'cache' => false,          // en production : un dossier de cache (ex. storage/cache)
            'autoescape' => 'html',    // échappement automatique
        ]);
    }

    public function render(string $template, array $data = []): string
    {
        return $this->twig->render($template . '.html.twig', $data);
    }
}
```

`views/layout.html.twig` :

```twig
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>{{ pageTitle|default('Tâches') }} — Tâches MVC</title>
    <link rel="stylesheet" href="/style.css">
</head>
<body>
<header>
    <nav>
        <a href="/">Accueil</a>
        <a href="/tasks">Tâches</a>
    </nav>
</header>
<main>
    {% block content %}{% endblock %}
</main>
</body>
</html>
```

`views/home.html.twig` :

```twig
{% extends 'layout.html.twig' %}

{% block content %}
    <h1>Bienvenue</h1>
    <p>Cette application de gestion de tâches est construite avec un mini-framework MVC écrit à la main.</p>
    <p><a class="button" href="/tasks">Voir mes tâches</a></p>
{% endblock %}
```

`views/errors/404.html.twig` :

```twig
{% extends 'layout.html.twig' %}

{% block content %}
    <h1>404</h1>
    <p>Cette page n'existe pas.</p>
    <p><a href="/tasks">← Retour aux tâches</a></p>
{% endblock %}
```

`views/tasks/index.html.twig` :

```twig
{% extends 'layout.html.twig' %}

{% block content %}
    <h1>Mes tâches</h1>

    <form method="get" action="/tasks">
        <input type="text" name="q" placeholder="Rechercher…">
    </form>

    <p><a class="button" href="/tasks/create">+ Nouvelle tâche</a></p>

    <p>{{ tasks|length }} tâche(s)</p>
    <p>
        <a href="/tasks">Toutes</a> |
        <a href="/tasks?filter=done">Terminées</a> |
        <a href="/tasks?filter=todo">À faire</a>
    </p>

    {% if tasks is empty %}
        <p>Aucune tâche pour l'instant.</p>
    {% else %}
        <ul class="tasks">
            {% for task in tasks %}
                <li class="{{ task.done ? 'done' : '' }}">
                    <a href="/tasks/{{ task.id }}">{{ task.title }}</a>

                    <a class="button" href="/tasks/{{ task.id }}/edit">Modifier</a>

                    <form method="post" action="/tasks/{{ task.id }}/toggle">
                        <button type="submit">{{ task.done ? 'Rouvrir' : 'Terminer' }}</button>
                    </form>

                    <form method="post" action="/tasks/{{ task.id }}/delete"
                          onsubmit="return confirm('Supprimer cette tâche ?');">
                        <button type="submit" class="danger">Supprimer</button>
                    </form>
                </li>
            {% endfor %}
        </ul>
    {% endif %}
{% endblock %}
```

`views/tasks/show.html.twig` :

```twig
{% extends 'layout.html.twig' %}

{% block content %}
    <h1>{{ task.title }}</h1>

    <p>Statut : {{ task.done ? 'terminée' : 'à faire' }}</p>
    <p>Créée le {{ task.createdAt|date('d/m/Y à H:i') }}</p>

    <p><a href="/tasks">← Retour à la liste</a></p>
{% endblock %}
```

`views/tasks/form.html.twig` :

```twig
{% extends 'layout.html.twig' %}

{% block content %}
    <h1>{{ pageTitle }}</h1>

    {% if error %}
        <p class="error">{{ error }}</p>
    {% endif %}

    <form method="post" action="{{ action }}">
        <label for="title">Titre</label>
        <input type="text" id="title" name="title" value="{{ old }}" autofocus>

        <button type="submit">Enregistrer</button>
        <a href="/tasks">Annuler</a>
    </form>
{% endblock %}
```

`views/stats.html.twig` (si tu as fait l'exercice de l'étape 10) :

```twig
{% extends 'layout.html.twig' %}

{% block content %}
    <h1>Statistiques</h1>
    <p>Total : <strong>{{ total }}</strong> tâche(s)</p>
    <p>Terminées : <strong>{{ done }}</strong></p>
    <p><a href="/tasks">← Retour aux tâches</a></p>
{% endblock %}
```

Tu peux maintenant **supprimer les anciennes vues `.php`**, ainsi que `src/helpers.php` et la ligne `"files": [...]` de `composer.json` (Twig fait l'échappement), puis lancer `composer dump-autoload`.

### Explication

* **`{{ ... }}`** affiche une valeur, **échappée automatiquement** : `{{ task.title }}` est sûr, sans `e()`. Pour afficher du HTML brut, il faudrait écrire volontairement `{{ valeur|raw }}`.
* **`{% ... %}`** contient de la logique : `{% if %}`, `{% for %}`, `{% extends %}`, `{% block %}`.
* **`{% extends 'layout.html.twig' %}`** + **`{% block content %}`** remplacent notre système `$content` : le layout définit des *blocs* que chaque page remplit.
* `task.title` : Twig lit aussi bien une propriété publique qu'une méthode ou une clé de tableau. Une seule syntaxe pour tout.
* `|length`, `|date('d/m/Y à H:i')`, `|default('…')` sont des **filtres** qu'on enchaîne avec `|`.
* `Controller::view()` est **inchangée** : elle appelle toujours `new View(...)` puis `render('tasks/index', $data)`. C'est le modèle MVC qui paie : on a changé la technique de vue sans toucher aux contrôleurs.
* `'cache' => false` : Twig *compile* chaque template en PHP. En production, on active un dossier de cache pour éviter de recompiler.

### Test

Refais le scénario complet. L'application est **identique**. Essaie ensuite d'ajouter une tâche dont le titre est `<script>alert('x')</script>` : Twig l'affiche comme **texte**, sans l'exécuter.

> **Dans Laravel :** le moteur s'appelle **Blade**, avec une syntaxe proche : `{{ $task->title }}` (échappé), `@foreach`, `@if`, `@extends('layout')`, `@section('content')`. Si tu comprends Twig, Blade te semblera familier.

### Petit exercice

Extrais le `<li>` d'une tâche dans un fichier `views/tasks/_item.html.twig` et appelle-le depuis `index.html.twig` avec `{% include %}`.

### Correction

`views/tasks/_item.html.twig` :

```twig
<li class="{{ task.done ? 'done' : '' }}">
    <a href="/tasks/{{ task.id }}">{{ task.title }}</a>

    <a class="button" href="/tasks/{{ task.id }}/edit">Modifier</a>

    <form method="post" action="/tasks/{{ task.id }}/toggle">
        <button type="submit">{{ task.done ? 'Rouvrir' : 'Terminer' }}</button>
    </form>

    <form method="post" action="/tasks/{{ task.id }}/delete"
          onsubmit="return confirm('Supprimer cette tâche ?');">
        <button type="submit" class="danger">Supprimer</button>
    </form>
</li>
```

Dans `index.html.twig`, la boucle devient :

```twig
            {% for task in tasks %}
                {% include 'tasks/_item.html.twig' %}
            {% endfor %}
```

Le partial `_item` hérite des variables du template appelant (`task` ici).

---

## Ce que font réellement les frameworks

Tu as construit toi-même les pièces que Laravel et Symfony fournissent. Voici la correspondance :

| Ce que tu as écrit | Dans Laravel | Dans Symfony |
|---|---|---|
| `public/index.php` (point d'entrée) | `public/index.php` | `public/index.php` |
| `Request` | `Illuminate\Http\Request` | `HttpFoundation\Request` |
| `Router` + `config/routes.php` | `routes/web.php` | attributs `#[Route]` ou `config/routes.yaml` |
| Contrôleurs dans `src/Controller` | `app/Http/Controllers` | `src/Controller` |
| `Task` + `TaskRepository` | Modèle **Eloquent** (`App\Models\Task`) | Entité **Doctrine** + Repository |
| `View` + vues PHP | `view()` + **Blade** | **Twig** |
| `Container` (autowiring) | **Service Container** | **Service Container** (DependencyInjection) |
| `config/database.php` | `config/database.php` + `.env` | `.env` + `config/packages/` |
| `Task::validateTitle()` | `$request->validate([...])` | composant **Validator** / Form |
| Redirection après POST | `redirect()->route(...)` | `$this->redirectToRoute(...)` |

**Ce que les frameworks ajoutent en plus** (et que tu n'as pas encore) : middlewares (authentification, CSRF), sessions et messages flash, migrations de base de données, ORM complet, validation avancée, mise en cache, outils en ligne de commande, tests intégrés, gestion des erreurs et des logs.

**Ce que tu peux retenir :** un framework n'a rien de magique. C'est un routeur, des contrôleurs, un conteneur et un moteur de vues, exactement comme ce que tu viens d'écrire, en plus complet et plus solide.

---

## Et maintenant ?

1. **Remplace tes briques par de vraies bibliothèques** pour voir la différence : `nikic/fast-route` (routeur), `php-di/php-di` (conteneur), `vlucas/phpdotenv` (fichier `.env`).
2. **Ajoute ce qui manque** à ton mini-framework : sessions, messages flash, jeton CSRF, authentification (en réutilisant `password_hash` et les sessions vus dans le premier cours).
3. **Apprends un framework :** **Laravel** est le plus populaire et le plus accessible pour commencer. **Symfony** est très utilisé en entreprise. Tu retrouveras dans chacun l'architecture de ce cours.
4. **Écris des tests** (PHPUnit ou Pest) : ton `TaskRepository` et ton `Container` s'y prêtent très bien.
5. **Crée une API JSON** : au lieu de rendre une vue, un contrôleur retourne du JSON (`json_encode`) avec le bon code HTTP. C'est la base des applications modernes (front JavaScript, application mobile).
