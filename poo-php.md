# Cours PHP 8 : la programmation orientée objet (POO)

Ce cours suppose que tu connais les bases de PHP (variables, fonctions, tableaux, formulaires, PDO). Il est court, pratique et progressif : **notion → exemple → explication → exercice → correction**, puis un mini-projet réaliste.

**Comment exécuter les exemples :**

* Crée un fichier `test.php`, copie le code, puis lance `php test.php` dans le terminal.
* Les `\n` font un retour à la ligne dans le terminal. Dans un navigateur, entoure l'affichage de `<pre>` ou remplace `\n` par `<br>`.
* Les exemples commencent par `declare(strict_types=1);` : PHP refuse alors les conversions de type silencieuses (`"abc"` pour un `int`, par exemple). C'est la pratique moderne recommandée.

**Plan :**

1. Pourquoi utiliser la POO
2. Classes et objets
3. Propriétés
4. Méthodes
5. Le constructeur `__construct`
6. `public`, `private`, `protected`
7. Getters et setters
8. `static`
9. Héritage
10. Interfaces
11. `namespace`
12. `use` (et l'autoload)
13. Mini-projet : gestion de produits

---

## 1. Pourquoi utiliser la POO

### Notion

Dans un vrai projet, tu manipules des « choses » : un utilisateur, un produit, une commande. Avec les tableaux et les fonctions, les **données** d'un côté et le **code qui les utilise** de l'autre sont dispersés, et rien ne protège tes données.

La POO regroupe les deux dans une **classe** :

* **les données** (propriétés) : le nom, le prix, l'email…
* **les actions** (méthodes) : calculer le prix TTC, changer le mot de passe…

Avantages concrets :

* le code est **rangé** : tout ce qui concerne un produit est au même endroit ;
* les données sont **protégées** (on empêche un prix négatif) ;
* le code est **réutilisable** et plus facile à modifier ;
* tous les frameworks modernes (Laravel, Symfony) sont écrits en POO.

### Exemple

```php
<?php

// --- Sans POO : un tableau + une fonction ---
function aireRectangle(array $rectangle): float
{
    return $rectangle["largeur"] * $rectangle["hauteur"];
}

$r1 = ["largeur" => 5, "hauteur" => 3];
echo "Aire (tableau) : " . aireRectangle($r1) . "\n";   // 15

// --- Avec POO : une classe qui contient données ET action ---
class Rectangle
{
    public float $largeur = 0;
    public float $hauteur = 0;

    public function aire(): float
    {
        return $this->largeur * $this->hauteur;
    }
}

$r2 = new Rectangle();
$r2->largeur = 5;
$r2->hauteur = 3;
echo "Aire (objet) : " . $r2->aire() . "\n";              // 15
```

### Explication

* Dans la version tableau, rien n'empêche d'écrire `"largeru"` (faute de frappe) ou `"abc"` comme largeur, et la fonction n'est liée à aucune donnée.
* Dans la version objet, `Rectangle` **contient** ses propriétés (`largeur`, `hauteur`) et sa méthode (`aire()`), avec des types déclarés.
* Ne t'inquiète pas si la syntaxe est nouvelle : chaque élément est détaillé dans les chapitres suivants.

### Petit exercice

Tu dois gérer des livres (titre, auteur, nombre de pages) et pouvoir dire si un livre est « long » (plus de 300 pages). Sans coder, liste les **propriétés** et les **méthodes** d'une classe `Livre`.

### Correction

* Propriétés : `titre`, `auteur`, `pages`.
* Méthode : `estLong()` qui retourne `true` si `pages > 300`.

C'est exactement ce que tu coderas dans les chapitres 3 à 5.

---

## 2. Classes et objets

### Notion

* Une **classe** est un plan, un modèle (le plan d'une maison).
* Un **objet** (ou **instance**) est une réalisation concrète de ce plan (une maison construite).
* On crée un objet avec le mot-clé `new`. À partir d'une seule classe, tu peux créer autant d'objets que tu veux, indépendants les uns des autres.

Convention : les noms de classe s'écrivent en **PascalCase** (`Produit`, `CompteBancaire`).

### Exemple

```php
<?php

declare(strict_types=1);

class Chien
{
}

$rex = new Chien();
$mila = new Chien();

var_dump($rex instanceof Chien);   // bool(true)
var_dump($rex === $mila);          // bool(false) : deux objets distincts
var_dump($rex);                    // object(Chien)#1 (0) {}
```

### Explication

* `class Chien { }` déclare une classe (vide pour l'instant).
* `new Chien()` crée un objet. `$rex` et `$mila` sont deux objets différents issus du même plan.
* `instanceof` vérifie qu'un objet est bien une instance d'une classe.
* `#1` dans `var_dump` est l'identifiant interne de l'objet.

### Petit exercice

Crée une classe `Voiture`, instancie-la deux fois, puis vérifie avec `instanceof` que les deux objets sont bien des `Voiture`.

### Correction

```php
<?php

declare(strict_types=1);

class Voiture
{
}

$v1 = new Voiture();
$v2 = new Voiture();

var_dump($v1 instanceof Voiture);   // bool(true)
var_dump($v2 instanceof Voiture);   // bool(true)
```

---

## 3. Propriétés

### Notion

Les **propriétés** sont les variables d'un objet. Elles se déclarent dans la classe, avec un **type**, et se lisent/écrivent avec la flèche `->`.

### Exemple

```php
<?php

declare(strict_types=1);

class Livre
{
    public string $titre = "";
    public string $auteur = "";
    public int $pages = 0;
    public ?string $isbn = null;   // ?string = texte ou null
}

$livre = new Livre();
$livre->titre = "Le Petit Prince";
$livre->auteur = "Antoine de Saint-Exupéry";
$livre->pages = 96;

echo "{$livre->titre} de {$livre->auteur} ({$livre->pages} pages)\n";

$autre = new Livre();
echo "Titre du second livre : [" . $autre->titre . "]\n";   // []

try {
    $livre->pages = "beaucoup";   // mauvais type
} catch (TypeError $e) {
    echo "Erreur : " . $e->getMessage() . "\n";
}
```

### Explication

* `public string $titre = "";` : visibilité, type, nom, valeur par défaut.
* `$livre->titre` lit la propriété ; `$livre->titre = "..."` la modifie.
* Chaque objet a **ses propres valeurs** : `$autre` n'est pas affecté par `$livre`.
* Avec `strict_types=1`, assigner un texte à un `int` lève une `TypeError` : les types te protègent.
* Dans une chaîne, `{$objet->propriete}` permet d'utiliser une propriété avec des accolades.
* Une propriété typée **sans valeur par défaut** est « non initialisée » : la lire avant de lui donner une valeur provoque une erreur. Le constructeur (chapitre 5) règle ce problème.

### Petit exercice

Crée une classe `Produit` avec les propriétés `nom` (texte), `prix` (décimal) et `stock` (entier). Crée un produit « Clavier » à 49.90 € avec 12 en stock et affiche ses informations.

### Correction

```php
<?php

declare(strict_types=1);

class Produit
{
    public string $nom = "";
    public float $prix = 0.0;
    public int $stock = 0;
}

$produit = new Produit();
$produit->nom = "Clavier";
$produit->prix = 49.90;
$produit->stock = 12;

echo "{$produit->nom} : {$produit->prix} € ({$produit->stock} en stock)\n";
```

---

## 4. Méthodes

### Notion

Les **méthodes** sont les fonctions d'un objet. Elles peuvent lire et modifier ses propriétés grâce à **`$this`**, qui désigne « l'objet courant ».

### Exemple

```php
<?php

declare(strict_types=1);

class CompteBancaire
{
    public float $solde = 0;

    public function deposer(float $montant): void
    {
        $this->solde += $montant;
    }

    public function retirer(float $montant): bool
    {
        if ($montant > $this->solde) {
            return false;   // fonds insuffisants
        }

        $this->solde -= $montant;
        return true;
    }

    public function afficher(): string
    {
        return "Solde : " . number_format($this->solde, 2, ",", " ") . " €";
    }
}

$compte = new CompteBancaire();
$compte->deposer(100);
$compte->retirer(30);
echo $compte->afficher() . "\n";             // Solde : 70,00 €

$ok = $compte->retirer(500);
var_dump($ok);                               // bool(false)
echo $compte->afficher() . "\n";             // Solde : 70,00 €
```

### Explication

* Une méthode se déclare comme une fonction, précédée de sa visibilité (`public`).
* `$this->solde` accède à la propriété **de l'objet sur lequel on appelle la méthode**.
* `$compte->deposer(100)` appelle la méthode avec la flèche `->`.
* Le type de retour (`: void`, `: bool`, `: string`) documente et sécurise le code. `void` signifie « ne retourne rien ».

### Petit exercice

Reprends la classe `Rectangle` (propriétés `largeur` et `hauteur`) et ajoute les méthodes `aire()`, `perimetre()` et `estCarre()` (retourne `true` si largeur et hauteur sont égales).

### Correction

```php
<?php

declare(strict_types=1);

class Rectangle
{
    public float $largeur = 0;
    public float $hauteur = 0;

    public function aire(): float
    {
        return $this->largeur * $this->hauteur;
    }

    public function perimetre(): float
    {
        return 2 * ($this->largeur + $this->hauteur);
    }

    public function estCarre(): bool
    {
        return $this->largeur === $this->hauteur;
    }
}

$r = new Rectangle();
$r->largeur = 4;
$r->hauteur = 4;

echo "Aire : " . $r->aire() . "\n";             // 16
echo "Périmètre : " . $r->perimetre() . "\n";   // 16
var_dump($r->estCarre());                       // bool(true)
```

---

## 5. Le constructeur `__construct`

### Notion

Le **constructeur** est une méthode spéciale, appelée automatiquement quand on écrit `new`. Il sert à donner à l'objet ses valeurs de départ, pour qu'il soit **toujours valide dès sa création**.

PHP 8 propose la **promotion de propriétés** : tu déclares les propriétés directement dans les paramètres du constructeur, sans les répéter ailleurs.

### Exemple

```php
<?php

declare(strict_types=1);

class Utilisateur
{
    public function __construct(
        public string $nom,
        public string $email,
        public bool $actif = true,
    ) {
    }

    public function presentation(): string
    {
        $statut = $this->actif ? "actif" : "inactif";

        return "{$this->nom} ({$this->email}) - $statut";
    }
}

$alice = new Utilisateur("Alice", "alice@example.com");
$bob = new Utilisateur(email: "bob@example.com", nom: "Bob", actif: false);

echo $alice->presentation() . "\n";   // Alice (alice@example.com) - actif
echo $bob->presentation() . "\n";     // Bob (bob@example.com) - inactif
```

### Explication

* `public string $nom` dans les parenthèses **déclare la propriété ET l'initialise** avec l'argument reçu.
* `bool $actif = true` : valeur par défaut, donc l'argument est facultatif.
* Les **arguments nommés** (`email: ...`) permettent de passer les valeurs dans n'importe quel ordre et rendent l'appel lisible.
* On ne peut plus créer un `Utilisateur` sans nom ni email : l'objet est toujours complet.

Sans promotion (ancienne écriture, que tu verras encore dans beaucoup de code), cela donnerait :

```php
class Utilisateur
{
    public string $nom;
    public string $email;

    public function __construct(string $nom, string $email)
    {
        $this->nom = $nom;
        $this->email = $email;
    }
}
```

La version promue fait la même chose en quatre fois moins de code.

### Petit exercice

Crée une classe `Produit` avec un constructeur (promotion) prenant `nom`, `prix` et `stock` (0 par défaut), et une méthode `prixTtc(float $tva = 20.0): float`. Affiche le prix TTC d'un produit à 50 €.

### Correction

```php
<?php

declare(strict_types=1);

class Produit
{
    public function __construct(
        public string $nom,
        public float $prix,
        public int $stock = 0,
    ) {
    }

    public function prixTtc(float $tva = 20.0): float
    {
        return $this->prix * (1 + $tva / 100);
    }
}

$produit = new Produit("Souris", 50);
echo "Prix TTC : " . $produit->prixTtc() . " €\n";          // 60
echo "TVA 5,5 % : " . $produit->prixTtc(5.5) . " €\n";      // 52.75
```

---

## 6. `public`, `private`, `protected`

### Notion

La **visibilité** contrôle qui peut accéder à une propriété ou à une méthode :

| Mot-clé | Accessible depuis |
|---|---|
| `public` | partout |
| `private` | uniquement **dans la classe elle-même** |
| `protected` | dans la classe **et ses classes filles** (voir chapitre 9) |

Règle pratique : **tout est `private` par défaut**, et tu n'ouvres en `public` que ce qui doit vraiment l'être. C'est le principe d'**encapsulation** : l'objet protège son état interne.

### Exemple

```php
<?php

declare(strict_types=1);

class Compte
{
    public function __construct(private float $solde = 0)
    {
    }

    public function deposer(float $montant): void
    {
        $this->solde += $montant;
    }

    public function solde(): float
    {
        return $this->solde;
    }

    private function journaliser(string $message): void
    {
        // méthode interne, invisible de l'extérieur
    }
}

$compte = new Compte(100);
$compte->deposer(50);
echo $compte->solde() . "\n";   // 150

try {
    echo $compte->solde;        // accès direct interdit
} catch (Error $e) {
    echo "Erreur : " . $e->getMessage() . "\n";
}

try {
    $compte->journaliser("test");
} catch (Error $e) {
    echo "Erreur : " . $e->getMessage() . "\n";
}
```

### Explication

* `private float $solde` : seule la classe `Compte` peut lire ou modifier le solde.
* Pour agir sur le solde, l'extérieur doit passer par les méthodes publiques (`deposer()`, `solde()`), qui peuvent **vérifier les règles** (montant positif, etc.).
* L'accès direct `$compte->solde` lève une `Error` : « Cannot access private property Compte::$solde ».
* `protected` fonctionne comme `private`, mais les classes qui **héritent** peuvent aussi accéder à l'élément. Nous le verrons au chapitre 9.

### Petit exercice

Crée une classe `Coffre` avec un code secret `private` (donné au constructeur) et une méthode publique `ouvrir(string $code): bool` qui retourne `true` seulement si le code est correct. Vérifie qu'on ne peut pas lire `$coffre->code` directement.

### Correction

```php
<?php

declare(strict_types=1);

class Coffre
{
    public function __construct(private string $code)
    {
    }

    public function ouvrir(string $code): bool
    {
        return $code === $this->code;
    }
}

$coffre = new Coffre("1234");

var_dump($coffre->ouvrir("0000"));   // bool(false)
var_dump($coffre->ouvrir("1234"));   // bool(true)

try {
    echo $coffre->code;
} catch (Error $e) {
    echo "Interdit : " . $e->getMessage() . "\n";
}
```

---

## 7. Getters et setters

### Notion

Quand une propriété est `private`, on propose des méthodes publiques pour y accéder :

* un **getter** lit la valeur (`getPrix()`) ;
* un **setter** la modifie (`setPrix()`) **après avoir vérifié qu'elle est valide**.

C'est l'intérêt principal : **l'objet refuse les valeurs incorrectes**. Pour signaler une erreur, on **lève une exception** avec `throw`, que l'appelant peut attraper avec `try / catch`.

Astuce moderne : pour une valeur qui ne doit **jamais changer** après la création (un identifiant), utilise `readonly`.

### Exemple

```php
<?php

declare(strict_types=1);

class Produit
{
    private float $prix;

    public function __construct(
        public readonly int $id,
        private string $nom,
        float $prix,
    ) {
        $this->setPrix($prix);   // on réutilise le setter : la validation s'applique aussi à la création
    }

    public function getNom(): string
    {
        return $this->nom;
    }

    public function getPrix(): float
    {
        return $this->prix;
    }

    public function setPrix(float $prix): void
    {
        if ($prix < 0) {
            throw new InvalidArgumentException("Le prix ne peut pas être négatif.");
        }

        $this->prix = $prix;
    }
}

$produit = new Produit(1, "Clavier", 49.90);
$produit->setPrix(39.90);
echo $produit->getNom() . " : " . $produit->getPrix() . " €\n";   // Clavier : 39.9 €

try {
    $produit->setPrix(-5);
} catch (InvalidArgumentException $e) {
    echo "Erreur : " . $e->getMessage() . "\n";
}

try {
    $produit->id = 99;   // readonly : impossible de modifier
} catch (Error $e) {
    echo "Erreur : " . $e->getMessage() . "\n";
}
```

### Explication

* `private float $prix` n'est pas promue dans le constructeur, car on veut **valider** avant d'assigner : on appelle donc `setPrix()`.
* `throw new InvalidArgumentException(...)` interrompt l'exécution et signale l'erreur. `try / catch` permet de la gérer proprement sans faire planter le script.
* `public readonly int $id` : lisible partout, assignable **une seule fois** (dans le constructeur). Essayer de la modifier lève une `Error`.
* Ne crée pas de getter/setter pour tout par réflexe : **seulement ceux dont tu as besoin**. Une propriété sans setter est simplement une valeur non modifiable de l'extérieur.

### Petit exercice

Crée une classe `Utilisateur` avec un `email` privé. Le constructeur reçoit l'email, `getEmail()` le retourne et `setEmail()` refuse tout email invalide (utilise `filter_var` avec `FILTER_VALIDATE_EMAIL`) en levant une `InvalidArgumentException`.

### Correction

```php
<?php

declare(strict_types=1);

class Utilisateur
{
    private string $email;

    public function __construct(string $email)
    {
        $this->setEmail($email);
    }

    public function getEmail(): string
    {
        return $this->email;
    }

    public function setEmail(string $email): void
    {
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException("Email invalide : $email");
        }

        $this->email = $email;
    }
}

$user = new Utilisateur("alice@example.com");
echo $user->getEmail() . "\n";

try {
    $user->setEmail("pas-un-email");
} catch (InvalidArgumentException $e) {
    echo "Erreur : " . $e->getMessage() . "\n";
}
```

---

## 8. `static`

### Notion

Une propriété ou une méthode **`static`** appartient à la **classe** elle-même, et non à un objet particulier. On l'appelle avec `::` sans créer d'objet.

Trois usages courants :

* une **constante de classe** (`const`) : une valeur fixe liée à la classe ;
* un **compteur ou une valeur partagée** entre tous les objets ;
* une **méthode de création nommée** (`fromArray()`), très utilisée pour construire un objet à partir de données (base de données, JSON…).

À l'intérieur de la classe, on accède aux éléments statiques avec `self::`.

### Exemple

```php
<?php

declare(strict_types=1);

class Produit
{
    public const TVA = 20.0;              // constante de classe
    private static int $nombreCrees = 0;  // partagée par tous les objets

    public function __construct(
        public string $nom,
        public float $prix,
    ) {
        self::$nombreCrees++;
    }

    public static function fromArray(array $data): self
    {
        return new self($data["nom"], (float) $data["prix"]);
    }

    public static function nombreCrees(): int
    {
        return self::$nombreCrees;
    }

    public function prixTtc(): float
    {
        return $this->prix * (1 + self::TVA / 100);
    }
}

$stylo = new Produit("Stylo", 2.0);
$cahier = Produit::fromArray(["nom" => "Cahier", "prix" => "3.5"]);

echo Produit::nombreCrees() . " produits créés\n";                          // 2
echo "TVA : " . Produit::TVA . " %\n";                                       // 20 %
echo $cahier->nom . " TTC : " . number_format($cahier->prixTtc(), 2) . " €\n";   // 4.20 €
```

### Explication

* `Produit::TVA` et `Produit::nombreCrees()` s'appellent **sans objet**, directement sur la classe.
* `self::$nombreCrees` (avec le `$`) désigne la propriété statique ; `self::TVA` (sans `$`) la constante.
* `fromArray()` est une **fabrique** : elle convertit les données brutes (ici `"3.5"` en `float`) puis retourne un nouvel objet avec `new self(...)`. `self` désigne la classe courante.
* **Attention :** n'abuse pas de `static`. Une méthode statique ne peut pas utiliser `$this`, et un état partagé rend le code plus difficile à comprendre et à tester. Réserve-le aux constantes, aux fabriques et aux petits utilitaires.

### Petit exercice

Crée une classe `Temperature` avec une méthode statique `celsiusVersFahrenheit(float $c): float` (formule : `C × 9/5 + 32`) et une constante `ZERO_ABSOLU = -273.15`. Affiche la conversion de 20 °C.

### Correction

```php
<?php

declare(strict_types=1);

class Temperature
{
    public const ZERO_ABSOLU = -273.15;

    public static function celsiusVersFahrenheit(float $c): float
    {
        return $c * 9 / 5 + 32;
    }
}

echo "20 °C = " . Temperature::celsiusVersFahrenheit(20) . " °F\n";   // 68
echo "Zéro absolu : " . Temperature::ZERO_ABSOLU . " °C\n";
```

---

## 9. Héritage

### Notion

L'**héritage** permet à une classe (la **classe fille**) de reprendre les propriétés et méthodes d'une autre (la **classe mère**) avec `extends`, et d'y ajouter ou de **redéfinir** ce qui diffère.

* `protected` : la classe fille peut accéder à l'élément.
* `parent::` : appeler la version de la classe mère.
* `final` devant une classe : interdit d'en hériter.

Sers-toi de l'héritage quand une relation « **est un** » existe vraiment (un `Chien` *est un* `Animal`). Sinon, préfère assembler des objets entre eux (un `Panier` *contient* des `Produit`).

### Exemple

```php
<?php

declare(strict_types=1);

class Animal
{
    public function __construct(protected string $nom)
    {
    }

    public function presenter(): string
    {
        return "Je m'appelle {$this->nom}.";
    }

    public function parler(): string
    {
        return "{$this->nom} fait un bruit.";
    }
}

class Chien extends Animal
{
    public function parler(): string
    {
        return "{$this->nom} dit : Wouf !";
    }
}

class Chat extends Animal
{
    public function __construct(string $nom, private string $couleur)
    {
        parent::__construct($nom);
    }

    public function presenter(): string
    {
        return parent::presenter() . " Je suis un chat {$this->couleur}.";
    }

    public function parler(): string
    {
        return "{$this->nom} dit : Miaou !";
    }
}

$animaux = [
    new Chien("Rex"),
    new Chat("Mila", "noir"),
    new Animal("Bob"),
];

foreach ($animaux as $animal) {
    echo $animal->presenter() . " " . $animal->parler() . "\n";
}
```

Résultat :

```
Je m'appelle Rex. Rex dit : Wouf !
Je m'appelle Mila. Je suis un chat noir. Mila dit : Miaou !
Je m'appelle Bob. Bob fait un bruit.
```

### Explication

* `class Chien extends Animal` : `Chien` dispose de `presenter()` et `parler()` sans les réécrire.
* `Chien` **redéfinit** `parler()` pour son comportement propre.
* `Chat` a son propre constructeur : il doit appeler `parent::__construct($nom)` pour que la classe mère initialise `$nom`.
* `Chat::presenter()` **complète** la version de la mère avec `parent::presenter()`.
* `$nom` est `protected` : accessible dans les filles, mais pas depuis l'extérieur (`$rex->nom` lèverait une erreur).
* Dans la boucle, chaque objet réagit selon **sa propre classe** : on appelle `parler()` sans se soucier du type exact. C'est le **polymorphisme**.

### Petit exercice

Crée une classe `Personne` (`nom`, `prenom` en `protected`) avec une méthode `nomComplet()`. Crée `Employe`, qui en hérite et ajoute un `salaire`, avec une méthode `fiche()` qui affiche le nom complet et le salaire.

### Correction

```php
<?php

declare(strict_types=1);

class Personne
{
    public function __construct(
        protected string $nom,
        protected string $prenom,
    ) {
    }

    public function nomComplet(): string
    {
        return "{$this->prenom} {$this->nom}";
    }
}

class Employe extends Personne
{
    public function __construct(
        string $nom,
        string $prenom,
        private float $salaire,
    ) {
        parent::__construct($nom, $prenom);
    }

    public function fiche(): string
    {
        return $this->nomComplet() . " - salaire : " . number_format($this->salaire, 2, ",", " ") . " €";
    }
}

$employe = new Employe("Durand", "Marie", 2500);
echo $employe->fiche() . "\n";   // Marie Durand - salaire : 2 500,00 €
```

---

## 10. Interfaces

### Notion

Une **interface** est un **contrat** : elle liste les méthodes qu'une classe doit obligatoirement proposer, **sans dire comment elles fonctionnent**. Une classe qui `implements` l'interface s'engage à respecter ce contrat.

Pourquoi c'est très utile dans un vrai projet : ton code dépend du **contrat** et non d'une implémentation précise. Tu peux donc changer l'implémentation (envoyer par email, par SMS ; stocker dans un fichier JSON, dans MySQL…) **sans toucher au reste du code**.

### Exemple

```php
<?php

declare(strict_types=1);

interface Notifier
{
    public function envoyer(string $message): void;
}

class EmailNotifier implements Notifier
{
    public function __construct(private string $adresse)
    {
    }

    public function envoyer(string $message): void
    {
        echo "[Email à {$this->adresse}] $message\n";
    }
}

class SmsNotifier implements Notifier
{
    public function __construct(private string $numero)
    {
    }

    public function envoyer(string $message): void
    {
        echo "[SMS au {$this->numero}] $message\n";
    }
}

// Cette fonction accepte N'IMPORTE QUEL objet respectant le contrat Notifier
function alerter(Notifier $notifier, string $message): void
{
    $notifier->envoyer($message);
}

alerter(new EmailNotifier("alice@example.com"), "Ta commande est expédiée.");
alerter(new SmsNotifier("06 12 34 56 78"), "Ta commande est expédiée.");
```

Résultat :

```
[Email à alice@example.com] Ta commande est expédiée.
[SMS au 06 12 34 56 78] Ta commande est expédiée.
```

### Explication

* `interface Notifier` déclare seulement la **signature** de `envoyer()` : pas de corps, pas de propriétés.
* `implements Notifier` oblige la classe à définir `envoyer()` avec la même signature. Sinon, PHP renvoie une erreur fatale.
* `alerter(Notifier $notifier, ...)` accepte tout objet `Notifier`. Demain, tu ajoutes `SlackNotifier` sans modifier `alerter()`.
* Une classe peut implémenter plusieurs interfaces (`implements A, B`), alors qu'elle ne peut hériter que d'une seule classe.
* Par convention, le nom d'une interface décrit un rôle (`Notifier`) ou se termine par `Interface` (`ProduitRepositoryInterface`). Tu verras les deux dans les projets.

### Petit exercice

Crée une interface `Forme` avec une méthode `aire(): float`. Implémente-la dans `Cercle` (rayon) et `Carre` (côté). Mets les deux dans un tableau et affiche l'aire de chacune.

### Correction

```php
<?php

declare(strict_types=1);

interface Forme
{
    public function aire(): float;
}

class Cercle implements Forme
{
    public function __construct(private float $rayon)
    {
    }

    public function aire(): float
    {
        return M_PI * $this->rayon ** 2;
    }
}

class Carre implements Forme
{
    public function __construct(private float $cote)
    {
    }

    public function aire(): float
    {
        return $this->cote ** 2;
    }
}

$formes = [new Cercle(2), new Carre(3)];

foreach ($formes as $forme) {
    echo $forme::class . " : " . round($forme->aire(), 2) . "\n";
}
// Cercle : 12.57
// Carre : 9
```

---

## 11. `namespace`

### Notion

Dans un projet, les classes se multiplient, et deux classes peuvent avoir le même nom (une classe `Logger` à toi et une dans une bibliothèque). Un **namespace** (espace de noms) est comme un **dossier** qui organise et distingue les classes.

Le nom complet d'une classe = namespace + nom : `App\Models\Utilisateur`.

Convention universelle (PSR-4) : **le namespace reflète l'arborescence des dossiers**, avec **une classe par fichier** et un fichier qui porte le nom de la classe.

### Exemple

Structure :

```
demo-namespace/
├── src/
│   ├── Models/
│   │   └── Utilisateur.php
│   └── Services/
│       └── Mailer.php
└── index.php
```

`src/Models/Utilisateur.php` :

```php
<?php

declare(strict_types=1);

namespace App\Models;

class Utilisateur
{
    public function __construct(
        public string $nom,
        public string $email,
    ) {
    }
}
```

`src/Services/Mailer.php` :

```php
<?php

declare(strict_types=1);

namespace App\Services;

class Mailer
{
    public function envoyerBienvenue(\App\Models\Utilisateur $utilisateur): string
    {
        return "Mail de bienvenue envoyé à {$utilisateur->email}";
    }
}
```

`index.php` :

```php
<?php

declare(strict_types=1);

require __DIR__ . "/src/Models/Utilisateur.php";
require __DIR__ . "/src/Services/Mailer.php";

$user = new \App\Models\Utilisateur("Alice", "alice@example.com");
$mailer = new \App\Services\Mailer();

echo $mailer->envoyerBienvenue($user) . "\n";
echo $user::class . "\n";   // App\Models\Utilisateur
```

Lance : `php index.php`.

### Explication

* `namespace App\Models;` se place **juste après** `declare(strict_types=1);` (et avant tout autre code). Toutes les classes du fichier appartiennent à ce namespace.
* Le **`\` initial** (`\App\Models\Utilisateur`) indique un nom **complet**, en partant de la racine.
* `$user::class` (ou `Utilisateur::class`) retourne le nom complet de la classe sous forme de texte.
* **Piège classique :** dans un fichier qui a un namespace, une classe native de PHP comme `Exception` ou `InvalidArgumentException` est cherchée *dans le namespace courant* et n'est pas trouvée. Écris `\InvalidArgumentException` ou importe-la avec `use` (chapitre suivant).
* Le namespace ne change pas l'emplacement du fichier : c'est à toi (ou à l'autoload) de charger le bon fichier.

### Petit exercice

Crée une classe `Texte` dans le namespace `App\Utils` avec une méthode statique `majuscule(string $t): string`. Utilise-la depuis un `index.php` avec son nom complet.

### Correction

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

require __DIR__ . "/src/Utils/Texte.php";

echo \App\Utils\Texte::majuscule("bonjour") . "\n";   // BONJOUR
```

---

## 12. `use` (et l'autoload)

### Notion

Écrire le nom complet partout est pénible. Le mot-clé **`use`** **importe** une classe : tu peux ensuite l'appeler par son nom court.

Et comme multiplier les `require` devient ingérable, on utilise l'**autoload** : PHP charge automatiquement le bon fichier **au moment où tu utilises une classe**, en déduisant son chemin depuis son namespace.

### Exemple

Reprenons le projet du chapitre précédent et ajoutons un fichier `autoload.php` :

```
demo-namespace/
├── autoload.php
├── index.php
└── src/
    ├── Models/Utilisateur.php
    └── Services/Mailer.php
```

`autoload.php` :

```php
<?php

declare(strict_types=1);

spl_autoload_register(static function (string $class): void {
    $prefix = "App\\";

    // On ne gère que les classes du namespace App\
    if (!str_starts_with($class, $prefix)) {
        return;
    }

    // App\Models\Utilisateur -> src/Models/Utilisateur.php
    $relative = substr($class, strlen($prefix));
    $file = __DIR__ . "/src/" . str_replace("\\", "/", $relative) . ".php";

    if (is_file($file)) {
        require $file;
    }
});
```

`src/Services/Mailer.php` (avec `use`) :

```php
<?php

declare(strict_types=1);

namespace App\Services;

use App\Models\Utilisateur;

class Mailer
{
    public function envoyerBienvenue(Utilisateur $utilisateur): string
    {
        return "Mail de bienvenue envoyé à {$utilisateur->email}";
    }
}
```

`index.php` :

```php
<?php

declare(strict_types=1);

use App\Models\Utilisateur;
use App\Services\Mailer;

require __DIR__ . "/autoload.php";

$user = new Utilisateur("Alice", "alice@example.com");
$mailer = new Mailer();

echo $mailer->envoyerBienvenue($user) . "\n";
```

Autres formes utiles de `use` :

```php
use App\Models\Utilisateur as User;          // alias (utile en cas de conflit de noms)
use App\Models\{Utilisateur, Produit};       // import groupé
use InvalidArgumentException;                // classe native de PHP dans un fichier avec namespace
```

### Explication

* `use App\Models\Utilisateur;` crée un raccourci : `Utilisateur` désigne maintenant `App\Models\Utilisateur`. `use` **n'inclut pas** de fichier, il ne fait que donner un nom court.
* L'ordre dans un fichier est toujours : `declare` → `namespace` → `use` → le code.
* `spl_autoload_register()` enregistre une fonction appelée par PHP dès qu'une classe inconnue est utilisée. Elle transforme `App\Models\Utilisateur` en `src/Models/Utilisateur.php` et charge le fichier.
* Il suffit d'un seul `require "autoload.php"` au point d'entrée. Les classes se chargent ensuite toutes seules, à condition de **respecter la règle : namespace = dossiers, fichier = nom de la classe**.
* Dans un vrai projet, **Composer** génère cet autoload à ta place (voir « Et maintenant ? »), selon exactement le même principe.

### Petit exercice

Dans un nouveau projet, crée :

* `App\Models\Livre` (titre, auteur) ;
* `App\Services\Bibliotheque`, qui stocke une liste de livres avec `ajouter(Livre $livre)` et `lister(): array` ;
* un `index.php` qui utilise `use` et l'autoload.

### Correction

Structure :

```
bibliotheque/
├── autoload.php      (le même que ci-dessus)
├── index.php
└── src/
    ├── Models/Livre.php
    └── Services/Bibliotheque.php
```

`src/Models/Livre.php` :

```php
<?php

declare(strict_types=1);

namespace App\Models;

class Livre
{
    public function __construct(
        public string $titre,
        public string $auteur,
    ) {
    }
}
```

`src/Services/Bibliotheque.php` :

```php
<?php

declare(strict_types=1);

namespace App\Services;

use App\Models\Livre;

class Bibliotheque
{
    /** @var Livre[] */
    private array $livres = [];

    public function ajouter(Livre $livre): void
    {
        $this->livres[] = $livre;
    }

    /** @return Livre[] */
    public function lister(): array
    {
        return $this->livres;
    }
}
```

`index.php` :

```php
<?php

declare(strict_types=1);

use App\Models\Livre;
use App\Services\Bibliotheque;

require __DIR__ . "/autoload.php";

$bibliotheque = new Bibliotheque();
$bibliotheque->ajouter(new Livre("Le Petit Prince", "Saint-Exupéry"));
$bibliotheque->ajouter(new Livre("Germinal", "Zola"));

foreach ($bibliotheque->lister() as $livre) {
    echo "{$livre->titre} - {$livre->auteur}\n";
}
```

---

## 13. Mini-projet : gestion de produits

Tu vas assembler **tout** ce que tu viens d'apprendre dans une petite application web : ajouter, lister, modifier le prix et supprimer des produits, avec stockage dans un fichier JSON.

### Structure des fichiers

```
boutique/
├── autoload.php
├── data/
│   └── produits.json                  (créé automatiquement)
├── public/
│   └── index.php                      (point d'entrée : le contrôleur)
├── src/
│   ├── Entity/
│   │   └── Produit.php                (un produit + ses règles)
│   ├── Repository/
│   │   ├── ProduitRepositoryInterface.php   (le contrat de stockage)
│   │   └── JsonProduitRepository.php        (stockage dans un fichier JSON)
│   └── Service/
│       └── CatalogueService.php       (calculs sur le catalogue)
└── templates/
    └── produits.php                   (l'affichage HTML)
```

**Pourquoi cette organisation ?**

| Élément | Rôle | Notion du cours |
|---|---|---|
| `public/` | Seul dossier accessible depuis le navigateur | Sécurité : `data/` et `src/` restent inaccessibles |
| `Entity\Produit` | Représente un produit, valide ses données | Classes, propriétés privées, setters, `readonly`, `static` |
| `ProduitRepositoryInterface` | Décrit **ce qu'on peut faire** avec les produits | Interface |
| `JsonProduitRepository` | Décrit **comment** on le fait (JSON) | `implements`, méthodes privées |
| `CatalogueService` | Calculs métier, dépend du contrat, pas du JSON | Constructeur, interface |
| `templates/` | HTML sans logique métier | Séparation affichage / logique |
| `autoload.php` | Charge les classes automatiquement | `namespace`, `use`, autoload |

Crée d'abord les dossiers `public`, `src/Entity`, `src/Repository`, `src/Service`, `templates` et `data`.

### Étape 1 : l'autoload

`autoload.php`

```php
<?php

declare(strict_types=1);

spl_autoload_register(static function (string $class): void {
    $prefix = "App\\";

    if (!str_starts_with($class, $prefix)) {
        return;
    }

    $relative = substr($class, strlen($prefix));
    $file = __DIR__ . "/src/" . str_replace("\\", "/", $relative) . ".php";

    if (is_file($file)) {
        require $file;
    }
});
```

### Étape 2 : l'entité `Produit`

`src/Entity/Produit.php`

```php
<?php

declare(strict_types=1);

namespace App\Entity;

use InvalidArgumentException;

final class Produit
{
    private string $nom;
    private float $prix;
    private int $stock;

    public function __construct(
        private readonly int $id,
        string $nom,
        float $prix,
        int $stock = 0,
    ) {
        $this->setNom($nom);
        $this->setPrix($prix);
        $this->setStock($stock);
    }

    public function getId(): int
    {
        return $this->id;
    }

    public function getNom(): string
    {
        return $this->nom;
    }

    public function getPrix(): float
    {
        return $this->prix;
    }

    public function getStock(): int
    {
        return $this->stock;
    }

    public function setNom(string $nom): void
    {
        $nom = trim($nom);

        if ($nom === "") {
            throw new InvalidArgumentException("Le nom est obligatoire.");
        }

        if (mb_strlen($nom) > 100) {
            throw new InvalidArgumentException("Le nom ne doit pas dépasser 100 caractères.");
        }

        $this->nom = $nom;
    }

    public function setPrix(float $prix): void
    {
        if ($prix < 0) {
            throw new InvalidArgumentException("Le prix ne peut pas être négatif.");
        }

        $this->prix = round($prix, 2);
    }

    public function setStock(int $stock): void
    {
        if ($stock < 0) {
            throw new InvalidArgumentException("Le stock ne peut pas être négatif.");
        }

        $this->stock = $stock;
    }

    public function prixTtc(float $tva = 20.0): float
    {
        return round($this->prix * (1 + $tva / 100), 2);
    }

    /** Convertit l'objet en tableau (pour l'enregistrer en JSON). */
    public function toArray(): array
    {
        return [
            "id" => $this->id,
            "nom" => $this->nom,
            "prix" => $this->prix,
            "stock" => $this->stock,
        ];
    }

    /** Reconstruit un objet à partir d'un tableau (lu depuis le JSON). */
    public static function fromArray(array $data): self
    {
        return new self(
            (int) $data["id"],
            (string) $data["nom"],
            (float) $data["prix"],
            (int) ($data["stock"] ?? 0),
        );
    }
}
```

**À noter :**

* Les trois propriétés validées sont `private` et passent par leurs **setters** : impossible d'avoir un produit sans nom ou avec un prix négatif.
* `id` est `readonly` : il ne change jamais après la création.
* `toArray()` et `fromArray()` (méthode `static`) font le pont entre objet et données brutes.
* `use InvalidArgumentException;` est nécessaire, car on est dans un namespace.

### Étape 3 : le contrat de stockage (interface)

`src/Repository/ProduitRepositoryInterface.php`

```php
<?php

declare(strict_types=1);

namespace App\Repository;

use App\Entity\Produit;

interface ProduitRepositoryInterface
{
    /** @return Produit[] */
    public function trouverTous(): array;

    public function trouver(int $id): ?Produit;

    public function enregistrer(Produit $produit): void;

    public function supprimer(int $id): void;

    public function prochainId(): int;
}
```

### Étape 4 : le stockage JSON

`src/Repository/JsonProduitRepository.php`

```php
<?php

declare(strict_types=1);

namespace App\Repository;

use App\Entity\Produit;

final class JsonProduitRepository implements ProduitRepositoryInterface
{
    public function __construct(private string $fichier)
    {
    }

    public function trouverTous(): array
    {
        return array_map(
            static fn (array $ligne): Produit => Produit::fromArray($ligne),
            $this->lire(),
        );
    }

    public function trouver(int $id): ?Produit
    {
        foreach ($this->trouverTous() as $produit) {
            if ($produit->getId() === $id) {
                return $produit;
            }
        }

        return null;
    }

    public function enregistrer(Produit $produit): void
    {
        $lignes = $this->lire();
        $existe = false;

        foreach ($lignes as $index => $ligne) {
            if ($ligne["id"] === $produit->getId()) {
                $lignes[$index] = $produit->toArray();   // mise à jour
                $existe = true;
                break;
            }
        }

        if (!$existe) {
            $lignes[] = $produit->toArray();             // nouveau produit
        }

        $this->ecrire($lignes);
    }

    public function supprimer(int $id): void
    {
        $lignes = array_filter(
            $this->lire(),
            static fn (array $ligne): bool => $ligne["id"] !== $id,
        );

        $this->ecrire(array_values($lignes));
    }

    public function prochainId(): int
    {
        $ids = array_column($this->lire(), "id");

        return $ids === [] ? 1 : max($ids) + 1;
    }

    private function lire(): array
    {
        if (!is_file($this->fichier)) {
            return [];
        }

        return json_decode((string) file_get_contents($this->fichier), true) ?? [];
    }

    private function ecrire(array $lignes): void
    {
        file_put_contents(
            $this->fichier,
            json_encode($lignes, JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE),
            LOCK_EX,
        );
    }
}
```

**À noter :**

* `implements ProduitRepositoryInterface` : la classe respecte le contrat.
* `lire()` et `ecrire()` sont `private` : ce sont des détails internes, le reste de l'application ne sait même pas qu'on utilise du JSON.
* Demain, tu pourras écrire un `PdoProduitRepository` (MySQL) qui implémente la **même** interface, sans rien changer d'autre.

### Étape 5 : le service

`src/Service/CatalogueService.php`

```php
<?php

declare(strict_types=1);

namespace App\Service;

use App\Entity\Produit;
use App\Repository\ProduitRepositoryInterface;

final class CatalogueService
{
    public function __construct(private ProduitRepositoryInterface $repository)
    {
    }

    public function valeurDuStock(): float
    {
        $total = 0.0;

        foreach ($this->repository->trouverTous() as $produit) {
            $total += $produit->getPrix() * $produit->getStock();
        }

        return $total;
    }

    /** @return Produit[] */
    public function produitsEnRupture(): array
    {
        return array_values(array_filter(
            $this->repository->trouverTous(),
            static fn (Produit $produit): bool => $produit->getStock() === 0,
        ));
    }
}
```

**À noter :** le service reçoit **l'interface**, pas `JsonProduitRepository`. Il fonctionnera avec n'importe quel stockage.

### Étape 6 : le contrôleur (point d'entrée)

`public/index.php`

```php
<?php

declare(strict_types=1);

use App\Entity\Produit;
use App\Repository\JsonProduitRepository;
use App\Service\CatalogueService;

require __DIR__ . "/../autoload.php";

// Pour changer de stockage, il suffirait de changer cette seule ligne
$repository = new JsonProduitRepository(__DIR__ . "/../data/produits.json");
$catalogue = new CatalogueService($repository);

$erreur = null;

if ($_SERVER["REQUEST_METHOD"] === "POST") {
    $action = $_POST["action"] ?? "";

    try {
        if ($action === "ajouter") {
            $produit = new Produit(
                $repository->prochainId(),
                (string) ($_POST["nom"] ?? ""),
                (float) ($_POST["prix"] ?? 0),
                (int) ($_POST["stock"] ?? 0),
            );
            $repository->enregistrer($produit);
        } elseif ($action === "prix") {
            $produit = $repository->trouver((int) ($_POST["id"] ?? 0));

            if ($produit === null) {
                throw new InvalidArgumentException("Produit introuvable.");
            }

            $produit->setPrix((float) ($_POST["prix"] ?? 0));
            $repository->enregistrer($produit);
        } elseif ($action === "supprimer") {
            $repository->supprimer((int) ($_POST["id"] ?? 0));
        }

        header("Location: index.php");
        exit;
    } catch (InvalidArgumentException $e) {
        $erreur = $e->getMessage();
    }
}

$produits = $repository->trouverTous();
$valeurStock = $catalogue->valeurDuStock();
$nbRuptures = count($catalogue->produitsEnRupture());

require __DIR__ . "/../templates/produits.php";
```

**À noter :**

* Le contrôleur ne contient **aucune règle de validation** : elles sont dans `Produit`. Il attrape simplement l'`InvalidArgumentException` et affiche son message.
* Comme au chapitre 6 du cours précédent, on redirige après un POST réussi.
* `index.php` est dans l'espace de noms global : `InvalidArgumentException` s'écrit donc sans `use`.

### Étape 7 : la vue

`templates/produits.php`

```php
<?php

/**
 * Variables fournies par le contrôleur :
 * @var App\Entity\Produit[] $produits
 * @var float                $valeurStock
 * @var int                  $nbRuptures
 * @var string|null          $erreur
 */

$e = static fn (string $texte): string => htmlspecialchars($texte, ENT_QUOTES, "UTF-8");
$euros = static fn (float $montant): string => number_format($montant, 2, ",", " ") . " €";
?>
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Gestion des produits</title>
    <style>
        body { font-family: system-ui, sans-serif; max-width: 860px; margin: 2rem auto; padding: 0 1rem; }
        table { border-collapse: collapse; width: 100%; }
        th, td { border-bottom: 1px solid #ddd; padding: 8px; text-align: left; }
        .erreur { background: #fee2e2; color: #b91c1c; padding: 8px; border-radius: 4px; }
        .rupture { color: #b91c1c; font-weight: bold; }
        form.inline { display: inline; }
        input[type="number"] { width: 90px; }
    </style>
</head>
<body>
    <h1>Gestion des produits</h1>

    <?php if ($erreur !== null): ?>
        <p class="erreur"><?= $e($erreur) ?></p>
    <?php endif; ?>

    <h2>Ajouter un produit</h2>
    <form method="post">
        <input type="hidden" name="action" value="ajouter">
        <input type="text" name="nom" placeholder="Nom" required>
        <input type="number" name="prix" placeholder="Prix" step="0.01" min="0" required>
        <input type="number" name="stock" placeholder="Stock" min="0" value="0">
        <button type="submit">Ajouter</button>
    </form>

    <h2>Catalogue</h2>
    <p>
        Valeur du stock : <strong><?= $euros($valeurStock) ?></strong>
        — Produits en rupture : <strong><?= $nbRuptures ?></strong>
    </p>

    <?php if ($produits === []): ?>
        <p>Aucun produit pour l'instant.</p>
    <?php else: ?>
        <table>
            <thead>
                <tr>
                    <th>Nom</th>
                    <th>Prix HT</th>
                    <th>Prix TTC</th>
                    <th>Stock</th>
                    <th>Actions</th>
                </tr>
            </thead>
            <tbody>
                <?php foreach ($produits as $produit): ?>
                    <tr>
                        <td><?= $e($produit->getNom()) ?></td>
                        <td><?= $euros($produit->getPrix()) ?></td>
                        <td><?= $euros($produit->prixTtc()) ?></td>
                        <td>
                            <?php if ($produit->getStock() === 0): ?>
                                <span class="rupture">Rupture</span>
                            <?php else: ?>
                                <?= $produit->getStock() ?>
                            <?php endif; ?>
                        </td>
                        <td>
                            <form class="inline" method="post">
                                <input type="hidden" name="action" value="prix">
                                <input type="hidden" name="id" value="<?= $produit->getId() ?>">
                                <input type="number" name="prix" step="0.01" min="0"
                                       value="<?= $produit->getPrix() ?>">
                                <button type="submit">Changer le prix</button>
                            </form>
                            <form class="inline" method="post"
                                  onsubmit="return confirm('Supprimer ce produit ?');">
                                <input type="hidden" name="action" value="supprimer">
                                <input type="hidden" name="id" value="<?= $produit->getId() ?>">
                                <button type="submit">Supprimer</button>
                            </form>
                        </td>
                    </tr>
                <?php endforeach; ?>
            </tbody>
        </table>
    <?php endif; ?>
</body>
</html>
```

### Étape 8 : lancer et tester

**Avec le serveur intégré** (depuis le dossier `boutique/`) :

```bash
php -S localhost:8000 -t public
```

Ouvre `http://localhost:8000`. L'option `-t public` indique que seul le dossier `public/` est exposé.

**Avec XAMPP/WAMP :** place le dossier `boutique` dans `htdocs` (ou `www`) et ouvre `http://localhost/boutique/public/`.

**Scénario de test :**

1. Ajoute « Clavier », 49.90 €, stock 12 : il apparaît avec son prix TTC (59,88 €).
2. Ajoute « Souris », 19.90 €, stock 0 : la mention **Rupture** s'affiche, le compteur de ruptures passe à 1.
3. Change le prix du clavier : le tableau et la valeur du stock sont mis à jour.
4. Ajoute un produit dont le nom ne contient que des espaces : le message « Le nom est obligatoire. » s'affiche (la validation vient de la classe `Produit`).
5. Supprime un produit, puis ouvre `data/produits.json` pour voir les données enregistrées.

*Si rien ne s'enregistre, vérifie que le dossier `data/` existe et que PHP peut y écrire.*

### Pour aller plus loin (exercices)

1. Ajoute une action « Modifier le stock » (`setStock()` existe déjà dans `Produit`).
2. Crée `src/Repository/PdoProduitRepository.php` qui implémente `ProduitRepositoryInterface` avec PDO/MySQL, puis change **une seule ligne** dans `public/index.php`. Le reste de l'application ne change pas : c'est l'intérêt de l'interface.
3. Reproduis la même architecture pour des **utilisateurs** : `Entity\Utilisateur` (email validé, mot de passe haché avec `password_hash`), un repository et un service d'inscription.

---

## Récapitulatif

| Notion | En une phrase |
|---|---|
| Classe / objet | Un plan et ses réalisations (`new`) |
| Propriétés | Les données de l'objet (`$this->nom`) |
| Méthodes | Les actions de l'objet |
| `__construct` | Initialise l'objet ; la promotion évite la répétition |
| `private` / `protected` / `public` | Protéger l'état interne ; tout en `private` par défaut |
| Getters / setters | Accès contrôlé, avec validation |
| `static` / `const` | Éléments liés à la classe, pas à un objet |
| Héritage | `extends` pour une vraie relation « est un » |
| Interface | Un contrat ; le code dépend du contrat, pas de l'implémentation |
| `namespace` / `use` | Organiser et importer les classes |
| Autoload | Charge les classes automatiquement (namespace = dossiers) |

---

## Et maintenant ?

1. **Composer et l'autoload PSR-4** : remplace ton `autoload.php` par celui de Composer et installe des bibliothèques.
2. **Exceptions personnalisées** : crée tes propres classes d'erreur (`ProduitIntrouvableException`).
3. **Énumérations (`enum`)** : représente proprement un statut (`Statut::Actif`, `Statut::Inactif`).
4. **Injection de dépendances** : passer les objets dont une classe a besoin par son constructeur, comme tu l'as fait avec `CatalogueService`.
5. **Tests unitaires** (PHPUnit ou Pest) : les classes bien découpées se testent facilement.
6. **MVC complet, puis un framework** (Laravel ou Symfony) : tu retrouveras partout ce que tu viens d'apprendre (entités, repositories, services, interfaces, namespaces).
