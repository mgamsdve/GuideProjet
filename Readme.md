# GuideProjet

Ce dépôt contient la base d'un projet web en architecture MVC, pensé pour gérer des salles et leurs réservations.

## À quoi sert ce repo

- `Guide-de-demarrage.md` : guide de démarrage technique pour structurer le projet, connecter la base de données et mettre en place le MVC.
- `vue-global.md` : vue d'ensemble du projet, de son architecture et de ses règles de fonctionnement.

## Structure générale

Le projet suit une organisation classique :

- `Assets/` pour les styles, images et scripts
- `Config/` pour la configuration, dont la connexion à la base de données
- `Controllers/` pour la logique de routage et de traitement
- `Models/` pour les requêtes SQL
- `Views/` pour l'affichage HTML/PHP
- `SQL/` pour les scripts de création et de remplissage de la base

## Documentation complète

Les deux versions Google Docs des fichiers de référence sont ici :

- [Guide de démarrage](https://docs.google.com/document/d/1kcZQ4TD_iVnpl2muefRgbzKPXGCY6HY7BEIKFl5jEms/edit?usp=sharing)
- [Vue globale du projet](https://docs.google.com/document/d/10xwMY_bJuB04vr-UccYx0ge9b2Uhkl1ibRsBFgzXw4A/edit?usp=sharing)
- [Google Slides] : https://docs.google.com/presentation/d/1oUtDvOm9PPslUE7LXXXopBRl5ctr8XTi/edit?usp=sharing&ouid=115021727874337636729&rtpof=true&sd=true 
- [Site en ligne] : https://bookroom-wheat.vercel.app/
  mot de passe admin : ```test123```
  utilisateur admin : ```secure@demo.be```
## Démarrage rapide

1. Lire le guide de démarrage pour comprendre la structure attendue.
2. Vérifier la connexion à la base de données dans `Config/connectDatabase.php`.
3. Construire le point d'entrée `index.php` et brancher les controllers.
4. Développer ensuite les models et les views selon les besoins du projet.
