# Cours PHP pratique : de zéro à une vraie petite application

Ce cours est court, progressif et basé sur **PHP 8.x**. Chaque notion est suivie d'un exemple que tu peux lancer tout de suite.

**Méthode :** explication → code → résultat → exercice → correction.

**Prérequis :** un ordinateur, un éditeur de texte (VS Code conseillé) et un terminal.

---

## Chapitre 1 : Installer et lancer PHP

### Ce que tu vas apprendre

Tu vas installer PHP, créer ton premier fichier et le lancer avec le serveur intégré. À la fin, tu auras déjà affiché une page web générée par PHP.

### 1. Notion

PHP est un langage qui s'exécute **sur ton ordinateur (le serveur)** et produit du HTML. Un fichier PHP a l'extension `.php` et le code PHP se place entre `<?php` et `?>`.

**Installation :**

```bash
# macOS (avec Homebrew)
brew install php

# Ubuntu / Debian
sudo apt install php-cli php-mysql php-mbstring

# Windows : télécharge PHP sur https://windows.php.net/download
# (ou installe XAMPP, qui inclut PHP et MySQL)
```

Vérifie l'installation :

```bash
php -v
```

Tu dois voir une version `8.x`.

#### Option B : utiliser XAMPP ou WAMP (tout-en-un)

**XAMPP** (Windows, macOS, Linux) et **WAMP** (Windows uniquement) installent en un clic PHP, le serveur Apache, MySQL/MariaDB et phpMyAdmin. C'est l'option la plus simple si tu veux tout avoir d'un coup pour les chapitres 7 à 10. Choisis une version avec **PHP 8.x**.

| | XAMPP | WAMP |
|---|---|---|
| Téléchargement | apachefriends.org | wampserver.com |
| Dossier d'installation (Windows) | `C:\xampp` | `C:\wamp64` |
| **Dossier de tes sites** | `C:\xampp\htdocs` | `C:\wamp64\www` |
| Démarrage | *XAMPP Control Panel* → **Start** sur Apache et MySQL | Lancer *WampServer* : l'icône dans la barre des tâches doit devenir **verte** |
| phpMyAdmin | `http://localhost/phpmyadmin` | `http://localhost/phpmyadmin` |
| PHP en terminal | `C:\xampp\php\php.exe` | `C:\wamp64\bin\php\php8.x.x\php.exe` |

*(Sur macOS, XAMPP utilise `/Applications/XAMPP/htdocs` ; sur Linux, `/opt/lampp/htdocs`.)*

**Mise en route :**

1. Installe XAMPP ou WAMP, puis démarre **Apache** et **MySQL** (voyants verts).
2. Crée ton dossier de travail **dans le dossier des sites** : `C:\xampp\htdocs\cours` (XAMPP) ou `C:\wamp64\www\cours` (WAMP).
3. Mets-y ton fichier `index.php` (voir l'exemple ci-dessous).
4. Ouvre `http://localhost/cours/index.php` dans ton navigateur.

Avec XAMPP/WAMP, **tu n'as pas besoin de `php -S`** : Apache sert déjà tes fichiers. Modifie ton fichier, enregistre, puis actualise la page (`F5`).

> **Pour tout le cours :** quand tu vois `http://localhost:8000/fichier.php`, remplace par `http://localhost/cours/fichier.php` si tu utilises XAMPP/WAMP.

**Utiliser la commande `php` dans le terminal (facultatif) :** si `php -v` répond « commande introuvable », appelle PHP avec son chemin complet (`C:\xampp\php\php.exe -v`) ou ajoute son dossier à la variable d'environnement `Path` de Windows (*Paramètres → Variables d'environnement → Path → Nouveau*), puis rouvre le terminal.

**Problèmes fréquents :**

| Problème | Solution |
|---|---|
| Apache ne démarre pas | Le port 80 est probablement occupé (Skype, IIS, autre serveur). Change le port dans `httpd.conf` (`Listen 8080`), puis utilise `http://localhost:8080/cours/`. |
| Le navigateur affiche le code PHP ou télécharge le fichier | Tu as ouvert le fichier directement (`file:///...`). Passe toujours par `http://localhost/...`. |
| Erreur 404 | Ton dossier n'est pas dans `htdocs` (XAMPP) ou `www` (WAMP), ou le nom dans l'URL est faux. |
| MySQL ne démarre pas | Un autre MySQL tourne déjà (port 3306). Arrête-le ou change le port. |

Tu peux aussi **mixer** : utiliser `php -S` pour servir tes pages et n'utiliser XAMPP/WAMP que pour MySQL (démarre seulement MySQL dans ce cas).

### 2. Exemple

Crée un dossier `cours` et, dedans, un fichier `index.php` :

```php
<?php

echo "Bonjour, je suis du PHP !";
```

Lance-le dans le terminal :

```bash
php index.php
```

Résultat :

```
Bonjour, je suis du PHP !
```

Maintenant, lance un **serveur local** depuis le dossier `cours` :

```bash
php -S localhost:8000
```

Ouvre `http://localhost:8000` dans ton navigateur. Tu vois ton message. Pour arrêter le serveur : `Ctrl + C`.

> **Avec XAMPP/WAMP :** place `index.php` dans `htdocs/cours` (XAMPP) ou `www/cours` (WAMP) et ouvre `http://localhost/cours/` (Apache doit être démarré). Pas besoin de `php -S`.

Mélange HTML et PHP :

```php
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <title>Ma première page</title>
</head>
<body>
    <h1><?php echo "Titre généré par PHP"; ?></h1>
    <p>Nous sommes le <?php echo date("d/m/Y"); ?>.</p>
</body>
</html>
```

### 3. Explication

* `<?php` ouvre le code PHP. Dans un fichier 100 % PHP, on ne ferme pas avec `?>`.
* `echo` affiche du texte.
* Chaque instruction se termine par un **point-virgule** `;`.
* `date("d/m/Y")` est une fonction PHP qui renvoie la date du jour.
* `php -S localhost:8000` lance un serveur de développement (jamais pour la production).

### 4. Petit exercice

Crée `exercice1.php` qui affiche ton prénom, puis sur une autre ligne l'année en cours (utilise `date("Y")`).

### 5. Correction

```php
<?php

echo "Je m'appelle Sarah.<br>";
echo "Nous sommes en " . date("Y");
```

Ouvre `http://localhost:8000/exercice1.php` (ou `http://localhost/cours/exercice1.php` avec XAMPP/WAMP). Le `<br>` fait un retour à la ligne en HTML, et le point `.` colle deux textes ensemble.

### À retenir

* PHP s'exécute côté serveur et génère du HTML.
* `php fichier.php` exécute un script dans le terminal.
* `php -S localhost:8000` lance un serveur local.
* `echo` affiche du texte, et chaque instruction finit par `;`.
* Avec XAMPP/WAMP, tes fichiers vont dans `htdocs` / `www` et s'ouvrent via `http://localhost/...`.

---

## Chapitre 2 : Les bases

### Ce que tu vas apprendre

Tu vas stocker des informations dans des variables, découvrir les types principaux et manipuler du texte. Tu feras aussi tes premiers calculs.

### 1. Notion

Une **variable** commence par `$` et stocke une valeur. PHP devine le type tout seul. Les types de base sont :

| Type | Exemple |
|------|---------|
| `string` (texte) | `"Bonjour"` |
| `int` (entier) | `42` |
| `float` (décimal) | `19.99` |
| `bool` (vrai/faux) | `true`, `false` |

### 2. Exemple

