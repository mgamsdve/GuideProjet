# Guide de démarrage d’un projet web (MVC + BDD)

## 1. Base de données (BDD)

### BDD vide
- Exemple de Nathan
- Être cohérent dans les nommages :
  - Choisir soit le pluriel, soit le singulier pour les entités (mais rester cohérent)
  - Utiliser des préfixes pour les champs (ex : `uti_nom`, `uti_email`, etc.)
  - Nommer correctement les clés étrangères  
    → Exemple : `bat_lieu_id` au lieu de `lieu_id`
- Créer la base de données vide en envoyant le MCD (Modèle Conceptuel de Données) à une IA

---

### BDD avec des données
- Remplir la base avec des données générées par IA (~20 lignes par table)
- Envoyer le schéma de la BDD à l’IA
- Demander des données variées :
  - emails différents
  - mots de passe différents
  - noms/prénoms différents
- Tester la connexion à la BDD :
  - via MySQL Workbench ou autre outil
  - soit en localhost
  - soit via le serveur de l’école

---

## 2. Structure du projet (MVC)

Créer les dossiers principaux (vides) :
- `Assets/`
- `Config/`
- `Controllers/`
- `Models/`
- `Views/`
- `SQL/`

---

## 3. Connexion à la base de données

Créer le fichier : `Config/connectDatabase.php`

```php
<?php
try {
    $pdo = new PDO(
        "mysql:host=10.10.67.227;dbname=Farhan;port=3306",
        "Farhan",
        "root",
        [
            PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
            PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_OBJ
        ]
    );
} catch (PDOException $e) {
    $message = $e->getMessage();
    die($message);
}
```

---

## 4. Point d’entrée du site

Créer le fichier : `index.php`

```php
<?php
session_start();
require_once("Config/connectDatabase.php");
```

---

## 5. Mise en place du MVC (3 éléments en parallèle)

### 5.1 Layout principal (base)

Créer : `Views/base.php`

```php
<!DOCTYPE html>
<html lang="fr">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="../Assets/CSS/style.css">
    <title><?= $title ?></title>
</head>

<body>
    <header class="glass">
        <!-- vide pour l’instant -->
    </header>

    <main class="fade-in">
        <?php require_once($template); ?>
    </main>

    <footer class="fade-in">
        <!-- vide pour l’instant -->
    </footer>
</body>

</html>
```

---

### 5.2 Controller principal

Créer : `Controllers/<tonProjet>Controller.php`

```php
<?php
$uri = $_SERVER["REQUEST_URI"];

if ($uri == "/index.php" || $uri == "/") {
    $template = "Views/tonProjet/pageAccueil.php";
    $title = "Accueil";
}
```

---

### 5.3 Vue (page)

Créer : `Views/tonProjet/pageAccueil.php`

```php
<h1>Bienvenue sur mon projet</h1>
<h2>Accueil</h2>

<!-- Développer ici le contenu HTML de la page d’accueil -->
```

---

### 5.4 Relier le controller à l’index

Modifier `index.php` :

```php
<?php
require_once("Config/connectDatabase.php");
require_once("Controllers/<tonProjet>Controller.php");
```

---

## 6. Lancer le projet

* Lancer `index.php` avec une extension type **PHP Server**
* Vérifier que la page s’affiche :

```
Bienvenue sur mon projet
Accueil
```

---

## 7. Première requête sur la BDD

### 7.1 Créer un Model

Exemple : `Models/carrelageModel.php`

```php
<?php

// ===============================
// SELECT ALL PRODUITS (CARRELAGE)
// ===============================
function selectAllCarrelage($pdo) {
    try {
        $query = "SELECT * FROM produit";
        $stmt = $pdo->prepare($query);
        $stmt->execute();
        return $stmt->fetchAll(PDO::FETCH_OBJ);

    } catch (PDOException $e) {
        die($e->getMessage());
    }
}
```

---

### 7.2 Mettre à jour le Controller

Exemple : `Controllers/carrelageController.php`

```php
<?php
$uri = $_SERVER["REQUEST_URI"];

require_once("Models/carrelageModel.php");

if ($uri === "/index.php" || $uri === "/home") {
    $carrelages = selectAllCarrelage($pdo);
    $template = "Views/Carrelage/pageAccueil.php";
}

require_once("Views/base.php");
```

---

## 8. Résumé du fonctionnement

* `index.php` = point d’entrée
* `Controller` = logique + choix de la vue
* `Model` = accès aux données (BDD)
* `Views` = affichage HTML
* `base.php` = structure globale du site

---

## 9. Conseils importants

* Toujours séparer :

  * logique (Controller)
  * données (Model)
  * affichage (View)
* Ne jamais faire de requêtes SQL directement dans les Views
* Toujours utiliser `prepare()` pour éviter les injections SQL
* Garder une structure claire et cohérente dès le début

---
