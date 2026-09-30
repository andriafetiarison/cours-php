# Cours PHP : Composer, le gestionnaire de dépendances

Ce cours suppose que tu connais les bases de PHP et la POO (classes, `namespace`, `use`). Il est court et pratique : **explication → commande → exemple → exercice → correction**, puis un petit projet complet.

**Conseils avant de commencer :**

* Travaille dans un dossier de test, par exemple `~/php/composer-cours/`.
* Les commandes `composer ...` se tapent dans le **terminal**, à la racine de ton projet (là où se trouve, ou se trouvera, `composer.json`).
* Les numéros de version affichés dans ce cours sont des **exemples** : chez toi, Composer installera les versions les plus récentes au moment où tu lances la commande.

**Plan :**

1. Qu'est-ce que Composer et pourquoi l'utiliser ?
2. Installation et vérification
3. `composer init`
4. `composer.json`
5. `composer require`
6. Installer une bibliothèque
7. `vendor/`
8. `composer install`
9. `composer update`
10. `composer.lock`
11. Autoload
12. Autoload PSR-4
13. Charger mes propres classes automatiquement
14. Petit projet : un inventaire en ligne de commande

---

## 1. Qu'est-ce que Composer et pourquoi l'utiliser ?

### Explication

**Composer** est le gestionnaire de dépendances de PHP. Une **dépendance** est un morceau de code écrit par quelqu'un d'autre (une *bibliothèque* ou *package*) que tu utilises dans ton projet : gestion des dates, envoi d'emails, journal d'erreurs, génération de PDF…

Les milliers de packages PHP sont référencés sur **[packagist.org](https://packagist.org)**, le catalogue officiel. Composer va les y chercher pour toi.

Sans Composer, il faudrait télécharger chaque bibliothèque à la main, écrire des dizaines de `require`, vérifier les versions compatibles et récupérer aussi les bibliothèques dont **elle** dépend. Avec Composer, une commande suffit.

Composer fait trois choses :

1. **Installe** les bibliothèques (et leurs dépendances) dans le dossier `vendor/` ;
2. **Gère les versions** (quelles versions, compatibles entre elles) ;
3. **Génère un autoload** : un seul `require` charge ensuite toutes les classes, les tiennes comme celles des bibliothèques.

### Commande

Pas encore de commande : on installe Composer au chapitre suivant. Voici déjà celle que tu utiliseras le plus souvent :

```bash
composer require vendor/package
```

### Exemple

Sans Composer, pour utiliser une bibliothèque de dates :

```php
<?php

require_once 'lib/carbon/src/Carbon/Carbon.php';
require_once 'lib/carbon/src/Carbon/CarbonInterface.php';
require_once 'lib/symfony-translation/Translator.php';
// ... et des dizaines d'autres fichiers, dont ceux des dépendances
```

Avec Composer :

```php
<?php

require_once __DIR__ . '/vendor/autoload.php';   // une seule ligne, pour tout
```

### Exercice

Cite trois problèmes qu'un développeur rencontre s'il gère ses bibliothèques à la main, et que Composer résout.

### Correction

* Télécharger, copier et mettre à jour chaque bibliothèque manuellement.
* Écrire et maintenir de nombreux `require`, y compris pour les dépendances des dépendances.
* Ne pas savoir quelles versions sont compatibles entre elles, ni retrouver les mêmes versions sur un autre ordinateur ou sur le serveur.

---

## 2. Installation et vérification

### Explication

Composer est lui-même un programme en PHP. Il faut donc que la commande `php` fonctionne d'abord (`php -v`, vu dans le premier cours).

### Commande

**Windows (avec ou sans XAMPP/WAMP) :** télécharge et lance `Composer-Setup.exe` depuis **getcomposer.org/download**. L'installateur te demande quel PHP utiliser : indique le `php.exe` de ton installation (par exemple `C:\xampp\php\php.exe`). Il ajoute Composer au `PATH`.

**macOS :**

```bash
brew install composer
```

**Linux (Ubuntu/Debian) :** le plus fiable est l'installateur officiel (la page getcomposer.org/download donne aussi la procédure de vérification de l'empreinte) :

```bash
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php
php -r "unlink('composer-setup.php');"
sudo mv composer.phar /usr/local/bin/composer
```

**Vérification :**

```bash
composer --version
```

### Exemple

Résultat attendu (les numéros varient) :

```
Composer version 2.8.x 2025-xx-xx xx:xx:xx
PHP version 8.x.x (/chemin/vers/php)
```

**Si ça ne marche pas :**

| Problème | Solution |
|---|---|
| `composer` n'est pas reconnu | **Ferme et rouvre** le terminal (le `PATH` est lu au démarrage). Sinon, vérifie l'installation. |
| Erreur « The openssl extension is required » | Active-la dans `php.ini` : enlève le `;` devant `extension=openssl` (XAMPP : `C:\xampp\php\php.ini`), puis relance le terminal. |
| Téléchargements lents ou avertissement sur `zip` | Active `extension=zip` dans `php.ini`. |

### Exercice

Vérifie que Composer est installé et affiche la liste de ses commandes.