```php
<?php

$prenom = "Jean";
$age = 25;
$prix = 19.99;
$estMajeur = true;

// Interpolation : les variables dans des guillemets doubles sont remplacées
echo "Je m'appelle $prenom et j'ai $age ans.\n";

// Concaténation avec le point
echo "Bonjour " . $prenom . "\n";

// Opérateurs
$total = $prix * 3;
echo "Total : $total €\n";
echo "Reste de la division : " . (10 % 3) . "\n";

// Quelques fonctions sur les chaînes
echo strtoupper($prenom) . "\n";   // JEAN
echo strlen($prenom) . "\n";       // 4

// Voir le type d'une variable
var_dump($age);        // int(25)
var_dump($estMajeur);  // bool(true)
```

### 3. Explication

* `"...$prenom..."` (guillemets doubles) remplace la variable par sa valeur. Avec des guillemets simples `'...'`, rien n'est remplacé.
* `.` concatène (colle) des textes.
* `+ - * / %` sont les opérateurs de calcul (`%` = reste de la division).
* `\n` est un retour à la ligne dans le terminal (en HTML, utilise `<br>`).
* `var_dump()` affiche le type et la valeur : très utile pour déboguer.
* Pour des expressions, utilise les accolades : `"Prix : {$produit['prix']} €"`.

### 4. Petit exercice

Crée un script avec deux variables : `$prixHT` (50) et `$tva` (20). Calcule et affiche le prix TTC.

### 5. Correction

```php
<?php

$prixHT = 50;
$tva = 20;

$prixTTC = $prixHT + ($prixHT * $tva / 100);

echo "Prix HT : $prixHT €\n";
echo "Prix TTC : $prixTTC €\n";
```

Résultat : `Prix TTC : 60 €`.

### À retenir

* Une variable commence par `$`.
* Types de base : `string`, `int`, `float`, `bool`.
* Guillemets doubles = interpolation ; guillemets simples = texte brut.
* `.` concatène, `var_dump()` aide à comprendre ce qu'on manipule.

---

## Chapitre 3 : Conditions et boucles

### Ce que tu vas apprendre

Tu vas faire prendre des décisions à ton programme et répéter des actions. Ce sont les deux briques de toute logique de programmation.

### 1. Notion

* `if / elseif / else` : exécuter du code selon une condition.
* `match` : choisir une valeur selon un cas (version moderne de `switch`).
* `for`, `while`, `foreach` : répéter du code.

Comparaisons : `==` (égal), `===` (égal et même type, à privilégier), `!=`, `<`, `>`, `<=`, `>=`. Combinaisons : `&&` (et), `||` (ou), `!` (non).

### 2. Exemple

```php
<?php

$note = 14;

if ($note >= 16) {
    echo "Très bien\n";
} elseif ($note >= 10) {
    echo "Reçu\n";
} else {
    echo "Recalé\n";
}

// match
$jour = 3;
$nomJour = match ($jour) {
    1 => "Lundi",
    2 => "Mardi",
    3 => "Mercredi",
    default => "Autre jour",
};
echo "Jour : $nomJour\n";

// for : compter de 1 à 5
for ($i = 1; $i <= 5; $i++) {
    echo "Tour numéro $i\n";
}

// while : tant que la condition est vraie
$compteur = 3;
while ($compteur > 0) {
    echo "Compte à rebours : $compteur\n";
    $compteur--;
}

// foreach : parcourir une liste
$fruits = ["pomme", "banane", "cerise"];
foreach ($fruits as $fruit) {
    echo "J'aime la $fruit\n";
}
```

### 3. Explication

* `if (...) { ... }` : le code entre accolades s'exécute si la condition est vraie.
* `match` renvoie une valeur ; `default` couvre tous les autres cas.
* `for ($i = 1; $i <= 5; $i++)` : départ, condition, incrément.
* `$compteur--` retire 1 à la variable. Sans cela, le `while` tournerait à l'infini.
* `foreach` est la boucle la plus utilisée pour parcourir des listes (chapitre suivant).

### 4. Petit exercice

Affiche la table de multiplication de 7 (de 1 à 10). Puis, pour chaque nombre de 1 à 10, affiche s'il est pair ou impair.

### 5. Correction

```php
<?php

for ($i = 1; $i <= 10; $i++) {
    echo "7 x $i = " . (7 * $i) . "\n";
}

echo "\n";

for ($i = 1; $i <= 10; $i++) {
    if ($i % 2 === 0) {
        echo "$i est pair\n";
    } else {
        echo "$i est impair\n";
    }
}
```

### À retenir

* `if / elseif / else` pour décider, `match` pour choisir une valeur.
* Utilise `===` plutôt que `==`.
* `for` quand on sait combien de fois répéter, `while` quand on attend une condition.
* `foreach` parcourt les tableaux.

---

## Chapitre 4 : Fonctions et tableaux

### Ce que tu vas apprendre

Tu vas regrouper du code réutilisable dans des fonctions et organiser des données dans des tableaux. C'est la base pour manipuler des listes d'utilisateurs, de produits ou de tâches.

### 1. Notion

Une **fonction** est un bloc de code nommé qu'on peut appeler plusieurs fois. Elle reçoit des **paramètres** et peut **retourner** un résultat.

Un **tableau** stocke plusieurs valeurs. Un **tableau associatif** associe une clé à chaque valeur.

### 2. Exemple

```php
<?php

declare(strict_types=1);

// Fonction avec paramètres typés et valeur de retour
function additionner(int $a, int $b): int
{
    return $a + $b;
}

echo additionner(3, 4) . "\n"; // 7

// Paramètre avec valeur par défaut
function saluer(string $nom, string $salutation = "Bonjour"): string
{
    return "$salutation $nom !";
}

echo saluer("Léa") . "\n";
echo saluer("Tom", "Salut") . "\n";

// Tableau simple
$couleurs = ["rouge", "vert", "bleu"];
$couleurs[] = "jaune"; // ajouter un élément
echo $couleurs[0] . "\n";       // rouge
echo count($couleurs) . "\n";   // 4

// Tableau associatif
$utilisateur = [
    "nom" => "Durand",
    "prenom" => "Marie",
    "age" => 30,
];
echo $utilisateur["prenom"] . " a " . $utilisateur["age"] . " ans\n";

// Tableau de tableaux
$produits = [
    ["nom" => "Stylo", "prix" => 1.5],
    ["nom" => "Cahier", "prix" => 3.0],
    ["nom" => "Sac", "prix" => 25.0],
];

foreach ($produits as $produit) {
    echo $produit["nom"] . " : " . $produit["prix"] . " €\n";
}

// foreach avec clé et valeur
foreach ($utilisateur as $cle => $valeur) {
    echo "$cle = $valeur\n";
}
```

### 3. Explication

* `function nom(type $param): type` : on déclare les types pour éviter des erreurs.
* `declare(strict_types=1);` (tout en haut du fichier) rend PHP strict sur les types.
* `return` renvoie le résultat et termine la fonction.
* `$tableau[] = valeur` ajoute à la fin.
* `$tableau["cle"]` lit une valeur par sa clé.
* `count()` donne le nombre d'éléments.

### 4. Petit exercice

Écris une fonction `calculerMoyenne(array $notes): float` qui retourne la moyenne d'un tableau de notes. Teste-la avec `[12, 15, 9, 18]`.

### 5. Correction

```php
<?php

function calculerMoyenne(array $notes): float
{
    return array_sum($notes) / count($notes);
}

$notes = [12, 15, 9, 18];
echo "Moyenne : " . calculerMoyenne($notes) . "\n"; // 13.5
```

### À retenir

* Une fonction se déclare avec `function`, reçoit des paramètres et retourne avec `return`.
* `[]` crée un tableau ; `"cle" => valeur` crée un tableau associatif.
* `foreach ($tableau as $cle => $valeur)` parcourt tout.
* `count()` et `array_sum()` sont deux fonctions de tableau très utiles.

---

## Chapitre 5 : Formulaires

### Ce que tu vas apprendre

Tu vas créer un formulaire HTML, récupérer ce que l'utilisateur saisit avec PHP, le valider et afficher le résultat. C'est ce qui rend un site vraiment dynamique.

