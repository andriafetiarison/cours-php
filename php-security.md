# Cours PHP : sécuriser une application web

Ce cours est court et très pratique. Chaque sujet suit la même méthode : **mauvais exemple → pourquoi c'est dangereux → bonne solution → exercice → correction**. Il se termine par une **checklist** à parcourir avant toute mise en ligne.

**Règle d'or :** ne jamais faire confiance à ce qui vient de l'extérieur. C'est vrai pour `$_GET`, `$_POST`, `$_COOKIE`, `$_FILES`, les en-têtes HTTP (`$_SERVER`), et même pour des données déjà enregistrées en base si elles viennent d'utilisateurs.

**Cinq principes qui reviennent partout :**

| Principe | En pratique |
|---|---|
| **Valider à l'entrée** | Refuser tout ce qui n'a pas la forme attendue |
| **Échapper à la sortie** | Protéger chaque donnée au moment de l'afficher |
| **Séparer le code des données** | Requêtes préparées, jamais de concaténation |
| **Défense en profondeur** | Plusieurs protections superposées : si l'une échoue, une autre reste |
| **Moindre privilège** | Chaque compte, fichier ou utilisateur n'a que les droits nécessaires |

> **Attention :** les « mauvais exemples » sont **volontairement vulnérables** et simplifiés. Ne les utilise jamais dans un vrai projet. Teste uniquement sur ta propre machine et tes propres applications.

**Plan :**

1. Validation des données
2. XSS
3. Injection SQL
4. Requêtes préparées PDO
5. CSRF
6. Sessions sécurisées
7. Mots de passe
8. Upload de fichiers sécurisé
9. Gestion des secrets avec `.env`
10. HTTPS
11. Checklist avant mise en ligne

---

## 1. Validation des données

### Mauvais exemple

```php
<?php
// inscription.php : on fait confiance à ce que le navigateur envoie
$email = $_POST['email'];
$age = $_POST['age'];
$role = $_POST['role'];

$pdo->prepare('INSERT INTO users (email, age, role) VALUES (?, ?, ?)')
    ->execute([$email, $age, $role]);

echo 'Compte créé !';
```

Avec ce formulaire, qui se « protège » uniquement dans le navigateur :

```html
<form method="post">
    <input type="email" name="email" required>
    <input type="number" name="age" min="18" max="120" required>
    <input type="hidden" name="role" value="member">
    <button>Créer mon compte</button>
</form>
```

### Pourquoi c'est dangereux

Les attributs HTML (`required`, `min`, `type="email"`) ne sont qu'une **aide pour l'utilisateur honnête**. N'importe qui peut envoyer une requête POST sans passer par ton formulaire (avec `curl`, un outil, ou en modifiant la page). Il peut donc envoyer un âge de `-5`, un email vide, ou `role=admin`. **Seule la validation côté serveur compte.**

### Bonne solution

Valider **côté serveur**, avec des règles précises et des **listes blanches** (on liste ce qui est autorisé, on refuse tout le reste).

```php
<?php

declare(strict_types=1);

/** Lit un champ texte : retourne '' si absent ou si ce n'est pas un texte (ex. tableau). */
function field(array $data, string $key): string
{
    $value = $data[$key] ?? '';

    return is_string($value) ? trim($value) : '';
}

/**
 * @return array{errors: array<string, string>, data: array<string, mixed>}
 */
function validateRegistration(array $input): array
{
    $errors = [];

    $name = field($input, 'name');
    if (mb_strlen($name) < 2 || mb_strlen($name) > 50) {
        $errors['name'] = 'Le nom doit contenir entre 2 et 50 caractères.';
    }

    $email = field($input, 'email');
    if (filter_var($email, FILTER_VALIDATE_EMAIL) === false) {
        $errors['email'] = 'Adresse email invalide.';
    }

    $age = filter_var(field($input, 'age'), FILTER_VALIDATE_INT, [
        'options' => ['min_range' => 18, 'max_range' => 120],
    ]);
    if ($age === false) {
        $errors['age'] = "L'âge doit être un entier entre 18 et 120.";
    }

    $role = field($input, 'role');
    if (!in_array($role, ['member', 'author'], true)) {   // liste blanche
        $errors['role'] = 'Rôle invalide.';
    }

    return [
        'errors' => $errors,
        'data' => ['name' => $name, 'email' => $email, 'age' => $age, 'role' => $role],
    ];
}

// Utilisation
$result = validateRegistration($_POST);

if ($result['errors'] !== []) {
    http_response_code(422);
    // réafficher le formulaire avec les messages d'erreur
} else {
    // enregistrer $result['data'] avec une requête préparée (chapitre 4)
}
```

**Points importants :**