### Correction

```bash
composer --version
composer list
```

`composer list` affiche toutes les commandes. `composer help require` détaille une commande en particulier.

---

## 3. `composer init`

### Explication

`composer init` est un assistant qui pose quelques questions et **crée le fichier `composer.json`** de ton projet. C'est la carte d'identité du projet.

### Commande

Crée un dossier, puis lance l'assistant :

```bash
mkdir mon-projet
cd mon-projet
composer init
```

### Exemple

Voici les questions (le texte exact peut varier selon la version) :

```
Package name (<vendor>/<name>) [user/mon-projet]: moi/mon-projet
Description []: Mon premier projet avec Composer
Author [n to skip]:                       ← appuie sur Entrée pour ignorer
Minimum Stability []:                     ← Entrée
Package Type []: project
License []:                               ← Entrée

Would you like to define your dependencies (require) interactively [yes]? no
Would you like to define your dev dependencies (require-dev) interactively [yes]? no
Add PSR-4 autoload mapping? [src/, n to skip]: n      ← on le fera à la main plus tard

Do you confirm generation [yes]? yes
```

Un fichier `composer.json` apparaît dans ton dossier.

* **Package name** : `vendeur/nom`, en minuscules (par exemple ton pseudo + le nom du projet).
* **Package Type** : `project` pour une application, `library` pour une bibliothèque que d'autres installeront.
* Tu peux répondre `no` aux questions sur les dépendances : on les ajoutera avec `composer require`, plus simple.

> `composer init` est **facultatif** : `composer require` crée `composer.json` s'il n'existe pas. Mais il est utile pour comprendre la structure.

### Exercice

Crée un dossier `agenda` et génère son `composer.json` avec `composer init` : nom `moi/agenda`, type `project`, aucune dépendance.

### Correction

```bash
mkdir agenda
cd agenda
composer init
```

Réponds `moi/agenda` au nom, `project` au type, et `no` aux deux questions de dépendances. Vérifie avec `ls` (ou `dir`) que `composer.json` existe.

---

## 4. `composer.json`

### Explication

`composer.json` décrit ton projet : **de quoi il a besoin** et **comment charger tes classes**. C'est **toi** qui le modifies (directement, ou via des commandes comme `composer require`).

Les sections principales :

| Clé | Rôle |
|---|---|
| `name` | Nom du projet (`vendeur/nom`) |
| `description` | Courte description |
| `type` | `project` ou `library` |
| `require` | Dépendances nécessaires **au fonctionnement** |
| `require-dev` | Dépendances utiles **uniquement en développement** (tests, outils) |
| `autoload` | Comment charger **tes** classes (chapitres 11 à 13) |

**Les versions** s'écrivent avec des contraintes :

| Contrainte | Signification |
|---|---|
| `^3.0` | Toute version `3.x` à partir de `3.0` (jamais la `4.0`). **La plus courante.** |
| `~3.1.0` | De `3.1.0` jusqu'à `3.1.x` (pas `3.2`) |
| `3.1.2` | Exactement cette version |
| `*` | N'importe quelle version (déconseillé) |

Règle d'or : **`^` signifie « mises à jour sûres »**. Une nouvelle version majeure (4.0) peut casser ton code, elle n'est donc jamais installée automatiquement.

### Commande

```bash
composer validate
```

Cette commande vérifie que ton `composer.json` est correct.

### Exemple

```json
{
    "name": "moi/agenda",
    "description": "Un petit agenda en PHP",
    "type": "project",
    "require": {
        "php": ">=8.1",
        "nesbot/carbon": "^3.0"
    },
    "require-dev": {
        "fakerphp/faker": "^1.23"
    },
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```

* `"php": ">=8.1"` impose une version minimale de PHP. Composer refusera d'installer le projet sur un PHP trop ancien.
* Dans `"App\\"`, la barre oblique inverse est **doublée** : c'est la règle du JSON (`\\` représente un seul `\`).
* C'est du **JSON strict** : guillemets doubles partout, **pas de virgule après le dernier élément**, pas de commentaires.

### Exercice

Écris le `composer.json` d'un projet `moi/notes` qui :