### 1. Notion

* **GET** : les données passent dans l'URL (`?nom=Jean`). Utile pour la recherche ou les filtres.
* **POST** : les données passent dans le corps de la requête. À utiliser pour envoyer, créer ou modifier.

PHP les récupère dans `$_GET` et `$_POST` (des tableaux associatifs).

**Règle d'or :** ne fais jamais confiance à ce que l'utilisateur envoie, et échappe toujours ce que tu affiches avec `htmlspecialchars()`.

### 2. Exemple

Fichier `formulaire.php` (le formulaire et son traitement dans le même fichier) :

```php
<?php

$erreurs = [];
$nom = "";
$email = "";
$succes = false;

if ($_SERVER["REQUEST_METHOD"] === "POST") {
    $nom = trim($_POST["nom"] ?? "");
    $email = trim($_POST["email"] ?? "");

    if ($nom === "") {
        $erreurs[] = "Le nom est obligatoire.";
    }

    if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
        $erreurs[] = "L'email n'est pas valide.";
    }

    if (count($erreurs) === 0) {
        $succes = true;
    }
}
?>
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <title>Inscription</title>
</head>
<body>
    <h1>Inscription</h1>

    <?php if ($succes): ?>
        <p style="color: green;">
            Merci <?= htmlspecialchars($nom) ?>, nous vous écrirons à
            <?= htmlspecialchars($email) ?>.
        </p>
    <?php endif; ?>

    <?php foreach ($erreurs as $erreur): ?>
        <p style="color: red;"><?= htmlspecialchars($erreur) ?></p>
    <?php endforeach; ?>

    <form method="post">
        <p>
            <label>Nom :
                <input type="text" name="nom" value="<?= htmlspecialchars($nom) ?>">
            </label>
        </p>
        <p>
            <label>Email :
                <input type="text" name="email" value="<?= htmlspecialchars($email) ?>">
            </label>
        </p>
        <button type="submit">Envoyer</button>
    </form>
</body>
</html>
```

Teste avec un nom vide, un mauvais email, puis des données correctes.

### 3. Explication

* `$_SERVER["REQUEST_METHOD"] === "POST"` : on traite seulement si le formulaire a été envoyé.
* `$_POST["nom"] ?? ""` : si la clé n'existe pas, on prend `""` (opérateur `??`).
* `trim()` supprime les espaces au début et à la fin.
* `filter_var(..., FILTER_VALIDATE_EMAIL)` vérifie un email.
* `<?= ... ?>` est un raccourci pour `<?php echo ... ?>`.
* `htmlspecialchars()` protège contre l'injection de HTML/JavaScript (faille XSS).
* On remet les valeurs dans les champs (`value="..."`) pour que l'utilisateur ne retape pas tout en cas d'erreur.

### 4. Petit exercice

Crée un formulaire « Calculatrice » avec deux nombres et une opération (addition ou multiplication, via une liste `<select>`). Affiche le résultat avec `match`. Refuse les valeurs non numériques.

### 5. Correction

```php
<?php

$resultat = null;
$erreur = "";

if ($_SERVER["REQUEST_METHOD"] === "POST") {
    $a = $_POST["a"] ?? "";
    $b = $_POST["b"] ?? "";
    $operation = $_POST["operation"] ?? "";

    if (!is_numeric($a) || !is_numeric($b)) {
        $erreur = "Entre deux nombres valides.";
    } else {
        $resultat = match ($operation) {
            "addition" => $a + $b,
            "multiplication" => $a * $b,
            default => null,
        };

        if ($resultat === null) {
            $erreur = "Opération inconnue.";
        }
    }
}
?>
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <title>Calculatrice</title>
</head>
<body>
    <h1>Calculatrice</h1>

    <?php if ($erreur !== ""): ?>
        <p style="color: red;"><?= htmlspecialchars($erreur) ?></p>
    <?php endif; ?>

    <?php if ($resultat !== null): ?>
        <p>Résultat : <strong><?= htmlspecialchars((string) $resultat) ?></strong></p>
    <?php endif; ?>

    <form method="post">
        <input type="text" name="a" placeholder="Nombre 1">
        <select name="operation">
            <option value="addition">+</option>
            <option value="multiplication">×</option>
        </select>
        <input type="text" name="b" placeholder="Nombre 2">
        <button type="submit">Calculer</button>
    </form>
</body>
</html>
```

### À retenir

* `$_GET` et `$_POST` contiennent les données envoyées.
* Toujours valider (`trim`, `filter_var`, `is_numeric`…) côté serveur.
* Toujours échapper l'affichage avec `htmlspecialchars()`.
* `??` donne une valeur par défaut quand une clé n'existe pas.

---

## Chapitre 6 : Fichiers et JSON

### Ce que tu vas apprendre

Tu vas lire et écrire dans un fichier, puis utiliser le format JSON pour sauvegarder des données structurées. Cela permet de créer de petites applications sans base de données.

### 1. Notion

* `file_put_contents()` écrit dans un fichier (le crée s'il n'existe pas).
* `file_get_contents()` lit tout le contenu.
* **JSON** est un format texte pour représenter des données. `json_encode()` transforme un tableau PHP en JSON, `json_decode()` fait l'inverse.

### 2. Exemple

```php
<?php

// Écrire dans un fichier
file_put_contents("note.txt", "Ma première note\n");

// Ajouter à la fin sans écraser
file_put_contents("note.txt", "Deuxième ligne\n", FILE_APPEND);

// Lire le fichier
echo file_get_contents("note.txt");

// JSON : tableau -> texte
$personne = ["nom" => "Martin", "age" => 28, "langages" => ["PHP", "JS"]];
$json = json_encode($personne, JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE);
file_put_contents("personne.json", $json);

// JSON : texte -> tableau
$contenu = file_get_contents("personne.json");
$donnees = json_decode($contenu, true);
echo $donnees["nom"] . " connaît " . implode(", ", $donnees["langages"]) . "\n";
```

Exemple concret : un mini livre d'or (`livre-or.php`).

```php
<?php

$fichier = "messages.json";

// Charger les messages existants
$messages = [];
if (file_exists($fichier)) {
    $messages = json_decode(file_get_contents($fichier), true) ?? [];
}

// Ajouter un message
if ($_SERVER["REQUEST_METHOD"] === "POST") {
    $auteur = trim($_POST["auteur"] ?? "");
    $texte = trim($_POST["texte"] ?? "");

    if ($auteur !== "" && $texte !== "") {
        $messages[] = [
            "auteur" => $auteur,
            "texte" => $texte,
            "date" => date("d/m/Y H:i"),
        ];
        file_put_contents(
            $fichier,
            json_encode($messages, JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE)
        );
        header("Location: livre-or.php"); // évite le renvoi du formulaire au rafraîchissement
        exit;
    }
}
?>
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <title>Livre d'or</title>
</head>
<body>
    <h1>Livre d'or</h1>

    <form method="post">
        <input type="text" name="auteur" placeholder="Ton nom">
        <br>
        <textarea name="texte" placeholder="Ton message"></textarea>
        <br>
        <button type="submit">Publier</button>
    </form>

    <?php foreach (array_reverse($messages) as $message): ?>
        <p>
            <strong><?= htmlspecialchars($message["auteur"]) ?></strong>
            (<?= $message["date"] ?>)<br>
            <?= nl2br(htmlspecialchars($message["texte"])) ?>
        </p>
    <?php endforeach; ?>
</body>
</html>
```

### 3. Explication

* `FILE_APPEND` ajoute à la suite au lieu d'écraser.
* `json_decode($texte, true)` : le `true` donne un tableau associatif.
* `file_exists()` évite une erreur si le fichier n'existe pas encore.
* `header("Location: ...")` + `exit` : **redirection après POST**, pour éviter de renvoyer le formulaire en actualisant la page.
* `array_reverse()` montre les messages les plus récents d'abord.
* `nl2br()` transforme les retours à la ligne en `<br>`.

