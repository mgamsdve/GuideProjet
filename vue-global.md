# Vue globale du projet

## 1. Objectif du projet

Ce projet est une application web MVC de gestion de salles.

Fonctionnalités principales :
- consulter les salles et leurs détails
- consulter les équipements
- créer et gérer des réservations utilisateur
- administrer les données (users, salles, équipements, catégories, contenances, réservations)

---

## 2. Architecture générale

Structure principale :
- Assets : styles, images, scripts front
- Config : connexion à la base de données
- Controllers : logique de routage et traitement
- Models : accès aux données (SQL)
- Views : affichage HTML/PHP
- SQL : scripts de création et d'alimentation de la base

Principe MVC appliqué :
- Controller : lit l'URL, contrôle les droits, choisit la vue, traite les POST
- Model : exécute les requêtes SQL (SELECT/INSERT/UPDATE/DELETE)
- View : affiche les données, envoie les formulaires

---

## 3. Point d'entrée et cycle de requête

Le point d'entrée est index.php.

Cycle d'une requête :
1. session_start démarre la session.
2. La connexion BDD est chargée.
3. Les contrôleurs sont inclus.
4. Chaque contrôleur vérifie s'il doit traiter l'URL.
5. Si un template est défini, Views/base.php est affiché.

Schéma mental simple :
URL -> Controller -> Model (si besoin) -> Template -> Views/base.php

---

## 4. Routage du projet

Routes publiques :
- / ou /index.php : accueil
- /Salle : liste des salles
- /Salle/Details/{numero} : détail d'une salle
- /Equipement : liste des équipements
- /connexion : connexion
- /inscription : inscription

Routes utilisateur connecté :
- /Reservation : mes réservations
- /create-reservation : créer une réservation
- /Profil : modifier son profil
- /deconnexion : se déconnecter

Routes admin (rôle admin obligatoire) :
- /admin
- /admin/users
- /admin/categories
- /admin/salles
- /admin/equipements
- /admin/contenances
- /admin/reservations

---

## 5. Gestion des sessions

La session stocke l'utilisateur connecté dans $_SESSION["user"].

Moments clés :
- connexion réussie : l'objet user est mis en session
- pages protégées : vérification isset($_SESSION["user"])
- actions utilisateur : on lit l'id depuis la session
- mise à jour profil : updateSession pour rafraîchir les données en mémoire
- déconnexion : session_destroy

Pourquoi c'est important :
- la session évite de redemander l'authentification à chaque page
- elle permet de savoir qui agit sur les données

---

## 6. Permissions et rôles

Rôles utilisés :
- utilisateur
- admin

Règle principale :
- toute route commençant par /admin est protégée
- si non connecté : redirection vers /connexion
- si connecté mais non admin : redirection vers /

Sécurité utile à retenir :
- cacher un lien dans la navbar ne suffit pas
- la vraie sécurité doit toujours être validée côté contrôleur

---

## 7. Navbar dynamique selon l'état utilisateur

La navbar adapte son contenu :
- invité : Connexion, Inscription
- connecté : Mes réservations, Profil, Déconnexion, avatar/prénom
- admin connecté : menu Admin supplémentaire (dashboard + CRUD admin)

Impact pédagogique :
- l'interface guide l'utilisateur
- le rôle visible correspond à ses permissions réelles

---

## 8. Flux CRUD utilisé dans tout le projet

Pattern commun (surtout en admin) :
1. La page affiche un formulaire avec method POST et un champ action.
2. Le contrôleur lit $_POST["action"].
3. Selon l'action, il appelle la fonction Model appropriée.
4. Redirection vers la même page (Post/Redirect/Get).
5. Rechargement des données en GET.

Avantage :
- logique claire
- évite le double envoi lors d'un refresh
- structure homogène entre modules

---

## 9. Relations de données importantes

Tables de liaison :
- Reservation : lie Salle + Utilisateur
- Contenance : lie Salle + Equipement

Lecture des données :
- les IDs servent au lien technique
- les JOIN servent à afficher des noms lisibles côté interface

Exemple pratique :
- pour afficher une réservation, on joint Reservation, Salle, Utilisateur
- pour afficher une contenance, on joint Contenance, Salle, Equipement

---

## 10. Spécificités métier à connaître

Réservation utilisateur :
- nécessite connexion
- validations dates (champs obligatoires, début < fin, durée max)
- insertion liée à l'utilisateur en session

Administration :
- CRUD complet sur plusieurs entités
- formulaires en table avec actions add/update/delete
- contrôles de droits systématiques sur /admin

---

## 11. Frontend et rendu

Layout global :
- Views/base.php inclut header, main, footer
- style chargé via Assets/CSS/style.css
- composants partagés dans Views/Components

Particularité actuelle :
- le rendu admin est piloté en CSS via la structure .admin-layout (pas par condition PHP dans base)

---

## 12. Où modifier quoi (guide rapide)

Besoin de modifier une route :
- Controllers/*.php

Besoin de modifier une requête SQL :
- Models/*.php

Besoin de modifier un affichage :
- Views/**/*.php

Besoin de modifier l'apparence :
- Assets/CSS/partials/*.css

Besoin de modifier la navigation :
- Views/Components/header.php

---

## 13. Vérification rapide après modification

Checklist simple :
1. Vérifier la syntaxe PHP des fichiers modifiés.
2. Tester le parcours invité -> connexion -> action -> déconnexion.
3. Tester un compte utilisateur standard.
4. Tester un compte admin (accès /admin et CRUD).
5. Vérifier les redirections après POST.
6. Vérifier que la navbar change correctement selon le rôle.

---

## 14. Résumé en une phrase

Le projet repose sur un MVC classique, avec session utilisateur, permissions par rôle, CRUD homogène en POST + redirection, et une séparation claire entre logique, données et affichage.
