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

```html
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

```html
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

## 10. Relations entre tables : `SELECT` et "double SELECT" (2 clés étrangères)

Certaines tables ne stockent pas juste des données simples.
Elles servent de **table de liaison** entre plusieurs entités.

Exemples dans ce projet :
- `Reservation` relie une `Salle` et un `Utilisateur`
- `Contenance` relie une `Salle` et un `Equipement`

Concrètement, on enregistre dans une même ligne :
- l'ID de la première table (`id_salle`)
- l'ID de la deuxième table (`id_utilisateur` ou `id_equipement`)
- puis les données métier (dates, quantité, etc.)

Extrait SQL (structure logique d'une table de liaison) :

```sql
CREATE TABLE Reservation (
    id_reservation INT PRIMARY KEY AUTO_INCREMENT,
    id_salle INT NOT NULL,
    id_utilisateur INT NOT NULL,
    res_dateDebut DATE NOT NULL,
    res_dateFin DATE NOT NULL,
    FOREIGN KEY (id_salle) REFERENCES Salle(id_salle),
    FOREIGN KEY (id_utilisateur) REFERENCES Utilisateur(id_utilisateur)
);
```

---

### 10.1 Comment se passe l'`INSERT` avec 2 IDs

Cas `Reservation` :
- L'utilisateur choisit une salle (donc un `id_salle`)
- L'`id_utilisateur` vient de la session
- On ajoute aussi `dateDebut` et `dateFin`

```php
$query = 'INSERT INTO Reservation (id_salle, id_utilisateur, res_dateDebut, res_dateFin)
          VALUES (:idSalle, :idUtilisateur, :dateDebut, :dateFin)';
```

Extrait du modèle (liaison des valeurs au moment de l'insert) :

```php
$insertReservation->execute([
    "idSalle" => $_POST["id_salle"],
    "idUtilisateur" => $_SESSION["user"]->id_utilisateur,
    "dateDebut" => $_POST["dateDebut"],
    "dateFin" => $_POST["dateFin"]
]);
```

Cas `Contenance` :
- L'admin choisit une salle (`id_salle`)
- L'admin choisit un équipement (`id_equipement`)
- Il saisit la quantité

```php
$query = 'INSERT INTO Contenance (id_equipement, id_salle, cont_quantite)
          VALUES (:idEquipement, :idSalle, :quantite)';
```

Extrait vue admin (2 listes déroulantes pour sélectionner les 2 IDs) :

```php
<select name="id_salle" id="contenance_salle">
    <?php foreach ($salles as $salle) : ?>
        <option value="<?= $salle->id_salle ?>"><?= $salle->sal_nom ?></option>
    <?php endforeach ?>
</select>

<select name="id_equipement" id="contenance_equipement">
    <?php foreach ($equipements as $equipement) : ?>
        <option value="<?= $equipement->id_equipement ?>"><?= $equipement->equi_nom ?></option>
    <?php endforeach ?>
</select>
```

---

### 10.2 Pourquoi on parle de "double SELECT"

Pour afficher une ligne compréhensible à l'écran, il faut souvent récupérer les infos de plusieurs tables.

Exemple : pour afficher une réservation, on veut voir :
- le nom de la salle
- le nom/prénom de l'utilisateur
- les dates

Donc on fait un `SELECT` avec `JOIN` :

```sql
SELECT Reservation.*, Salle.sal_nom, Utilisateur.uti_nom, Utilisateur.uti_prenom
FROM Reservation
INNER JOIN Salle ON Salle.id_salle = Reservation.id_salle
INNER JOIN Utilisateur ON Utilisateur.id_utilisateur = Reservation.id_utilisateur
```

Extrait équivalent pour les contenances :

```sql
SELECT Contenance.*, Equipement.equi_nom, Salle.sal_nom, Salle.sal_numero
FROM Contenance
INNER JOIN Equipement ON Equipement.id_equipement = Contenance.id_equipement
INNER JOIN Salle ON Salle.id_salle = Contenance.id_salle
ORDER BY Salle.sal_nom, Equipement.equi_nom;
```

Même logique pour `Contenance` :
- on joint `Contenance` + `Salle` + `Equipement`

Idée clé à retenir :
- les IDs servent au lien technique en base
- les `JOIN` servent à afficher des données lisibles dans l'interface

Extrait contrôleur (chargement des données nécessaires à l'affichage) :

```php
$salles = selectAllSalle($pdo);
$equipements = selectAllEquipements($pdo);
$contenances = selectAllContenances($pdo);
```

---

## 11. Gestion des sessions utilisateur

La session permet de "se souvenir" de l'utilisateur connecté entre les pages.

### 11.1 Démarrage de session

Dans `index.php`, on lance :

```php
session_start();
```

Sans ça, `$_SESSION` n'existe pas.

Extrait réel du point d'entrée :

```php
session_start();
require_once("Config/connectDatabase.php");
require_once("Controllers/userController.php");
```

---

### 11.2 Connexion (login)

Après vérification email + mot de passe, on stocke l'utilisateur en session :

```php
$_SESSION["user"] = $user;
```

Extrait type (contrôleur de connexion) :

```php
$user = selectUserByEmailAndPassword($pdo);
if ($user) {
    $_SESSION["user"] = $user;
    header("location:/");
    exit;
}
```

Ensuite, toutes les pages peuvent savoir :
- qui est connecté
- son rôle
- son id

---

### 11.3 Utilisation de la session dans le projet

Exemples courants :
- vérifier si l'utilisateur est connecté (`isset($_SESSION["user"])`)
- récupérer son ID pour créer une réservation personnelle
- afficher son prénom dans la navbar

Extrait (protéger une page réservée) :

```php
if (!isset($_SESSION["user"])) {
    header("location:/connexion");
    exit;
}
```

Extrait (lier une réservation à l'utilisateur connecté) :

```php
$userId = $_SESSION["user"]->id_utilisateur;
```

Extrait (affichage dans la navbar) :

```php
<span><?= htmlspecialchars($_SESSION["user"]->uti_prenom) ?></span>
```

---

### 11.4 Mise à jour et suppression de session

- Si l'utilisateur modifie son profil, on peut recharger ses données en session (`updateSession`)
- À la déconnexion, on détruit la session :

```php
session_destroy();
```

Extrait (mise à jour de session après modification du profil) :

```php
updateUser($pdo);
updateSession($pdo);
header("location:/Profil");
```

---

## 12. Permissions par rôle et navbar dynamique

Dans ce projet, il y a au minimum 2 rôles :
- `admin`
- `utilisateur`

### 12.1 Contrôle d'accès par rôle

Pour les routes `/admin`, le contrôleur vérifie :
1. Utilisateur connecté
2. Rôle = `admin`

Si une condition échoue : redirection.

Ce contrôle protège les pages même si quelqu'un tape l'URL manuellement.

Extrait réel (contrôle admin côté serveur) :

```php
if (strpos($uri, "/admin") === 0) {
    if (!isset($_SESSION["user"])) {
        header("location:/connexion");
        exit;
    }

    if ($_SESSION["user"]->uti_role != "admin") {
        header("location:/");
        exit;
    }
}
```

---

### 12.2 Navbar qui change selon l'état utilisateur

La navbar est conditionnelle :

Si personne n'est connecté :
- boutons `Connexion` et `Inscription`

Si un utilisateur est connecté :
- liens utilisateur (profil, réservations, etc.)
- avatar + prénom
- bouton `Déconnexion`

Si l'utilisateur connecté est admin :
- menu déroulant `Admin`
- accès vers dashboard et pages de gestion (`users`, `categories`, `salles`, `equipements`, `contenances`, `reservations`)

Extrait navbar (partie invité) :

```php
<?php if (isset($_SESSION["user"])) : ?>
    <!-- contenu utilisateur connecté -->
<?php else : ?>
    <a href="/connexion" class="btn-outline-sm">Connexion</a>
    <a href="/inscription" class="btn-sm">Inscription</a>
<?php endif ?>
```

Extrait navbar (partie admin) :

```php
<?php if ($_SESSION["user"]->uti_role == "admin") : ?>
    <div class="navbar-dropdown-menu">
        <a href="/admin">Dashboard</a>
        <a href="/admin/contenances">Contenances</a>
        <a href="/admin/reservations">Réservations</a>
    </div>
<?php endif ?>
```

---

### 12.3 Pourquoi c'est important

- L'interface (navbar) guide l'utilisateur selon son rôle
- Les contrôleurs appliquent la vraie sécurité côté serveur
- Les deux ensemble donnent une application claire et sécurisée

Extrait récapitulatif (bonne pratique) :

```php
// Vue : masquer/afficher des liens selon la session
// Contrôleur : toujours revérifier les droits avant traitement
```

---
