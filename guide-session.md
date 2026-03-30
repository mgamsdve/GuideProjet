# Guide session - inscription, connexion et utilisateur connecté

Ce document sert de base simple pour créer un petit projet MVC avec :
- inscription
- connexion
- session utilisateur
- déconnexion
- affichage conditionnel selon l'état de connexion

L'idée est d'avoir un projet propre et minimal, que tu peux copier puis adapter.

---

## 1. Structure minimale du projet

Fichiers utiles :
- `index.php`
- `Config/connectDatabase.php`
- `Controllers/userController.php`
- `Models/userModel.php`
- `Views/base.php`
- `Views/Components/header.php`
- `Views/Components/footer.php`
- `Views/users/pageInscription.php`
- `Views/users/pageConnexion.php`
- `Views/users/pageProfil.php`

Arborescence conseillée :

```text
Projet/
├── index.php
├── Config/
│   └── connectDatabase.php
├── Controllers/
│   └── userController.php
├── Models/
│   └── userModel.php
├── Views/
│   ├── base.php
│   ├── Components/
│   │   ├── header.php
│   │   └── footer.php
│   └── users/
│       ├── pageInscription.php
│       ├── pageConnexion.php
│       └── pageProfil.php
```

---

## 2. Base de données minimale

Table `Utilisateur` conseillée :

```sql
CREATE TABLE Utilisateur (
    id_utilisateur INT AUTO_INCREMENT PRIMARY KEY,
    uti_nom VARCHAR(100) NOT NULL,
    uti_prenom VARCHAR(100) NOT NULL,
    uti_email VARCHAR(255) NOT NULL UNIQUE,
    uti_mdp VARCHAR(255) NOT NULL,
    uti_role VARCHAR(50) NOT NULL DEFAULT 'utilisateur'
);
```

Explication rapide :
- `uti_email` doit être unique pour éviter deux comptes avec la même adresse
- `uti_mdp` contient un mot de passe hashé, jamais en clair
- `uti_role` permet de gérer les permissions plus tard

---

## 3. Connexion à la base de données

### Fichier : `Config/connectDatabase.php`

```php
<?php
try {
    $pdo = new PDO(
        "mysql:host=localhost;dbname=nom_de_ta_base;port=3306",
        "root",
        "",
        [
            PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
            PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_OBJ
        ]
    );
} catch (PDOException $e) {
    die($e->getMessage());
}
```

Ce fichier sert à créer la connexion à la base avec `$pdo`.

---

## 4. Point d'entrée du projet

### Fichier : `index.php`

```php
<?php
session_start();

require_once("Config/connectDatabase.php");
require_once("Controllers/userController.php");
```

Pourquoi `session_start()` ?
- pour pouvoir utiliser `$_SESSION`
- sans ça, la session ne fonctionne pas

---

## 5. Layout principal

### Fichier : `Views/base.php`

```php
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title><?= $title ?></title>
</head>
<body>
    <header>
        <?php require_once("Views/Components/header.php"); ?>
    </header>

    <main>
        <?php require_once($template); ?>
    </main>

    <footer>
        <?php require_once("Views/Components/footer.php"); ?>
    </footer>
</body>
</html>
```

Rôle de `base.php` :
- structure commune à toutes les pages
- header, contenu central, footer

---

## 6. Navbar / header dynamique

### Fichier : `Views/Components/header.php`

```php
<nav>
    <a href="/">Accueil</a>

    <?php if (isset($_SESSION["user"])) : ?>
        <a href="/Profil">Mon profil</a>
        <a href="/deconnexion">Déconnexion</a>
        <span>
            Bonjour <?= htmlspecialchars($_SESSION["user"]->uti_prenom) ?>
        </span>
    <?php else : ?>
        <a href="/connexion">Connexion</a>
        <a href="/inscription">Inscription</a>
    <?php endif; ?>
</nav>
```

Comment ça marche :
- si personne n'est connecté, on affiche les liens de base
- si un utilisateur est connecté, on affiche son prénom et la déconnexion

---

## 7. Le modèle utilisateur

### Fichier : `Models/userModel.php`