### 4. Petit exercice

Crée un compteur de visites : à chaque chargement de la page, lis un nombre dans `compteur.txt`, ajoute 1, sauvegarde-le et affiche « Tu es le visiteur n° X ».

### 5. Correction

```php
<?php

$fichier = "compteur.txt";

$visites = file_exists($fichier) ? (int) file_get_contents($fichier) : 0;
$visites++;

file_put_contents($fichier, (string) $visites);

echo "Tu es le visiteur n° $visites";
```

### À retenir

* `file_put_contents()` écrit, `file_get_contents()` lit.
* `json_encode()` et `json_decode($texte, true)` convertissent tableau ↔ JSON.
* Toujours tester `file_exists()` avant de lire.
* Redirige après un POST réussi avec `header("Location: ...")` puis `exit`.

---

## Chapitre 7 : MySQL avec PDO

### Ce que tu vas apprendre

Tu vas te connecter à une base MySQL avec PDO et réaliser les quatre opérations de base : créer, lire, modifier, supprimer (CRUD). Tu apprendras aussi à te protéger des injections SQL.

### 1. Notion

**PDO** est l'outil standard de PHP pour parler à une base de données. Les **requêtes préparées** (`prepare` + `execute`) envoient les valeurs séparément du SQL : c'est indispensable pour éviter les injections SQL.

**Préparation :** il faut un serveur MySQL (ou MariaDB) en marche.

* **XAMPP** : dans le *Control Panel*, clique sur **Start** pour MySQL (et pour Apache si tu veux utiliser phpMyAdmin).
* **WAMP** : lance *WampServer* et attends l'icône **verte**.
* **Autre** (MAMP, Docker, installation directe) : démarre ton serveur MySQL/MariaDB.