* `field()` gère le cas où un attaquant envoie `name[]=x` (un tableau au lieu d'un texte) : sans cette précaution, PHP produirait une erreur.
* `filter_var(..., FILTER_VALIDATE_INT, ['options' => [...]])` vérifie que c'est un entier **dans une plage**. On compare avec `=== false`, car `0` est une valeur valide mais « fausse ».
* `in_array(..., true)` (mode strict) : liste blanche pour les valeurs prévues (rôles, catégories, tris).
* **Valider** (« cette donnée est-elle acceptable ? ») et **échapper** (« comment l'afficher sans danger ? ») sont **deux étapes différentes** : on ne remplace jamais l'une par l'autre.

### Exercice

Écris `validateProduct(array $input): array` (même forme de retour que ci-dessus) avec ces règles :

* `name` : texte de 1 à 100 caractères ;
* `price` : nombre décimal entre 0 et 100 000 ;
* `category` : uniquement `book`, `game` ou `tool` ;
* `stock` : entier entre 0 et 1 000.

### Correction

```php
function validateProduct(array $input): array
{
    $errors = [];

    $name = field($input, 'name');
    if ($name === '' || mb_strlen($name) > 100) {
        $errors['name'] = 'Le nom doit contenir entre 1 et 100 caractères.';
    }

    $price = filter_var(field($input, 'price'), FILTER_VALIDATE_FLOAT, [
        'options' => ['min_range' => 0, 'max_range' => 100000],
    ]);
    if ($price === false) {
        $errors['price'] = 'Le prix doit être un nombre entre 0 et 100 000.';
    }

    $category = field($input, 'category');
    if (!in_array($category, ['book', 'game', 'tool'], true)) {
        $errors['category'] = 'Catégorie invalide.';
    }

    $stock = filter_var(field($input, 'stock'), FILTER_VALIDATE_INT, [
        'options' => ['min_range' => 0, 'max_range' => 1000],
    ]);
    if ($stock === false) {
        $errors['stock'] = 'Le stock doit être un entier entre 0 et 1 000.';
    }

    return [
        'errors' => $errors,
        'data' => compact('name', 'price', 'category', 'stock'),
    ];
}
```

---

## 2. XSS (Cross-Site Scripting)

### Mauvais exemple

```php
<?php
// bonjour.php?nom=Alice
echo '<h1>Bonjour ' . $_GET['nom'] . '</h1>';

// Livre d'or : les commentaires enregistrés en base sont réaffichés tels quels
foreach ($comments as $comment) {
    echo '<p>' . $comment['text'] . '</p>';
}
```

### Pourquoi c'est dangereux

Si quelqu'un envoie `?nom=<script>alert('XSS')</script>`, ou poste un commentaire contenant du JavaScript, **le navigateur des visiteurs exécutera ce script** comme s'il venait de ton site. Il peut alors lire des informations de la page, effectuer des actions au nom de l'utilisateur connecté, ou afficher un faux formulaire de connexion. Dans le cas du livre d'or (XSS « stocké »), **tous les visiteurs** sont touchés.

### Bonne solution

**Échapper toute donnée au moment de l'afficher**, avec `htmlspecialchars()`. On en fait une petite fonction :

```php
<?php

declare(strict_types=1);

function e(mixed $value): string
{
    return htmlspecialchars((string) $value, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8');
}
```

```php
<h1>Bonjour <?= e($_GET['nom'] ?? '') ?></h1>

<?php foreach ($comments as $comment): ?>
    <p><?= e($comment['text']) ?></p>
<?php endforeach; ?>

<!-- Dans un attribut : ENT_QUOTES protège aussi les guillemets -->
<input type="text" name="title" value="<?= e($title) ?>">
```

Les liens demandent une précaution en plus : une URL de type `javascript:...` serait dangereuse même échappée.

```php
function safeUrl(string $url): string
{
    $scheme = parse_url($url, PHP_URL_SCHEME);

    return in_array($scheme, ['http', 'https'], true) ? $url : '#';
}
```

```php
<a href="<?= e(safeUrl((string) $user['website'])) ?>">Site web</a>
```

**À retenir :**

* On **échappe à l'affichage**, pas avant l'enregistrement : la base garde la donnée brute.
* Avec **Twig**, `{{ variable }}` est échappé automatiquement (c'est un grand avantage, voir le cours MVC).
* Pour passer des données PHP à du JavaScript : `json_encode($data, JSON_HEX_TAG | JSON_HEX_AMP | JSON_HEX_APOS | JSON_HEX_QUOT)`.
* **Défense en plus** : l'en-tête `Content-Security-Policy` limite les scripts autorisés. Exemple : `header("Content-Security-Policy: default-src 'self'");` (il bloque aussi les scripts « inline » : à adapter selon ton application).

### Exercice

Corrige ce fragment pour qu'il ne soit plus vulnérable au XSS :

```php
<p>Commentaire de <?= $_POST['author'] ?> :</p>
<a href="<?= $user['website'] ?>">Site</a>
<input type="text" name="title" value="<?= $title ?>">
```

### Correction

```php
<p>Commentaire de <?= e($_POST['author'] ?? '') ?> :</p>
<a href="<?= e(safeUrl((string) $user['website'])) ?>">Site</a>
<input type="text" name="title" value="<?= e($title) ?>">
```

---

## 3. Injection SQL

### Mauvais exemple

```php
<?php
// login.php
$email = $_POST['email'];
$password = $_POST['password'];

$sql = "SELECT * FROM users WHERE email = '$email' AND password = '$password'";
$user = $pdo->query($sql)->fetch();

if ($user) {
    echo 'Connecté !';
}
```

### Pourquoi c'est dangereux

Les valeurs de l'utilisateur sont **collées dans la requête SQL**. Si le champ email contient :

```
' OR '1'='1' -- 
```

la requête devient :

```sql
SELECT * FROM users WHERE email = '' OR '1'='1' -- ' AND password = ''
```

`--` commence un commentaire SQL : tout ce qui suit est ignoré, y compris la vérification du mot de passe. La condition `'1'='1'` est toujours vraie, donc la requête renvoie un utilisateur sans connaître aucun mot de passe. Une injection SQL permet aussi, selon les cas, de **lire, modifier ou supprimer** des données.

### Bonne solution

Ne **jamais** concaténer de données utilisateur dans du SQL. On envoie la requête avec des **marqueurs** (`:email`), puis les valeurs séparément : la base les traite comme de simples données, jamais comme du code.

```php
<?php

$stmt = $pdo->prepare('SELECT id, password_hash FROM users WHERE email = :email');
$stmt->execute(['email' => $email]);
$user = $stmt->fetch();

if ($user !== false && password_verify($password, $user['password_hash'])) {
    echo 'Connecté !';
}
```

Remarque : on ne compare plus le mot de passe dans le SQL. On récupère le **hash** et on utilise `password_verify()` (chapitre 7). Le chapitre suivant détaille les requêtes préparées.

### Exercice

Cette suppression est vulnérable. Explique ce que provoque `?id=1 OR 1=1`, puis corrige le code.

```php
$pdo->exec('DELETE FROM tasks WHERE id = ' . $_GET['id']);
```

### Correction

Avec `?id=1 OR 1=1`, la requête devient `DELETE FROM tasks WHERE id = 1 OR 1=1` : **toutes les tâches sont supprimées**.

Version corrigée :

```php
$id = filter_var($_GET['id'] ?? null, FILTER_VALIDATE_INT);

if ($id === false) {
    http_response_code(400);
    exit('Identifiant invalide.');
}

$stmt = $pdo->prepare('DELETE FROM tasks WHERE id = :id');
$stmt->execute(['id' => $id]);
```

(Supprimer avec une requête GET pose aussi un problème de CSRF : voir le chapitre 5.)

---

## 4. Requêtes préparées PDO

### Mauvais exemple

Utiliser `prepare()` ne suffit pas si on y colle quand même des variables :

```php
<?php
$q = $_GET['q'] ?? '';
$sort = $_GET['sort'] ?? 'id';

$stmt = $pdo->prepare("SELECT * FROM products WHERE name LIKE '%$q%' ORDER BY $sort");
$stmt->execute();
```

### Pourquoi c'est dangereux

* `$q` est inséré **dans le texte de la requête** avant même la préparation : la protection est contournée, c'est exactement la même faille que le chapitre précédent.
* Les marqueurs (`:q`, `?`) ne peuvent remplacer que des **valeurs**, pas des **noms de colonnes** ni des mots-clés (`ORDER BY`). Les débutants concaténent donc ces parties, ce qui rouvre la faille.

### Bonne solution

```php
<?php

declare(strict_types=1);

// 1. Connexion sécurisée
$pdo = new PDO($dsn, $user, $pass, [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,         // erreurs signalées par des exceptions
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    PDO::ATTR_EMULATE_PREPARES => false,                 // vraies requêtes préparées côté MySQL
]);

// 2. Valeurs : toujours avec des marqueurs
$q = trim((string) ($_GET['q'] ?? ''));

$stmt = $pdo->prepare('SELECT * FROM products WHERE name LIKE :q');
$stmt->execute(['q' => '%' . $q . '%']);      // les % font partie de la VALEUR, pas du SQL
$products = $stmt->fetchAll();

// 3. Parties dynamiques du SQL (tri, colonne) : liste blanche
$allowedSorts = ['id', 'name', 'price'];
$sort = in_array($_GET['sort'] ?? '', $allowedSorts, true) ? $_GET['sort'] : 'id';

$stmt = $pdo->prepare("SELECT * FROM products ORDER BY $sort");   // $sort vient de NOTRE liste
```

**Autres cas courants :**

```php
// LIMIT : on précise le type entier
$stmt = $pdo->prepare('SELECT * FROM products LIMIT :limit');
$stmt->bindValue(':limit', 20, PDO::PARAM_INT);
$stmt->execute();

// IN (...) : un marqueur par valeur
$ids = [1, 2, 3];
if ($ids !== []) {   // IN () vide est invalide en SQL
    $placeholders = implode(',', array_fill(0, count($ids), '?'));
    $stmt = $pdo->prepare("SELECT * FROM tasks WHERE id IN ($placeholders)");
    $stmt->execute($ids);
}
```

**Bonnes pratiques complémentaires :**

* **N'affiche jamais** une erreur SQL à l'utilisateur : attrape `PDOException`, écris-la dans un journal, affiche un message générique.
* Utilise un **compte MySQL dédié** à l'application, avec uniquement `SELECT`, `INSERT`, `UPDATE`, `DELETE` sur sa propre base : jamais `root` en production.
* Un nom de marqueur ne s'utilise qu'une seule fois par requête avec les vraies requêtes préparées : donne-lui un nom différent si besoin.

### Exercice

Écris `searchProducts(PDO $pdo, string $q, ?string $category, string $sort, int $limit): array` qui :

* cherche `$q` dans le nom ;
* filtre par catégorie si elle est fournie ;
* trie par `id`, `name` ou `price` (valeur par défaut : `id`) ;
* limite le nombre de résultats entre 1 et 100.

### Correction

```php
function searchProducts(PDO $pdo, string $q, ?string $category, string $sort, int $limit): array
{
    $allowedSorts = ['id', 'name', 'price'];
    if (!in_array($sort, $allowedSorts, true)) {
        $sort = 'id';
    }

    $limit = max(1, min($limit, 100));

    $sql = 'SELECT * FROM products WHERE name LIKE :q';
    $params = ['q' => '%' . $q . '%'];

    if ($category !== null && $category !== '') {
        $sql .= ' AND category = :category';
        $params['category'] = $category;
    }

    $sql .= " ORDER BY $sort LIMIT :limit";

    $stmt = $pdo->prepare($sql);

    foreach ($params as $name => $value) {
        $stmt->bindValue(':' . $name, $value);
    }
    $stmt->bindValue(':limit', $limit, PDO::PARAM_INT);

    $stmt->execute();

    return $stmt->fetchAll();
}
```

---

## 5. CSRF (Cross-Site Request Forgery)

### Mauvais exemple

```php
<?php
// delete.php?id=3 : une action sensible déclenchée par un simple lien (GET)
session_start();

if (isset($_SESSION['user_id'])) {
    $pdo->prepare('DELETE FROM tasks WHERE id = ?')->execute([$_GET['id']]);
}
```

Ou un formulaire POST **sans aucune vérification** :

```html
<form method="post" action="/tasks/3/delete">
    <button>Supprimer</button>
</form>
```

### Pourquoi c'est dangereux

Le navigateur envoie **automatiquement les cookies de session** à ton site, quel que soit le site depuis lequel la requête est déclenchée. Si tu es connecté à ton application, une **autre page web** peut déclencher une action à ta place, sans que tu t'en rendes compte. Avec l'exemple GET, il suffit d'une image cachée sur une autre page :

```html
<img src="https://ton-site.com/delete.php?id=3" width="0" height="0">
```

Un formulaire POST envoyé automatiquement par une page malveillante fonctionne de la même façon s'il n'y a pas de jeton.

### Bonne solution

Trois protections qui se combinent :

1. **Jamais de modification en GET** : utilise POST pour tout ce qui crée, modifie ou supprime.
2. **Un jeton CSRF** : une valeur aléatoire propre à la session, placée dans chaque formulaire et vérifiée à la réception. Un site tiers ne peut pas la connaître.
3. **Cookie de session `SameSite=Lax`** (chapitre 6) : le navigateur n'envoie pas le cookie dans la plupart des requêtes venant d'un autre site.

```php
<?php

declare(strict_types=1);

final class Csrf
{
    /** Retourne le jeton de la session (le crée au besoin). */
    public static function token(): string
    {
        if (empty($_SESSION['csrf_token'])) {
            $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
        }

        return $_SESSION['csrf_token'];
    }

    /** Vérifie le jeton reçu. */
    public static function verify(mixed $token): bool
    {
        return isset($_SESSION['csrf_token'])
            && is_string($token)
            && hash_equals($_SESSION['csrf_token'], $token);
    }
}
```

Dans **chaque formulaire POST** :

```php
<form method="post" action="/tasks/3/delete">
    <input type="hidden" name="csrf_token" value="<?= e(Csrf::token()) ?>">
    <button type="submit">Supprimer</button>
</form>
```

À la **réception** (dans le contrôleur, ou pour toutes les requêtes POST à un seul endroit) :

```php
if ($_SERVER['REQUEST_METHOD'] === 'POST' && !Csrf::verify($_POST['csrf_token'] ?? null)) {
    http_response_code(403);
    exit('Requête refusée (jeton CSRF invalide).');
}
```

* `random_bytes()` produit un aléa cryptographiquement sûr (jamais `rand()` ni `uniqid()` pour la sécurité).
* `hash_equals()` compare deux textes en temps constant (évite les fuites d'information par la durée de la comparaison).
* La session doit être démarrée avant d'utiliser `Csrf` (voir chapitre 6).
* Pour des appels JavaScript (fetch), envoie le jeton dans un en-tête `X-CSRF-Token` et vérifie-le de la même façon.

### Exercice

Dans l'application de tâches du cours MVC, protège la suppression : ajoute le jeton au formulaire de `tasks/index.php` et vérifie-le dans `TaskController::delete()`.

### Correction

Vue (`views/tasks/index.php`) :

```php
<form method="post" action="/tasks/<?= $task->id ?>/delete"
      onsubmit="return confirm('Supprimer cette tâche ?');">
    <input type="hidden" name="csrf_token" value="<?= e(Csrf::token()) ?>">
    <button type="submit" class="danger">Supprimer</button>
</form>
```

*(Ajoute `use App\Core\Csrf;` en haut de la vue, ou fournis `Csrf::token()` via une fonction d'aide.)*

Contrôleur :

```php
public function delete(Request $request, int $id): void
{
    if (!Csrf::verify($request->input('csrf_token'))) {
        http_response_code(403);
        exit('Requête refusée (jeton CSRF invalide).');
    }

    $this->tasks->delete($id);
    $this->redirect('/tasks');
}
```

Pense à placer la classe dans `src/Core/Csrf.php` (namespace `App\Core`) et à appeler `startSecureSession()` (chapitre suivant) au début de `public/index.php`, car le jeton est stocké en session. Applique ensuite la même vérification à **toutes** les actions POST (création, modification, bascule).

---

## 6. Sessions sécurisées

### Mauvais exemple

```php
<?php
session_start();   // réglages par défaut

if ($email === 'alice@example.com' && $password === 'secret') {
    // On stocke trop d'informations, mot de passe compris
    $_SESSION['user'] = ['email' => $email, 'password' => $password, 'is_admin' => true];
    // Pas de session_regenerate_id()
}

// Déconnexion incomplète : la session reste valide
if (isset($_GET['logout'])) {
    unset($_SESSION['user']);
}
```

### Pourquoi c'est dangereux

* **Fixation de session** : si l'identifiant de session existait avant la connexion et n'est pas renouvelé, quelqu'un qui le connaissait garde l'accès une fois que l'utilisateur s'est connecté.
* Un cookie de session **lisible par JavaScript** peut être volé par une faille XSS.
* Un cookie **envoyé en HTTP** peut être intercepté sur un réseau non sûr.
* **Stocker le mot de passe** ou des droits en session les expose inutilement.
* Une session **qui n'expire jamais** reste exploitable sur un ordinateur partagé.

### Bonne solution

```php
<?php

declare(strict_types=1);

function isHttps(): bool
{
    return !empty($_SERVER['HTTPS']) && $_SERVER['HTTPS'] !== 'off';
}

function startSecureSession(): void
{
    if (session_status() === PHP_SESSION_ACTIVE) {
        return;
    }

    ini_set('session.use_strict_mode', '1');   // refuse les identifiants de session inconnus
    ini_set('session.use_only_cookies', '1');  // jamais d'identifiant dans l'URL

    session_set_cookie_params([
        'lifetime' => 0,           // cookie supprimé à la fermeture du navigateur
        'path' => '/',
        'secure' => isHttps(),     // envoyé uniquement en HTTPS (quand le site l'est)
        'httponly' => true,        // inaccessible depuis JavaScript
        'samesite' => 'Lax',       // pas envoyé dans la plupart des requêtes inter-sites
    ]);

    session_start();
}
```

À la **connexion** : renouveler l'identifiant et ne stocker que l'identifiant de l'utilisateur.

```php
session_regenerate_id(true);               // true : supprime l'ancienne session
$_SESSION['user_id'] = (int) $user['id'];  // rien d'autre : on relit l'utilisateur en base si besoin
```

À la **déconnexion** : tout nettoyer, y compris le cookie.

```php
function logout(): void
{
    $_SESSION = [];

    if (ini_get('session.use_cookies')) {
        $p = session_get_cookie_params();
        setcookie(session_name(), '', [
            'expires' => time() - 3600,
            'path' => $p['path'],
            'domain' => $p['domain'],
            'secure' => $p['secure'],
            'httponly' => $p['httponly'],
            'samesite' => $p['samesite'] ?? 'Lax',
        ]);
    }

    session_destroy();
}
```

**Les trois attributs du cookie à connaître :**

| Attribut | Rôle |
|---|---|
| `HttpOnly` | Le JavaScript ne peut pas lire le cookie (limite les dégâts d'un XSS) |
| `Secure` | Le cookie ne voyage qu'en HTTPS (la valeur est `isHttps()` pour que ça marche aussi en local en HTTP) |
| `SameSite` | Limite l'envoi du cookie depuis d'autres sites (aide contre le CSRF) |

### Exercice

Ajoute une **expiration après 30 minutes d'inactivité** : écris `enforceSessionTimeout()`, à appeler après `startSecureSession()`.

### Correction

```php
function enforceSessionTimeout(int $maxIdleSeconds = 1800): void
{
    $now = time();

    if (isset($_SESSION['last_activity']) && ($now - $_SESSION['last_activity']) > $maxIdleSeconds) {
        logout();
        header('Location: /login?expired=1');
        exit;
    }

    $_SESSION['last_activity'] = $now;   // l'activité repousse l'expiration
}
```

Utilisation, en haut de `public/index.php` :

```php
startSecureSession();
enforceSessionTimeout();
```

---

## 7. Mots de passe : `password_hash()` et `password_verify()`

### Mauvais exemple

```php
<?php
// Inscription
$hash = md5($_POST['password']);
$pdo->prepare('INSERT INTO users (email, password) VALUES (?, ?)')->execute([$email, $hash]);

// Connexion
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = ? AND password = ?');
$stmt->execute([$email, md5($password)]);
```

(Stocker le mot de passe **en clair** serait encore pire.)

### Pourquoi c'est dangereux

* `md5` et `sha1` sont conçus pour être **très rapides** : un attaquant qui obtient ta base de données peut tester des milliards de mots de passe par seconde, ou utiliser des tables de hash précalculées.
* Sans **sel** (valeur aléatoire propre à chaque mot de passe), deux utilisateurs ayant le même mot de passe ont le même hash, et un hash craqué révèle tous les comptes identiques.
* Beaucoup d'utilisateurs **réutilisent leur mot de passe** ailleurs : une fuite chez toi compromet leurs autres comptes.

### Bonne solution

PHP fournit tout ce qu'il faut : un algorithme **lent exprès**, un **sel aléatoire** automatique, et une vérification sûre.

```php
<?php

declare(strict_types=1);

// --- Inscription : on enregistre le hash, jamais le mot de passe ---
$hash = password_hash($password, PASSWORD_DEFAULT);

$stmt = $pdo->prepare('INSERT INTO users (email, password_hash) VALUES (:email, :hash)');
$stmt->execute(['email' => $email, 'hash' => $hash]);

// --- Connexion ---
$stmt = $pdo->prepare('SELECT id, password_hash FROM users WHERE email = :email');
$stmt->execute(['email' => $email]);
$user = $stmt->fetch();

if ($user !== false && password_verify($password, $user['password_hash'])) {
    // Si PHP améliore l'algorithme par défaut, on met le hash à jour au passage
    if (password_needs_rehash($user['password_hash'], PASSWORD_DEFAULT)) {
        $update = $pdo->prepare('UPDATE users SET password_hash = :hash WHERE id = :id');
        $update->execute(['hash' => password_hash($password, PASSWORD_DEFAULT), 'id' => $user['id']]);
    }

    session_regenerate_id(true);
    $_SESSION['user_id'] = (int) $user['id'];
} else {
    $error = 'Email ou mot de passe incorrect.';   // même message dans les deux cas
}
```

La table :

```sql
CREATE TABLE users (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(150) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL      -- 255 : la taille des hash peut évoluer
);
```

**Règles à retenir :**

* `password_hash()` génère le sel et l'inclut dans le résultat ; `password_verify()` le relit tout seul : **tu n'as rien à gérer**.
* `PASSWORD_DEFAULT` choisit l'algorithme recommandé par PHP (aujourd'hui bcrypt) : n'écris pas toi-même le nom de l'algorithme sauf besoin précis.
* **Ne modifie pas** le mot de passe saisi (pas de `trim`, pas de limitation de caractères spéciaux).
* Exige une longueur minimale raisonnable (12 caractères ou plus). Avec bcrypt, ce qui dépasse **72 octets** est ignoré : refuse les mots de passe plus longs plutôt que de les tronquer en silence.
* **Message d'erreur identique** pour « email inconnu » et « mauvais mot de passe » (on ne révèle pas quels comptes existent).
* Ajoute une **limitation des tentatives** (par exemple, une pause après plusieurs échecs) pour freiner les essais en série.
* Ne envoie **jamais** un mot de passe par email : pour un oubli, envoie un lien de réinitialisation à usage unique et limité dans le temps.

### Exercice

Écris deux fonctions :

* `registerUser(PDO $pdo, string $email, string $password): void` : vérifie l'email, impose 12 caractères minimum (72 octets maximum), refuse un email déjà utilisé, puis enregistre le hash ;
* `authenticate(PDO $pdo, string $email, string $password): ?int` : retourne l'identifiant de l'utilisateur, ou `null` si la connexion échoue (avec mise à jour du hash si nécessaire).

### Correction

```php
function registerUser(PDO $pdo, string $email, string $password): void
{
    if (filter_var($email, FILTER_VALIDATE_EMAIL) === false) {
        throw new InvalidArgumentException('Email invalide.');
    }

    if (mb_strlen($password) < 12) {
        throw new InvalidArgumentException('Le mot de passe doit contenir au moins 12 caractères.');
    }

    if (strlen($password) > 72) {
        throw new InvalidArgumentException('Mot de passe trop long (72 octets maximum).');
    }

    $stmt = $pdo->prepare('SELECT id FROM users WHERE email = :email');
    $stmt->execute(['email' => $email]);

    if ($stmt->fetch() !== false) {
        throw new InvalidArgumentException('Cet email est déjà utilisé.');
    }

    $stmt = $pdo->prepare('INSERT INTO users (email, password_hash) VALUES (:email, :hash)');
    $stmt->execute([
        'email' => $email,
        'hash' => password_hash($password, PASSWORD_DEFAULT),
    ]);
}

function authenticate(PDO $pdo, string $email, string $password): ?int
{
    $stmt = $pdo->prepare('SELECT id, password_hash FROM users WHERE email = :email');
    $stmt->execute(['email' => $email]);
    $user = $stmt->fetch();

    if ($user === false || !password_verify($password, $user['password_hash'])) {
        return null;
    }

    if (password_needs_rehash($user['password_hash'], PASSWORD_DEFAULT)) {
        $update = $pdo->prepare('UPDATE users SET password_hash = :hash WHERE id = :id');
        $update->execute([
            'hash' => password_hash($password, PASSWORD_DEFAULT),
            'id' => $user['id'],
        ]);
    }

    return (int) $user['id'];
}
```

*(La contrainte `UNIQUE` sur la colonne `email` reste indispensable : c'est elle qui garantit l'unicité même si deux inscriptions arrivent en même temps.)*

---

## 8. Upload de fichiers sécurisé

### Mauvais exemple

```html
<form method="post" enctype="multipart/form-data">
    <input type="file" name="avatar">
    <button>Envoyer</button>
</form>
```

```php
<?php
// Le fichier est copié tel quel, avec le nom donné par l'utilisateur,
// dans un dossier accessible depuis le navigateur
move_uploaded_file($_FILES['avatar']['tmp_name'], 'uploads/' . $_FILES['avatar']['name']);
```

### Pourquoi c'est dangereux

* Si quelqu'un envoie un fichier **`.php`** et que le dossier `uploads/` est accessible, il suffit d'ouvrir son URL pour que **le serveur exécute son code** : c'est la prise de contrôle du site.
* Le **nom** et le **type** (`$_FILES[...]['name']`, `['type']`) viennent du navigateur : ils sont falsifiables.
* Sans limite de taille, on peut saturer le disque ; sans nom unique, on peut **écraser** un fichier existant.
* Certains formats (SVG, HTML) peuvent contenir du JavaScript.

### Bonne solution

Le principe : **ne rien croire de ce que dit le client**, vérifier le **contenu réel**, **renommer** le fichier, et le stocker **hors du dossier public**.

```php
<?php

declare(strict_types=1);

/**
 * @param array<string, string> $allowedMimes  type MIME autorisé => extension à utiliser
 * @return string  le nom du fichier enregistré
 */
function saveUpload(array $file, string $destDir, array $allowedMimes, int $maxBytes): string
{
    if (($file['error'] ?? UPLOAD_ERR_NO_FILE) !== UPLOAD_ERR_OK) {
        throw new RuntimeException("Échec de l'envoi du fichier.");
    }

    if (!is_uploaded_file($file['tmp_name'])) {
        throw new RuntimeException('Fichier invalide.');
    }

    if (filesize($file['tmp_name']) > $maxBytes) {
        throw new RuntimeException('Fichier trop volumineux.');
    }

    // Type réel, détecté à partir du CONTENU (jamais $_FILES['type'] ni l'extension d'origine)
    $mime = (new finfo(FILEINFO_MIME_TYPE))->file($file['tmp_name']);

    if (!isset($allowedMimes[$mime])) {
        throw new RuntimeException('Type de fichier non autorisé.');
    }

    // Pour une image, on vérifie en plus que c'en est vraiment une
    if (str_starts_with($mime, 'image/') && getimagesize($file['tmp_name']) === false) {
        throw new RuntimeException("Ce fichier n'est pas une image valide.");
    }

    // Nom généré par le serveur : aléatoire, avec une extension que NOUS choisissons
    $name = bin2hex(random_bytes(16)) . '.' . $allowedMimes[$mime];

    if (!move_uploaded_file($file['tmp_name'], $destDir . '/' . $name)) {
        throw new RuntimeException("Impossible d'enregistrer le fichier.");
    }

    return $name;
}
```

Utilisation (avatar : images uniquement, 2 Mo maximum, **dossier hors de `public/`**) :

```php
try {
    $avatar = saveUpload($_FILES['avatar'] ?? [], __DIR__ . '/../storage/avatars', [
        'image/jpeg' => 'jpg',
        'image/png' => 'png',
        'image/webp' => 'webp',
    ], 2 * 1024 * 1024);

    // enregistrer $avatar (le nom du fichier) en base, avec une requête préparée
} catch (RuntimeException $e) {
    $error = $e->getMessage();
}
```

Pour **afficher** un fichier stocké hors de `public/`, un petit script PHP le renvoie après avoir vérifié le nom :

```php
// /files/{name}
if (!preg_match('/^[a-f0-9]{32}\.(jpg|png|webp)$/', $name)) {
    http_response_code(404);
    exit;
}

$path = __DIR__ . '/../storage/avatars/' . $name;

if (!is_file($path)) {
    http_response_code(404);
    exit;
}

$types = ['jpg' => 'image/jpeg', 'png' => 'image/png', 'webp' => 'image/webp'];

header('Content-Type: ' . $types[pathinfo($name, PATHINFO_EXTENSION)]);
header('X-Content-Type-Options: nosniff');
header('Content-Length: ' . filesize($path));
readfile($path);
```

**Si tu dois absolument stocker dans un dossier public**, empêche l'exécution de code dedans (Apache 2.4, fichier `.htaccess` dans ce dossier) :

```apache
<FilesMatch "\.(php|phtml|phar)$">
    Require all denied
</FilesMatch>
```

**Rappels :**

* le formulaire doit avoir `enctype="multipart/form-data"` ;
* n'autorise **que** les types dont tu as besoin (pas de SVG ni de HTML) ;
* dans `php.ini`, `upload_max_filesize` (2 Mo par défaut) et `post_max_size` fixent les limites du serveur ;
* crée le dossier de destination avec des droits restreints (par exemple `mkdir($dir, 0750, true)`).

### Exercice

Utilise `saveUpload()` pour permettre l'envoi d'un **document PDF** de 5 Mo maximum. Écris le formulaire et le traitement, en précisant ce qu'il faut régler dans `php.ini`.

### Correction

Formulaire :

```php
<form method="post" enctype="multipart/form-data">
    <input type="hidden" name="csrf_token" value="<?= e(Csrf::token()) ?>">
    <input type="file" name="document" accept="application/pdf">
    <button type="submit">Envoyer</button>
</form>
```

Traitement :

```php
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    if (!Csrf::verify($_POST['csrf_token'] ?? null)) {
        http_response_code(403);
        exit('Jeton CSRF invalide.');
    }

    try {
        $name = saveUpload(
            $_FILES['document'] ?? [],
            __DIR__ . '/../storage/documents',
            ['application/pdf' => 'pdf'],
            5 * 1024 * 1024,
        );

        // enregistrer $name en base
    } catch (RuntimeException $e) {
        $error = $e->getMessage();
    }
}
```

Dans `php.ini`, `upload_max_filesize` (2 Mo par défaut) doit être porté à au moins `5M`, et `post_max_size` doit être supérieur ou égal à cette valeur ; redémarre ensuite le serveur. Le champ `accept` du formulaire n'est qu'une aide : la vraie vérification est celle de `saveUpload()`.

---

## 9. Gestion des secrets avec `.env`

### Mauvais exemple

```php
<?php
// config/database.php : fichier envoyé sur Git avec le reste du code
return [
    'host' => 'localhost',
    'user' => 'root',
    'pass' => 'Sup3rSecret!',
    'api_key' => 'sk_live_xxxxxxxxxxxxxxxx',
];
```

### Pourquoi c'est dangereux

* Git **conserve tout l'historique** : même si tu supprimes le secret plus tard, il reste récupérable dans les anciennes versions.
* Un dépôt public (ou qui le devient un jour) expose immédiatement les mots de passe et les clés. Des robots parcourent GitHub en permanence pour trouver des clés.
* Les mêmes identifiants en développement et en production : une erreur locale peut toucher les vraies données.

### Bonne solution

Les secrets vont dans un fichier **`.env`**, **hors du dépôt Git** et **hors du dossier public**. Le code, lui, ne contient que des **noms de variables**.

```bash
composer require vlucas/phpdotenv
```

`.env` (jamais envoyé sur Git) :

```
APP_ENV=local
APP_DEBUG=true

DB_HOST=localhost
DB_NAME=mvc_taches
DB_USER=root
DB_PASS=
```

`.env.example` (envoyé sur Git : il montre quelles variables existent, **sans les vraies valeurs**) :

```
APP_ENV=local
APP_DEBUG=true

DB_HOST=localhost
DB_NAME=
DB_USER=
DB_PASS=
```

`.gitignore` :

```
/vendor/
.env
```

Chargement au démarrage (`public/index.php`) :

```php
use Dotenv\Dotenv;

require __DIR__ . '/../vendor/autoload.php';

$dotenv = Dotenv::createImmutable(__DIR__ . '/..');   // dossier qui contient .env
$dotenv->load();
$dotenv->required(['DB_HOST', 'DB_NAME', 'DB_USER'])->notEmpty();   // échoue clairement s'il manque quelque chose
```

`config/database.php` :

```php
<?php

declare(strict_types=1);

return [
    'host' => $_ENV['DB_HOST'],
    'name' => $_ENV['DB_NAME'],
    'user' => $_ENV['DB_USER'],
    'pass' => $_ENV['DB_PASS'] ?? '',
    'charset' => 'utf8mb4',
];
```

**Bonnes pratiques :**

* En **production**, chaque environnement a **son propre `.env`** (ou de vraies variables d'environnement du serveur), avec des mots de passe **différents** de ceux du développement.
* Le `.env` doit être **inaccessible depuis le web** : c'est le cas si la racine du site est le dossier `public/` (voir le cours MVC).
* **Si un secret a été committé par erreur :** considère-le comme **compromis**. Change-le immédiatement (nouveau mot de passe, nouvelle clé) : supprimer le fichier du dépôt ne suffit pas. Puis retire le fichier du suivi avec `git rm --cached .env` et ajoute-le au `.gitignore`.
* `APP_DEBUG=false` en production : jamais de messages d'erreur détaillés pour les visiteurs.

### Exercice

Migre le fichier `config/database.php` de ton projet MVC vers un `.env` : crée `.env`, `.env.example`, mets à jour `.gitignore` et `config/database.php`, puis charge le `.env` au démarrage.

### Correction

1. `composer require vlucas/phpdotenv`
2. Crée `.env` avec tes vraies valeurs (`DB_HOST`, `DB_NAME`, `DB_USER`, `DB_PASS`) et `.env.example` avec les mêmes clés sans valeurs sensibles.
3. Ajoute `.env` à `.gitignore`.
4. Dans `public/index.php`, charge le `.env` juste après `require __DIR__ . '/../vendor/autoload.php';` (code ci-dessus).
5. Remplace les valeurs de `config/database.php` par `$_ENV['DB_HOST']`, etc.
6. Teste : l'application fonctionne ; en supprimant temporairement `DB_NAME` du `.env`, elle doit s'arrêter avec un message clair.
7. Si le mot de passe a déjà été committé, **change-le** côté MySQL.

---

## 10. HTTPS

### Mauvais exemple

```html
<!-- Site servi en HTTP simple : le mot de passe voyage en clair -->
<form method="post" action="http://mon-site.com/login">
    <input type="email" name="email">
    <input type="password" name="password">
    <button>Connexion</button>
</form>
```

Et un cookie de session envoyé sans l'attribut `Secure`.

### Pourquoi c'est dangereux

En HTTP, tout ce qui circule entre le navigateur et le serveur (mots de passe, cookies de session, données personnelles) est **lisible et modifiable** par quiconque se trouve sur le chemin : Wi-Fi public, réseau compromis, fournisseur d'accès. Un cookie de session intercepté permet de **prendre la place de l'utilisateur**. Les navigateurs affichent d'ailleurs « Non sécurisé » pour ces sites.

### Bonne solution

**HTTPS partout** : le trafic est chiffré et le serveur est authentifié grâce à un certificat. Les étapes :

1. **Obtenir un certificat** : Let's Encrypt est **gratuit** (via Certbot, ou en un clic dans le panneau de la plupart des hébergeurs).
2. **Rediriger tout le trafic HTTP vers HTTPS** (Apache, dans `public/.htaccess`) :

```apache
RewriteEngine On
RewriteCond %{HTTPS} !=on
RewriteRule ^ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
```

3. **Activer HSTS** (le navigateur refuse ensuite le HTTP pour ce site) et quelques en-têtes de sécurité :

```php
<?php

declare(strict_types=1);

function sendSecurityHeaders(): void
{
    header('X-Content-Type-Options: nosniff');                       // pas de "devinette" de type de fichier
    header('X-Frame-Options: DENY');                                  // interdit d'afficher le site dans une iframe
    header('Referrer-Policy: strict-origin-when-cross-origin');

    if (isHttps()) {
        // Commence avec une durée courte pour tester, puis augmente (ex. 31536000 = 1 an)
        header('Strict-Transport-Security: max-age=300');
    }
}
```

4. **Cookies `Secure`** : c'est déjà fait avec `'secure' => isHttps()` dans `startSecureSession()` (chapitre 6).

**À savoir :**

* En **développement local**, `http://localhost` suffit : c'est pour cela que `isHttps()` conditionne les réglages.
* **Derrière un proxy ou un équilibreur de charge** (très fréquent chez les hébergeurs), PHP ne voit parfois pas HTTPS directement : le proxy transmet l'information dans l'en-tête `X-Forwarded-Proto`. Ne fais confiance à cet en-tête que si le proxy est le tien ou celui de ton hébergeur (n'importe qui peut envoyer cet en-tête s'il atteint directement ton serveur).
* HSTS est **difficile à annuler** : n'active-le qu'après avoir vérifié que tout le site fonctionne en HTTPS, et n'ajoute `includeSubDomains` que si tous tes sous-domaines le supportent.
* Le renouvellement du certificat est **automatique** avec Certbot ou ton hébergeur : vérifie qu'il l'est.

### Exercice

1. Écris la redirection HTTP → HTTPS pour Apache.
2. Appelle `sendSecurityHeaders()` au début de `public/index.php`.
3. Explique comment vérifier que les en-têtes sont bien envoyés.

### Correction

1. Le bloc `.htaccess` ci-dessus, dans `public/.htaccess`, **avant** la règle qui redirige vers `index.php`.
2. Dans `public/index.php`, juste après `startSecureSession()` :

```php
sendSecurityHeaders();
```

3. Dans le terminal : `curl -I https://ton-site.com` (affiche les en-têtes de réponse), ou dans le navigateur, outils de développement (`F12`) → onglet **Réseau** → clique sur la requête principale → **En-têtes de réponse**. Vérifie aussi que `curl -I http://ton-site.com` répond `301` avec une `Location` en `https://`.

---

## 11. Checklist avant mise en ligne

Parcours cette liste **avant chaque mise en production**, case par case.

### Données et affichage

- [ ] Toutes les entrées sont **validées côté serveur** (type, longueur, plage, liste blanche).
- [ ] Toute donnée affichée est **échappée** (`e()` / `htmlspecialchars`, ou Twig avec échappement automatique).
- [ ] Les liens issus de données utilisateur n'acceptent que `http` et `https`.
- [ ] Un en-tête `Content-Security-Policy` est défini (même simple).

### Base de données

- [ ] **Aucune** requête SQL ne contient de variable concaténée : requêtes préparées partout.
- [ ] Les parties dynamiques (tri, colonnes) passent par une **liste blanche**.
- [ ] PDO est configuré avec `ERRMODE_EXCEPTION` et les erreurs SQL ne sont **jamais affichées** aux visiteurs.
- [ ] L'application utilise un **compte MySQL dédié**, avec uniquement les droits nécessaires (pas `root`).

### Authentification et sessions

- [ ] Les mots de passe sont stockés avec `password_hash()` et vérifiés avec `password_verify()`.
- [ ] La longueur minimale est imposée (12 caractères ou plus) et le message d'erreur de connexion est **identique** pour tous les échecs.
- [ ] Les tentatives de connexion sont **limitées**.
- [ ] `session_regenerate_id(true)` est appelé à la connexion.
- [ ] Le cookie de session est `HttpOnly`, `Secure` (en HTTPS) et `SameSite`.
- [ ] La déconnexion détruit la session **et** le cookie ; la session expire après inactivité.
- [ ] Chaque page ou action protégée **vérifie** l'authentification et les droits (pas seulement le menu qui cache un lien).

### Formulaires et actions

- [ ] Toutes les actions qui modifient des données sont en **POST**, jamais en GET.
- [ ] Chaque formulaire POST contient un **jeton CSRF**, vérifié côté serveur.

### Fichiers envoyés

- [ ] Le type est vérifié sur le **contenu** (`finfo`), la taille est limitée, la liste des types est blanche.
- [ ] Les fichiers sont **renommés** par le serveur et stockés **hors de `public/`** (ou sans exécution de PHP possible).
- [ ] Les limites `upload_max_filesize` et `post_max_size` sont adaptées.

### Configuration et secrets

- [ ] Les mots de passe et clés sont dans un **`.env`** (ou des variables d'environnement), **absent de Git** (`.gitignore`).
- [ ] Les identifiants de production sont **différents** de ceux du développement.
- [ ] Aucun secret n'a jamais été committé (sinon : changés).
- [ ] Seul le dossier `public/` est accessible depuis le web (`src/`, `vendor/`, `config/`, `.env` ne le sont pas).

### Serveur et HTTPS

- [ ] Le site fonctionne en **HTTPS**, le HTTP est redirigé en 301, le certificat se renouvelle automatiquement.
- [ ] Les en-têtes `Strict-Transport-Security`, `X-Content-Type-Options`, `Referrer-Policy` sont présents.
- [ ] `display_errors` est sur `Off` et `log_errors` sur `On` ; `APP_DEBUG=false`.
- [ ] Les fichiers de test ou de diagnostic (`phpinfo()`, `test.php`, `.sql`, sauvegardes) ont été **supprimés** du serveur.
- [ ] La version de PHP est **encore maintenue** (voir php.net/supported-versions.php).

### Dépendances et exploitation

- [ ] `composer audit` ne signale aucune faille connue ; `composer install --no-dev --optimize-autoloader` est utilisé en production.
- [ ] Les comptes et mots de passe par défaut ont été supprimés ou changés.
- [ ] Des **sauvegardes** de la base existent et ont été **testées** (restauration réussie).
- [ ] Un **journal d'erreurs** existe et quelqu'un le consulte.

### Test final

- [ ] J'ai essayé de saisir `<script>alert(1)</script>` et une apostrophe `'` dans **chaque** champ.
- [ ] J'ai essayé d'ouvrir les pages protégées **sans être connecté**, et celles d'un autre utilisateur.
- [ ] J'ai essayé d'envoyer un formulaire **sans jeton CSRF** : il est refusé.

---

## Pour aller plus loin

* **OWASP Top 10** (owasp.org) : la liste des risques les plus courants pour les applications web. Tu viens de traiter plusieurs d'entre eux.
* **OWASP Cheat Sheet Series** : des fiches très pratiques sur les sessions, les mots de passe, le CSRF, etc.
* **`composer audit`** : à lancer régulièrement pour détecter les bibliothèques vulnérables.
* **Frameworks (Laravel, Symfony)** : ils intègrent déjà l'échappement automatique, la protection CSRF, le hachage des mots de passe, la validation et la gestion du `.env`. Comprendre ce chapitre te permet de savoir **ce qu'ils font pour toi**, et de ne pas les contourner par mégarde.
* **Un principe pour finir :** la sécurité n'est pas une étape finale, c'est une **habitude**. Valide, échappe, prépare, vérifie, et mets à jour.