```php
<?php

function selectUserByEmail($pdo, $email)
{
    try {
        $query = 'SELECT * FROM Utilisateur WHERE uti_email = :email';
        $stmt = $pdo->prepare($query);
        $stmt->execute([
            "email" => $email
        ]);
        return $stmt->fetch();
    } catch (PDOException $e) {
        die($e->getMessage());
    }
}

function insertUser($pdo)
{
    try {
        $query = 'INSERT INTO Utilisateur (uti_nom, uti_prenom, uti_email, uti_mdp, uti_role)
                  VALUES (:nom, :prenom, :email, :mdp, :role)';
        $stmt = $pdo->prepare($query);
        $stmt->execute([
            "nom" => $_POST["nom"],
            "prenom" => $_POST["prenom"],
            "email" => $_POST["email"],
            "mdp" => password_hash($_POST["mdp"], PASSWORD_DEFAULT),
            "role" => "utilisateur"
        ]);
    } catch (PDOException $e) {
        die($e->getMessage());
    }
}

function selectUserByEmailAndPassword($pdo)
{
    try {
        $query = 'SELECT * FROM Utilisateur WHERE uti_email = :email';
        $stmt = $pdo->prepare($query);
        $stmt->execute([
            "email" => $_POST["email"]
        ]);
        $user = $stmt->fetch();

        if ($user && password_verify($_POST["mdp"], $user->uti_mdp)) {
            return $user;
        }

        return false;
    } catch (PDOException $e) {
        die($e->getMessage());
    }
}

function updateUser($pdo)
{
    try {
        $query = 'UPDATE Utilisateur
                  SET uti_nom = :nom, uti_prenom = :prenom, uti_email = :email
                  WHERE id_utilisateur = :idUser';
        $stmt = $pdo->prepare($query);
        $stmt->execute([
            "nom" => $_POST["nom"],
            "prenom" => $_POST["prenom"],
            "email" => $_POST["email"],
            "idUser" => $_SESSION["user"]->id_utilisateur
        ]);
    } catch (PDOException $e) {
        die($e->getMessage());
    }
}
```

Points importants :
- toujours utiliser `prepare()`
- toujours utiliser `password_hash()` pour l'inscription
- toujours utiliser `password_verify()` pour la connexion

---

## 8. Le contrôleur utilisateur

### Fichier : `Controllers/userController.php`

```php
<?php
$uri = $_SERVER["REQUEST_URI"];
require_once("Models/userModel.php");

if ($uri == "/inscription") {
    $messageError = null;

    if (isset($_POST["envoyer"])) {
        if (
            empty($_POST["nom"]) ||
            empty($_POST["prenom"]) ||
            empty($_POST["email"]) ||
            empty($_POST["mdp"])
        ) {
            $messageError = "Tous les champs sont obligatoires.";
        } else {
            $userExist = selectUserByEmail($pdo, $_POST["email"]);

            if ($userExist === false) {
                insertUser($pdo);
                header("location:/connexion");
                exit;
            } else {
                $messageError = "Cette adresse email est déjà utilisée.";
            }
        }
    }

    $title = "Inscription";
    $template = "Views/users/pageInscription.php";
}
elseif ($uri == "/connexion") {
    $message = null;

    if (isset($_POST["envoyer"])) {
        $user = selectUserByEmailAndPassword($pdo);

        if ($user) {
            $_SESSION["user"] = $user;
            header("location:/");
            exit;
        } else {
            $message = "Email ou mot de passe incorrect.";
        }
    }

    $title = "Connexion";
    $template = "Views/users/pageConnexion.php";
}
elseif ($uri == "/Profil") {
    if (!isset($_SESSION["user"])) {
        header("location:/connexion");
        exit;
    }

    $erreur = null;

    if (isset($_POST["envoyer"])) {
        if (
            empty($_POST["nom"]) ||
            empty($_POST["prenom"]) ||
            empty($_POST["email"])
        ) {
            $erreur = "Tous les champs doivent être remplis.";
        } else {
            updateUser($pdo);
            header("location:/Profil");
            exit;
        }
    }

    $title = "Mon Profil";
    $template = "Views/users/pageProfil.php";
}
elseif ($uri == "/deconnexion") {
    session_destroy();
    header("location:/");
    exit;
}

if (isset($template)) {
    require_once("Views/base.php");
}
```