**Identifiants par défaut de XAMPP et WAMP :** utilisateur `root`, mot de passe **vide** (c'est ce que nous utilisons dans les exemples). Avec MAMP, le mot de passe est souvent `root`.

> Cette configuration convient pour apprendre en local, **jamais** pour un site en ligne.

**Créer la base avec phpMyAdmin (XAMPP/WAMP) :**

1. Ouvre `http://localhost/phpmyadmin`.
2. Clique sur **Nouvelle base de données**.
3. Nom : `cours_php`, interclassement : `utf8mb4_unicode_ci`, puis **Créer**.

Pour exécuter du SQL dans phpMyAdmin (créer une table, par exemple) : sélectionne ta base dans la colonne de gauche, ouvre l'onglet **SQL**, colle ta requête et clique sur **Exécuter**.

**Ou créer la base en SQL** (terminal MySQL ou onglet SQL de phpMyAdmin) :

```sql
CREATE DATABASE cours_php CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

Vérifie que l'extension est active : `php -m | grep pdo_mysql` (sur Windows : `php -m | findstr pdo_mysql`). Avec XAMPP et WAMP, elle est **activée par défaut**.

**Erreurs courantes :**

| Message | Cause probable |
|---|---|
| `could not find driver` | L'extension `pdo_mysql` n'est pas activée. Dans `php.ini`, enlève le `;` devant `extension=pdo_mysql` (XAMPP : `C:\xampp\php\php.ini` ; WAMP : menu *PHP → Extensions PHP*), puis redémarre Apache. |
| `Access denied for user 'root'` | Mauvais mot de passe : essaie `""` (XAMPP/WAMP) ou `"root"` (MAMP). |
| `Connection refused` / `No such file` | MySQL n'est pas démarré, ou il utilise un autre port (ajoute `;port=3307` dans le DSN). |
| `Unknown database 'cours_php'` | La base n'a pas été créée : fais-le dans phpMyAdmin. |

> **Attention avec WAMP :** la commande `php` du terminal peut utiliser un `php.ini` différent de celui d'Apache. Si un script marche dans le navigateur mais pas en terminal (ou l'inverse), c'est souvent la cause.
>
> **Astuce :** avec XAMPP/WAMP, tu peux lancer les exemples du chapitre 7 dans le navigateur (`http://localhost/cours/pdo.php`) au lieu du terminal. Pense alors à remplacer les `\n` par `<br>` ou à entourer l'affichage de `<pre>...</pre>`.

### 2. Exemple

Fichier `pdo.php` :

```php
<?php

// 1. Connexion
$pdo = new PDO(
    "mysql:host=localhost;dbname=cours_php;charset=utf8mb4",
    "root",   // adapte à ta configuration
    "",       // mot de passe (souvent vide en local)
    [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    ]
);

// 2. Créer une table
$pdo->exec("
    CREATE TABLE IF NOT EXISTS contacts (
        id INT AUTO_INCREMENT PRIMARY KEY,
        nom VARCHAR(100) NOT NULL,
        email VARCHAR(150) NOT NULL
    )
");

// 3. INSERT
$stmt = $pdo->prepare("INSERT INTO contacts (nom, email) VALUES (:nom, :email)");
$stmt->execute(["nom" => "Alice", "email" => "alice@example.com"]);
$stmt->execute(["nom" => "Bob", "email" => "bob@example.com"]);
echo "Dernier id inséré : " . $pdo->lastInsertId() . "\n";

// 4. SELECT (plusieurs lignes)
$stmt = $pdo->query("SELECT * FROM contacts");
foreach ($stmt->fetchAll() as $contact) {
    echo $contact["id"] . " - " . $contact["nom"] . " (" . $contact["email"] . ")\n";
}

// 5. SELECT d'une seule ligne
$stmt = $pdo->prepare("SELECT * FROM contacts WHERE id = :id");
$stmt->execute(["id" => 1]);
$contact = $stmt->fetch();
echo "Contact n°1 : " . $contact["nom"] . "\n";

// 6. UPDATE
$stmt = $pdo->prepare("UPDATE contacts SET nom = :nom WHERE id = :id");
$stmt->execute(["nom" => "Alice Martin", "id" => 1]);

// 7. DELETE
$stmt = $pdo->prepare("DELETE FROM contacts WHERE id = :id");
$stmt->execute(["id" => 2]);
echo "Contacts supprimés : " . $stmt->rowCount() . "\n";
```

Lance avec `php pdo.php`.

### 3. Explication

* Le DSN `mysql:host=...;dbname=...;charset=utf8mb4` décrit la base.
* `ERRMODE_EXCEPTION` fait que les erreurs SQL sont visibles au lieu d'échouer en silence.
* `FETCH_ASSOC` renvoie les lignes sous forme de tableaux associatifs.
* `:nom`, `:email` sont des **paramètres nommés** : jamais de variable collée directement dans le SQL !
* `fetchAll()` renvoie toutes les lignes, `fetch()` une seule (ou `false` s'il n'y en a pas).
* `rowCount()` donne le nombre de lignes touchées.

**Ne fais jamais ça :**

```php
// DANGEREUX : injection SQL possible
$pdo->query("SELECT * FROM contacts WHERE id = " . $_GET["id"]);
```

### 4. Petit exercice

Crée une table `produits` (`id`, `nom`, `prix`). Insère 3 produits, affiche uniquement ceux dont le prix est supérieur à 10, puis augmente tous les prix de 10 %.

### 5. Correction

```php
<?php

$pdo = new PDO(
    "mysql:host=localhost;dbname=cours_php;charset=utf8mb4",
    "root",
    "",
    [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    ]
);

$pdo->exec("
    CREATE TABLE IF NOT EXISTS produits (
        id INT AUTO_INCREMENT PRIMARY KEY,
        nom VARCHAR(100) NOT NULL,
        prix DECIMAL(8,2) NOT NULL
    )
");

$stmt = $pdo->prepare("INSERT INTO produits (nom, prix) VALUES (:nom, :prix)");
$stmt->execute(["nom" => "Stylo", "prix" => 2.50]);
$stmt->execute(["nom" => "Clavier", "prix" => 45.00]);
$stmt->execute(["nom" => "Souris", "prix" => 19.90]);

$stmt = $pdo->prepare("SELECT * FROM produits WHERE prix > :min");
$stmt->execute(["min" => 10]);
foreach ($stmt->fetchAll() as $produit) {
    echo $produit["nom"] . " : " . $produit["prix"] . " €\n";
}

$pdo->exec("UPDATE produits SET prix = prix * 1.10");
echo "Prix mis à jour.\n";
```

### À retenir

* PDO se connecte avec un DSN, un utilisateur et un mot de passe.
* Utilise **toujours** `prepare()` + `execute()` avec des paramètres.
* `fetchAll()` pour une liste, `fetch()` pour une ligne.
* CRUD = `INSERT`, `SELECT`, `UPDATE`, `DELETE`.

---

## Chapitre 8 : Sessions et authentification

### Ce que tu vas apprendre

Tu vas garder des informations d'une page à l'autre avec les sessions, puis créer un système d'inscription, de connexion et de déconnexion sécurisé avec des mots de passe hachés.

### 1. Notion

Le web « oublie » tout entre deux pages. Une **session** permet au serveur de mémoriser des données pour un visiteur (via un cookie). Elles sont stockées dans `$_SESSION`, après `session_start()`.

Un mot de passe ne se stocke **jamais en clair** :

* `password_hash($mdp, PASSWORD_DEFAULT)` crée une empreinte sécurisée ;
* `password_verify($mdp, $hash)` vérifie un mot de passe.

### 2. Exemple

Petit test de session (`session.php`) :

```php
<?php

session_start();

if (!isset($_SESSION["visites"])) {
    $_SESSION["visites"] = 0;
}

$_SESSION["visites"]++;

echo "Tu as chargé cette page " . $_SESSION["visites"] . " fois.";
```

Recharge la page plusieurs fois : le compteur monte.

Test du hachage :

```php
<?php

$hash = password_hash("secret123", PASSWORD_DEFAULT);
echo $hash . "\n"; // différent à chaque exécution

var_dump(password_verify("secret123", $hash)); // bool(true)
var_dump(password_verify("mauvais", $hash));   // bool(false)
```

**Système d'authentification complet.** Crée d'abord la table :

```sql
CREATE TABLE IF NOT EXISTS users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(150) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL
);
```

`db.php` (connexion réutilisable) :

```php
<?php

$pdo = new PDO(
    "mysql:host=localhost;dbname=cours_php;charset=utf8mb4",
    "root",
    "",
    [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    ]
);
```

`inscription.php` :

```php
<?php

require "db.php";

$erreur = "";

if ($_SERVER["REQUEST_METHOD"] === "POST") {
    $email = trim($_POST["email"] ?? "");
    $mdp = $_POST["password"] ?? "";

    if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
        $erreur = "Email invalide.";
    } elseif (strlen($mdp) < 8) {
        $erreur = "Le mot de passe doit faire au moins 8 caractères.";
    } else {
        $stmt = $pdo->prepare("SELECT id FROM users WHERE email = :email");
        $stmt->execute(["email" => $email]);

        if ($stmt->fetch()) {
            $erreur = "Cet email est déjà utilisé.";
        } else {
            $stmt = $pdo->prepare("INSERT INTO users (email, password) VALUES (:email, :password)");
            $stmt->execute([
                "email" => $email,
                "password" => password_hash($mdp, PASSWORD_DEFAULT),
            ]);
            header("Location: connexion.php");
            exit;
        }
    }
}
?>
<!DOCTYPE html>
<html lang="fr">
<head><meta charset="UTF-8"><title>Inscription</title></head>
<body>
    <h1>Inscription</h1>
    <?php if ($erreur): ?>
        <p style="color: red;"><?= htmlspecialchars($erreur) ?></p>
    <?php endif; ?>
    <form method="post">
        <p><input type="email" name="email" placeholder="Email" required></p>
        <p><input type="password" name="password" placeholder="Mot de passe" required></p>
        <button type="submit">Créer mon compte</button>
    </form>
    <p><a href="connexion.php">J'ai déjà un compte</a></p>
</body>
</html>
```

`connexion.php` :

```php
<?php

session_start();
require "db.php";

$erreur = "";

if ($_SERVER["REQUEST_METHOD"] === "POST") {
    $email = trim($_POST["email"] ?? "");
    $mdp = $_POST["password"] ?? "";

    $stmt = $pdo->prepare("SELECT * FROM users WHERE email = :email");
    $stmt->execute(["email" => $email]);
    $user = $stmt->fetch();

    if ($user && password_verify($mdp, $user["password"])) {
        session_regenerate_id(true);
        $_SESSION["user_id"] = $user["id"];
        $_SESSION["user_email"] = $user["email"];
        header("Location: profil.php");
        exit;
    }

    $erreur = "Email ou mot de passe incorrect.";
}
?>
<!DOCTYPE html>
<html lang="fr">
<head><meta charset="UTF-8"><title>Connexion</title></head>
<body>
    <h1>Connexion</h1>
    <?php if ($erreur): ?>
        <p style="color: red;"><?= htmlspecialchars($erreur) ?></p>
    <?php endif; ?>
    <form method="post">
        <p><input type="email" name="email" placeholder="Email" required></p>
        <p><input type="password" name="password" placeholder="Mot de passe" required></p>
        <button type="submit">Se connecter</button>
    </form>
    <p><a href="inscription.php">Créer un compte</a></p>
</body>
</html>
```

`profil.php` (page protégée) :

```php
<?php

session_start();

if (!isset($_SESSION["user_id"])) {
    header("Location: connexion.php");
    exit;
}
?>
<!DOCTYPE html>
<html lang="fr">
<head><meta charset="UTF-8"><title>Profil</title></head>
<body>
    <h1>Bienvenue <?= htmlspecialchars($_SESSION["user_email"]) ?></h1>
    <p><a href="deconnexion.php">Se déconnecter</a></p>
</body>
</html>
```

`deconnexion.php` :

```php
<?php

session_start();
$_SESSION = [];
session_destroy();

header("Location: connexion.php");
exit;
```

### 3. Explication

* `session_start()` doit être appelé **avant tout affichage**, sur chaque page qui utilise la session.
* À la connexion, on enregistre l'identité dans `$_SESSION` ; pour protéger une page, on vérifie sa présence.
* Même message d'erreur pour « email inconnu » et « mauvais mot de passe » : on ne révèle pas quels emails existent.
* `session_regenerate_id(true)` change l'identifiant de session à la connexion (protection contre le vol de session).
* `session_destroy()` + vider `$_SESSION` = déconnexion.

### 4. Petit exercice

Modifie `profil.php` pour afficher aussi la date d'inscription. Pour cela, ajoute d'abord une colonne `created_at` à la table, puis récupère l'utilisateur en base avec son `id` de session.

### 5. Correction

```sql
ALTER TABLE users ADD COLUMN created_at DATETIME DEFAULT CURRENT_TIMESTAMP;
```

```php
<?php

session_start();
require "db.php";

if (!isset($_SESSION["user_id"])) {
    header("Location: connexion.php");
    exit;
}

$stmt = $pdo->prepare("SELECT email, created_at FROM users WHERE id = :id");
$stmt->execute(["id" => $_SESSION["user_id"]]);
$user = $stmt->fetch();
?>
<!DOCTYPE html>
<html lang="fr">
<head><meta charset="UTF-8"><title>Profil</title></head>
<body>
    <h1>Bienvenue <?= htmlspecialchars($user["email"]) ?></h1>
    <p>Inscrit le <?= htmlspecialchars($user["created_at"]) ?></p>
    <p><a href="deconnexion.php">Se déconnecter</a></p>
</body>
</html>
```

### À retenir

* `session_start()` puis `$_SESSION` pour mémoriser des données entre pages.
* `password_hash()` pour stocker, `password_verify()` pour contrôler.
* Protège une page en testant `$_SESSION` et en redirigeant sinon.
* Déconnexion = vider `$_SESSION` et `session_destroy()`.

---

## Chapitre 9 : Organisation d'un petit projet

### Ce que tu vas apprendre

Tu vas ranger ton code pour qu'il reste lisible quand il grossit : fichiers séparés, HTML séparé de la logique, et une première idée très simple du MVC.

### 1. Notion

Quand tout est dans un seul fichier, ça devient vite illisible. Deux outils simples :

* `require` : inclut un autre fichier PHP (erreur fatale s'il manque).
* **Séparer la logique et l'affichage** : d'abord PHP (données, traitement), ensuite HTML (affichage).

**MVC en une phrase :**

* **M**odèle : le code qui parle à la base de données ;
* **V**ue : le HTML affiché ;
* **C**ontrôleur : le code qui reçoit la demande, appelle le modèle et choisit la vue.

Structure conseillée :

```
mon-projet/
├── config.php          # paramètres (base de données)
├── db.php              # connexion PDO
├── functions.php       # fonctions du modèle
├── index.php           # contrôleur de la page d'accueil
├── views/
│   ├── header.php
│   ├── footer.php
│   └── liste.php       # vue
└── public/
    └── style.css
```

### 2. Exemple

`config.php` :

```php
<?php

return [
    "db_host" => "localhost",
    "db_name" => "cours_php",
    "db_user" => "root",
    "db_pass" => "",
];
```

`db.php` :

```php
<?php

$config = require __DIR__ . "/config.php";

$pdo = new PDO(
    "mysql:host={$config['db_host']};dbname={$config['db_name']};charset=utf8mb4",
    $config["db_user"],
    $config["db_pass"],
    [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    ]
);
```

`functions.php` (le modèle) :

```php
<?php

function getContacts(PDO $pdo): array
{
    return $pdo->query("SELECT * FROM contacts ORDER BY nom")->fetchAll();
}

function e(string $texte): string
{
    return htmlspecialchars($texte, ENT_QUOTES, "UTF-8");
}
```

`index.php` (le contrôleur) :

```php
<?php

require __DIR__ . "/db.php";
require __DIR__ . "/functions.php";

$contacts = getContacts($pdo);
$titre = "Mes contacts";

require __DIR__ . "/views/liste.php";
```

`views/liste.php` (la vue) :

```php
<?php require __DIR__ . "/header.php"; ?>

<h1><?= e($titre) ?></h1>

<ul>
    <?php foreach ($contacts as $contact): ?>
        <li><?= e($contact["nom"]) ?> (<?= e($contact["email"]) ?>)</li>
    <?php endforeach; ?>
</ul>

<?php require __DIR__ . "/footer.php"; ?>
```

`views/header.php` :

```php
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <title><?= e($titre ?? "Mon site") ?></title>
</head>
<body>
```

`views/footer.php` :

```php
</body>
</html>
```

### 3. Explication

* `__DIR__` est le dossier du fichier courant : les chemins fonctionnent d'où que tu lances le script.
* `return [...]` dans `config.php` permet de récupérer le tableau avec `$config = require ...`.
* `e()` est un raccourci pour ne pas répéter `htmlspecialchars` partout.
* Le contrôleur prépare les **variables** (`$contacts`, `$titre`) que la vue utilise directement.
* La vue ne fait presque pas de logique : seulement `if`, `foreach` et `echo`.

**Bonnes pratiques essentielles :**

* Échappe toujours l'affichage, prépare toujours les requêtes SQL.
* Ne mets jamais de mots de passe en dur dans du code partagé (utilise un fichier de config ignoré par Git).
* Un fichier = une responsabilité.
* Nomme clairement : `getContacts()`, `ajouterTache()`.
* Valide les données côté serveur, même si le HTML les contrôle déjà.

### 4. Petit exercice

Ajoute une fonction `ajouterContact(PDO $pdo, string $nom, string $email): void` dans `functions.php`, puis crée `ajouter.php` (contrôleur) et `views/ajouter.php` (vue avec le formulaire).

### 5. Correction

Dans `functions.php` :

```php
function ajouterContact(PDO $pdo, string $nom, string $email): void
{
    $stmt = $pdo->prepare("INSERT INTO contacts (nom, email) VALUES (:nom, :email)");
    $stmt->execute(["nom" => $nom, "email" => $email]);
}
```

`ajouter.php` :

```php
<?php

require __DIR__ . "/db.php";
require __DIR__ . "/functions.php";

$titre = "Ajouter un contact";
$erreurs = [];
$nom = "";
$email = "";

if ($_SERVER["REQUEST_METHOD"] === "POST") {
    $nom = trim($_POST["nom"] ?? "");
    $email = trim($_POST["email"] ?? "");

    if ($nom === "") {
        $erreurs[] = "Le nom est obligatoire.";
    }
    if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
        $erreurs[] = "Email invalide.";
    }

    if (!$erreurs) {
        ajouterContact($pdo, $nom, $email);
        header("Location: index.php");
        exit;
    }
}

require __DIR__ . "/views/ajouter.php";
```

`views/ajouter.php` :

```php
<?php require __DIR__ . "/header.php"; ?>

<h1><?= e($titre) ?></h1>

<?php foreach ($erreurs as $erreur): ?>
    <p style="color: red;"><?= e($erreur) ?></p>
<?php endforeach; ?>

<form method="post">
    <p><input type="text" name="nom" value="<?= e($nom) ?>" placeholder="Nom"></p>
    <p><input type="text" name="email" value="<?= e($email) ?>" placeholder="Email"></p>
    <button type="submit">Ajouter</button>
</form>

<?php require __DIR__ . "/footer.php"; ?>
```

### À retenir

* `require` + `__DIR__` pour découper ton code en fichiers.
* Contrôleur (logique) → Modèle (base de données) → Vue (HTML).
* Une fonction utilitaire `e()` simplifie l'échappement.
* Structure claire = projet maintenable.

---

## Chapitre 10 : Projet final, gestionnaire de tâches (Todo List)

### Ce que tu vas apprendre

Tu vas construire une application complète, étape par étape, en réutilisant tout ce que tu as appris : formulaires, validation, PDO, organisation en fichiers. À la fin, tu pourras ajouter, afficher, modifier, terminer et supprimer des tâches.

### Où placer le projet ?

Choisis **une** des deux méthodes :

| | Méthode 1 : serveur intégré | Méthode 2 : XAMPP / WAMP |
|---|---|---|
| Dossier du projet | n'importe où, par exemple `~/todo` | `htdocs/todo` (XAMPP) ou `www/todo` (WAMP) |
| Démarrage | `php -S localhost:8000` depuis `todo/` | Démarrer **Apache** et **MySQL** |
| Adresse de l'application | `http://localhost:8000/` | `http://localhost/todo/` |
| Base de données | MySQL lancé (XAMPP, WAMP…) | MySQL de XAMPP/WAMP |

Dans les tests ci-dessous, je donne l'adresse de la méthode 1 ; avec XAMPP/WAMP, remplace simplement `http://localhost:8000/` par `http://localhost/todo/`.

Le projet n'utilise que des liens **relatifs** (`index.php`, `style.css`) : il fonctionne donc dans les deux cas sans rien modifier.

### Structure finale

```
todo/
├── config.php
├── db.php
├── functions.php
├── index.php          # liste des tâches
├── ajouter.php        # ajout
├── modifier.php       # modification
├── supprimer.php      # suppression
├── basculer.php       # terminer / rouvrir
├── views/
│   ├── header.php
│   └── footer.php
└── style.css
```

---

### Étape 1 : Créer la base de données

**On va :** créer la base `todo` et la table `taches`.

Avec **XAMPP/WAMP**, ouvre `http://localhost/phpmyadmin`, va dans l'onglet **SQL** (sans sélectionner de base), colle le code ci-dessous et clique sur **Exécuter**. Sinon, utilise le terminal (`mysql -u root -p`) :

```sql
CREATE DATABASE todo CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE todo;

CREATE TABLE taches (
    id INT AUTO_INCREMENT PRIMARY KEY,
    titre VARCHAR(150) NOT NULL,
    terminee TINYINT(1) NOT NULL DEFAULT 0,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

**Explication :** `terminee` vaut `0` (à faire) ou `1` (faite).

**Test :** `SHOW TABLES;` doit afficher `taches`. Dans phpMyAdmin, la base `todo` et sa table `taches` apparaissent dans la colonne de gauche.

---

### Étape 2 : Configuration et connexion

**On va :** créer le dossier `todo/` et les fichiers de connexion.

`config.php` :

```php
<?php

return [
    "db_host" => "localhost",
    "db_name" => "todo",
    "db_user" => "root",
    "db_pass" => "",
];
```

`db.php` :

```php
<?php

$config = require __DIR__ . "/config.php";

try {
    $pdo = new PDO(
        "mysql:host={$config['db_host']};dbname={$config['db_name']};charset=utf8mb4",
        $config["db_user"],
        $config["db_pass"],
        [
            PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
            PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        ]
    );
} catch (PDOException $e) {
    exit("Connexion à la base impossible. Vérifie config.php.");
}
```

**Explication :** `try / catch` attrape l'erreur de connexion et affiche un message simple.

**Test :** crée temporairement `test.php` avec `<?php require "db.php"; echo "Connexion OK";`, puis ouvre `http://localhost:8000/test.php` (après `php -S localhost:8000` dans `todo/`) ou `http://localhost/todo/test.php` (XAMPP/WAMP). Si tu vois une erreur, relis la table de dépannage du chapitre 7. Supprime ensuite `test.php`.

---

### Étape 3 : Les fonctions (le modèle)

**On va :** regrouper tout le code SQL dans un seul fichier.

`functions.php` :

```php
<?php

function e(string $texte): string
{
    return htmlspecialchars($texte, ENT_QUOTES, "UTF-8");
}

function getTaches(PDO $pdo): array
{
    return $pdo->query("SELECT * FROM taches ORDER BY terminee ASC, id DESC")->fetchAll();
}

function getTache(PDO $pdo, int $id): array|false
{
    $stmt = $pdo->prepare("SELECT * FROM taches WHERE id = :id");
    $stmt->execute(["id" => $id]);
    return $stmt->fetch();
}

function ajouterTache(PDO $pdo, string $titre): void
{
    $stmt = $pdo->prepare("INSERT INTO taches (titre) VALUES (:titre)");
    $stmt->execute(["titre" => $titre]);
}

function modifierTache(PDO $pdo, int $id, string $titre): void
{
    $stmt = $pdo->prepare("UPDATE taches SET titre = :titre WHERE id = :id");
    $stmt->execute(["titre" => $titre, "id" => $id]);
}

function basculerTache(PDO $pdo, int $id): void
{
    $stmt = $pdo->prepare("UPDATE taches SET terminee = 1 - terminee WHERE id = :id");
    $stmt->execute(["id" => $id]);
}

function supprimerTache(PDO $pdo, int $id): void
{
    $stmt = $pdo->prepare("DELETE FROM taches WHERE id = :id");
    $stmt->execute(["id" => $id]);
}

function validerTitre(string $titre): ?string
{
    if ($titre === "") {
        return "Le titre est obligatoire.";
    }
    if (mb_strlen($titre) > 150) {
        return "Le titre ne doit pas dépasser 150 caractères.";
    }
    return null; // pas d'erreur
}
```

**Explication :**

* Une fonction par action : ajouter, lire, modifier, basculer, supprimer.
* `1 - terminee` inverse 0 ↔ 1 directement en SQL.
* `validerTitre()` renvoie un message d'erreur, ou `null` si tout va bien (`?string` = texte ou `null`).
* `array|false` : `fetch()` renvoie `false` si la tâche n'existe pas.

**Test :** rien à afficher encore ; on vérifie à l'étape suivante.

---

### Étape 4 : Le gabarit de page et le style

**On va :** créer l'en-tête et le pied de page communs, plus un peu de CSS.

`views/header.php` :

```php
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title><?= e($titre ?? "Todo List") ?></title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
<main>
```

`views/footer.php` :

```php
</main>
</body>
</html>
```

`style.css` :

```css
body {
    font-family: system-ui, sans-serif;
    background: #f4f5f7;
    margin: 0;
}

main {
    max-width: 600px;
    margin: 40px auto;
    background: white;
    padding: 24px;
    border-radius: 8px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

h1 { margin-top: 0; }

ul { list-style: none; padding: 0; }

li {
    display: flex;
    align-items: center;
    gap: 8px;
    padding: 10px 0;
    border-bottom: 1px solid #eee;
}

li .titre { flex: 1; }
li.terminee .titre { text-decoration: line-through; color: #999; }

form.inline { display: inline; margin: 0; }

input[type="text"] {
    padding: 8px;
    width: 100%;
    box-sizing: border-box;
}

button, .bouton {
    padding: 6px 12px;
    border: none;
    border-radius: 4px;
    background: #3b82f6;
    color: white;
    cursor: pointer;
    text-decoration: none;
    font-size: 14px;
}

button.danger { background: #ef4444; }
button.gris, .bouton.gris { background: #6b7280; }

.erreur { color: #b91c1c; background: #fee2e2; padding: 8px; border-radius: 4px; }
```

**Explication :** `header.php` utilise `e()` : il est chargé après `functions.php` par chaque contrôleur. `$titre ?? "Todo List"` prend une valeur par défaut si `$titre` n'est pas défini.

---

### Étape 5 : Afficher les tâches (page d'accueil)

**On va :** créer `index.php`, qui affiche la liste et propose les actions.

`index.php` :

```php
<?php

require __DIR__ . "/db.php";
require __DIR__ . "/functions.php";

$titre = "Ma Todo List";
$taches = getTaches($pdo);

require __DIR__ . "/views/header.php";
?>

<h1>Ma Todo List</h1>

<p><a class="bouton" href="ajouter.php">+ Nouvelle tâche</a></p>

<?php if (count($taches) === 0): ?>
    <p>Aucune tâche pour l'instant. Ajoute la première !</p>
<?php else: ?>
    <ul>
        <?php foreach ($taches as $tache): ?>
            <li class="<?= $tache["terminee"] ? "terminee" : "" ?>">
                <span class="titre"><?= e($tache["titre"]) ?></span>

                <form class="inline" method="post" action="basculer.php">
                    <input type="hidden" name="id" value="<?= (int) $tache["id"] ?>">
                    <button type="submit" class="gris">
                        <?= $tache["terminee"] ? "Rouvrir" : "Terminer" ?>
                    </button>
                </form>

                <a class="bouton" href="modifier.php?id=<?= (int) $tache["id"] ?>">Modifier</a>

                <form class="inline" method="post" action="supprimer.php"
                      onsubmit="return confirm('Supprimer cette tâche ?');">
                    <input type="hidden" name="id" value="<?= (int) $tache["id"] ?>">
                    <button type="submit" class="danger">Supprimer</button>
                </form>
            </li>
        <?php endforeach; ?>
    </ul>
<?php endif; ?>

<?php require __DIR__ . "/views/footer.php"; ?>
```

**Explication :**

* La liste est chargée **avant** l'affichage (logique en haut, HTML en bas).
* Les actions qui **modifient** des données (terminer, supprimer) passent par un formulaire **POST**, pas par un simple lien.
* `(int)` force l'id en nombre entier.
* `onsubmit="return confirm(...)"` demande confirmation avant de supprimer.

**Test :** lance le serveur dans `todo/` (`php -S localhost:8000`) et ouvre `http://localhost:8000` (avec XAMPP/WAMP : `http://localhost/todo/`). Tu vois « Aucune tâche pour l'instant ». Pour vérifier l'affichage, ajoute une ligne à la main dans MySQL : `INSERT INTO taches (titre) VALUES ('Test');`.

---

### Étape 6 : Ajouter une tâche

**On va :** créer `ajouter.php` avec formulaire, validation et redirection.

`ajouter.php` :

```php
<?php

require __DIR__ . "/db.php";
require __DIR__ . "/functions.php";

$titre = "Ajouter une tâche";
$erreur = null;
$valeur = "";

if ($_SERVER["REQUEST_METHOD"] === "POST") {
    $valeur = trim($_POST["titre"] ?? "");
    $erreur = validerTitre($valeur);

    if ($erreur === null) {
        ajouterTache($pdo, $valeur);
        header("Location: index.php");
        exit;
    }
}

require __DIR__ . "/views/header.php";
?>

<h1>Ajouter une tâche</h1>

<?php if ($erreur): ?>
    <p class="erreur"><?= e($erreur) ?></p>
<?php endif; ?>

<form method="post">
    <p><input type="text" name="titre" value="<?= e($valeur) ?>" placeholder="Que dois-tu faire ?" autofocus></p>
    <button type="submit">Ajouter</button>
    <a class="bouton gris" href="index.php">Annuler</a>
</form>

<?php require __DIR__ . "/views/footer.php"; ?>
```

**Explication :**

* On traite uniquement si la requête est un POST.
* `validerTitre()` renvoie une erreur ou `null`.
* En cas de succès : insertion, puis **redirection** vers la liste (évite les doublons au rafraîchissement).
* En cas d'erreur, on réaffiche le formulaire avec la valeur saisie.

**Test :** clique sur « + Nouvelle tâche ». Envoie le formulaire vide : message d'erreur. Saisis « Apprendre PHP » : tu reviens à la liste avec ta tâche.

---

### Étape 7 : Modifier une tâche

**On va :** créer `modifier.php`, qui charge la tâche, affiche un formulaire prérempli et enregistre.

`modifier.php` :

```php
<?php

require __DIR__ . "/db.php";
require __DIR__ . "/functions.php";

$id = (int) ($_GET["id"] ?? $_POST["id"] ?? 0);
$tache = getTache($pdo, $id);

if ($tache === false) {
    http_response_code(404);
    exit("Tâche introuvable.");
}

$titre = "Modifier la tâche";
$erreur = null;
$valeur = $tache["titre"];

if ($_SERVER["REQUEST_METHOD"] === "POST") {
    $valeur = trim($_POST["titre"] ?? "");
    $erreur = validerTitre($valeur);

    if ($erreur === null) {
        modifierTache($pdo, $id, $valeur);
        header("Location: index.php");
        exit;
    }
}

require __DIR__ . "/views/header.php";
?>

<h1>Modifier la tâche</h1>

<?php if ($erreur): ?>
    <p class="erreur"><?= e($erreur) ?></p>
<?php endif; ?>

<form method="post">
    <input type="hidden" name="id" value="<?= $id ?>">
    <p><input type="text" name="titre" value="<?= e($valeur) ?>"></p>
    <button type="submit">Enregistrer</button>
    <a class="bouton gris" href="index.php">Annuler</a>
</form>

<?php require __DIR__ . "/views/footer.php"; ?>
```

**Explication :**

* L'`id` vient de l'URL (`?id=3`) au premier affichage, puis du champ caché au POST.
* Si la tâche n'existe pas (`false`), on renvoie une erreur 404 et on arrête.
* Le formulaire est prérempli avec le titre actuel.

**Test :** clique sur « Modifier », change le titre, enregistre. Essaie avec un titre vide, puis avec une URL invalide : `modifier.php?id=9999`.

---

### Étape 8 : Terminer et supprimer

**On va :** créer les deux petits scripts qui reçoivent les formulaires POST de la liste.

`basculer.php` :

```php
<?php

require __DIR__ . "/db.php";
require __DIR__ . "/functions.php";

if ($_SERVER["REQUEST_METHOD"] === "POST") {
    $id = (int) ($_POST["id"] ?? 0);
    if ($id > 0) {
        basculerTache($pdo, $id);
    }
}

header("Location: index.php");
exit;
```

`supprimer.php` :

```php
<?php

require __DIR__ . "/db.php";
require __DIR__ . "/functions.php";

if ($_SERVER["REQUEST_METHOD"] === "POST") {
    $id = (int) ($_POST["id"] ?? 0);
    if ($id > 0) {
        supprimerTache($pdo, $id);
    }
}

header("Location: index.php");
exit;
```

**Explication :** ces scripts n'affichent rien : ils agissent, puis redirigent vers la liste. On n'accepte que le POST, donc un simple clic sur un lien (ou un robot qui explore le site) ne peut rien supprimer.

**Test :**

1. Clique sur « Terminer » : la tâche est barrée et descend dans la liste.
2. Clique sur « Rouvrir » : elle redevient active.
3. Clique sur « Supprimer », confirme : elle disparaît.

---

### Étape 9 : Vérification finale

Fais le parcours complet :

1. Ajoute trois tâches.
2. Modifie-en une.
3. Termine-en une.
4. Supprime-en une.
5. Essaie d'ajouter une tâche vide et une tâche de plus de 150 caractères.

Si tout fonctionne, tu as construit une application PHP complète avec base de données, validation et interface.

### Pistes d'amélioration (facultatif)

* Ajouter une colonne `date_limite` et trier par échéance.
* Protéger l'application avec le système de connexion du chapitre 8 (et lier chaque tâche à un `user_id`).
* Ajouter un filtre « À faire / Terminées » avec `$_GET["filtre"]`.
* Ajouter un jeton CSRF dans les formulaires.

### À retenir

* Une application = base de données + modèle (fonctions) + contrôleurs + vues.
* Toute action qui modifie des données passe par un POST, suivi d'une redirection.
* Valide côté serveur, échappe à l'affichage, prépare tes requêtes.
* Avance par petites étapes et teste après chacune.

---

## Et maintenant ?

Tu sais écrire de vrais petits programmes PHP. Voici quoi apprendre ensuite, dans cet ordre :

1. **[La programmation orientée objet (POO)](poo-php.md)**: classes, objets, propriétés, méthodes, `namespace`. C'est la base de tout le PHP moderne.
2. **Composer** : le gestionnaire de dépendances de PHP. Il installe des bibliothèques (`composer require ...`) et charge automatiquement tes classes (autoload PSR-4).
3. **Le MVC en profondeur** : un routeur, des contrôleurs en classes, des modèles, des vues avec un moteur de templates (Twig, Blade).
4. **Sécurité** : protection CSRF, upload de fichiers sécurisé, validation avancée, `.env` pour les secrets, HTTPS.
5. **Laravel** (ou Symfony) : un framework complet qui gère routes, base de données (ORM), authentification, validation et bien plus. Il sera beaucoup plus facile à comprendre maintenant que tu connais les bases.
6. **Créer une API REST** : renvoyer du JSON, gérer les verbes HTTP (`GET`, `POST`, `PUT`, `DELETE`) et les codes de statut, pour alimenter une application JavaScript ou mobile.
7. **Tests automatisés** : PHPUnit ou Pest pour vérifier que ton code fonctionne et le garder fiable.
8. **Git et déploiement** : versionner ton code et mettre ton projet en ligne (hébergeur mutualisé, VPS ou Docker).

**Conseil final :** le meilleur moyen de progresser est de construire des projets : blog, carnet d'adresses, mini boutique, suivi de dépenses. Chaque projet te fera rencontrer de nouveaux problèmes à résoudre.

Bon code ! 🚀