* a pour description « Application de notes » et est de type `project` ;
* demande PHP 8.1 minimum ;
* dépend de `monolog/monolog` en version `^3.0` ;
* charge ses classes du namespace `App\` depuis le dossier `src/`.

### Correction

```json
{
    "name": "moi/notes",
    "description": "Application de notes",
    "type": "project",
    "require": {
        "php": ">=8.1",
        "monolog/monolog": "^3.0"
    },
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```

Teste-le avec `composer validate`.

---

## 5. `composer require`

### Explication

`composer require` **ajoute une bibliothèque** à ton projet. En une commande, Composer :

1. cherche le package sur Packagist ;
2. choisit la version la plus récente compatible avec ton PHP ;
3. **écrit la contrainte** dans `composer.json` ;
4. **télécharge** le package et ses dépendances dans `vendor/` ;
5. crée ou met à jour `composer.lock` et **régénère l'autoload**.

### Commande

```bash
composer require nesbot/carbon                 # installe la dernière version compatible
composer require nesbot/carbon:^3.0            # impose une contrainte de version
composer require monolog/monolog nesbot/carbon # plusieurs packages d'un coup
composer require --dev fakerphp/faker          # dépendance de développement seulement
composer remove nesbot/carbon                  # retire un package
```

### Exemple

```bash
cd agenda
composer require nesbot/carbon
```

Composer affiche quelque chose comme :

```
Using version ^3.x for nesbot/carbon
./composer.json has been updated
Running composer update nesbot/carbon
Loading composer repositories with package information
Updating dependencies
Lock file operations: 6 installs, 0 updates, 0 removals
  - Locking nesbot/carbon (3.x.x)
  ...
Writing lock file
Installing dependencies from lock file
Generating autoload files
```

Ouvre `composer.json` : la ligne `"nesbot/carbon": "^3.x"` a été ajoutée dans `require`. Un dossier `vendor/` et un fichier `composer.lock` sont apparus.

* `--dev` place le package dans `require-dev` : utile pour les outils dont la production n'a pas besoin (générateur de fausses données, tests…).
* Les packages s'écrivent toujours `vendeur/nom`, tels qu'ils apparaissent sur Packagist.

### Exercice

Dans ton projet `agenda`, installe `fakerphp/faker` comme dépendance **de développement**, puis vérifie dans `composer.json` qu'elle se trouve bien au bon endroit.

### Correction

```bash
composer require --dev fakerphp/faker
```

Dans `composer.json`, la ligne apparaît sous `"require-dev"` et non sous `"require"`.

---

## 6. Installer une bibliothèque

### Explication

Une fois le package installé, l'utiliser tient en **deux étapes** :

1. **Charger l'autoload** de Composer, une seule fois, en haut de ton point d'entrée :
   `require __DIR__ . '/vendor/autoload.php';`
2. **Importer la classe avec `use`** (son namespace est indiqué dans la documentation du package : page Packagist ou README sur GitHub).

### Commande

```bash
mkdir demo-carbon
cd demo-carbon
composer require nesbot/carbon
```

### Exemple

Crée `index.php` dans `demo-carbon/` :

```php
<?php

declare(strict_types=1);

use Carbon\Carbon;

require __DIR__ . '/vendor/autoload.php';

// La date du jour, en français
echo Carbon::now()->locale('fr')->translatedFormat('l j F Y') . "\n";

// Une date précise
$noel = Carbon::parse('2026-12-25');
echo "Noël 2026 : " . $noel->locale('fr')->translatedFormat('l j F Y') . "\n";

// Calculer une date future
echo "Dans 3 jours : " . Carbon::now()->addDays(3)->format('d/m/Y') . "\n";
```

Lance `php index.php`. Résultat (la première et la troisième ligne dépendent du jour où tu l'exécutes) :

```
mercredi 30 septembre 2026
Noël 2026 : vendredi 25 décembre 2026
Dans 3 jours : 03/10/2026
```

* `require __DIR__ . '/vendor/autoload.php';` est **la seule ligne d'inclusion** nécessaire pour toutes les bibliothèques.
* `use Carbon\Carbon;` importe la classe (comme au chapitre 12 du cours POO).
* `Carbon::now()`, `::parse()`, `->addDays()`, `->translatedFormat()` sont des méthodes de la bibliothèque : tu les trouves dans sa documentation.

### Exercice

Dans un nouveau dossier `demo-faker`, installe `fakerphp/faker` et affiche trois noms et adresses email fictifs en français.

### Correction

```bash
mkdir demo-faker
cd demo-faker
composer require fakerphp/faker
```

`index.php` :

```php
<?php

declare(strict_types=1);

use Faker\Factory;

require __DIR__ . '/vendor/autoload.php';

$faker = Factory::create('fr_FR');

for ($i = 0; $i < 3; $i++) {
    echo $faker->name() . " - " . $faker->email() . "\n";
}
```

---

## 7. `vendor/`

### Explication

`vendor/` est le dossier où Composer **range tout le code installé** : les bibliothèques demandées **et leurs propres dépendances**.

```
demo-carbon/
├── composer.json
├── composer.lock
├── index.php
└── vendor/
    ├── autoload.php       ← le seul fichier que TU utilises
    ├── composer/          ← mécanique interne de l'autoload
    ├── nesbot/carbon/     ← la bibliothèque demandée
    └── symfony/ ...       ← les dépendances de Carbon
```

Trois règles :

1. **Ne modifie jamais** un fichier dans `vendor/` : il serait écrasé à la prochaine mise à jour.
2. **Ne l'envoie pas sur Git** : il est volumineux et Composer peut le reconstruire à tout moment.
3. Tu n'as besoin que d'**un seul fichier** : `vendor/autoload.php`.

### Commande

```bash
ls vendor            # macOS / Linux
dir vendor           # Windows
```

### Exemple

Pour ignorer `vendor/` avec Git, crée un fichier `.gitignore` à la racine :

```
/vendor/
```

Pour voir où vit la classe `Carbon` :

```
vendor/nesbot/carbon/src/Carbon/Carbon.php
```

### Exercice

Liste le contenu de `vendor/` dans ton projet `demo-carbon`, retrouve le dossier de Carbon, puis crée le `.gitignore` adapté.

### Correction

```bash
ls vendor
ls vendor/nesbot/carbon
echo "/vendor/" > .gitignore
```

---

## 8. `composer install`

### Explication

`composer install` **installe toutes les dépendances déclarées** dans `composer.json`, en utilisant les versions **exactes** notées dans `composer.lock` (voir chapitre 10).

Tu l'utilises quand :

* tu **clones un projet** (il n'a pas de `vendor/`, puisqu'il est ignoré par Git) ;
* tu as **supprimé** `vendor/` ;
* tu **déploies** ton application sur un serveur.

### Commande

```bash
composer install
composer install --no-dev --optimize-autoloader   # pour la production
```

La seconde forme n'installe pas les dépendances de développement et accélère l'autoload.

### Exemple

Simulons un collègue qui récupère ton projet sans `vendor/` :

```bash
rm -rf vendor          # Windows : rmdir /s /q vendor
php index.php          # Erreur : le fichier vendor/autoload.php n'existe plus
composer install       # Composer reconstruit tout
php index.php          # Ça remarche
```

Message typique de `composer install` :

```
Installing dependencies from lock file (including require-dev)
Verifying lock file contents can be installed on current platform.
Package operations: 6 installs, 0 updates, 0 removals
  - Downloading ...
Generating autoload files
```

### Exercice

Dans `demo-carbon`, supprime `vendor/`, vérifie que le script échoue, puis restaure tout avec une seule commande.

### Correction

```bash
rm -rf vendor
php index.php       # échec : Failed opening required '.../vendor/autoload.php'
composer install
php index.php       # succès
```

---

## 9. `composer update`

### Explication

`composer update` cherche, pour chaque package, **la version la plus récente autorisée par les contraintes de `composer.json`**, l'installe et **réécrit `composer.lock`**.

Exemple : avec `"nesbot/carbon": "^3.0"`, l'update peut passer de `3.1.0` à `3.10.0`, mais **jamais** à `4.0.0`. Pour passer à une nouvelle version majeure, il faut changer la contrainte toi-même, après avoir lu le guide de migration du package.

### Commande

```bash
composer outdated                  # liste les packages qui ont une version plus récente
composer update                    # met à jour tous les packages
composer update nesbot/carbon      # met à jour un seul package
```

### Exemple

| Je veux… | Commande |
|---|---|
| Ajouter une nouvelle bibliothèque | `composer require vendeur/package` |
| Installer un projet existant, ou restaurer `vendor/` | `composer install` |
| Mettre à jour les bibliothèques | `composer update` |

```bash
composer outdated
# nesbot/carbon  3.1.0  3.10.0  ← une mise à jour existe
composer update nesbot/carbon
```

**Bonnes habitudes :** lance `composer update` **volontairement** (pas avant une livraison), teste ton application ensuite, puis enregistre le nouveau `composer.lock` dans Git.

### Exercice

Quelle commande utilises-tu :

1. pour savoir si des packages peuvent être mis à jour ?
2. pour mettre à jour uniquement Monolog ?

### Correction

1. `composer outdated`
2. `composer update monolog/monolog`

---

## 10. `composer.lock`

### Explication

`composer.lock` est généré automatiquement. Il **fige les versions exactes** de tous les packages installés (même les dépendances des dépendances), avec leur provenance.

Différence essentielle :

* `composer.json` dit : « je veux Carbon **en version 3.x** » (une *fourchette*).
* `composer.lock` dit : « voici **3.10.3** exactement » (une *photographie*).

Grâce à lui, **tous les développeurs et le serveur de production utilisent exactement les mêmes versions**. Sans lui, un collègue pourrait installer une version plus récente qui se comporte différemment.

Deux règles :

1. **Ne l'édite jamais à la main** : c'est Composer qui le gère.
2. **Envoie-le sur Git** (pour une application) : c'est lui qui garantit des installations identiques.

### Commande

```bash
composer show                     # liste les packages installés, avec leurs versions exactes
composer show nesbot/carbon       # détails d'un package
```

### Exemple

Un extrait de `composer.lock` :

```json
{
    "content-hash": "...",
    "packages": [
        {
            "name": "nesbot/carbon",
            "version": "3.10.3",
            "require": { "php": "^8.1", "...": "..." }
        }
    ]
}
```

Le cycle de vie d'un projet :

1. `composer require` → met à jour `composer.json` **et** `composer.lock` ;
2. `git add composer.json composer.lock` → tu enregistres **les deux** ;
3. un collègue clone le dépôt et lance `composer install` → il obtient **exactement tes versions**.

### Exercice

Parmi ces éléments, lesquels envoies-tu sur Git : `composer.json`, `composer.lock`, `vendor/`, `src/` ?

### Correction

`composer.json`, `composer.lock` et `src/`. Pas `vendor/` (il est reconstruit avec `composer install`).

---

## 11. Autoload

### Explication

Dans le cours POO, tu as écrit toi-même un `spl_autoload_register()`. **Composer le fait à ta place** : il génère `vendor/autoload.php`, qui charge automatiquement les classes **au moment où tu les utilises**. Tu n'écris donc plus jamais de `require` pour tes classes.

La section `autoload` de `composer.json` indique où se trouvent **tes** classes. Il existe plusieurs méthodes ; les deux à connaître :

| Méthode | Principe | Usage |
|---|---|---|
| `classmap` | Composer **scanne** des dossiers et mémorise toutes les classes trouvées | Ancien code **sans namespace** |
| `psr-4` | Le namespace correspond aux dossiers | **Standard moderne** (chapitre 12) |

Commençons par le cas le plus simple : **passer d'une liste de `require_once` à l'autoload**, sans toucher à tes classes.

### Commande

```bash
composer dump-autoload
```

`dump-autoload` **régénère** l'autoload. À lancer chaque fois que tu modifies la section `autoload` de `composer.json` (et, avec `classmap`, à chaque nouvelle classe).

### Exemple

**Avant :** un projet avec des classes et des `require_once`.

```
ancien-projet/
├── classes/
│   ├── User.php
│   └── Product.php
└── index.php
```

`classes/User.php` :

```php
<?php

declare(strict_types=1);

class User
{
    public function __construct(private string $name)
    {
    }

    public function hello(): string
    {
        return "Bonjour {$this->name} !";
    }
}
```

`classes/Product.php` :

```php
<?php

declare(strict_types=1);

class Product
{
    public function __construct(private string $label, private float $price)
    {
    }

    public function describe(): string
    {
        return "{$this->label} : {$this->price} €";
    }
}
```

`index.php` **avant** :

```php
<?php

require_once 'classes/User.php';
require_once 'classes/Product.php';

$user = new User("Alice");
echo $user->hello() . "\n";

$product = new Product("Clavier", 49.90);
echo $product->describe() . "\n";
```

**Après :** on branche Composer.

1. Crée `composer.json` à la racine (avec `composer init`, ou à la main) :

```json
{
    "autoload": {
        "classmap": ["classes/"]
    }
}
```

2. Génère l'autoload :

```bash
composer dump-autoload
```

```
Generating autoload files
Generated autoload files
```

3. `index.php` **après** :

```php
<?php

require __DIR__ . '/vendor/autoload.php';

$user = new User("Alice");
echo $user->hello() . "\n";

$product = new Product("Clavier", 49.90);
echo $product->describe() . "\n";
```

Résultat identique, mais **plus aucun `require_once` par classe** : une seule ligne charge tout.

**Limite de `classmap` :** à chaque nouvelle classe, il faut relancer `composer dump-autoload`, et elle ne tire pas parti des namespaces. C'est pourquoi on utilise plutôt PSR-4 (chapitres suivants).

### Exercice

Reprends le projet ci-dessus (`classes/User.php`, `classes/Product.php`, `index.php` avec des `require_once`) et remplace les `require_once` par l'autoload Composer avec `classmap`.

### Correction

1. Crée `composer.json` :

```json
{
    "autoload": {
        "classmap": ["classes/"]
    }
}
```

2. Lance `composer dump-autoload`.
3. Dans `index.php`, supprime les deux `require_once` et remplace-les par `require __DIR__ . '/vendor/autoload.php';`.
4. Teste avec `php index.php`.

---

## 12. Autoload PSR-4

### Explication

**PSR-4** est la règle standard de tous les projets PHP modernes : **le namespace d'une classe correspond à son chemin de fichier**. Tu déclares **une seule fois** la correspondance dans `composer.json`, et toutes les classes sont ensuite trouvées automatiquement.

```json
"autoload": {
    "psr-4": {
        "App\\": "src/"
    }
}
```

Cela se lit : « le namespace `App\` commence dans le dossier `src/` ».

Les **quatre règles** :

1. le préfixe de namespace **se termine par `\\`** dans le JSON (`"App\\"`) ;
2. ce qui suit le préfixe correspond aux **sous-dossiers** ;
3. le **nom du fichier = nom de la classe**, avec la même casse (`User.php`), car Linux distingue majuscules et minuscules ;
4. **une classe par fichier**.

| Classe | Fichier |
|---|---|
| `App\Models\User` | `src/Models/User.php` |
| `App\Services\Mailer` | `src/Services/Mailer.php` |
| `App\Services\Mail\SmtpMailer` | `src/Services/Mail/SmtpMailer.php` |

### Commande

```bash
composer dump-autoload
```

À lancer **une fois** après avoir ajouté ou modifié la section `autoload` de `composer.json`. Ensuite, les nouvelles classes placées au bon endroit sont trouvées automatiquement (en développement, tu n'as pas besoin de relancer la commande).

### Exemple

On transforme l'ancien projet : `classes/User.php` devient `src/Models/User.php`, avec un namespace.

```
projet/
├── composer.json
├── index.php
└── src/
    └── Models/
        └── User.php
```

`composer.json` :

```json
{
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```

`src/Models/User.php` :

```php
<?php

declare(strict_types=1);

namespace App\Models;

class User
{
    public function __construct(private string $name)
    {
    }

    public function hello(): string
    {
        return "Bonjour {$this->name} !";
    }
}
```

```bash
composer dump-autoload
```

`index.php` :

```php
<?php

declare(strict_types=1);

use App\Models\User;

require __DIR__ . '/vendor/autoload.php';

$user = new User("Alice");
echo $user->hello() . "\n";
```

Quand PHP rencontre `App\Models\User`, l'autoload retire le préfixe `App\`, transforme `Models\User` en `Models/User.php`, le cherche dans `src/` et charge `src/Models/User.php`.

### Exercice

1. Dans quel fichier doit se trouver la classe `App\Repository\UserRepository` ?
2. Que doit-on ajouter dans `composer.json` pour que le namespace `Tests\` soit chargé depuis le dossier `tests/` ?

### Correction

1. `src/Repository/UserRepository.php`
2. Une deuxième entrée dans `psr-4` :

```json
"autoload": {
    "psr-4": {
        "App\\": "src/",
        "Tests\\": "tests/"
    }
}
```

Puis `composer dump-autoload`.

---

## 13. Charger mes propres classes automatiquement

### Explication

Voici la **méthode à suivre** à chaque nouvelle classe, sans jamais écrire de `require` :

1. crée le **fichier** au bon endroit : `src/Dossier/NomDeLaClasse.php` ;
2. écris la **déclaration du namespace** qui correspond au dossier : `namespace App\Dossier;` ;
3. donne à la classe **le même nom que le fichier** ;
4. dans ton code, importe-la avec `use App\Dossier\NomDeLaClasse;`.

C'est tout. Le `require __DIR__ . '/vendor/autoload.php';` du point d'entrée s'occupe du reste.

### Commande

```bash
composer dump-autoload
```

Utile en cas de doute, ou en production avec `composer dump-autoload --optimize`.

### Exemple

On ajoute une classe `Calculatrice` au projet précédent.

`src/Services/Calculatrice.php` :

```php
<?php

declare(strict_types=1);

namespace App\Services;

class Calculatrice
{
    public function additionner(float $a, float $b): float
    {
        return $a + $b;
    }
}
```

`index.php` :

```php
<?php

declare(strict_types=1);

use App\Models\User;
use App\Services\Calculatrice;

require __DIR__ . '/vendor/autoload.php';

$user = new User("Alice");
echo $user->hello() . "\n";

$calc = new Calculatrice();
echo "2 + 3 = " . $calc->additionner(2, 3) . "\n";
```

Aucun `require` pour `Calculatrice` : Composer la trouve seul.

**Erreur `Class "App\Services\Calculatrice" not found` ? Vérifie dans l'ordre :**

| Cause possible | Vérification |
|---|---|
| Tu as oublié d'inclure l'autoload | `require __DIR__ . '/vendor/autoload.php';` en haut du script |
| Le namespace ne correspond pas au dossier | `namespace App\Services;` ↔ `src/Services/` |
| Le nom du fichier est faux (casse comprise) | `Calculatrice.php`, avec la majuscule |
| Le `use` est faux ou absent | `use App\Services\Calculatrice;` |
| `composer.json` modifié sans régénération | Lance `composer dump-autoload` |

**Bonus : migrer ton projet `boutique` du cours POO.** Il utilisait un `autoload.php` fait maison. Pour passer à Composer :

1. à la racine de `boutique/`, ajoute un `composer.json` avec `"psr-4": { "App\\": "src/" }` ;
2. lance `composer dump-autoload` ;
3. dans `public/index.php`, remplace `require __DIR__ . '/../autoload.php';` par `require __DIR__ . '/../vendor/autoload.php';` ;
4. supprime `autoload.php`.

Les namespaces `App\Entity`, `App\Repository`, `App\Service` fonctionnent sans autre changement.

### Exercice

Crée la classe `App\Utils\Texte` avec une méthode statique `majuscule(string $t): string`, chargée **uniquement par l'autoload Composer**, puis utilise-la dans `index.php`.

### Correction

Structure : `src/Utils/Texte.php` (le `composer.json` du chapitre 12 suffit).

`src/Utils/Texte.php` :

```php
<?php

declare(strict_types=1);

namespace App\Utils;

class Texte
{
    public static function majuscule(string $t): string
    {
        return mb_strtoupper($t);
    }
}
```

`index.php` :

```php
<?php

declare(strict_types=1);

use App\Utils\Texte;

require __DIR__ . '/vendor/autoload.php';

echo Texte::majuscule("bonjour") . "\n";   // BONJOUR
```

---

## 14. Petit projet : un inventaire en ligne de commande

Tu vas construire un petit programme qui gère un inventaire de produits et affiche un rapport. Il réunit :

* **plusieurs classes** (modèle, service, rapport, fabrique de logger) ;
* des **namespaces** ;
* **Composer** et l'**autoload PSR-4** ;
* **deux bibliothèques externes** : **Monolog** (journal d'événements dans un fichier) et **Carbon** (dates).

### Structure finale

```
inventaire/
├── .gitignore
├── composer.json
├── composer.lock                 (généré par Composer)
├── index.php                     (point d'entrée)
├── logs/
│   └── app.log                   (créé par Monolog)
├── src/
│   ├── Models/
│   │   └── Produit.php
│   ├── Services/
│   │   ├── Inventaire.php
│   │   └── Rapport.php
│   └── Support/
│       └── LoggerFactory.php
└── vendor/                       (généré par Composer)
```

### Étape 1 : créer le projet

```bash
mkdir inventaire
cd inventaire
composer init
```

Réponds comme au chapitre 3 : nom `moi/inventaire`, type `project`, `no` aux questions de dépendances et `n` à l'autoload (on le règle à l'étape 3).

### Étape 2 : installer les bibliothèques

```bash
composer require monolog/monolog nesbot/carbon
```

* **Monolog** écrit des messages dans un fichier de log.
* **Carbon** manipule et formate les dates.

Composer télécharge les deux packages (et leurs dépendances) dans `vendor/`, met à jour `composer.json` et crée `composer.lock`.

### Étape 3 : configurer l'autoload PSR-4

Ouvre `composer.json` et ajoute la section `autoload`. Le fichier final ressemble à ceci (les versions seront celles installées chez toi) :

```json
{
    "name": "moi/inventaire",
    "description": "Petit inventaire en ligne de commande",
    "type": "project",
    "require": {
        "monolog/monolog": "^3.0",
        "nesbot/carbon": "^3.0"
    },
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```

Puis régénère l'autoload :

```bash
composer dump-autoload
```

Crée aussi le `.gitignore` :

```
/vendor/
/logs/
```

### Étape 4 : le modèle

`src/Models/Produit.php`

```php
<?php

declare(strict_types=1);

namespace App\Models;

final class Produit
{
    public function __construct(
        public readonly string $nom,
        public readonly float $prix,
        public readonly int $stock,
    ) {
    }

    public function prixTtc(float $tva = 20.0): float
    {
        return round($this->prix * (1 + $tva / 100), 2);
    }

    public function valeurStock(): float
    {
        return $this->prix * $this->stock;
    }
}
```

### Étape 5 : la fabrique de logger (utilise Monolog)

`src/Support/LoggerFactory.php`

```php
<?php

declare(strict_types=1);

namespace App\Support;

use Monolog\Handler\StreamHandler;
use Monolog\Level;
use Monolog\Logger;

final class LoggerFactory
{
    public static function creer(string $fichier): Logger
    {
        $logger = new Logger('inventaire');
        $logger->pushHandler(new StreamHandler($fichier, Level::Info));

        return $logger;
    }
}
```

* `Logger('inventaire')` crée un journal nommé « inventaire ».
* `StreamHandler` indique **où écrire** (un fichier) et à partir de **quel niveau** (`Info` et plus grave).
* Les classes `Monolog\...` sont chargées par l'autoload de Composer, exactement comme les tiennes.

### Étape 6 : le service d'inventaire

`src/Services/Inventaire.php`

```php
<?php

declare(strict_types=1);

namespace App\Services;

use App\Models\Produit;
use Monolog\Logger;

final class Inventaire
{
    /** @var Produit[] */
    private array $produits = [];

    public function __construct(private Logger $logger)
    {
    }

    public function ajouter(Produit $produit): void
    {
        $this->produits[] = $produit;

        $this->logger->info('Produit ajouté', [
            'nom' => $produit->nom,
            'stock' => $produit->stock,
        ]);
    }

    /** @return Produit[] */
    public function tous(): array
    {
        return $this->produits;
    }

    public function valeurTotale(): float
    {
        return array_sum(array_map(
            static fn (Produit $produit): float => $produit->valeurStock(),
            $this->produits,
        ));
    }

    /** @return Produit[] */
    public function enRupture(): array
    {
        return array_values(array_filter(
            $this->produits,
            static fn (Produit $produit): bool => $produit->stock === 0,
        ));
    }
}
```

### Étape 7 : le rapport (utilise Carbon)

`src/Services/Rapport.php`

```php
<?php

declare(strict_types=1);

namespace App\Services;

use Carbon\Carbon;

final class Rapport
{
    public function __construct(private Inventaire $inventaire)
    {
    }

    public function generer(): string
    {
        $date = Carbon::now()->locale('fr')->translatedFormat('l j F Y à H:i');
        $ligne = str_repeat('-', 50);

        $lignes = ["Inventaire du $date", $ligne];

        foreach ($this->inventaire->tous() as $produit) {
            $lignes[] = sprintf(
                '%-15s %8.2f € HT %8.2f € TTC  stock : %d',
                $produit->nom,
                $produit->prix,
                $produit->prixTtc(),
                $produit->stock,
            );
        }

        $lignes[] = $ligne;
        $lignes[] = 'Valeur du stock : ' . number_format($this->inventaire->valeurTotale(), 2, ',', ' ') . ' €';
        $lignes[] = 'Produits en rupture : ' . count($this->inventaire->enRupture());

        return implode("\n", $lignes) . "\n";
    }
}
```

### Étape 8 : le point d'entrée

`index.php`

```php
<?php

declare(strict_types=1);

use App\Models\Produit;
use App\Services\Inventaire;
use App\Services\Rapport;
use App\Support\LoggerFactory;

require __DIR__ . '/vendor/autoload.php';

date_default_timezone_set('Europe/Paris');   // adapte-le à ton fuseau horaire

$logger = LoggerFactory::creer(__DIR__ . '/logs/app.log');
$inventaire = new Inventaire($logger);

$inventaire->ajouter(new Produit('Clavier', 49.90, 12));
$inventaire->ajouter(new Produit('Souris', 19.90, 0));
$inventaire->ajouter(new Produit('Webcam', 35.00, 5));

echo (new Rapport($inventaire))->generer();

$logger->info('Rapport généré');
```

Remarque : **un seul `require`** (l'autoload) pour charger tes cinq classes **et** les deux bibliothèques.

### Étape 9 : lancer et vérifier

```bash
php index.php
```

Résultat attendu (la date dépend du jour, les espaces peuvent légèrement varier) :

```
Inventaire du mercredi 30 septembre 2026 à 10:15
--------------------------------------------------
Clavier            49.90 € HT    59.88 € TTC  stock : 12
Souris             19.90 € HT    23.88 € TTC  stock : 0
Webcam             35.00 € HT    42.00 € TTC  stock : 5
--------------------------------------------------
Valeur du stock : 773,80 €
Produits en rupture : 1
```

Ouvre ensuite `logs/app.log` : Monolog y a écrit un journal.

```
[2026-09-30T10:15:00.123456+02:00] inventaire.INFO: Produit ajouté {"nom":"Clavier","stock":12} []
[2026-09-30T10:15:00.123789+02:00] inventaire.INFO: Produit ajouté {"nom":"Souris","stock":0} []
[2026-09-30T10:15:00.124012+02:00] inventaire.INFO: Produit ajouté {"nom":"Webcam","stock":5} []
[2026-09-30T10:15:00.124345+02:00] inventaire.INFO: Rapport généré [] []
```

*(Dans un navigateur, ce script afficherait tout sur une ligne : lance-le dans le terminal.)*

### Étape 10 : tester le « monde réel »

1. **Nouvelle classe :** ajoute `src/Models/Categorie.php` avec le bon namespace, utilise-la dans `index.php` avec `use` : aucun `require` à ajouter, ni `dump-autoload` à relancer.
2. **Simule un collègue :** supprime `vendor/` (`rm -rf vendor`), relance `php index.php` (erreur), puis `composer install` et `php index.php` (ça remarche, avec les mêmes versions grâce à `composer.lock`).
3. **Regarde ce que Composer a installé :** `composer show`.

### Pour aller plus loin

* Remplace `Level::Info` par `Level::Debug`, ajoute un `$logger->debug(...)` et observe le fichier.
* Ajoute une bibliothèque de ton choix avec `composer require` (par exemple `fakerphp/faker` en `--dev` pour générer de faux produits).
* Adapte le projet `boutique` du cours POO pour qu'il journalise chaque ajout de produit avec Monolog.

---

## Aide-mémoire des commandes

| Commande | Ce qu'elle fait |
|---|---|
| `composer --version` | Vérifie que Composer est installé |
| `composer init` | Crée `composer.json` via un assistant |
| `composer validate` | Vérifie `composer.json` |
| `composer require vendeur/package` | Ajoute une bibliothèque |
| `composer require --dev vendeur/package` | Ajoute une bibliothèque de développement |
| `composer remove vendeur/package` | Retire une bibliothèque |
| `composer install` | Installe les versions exactes de `composer.lock` |
| `composer update` | Met à jour selon les contraintes de `composer.json` |
| `composer outdated` | Liste les packages pouvant être mis à jour |
| `composer show` | Liste les packages installés |
| `composer dump-autoload` | Régénère l'autoload |

| Fichier / dossier | Rôle | Sur Git ? |
|---|---|---|
| `composer.json` | Ce dont le projet a besoin + l'autoload | Oui |
| `composer.lock` | Versions exactes installées | Oui (application) |
| `vendor/` | Code des bibliothèques + autoload | **Non** |
| `src/` | Tes classes | Oui |

---

## Et maintenant ?

1. **Tests automatisés** : installe PHPUnit ou Pest avec `composer require --dev phpunit/phpunit` et teste tes classes.
2. **Outils de qualité** : PHPStan (analyse statique) et PHP-CS-Fixer (style de code), installés en `--dev`.
3. **Scripts Composer** : la section `"scripts"` de `composer.json` crée des raccourcis (`composer test`, `composer lint`).
4. **Sécurité** : `composer audit` signale les bibliothèques installées qui ont des failles connues.
5. **Variables d'environnement** : `vlucas/phpdotenv` pour ranger mots de passe et clés dans un fichier `.env`.
6. **Démarrer un framework** : `composer create-project laravel/laravel mon-app` (ou Symfony). Tu y retrouveras `composer.json`, `vendor/` et l'autoload PSR-4 avec `App\` → `app/` ou `src/`.
7. **Publier ta propre bibliothèque** sur Packagist, une fois ton code propre, testé et documenté.