Comment ce contrôleur fonctionne :
- il lit l'URL
- il choisit la page
- il traite les formulaires envoyés en POST
- il protège les pages qui demandent une connexion

---

## 9. Page d'inscription

### Fichier : `Views/users/pageInscription.php`

```php
<h1>Inscription</h1>

<?php if (!empty($messageError)) : ?>
    <p style="color:red;"><?= htmlspecialchars($messageError) ?></p>
<?php endif; ?>

<form method="POST">
    <label for="nom">Nom</label>
    <input type="text" name="nom" id="nom">

    <label for="prenom">Prénom</label>
    <input type="text" name="prenom" id="prenom">

    <label for="email">Email</label>
    <input type="email" name="email" id="email">

    <label for="mdp">Mot de passe</label>
    <input type="password" name="mdp" id="mdp">

    <input type="submit" name="envoyer" value="Créer mon compte">
</form>
```

---

## 10. Page de connexion

### Fichier : `Views/users/pageConnexion.php`

```php
<h1>Connexion</h1>

<?php if (!empty($message)) : ?>
    <p style="color:red;"><?= htmlspecialchars($message) ?></p>
<?php endif; ?>

<form method="POST">
    <label for="email">Email</label>
    <input type="email" name="email" id="email">

    <label for="mdp">Mot de passe</label>
    <input type="password" name="mdp" id="mdp">

    <input type="submit" name="envoyer" value="Se connecter">
</form>
```

---

## 11. Page profil

### Fichier : `Views/users/pageProfil.php`

```php
<h1>Mon profil</h1>

<?php if (!empty($erreur)) : ?>
    <p style="color:red;"><?= htmlspecialchars($erreur) ?></p>
<?php endif; ?>

<form method="POST">
    <label for="nom">Nom</label>
    <input type="text" name="nom" id="nom" value="<?= htmlspecialchars($_SESSION["user"]->uti_nom) ?>">

    <label for="prenom">Prénom</label>
    <input type="text" name="prenom" id="prenom" value="<?= htmlspecialchars($_SESSION["user"]->uti_prenom) ?>">

    <label for="email">Email</label>
    <input type="email" name="email" id="email" value="<?= htmlspecialchars($_SESSION["user"]->uti_email) ?>">

    <input type="submit" name="envoyer" value="Modifier mon profil">
</form>
```

---

## 12. Ce qu'il faut retenir sur la session

Une session sert à :
- savoir si un utilisateur est connecté
- garder ses informations entre les pages
- lier les actions à cet utilisateur

Exemples de code importants :

```php
session_start();
```

```php
$_SESSION["user"] = $user;
```

```php
if (!isset($_SESSION["user"])) {
    header("location:/connexion");
    exit;
}
```

```php
session_destroy();
```

---

## 13. Version encore plus simple du flux

### Inscription
1. L'utilisateur remplit le formulaire.
2. Le contrôleur vérifie les champs.
3. Le mot de passe est hashé.
4. L'utilisateur est inséré en base.
5. Redirection vers la connexion.

### Connexion
1. L'utilisateur envoie email et mot de passe.
2. Le contrôleur cherche l'utilisateur en base.
3. `password_verify()` compare le mot de passe.
4. Si tout est bon, l'utilisateur est mis en session.
5. Redirection vers l'accueil.

### Profil
1. L'utilisateur modifie ses infos.
2. Le contrôleur vérifie qu'il est connecté.
3. Les données sont mises à jour en base.
4. La page se recharge.

---

## 14. Points de sécurité importants

À toujours faire :
- utiliser `prepare()`
- utiliser `password_hash()`
- utiliser `password_verify()`
- vérifier `isset($_SESSION["user"])` avant une page protégée
- rediriger après un POST avec `header("location:...")`

À éviter :
- stocker un mot de passe en clair
- faire des requêtes SQL dans les vues
- laisser une page protégée accessible sans contrôle

---

## 15. Résumé final

Si tu veux juste un mini projet propre avec inscription / connexion / session, il faut surtout :
- une table `Utilisateur`
- un `connectDatabase.php`
- un `userController.php`
- un `userModel.php`
- une base de vue `base.php`
- un `header.php` dynamique
- 3 pages : inscription, connexion, profil

Avec ça, tu as déjà un vrai petit système de comptes utilisateurs.