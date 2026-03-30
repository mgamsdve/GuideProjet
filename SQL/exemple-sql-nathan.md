```sql
-- SUPPRESSION DES TABLES (Ordre respecté pour les contraintes)

DROP TABLE IF EXISTS location;

DROP TABLE IF EXISTS bateau;

DROP TABLE IF EXISTS utilisateur;

DROP TABLE IF EXISTS categorie;

DROP TABLE IF EXISTS lieu;


-- TABLE utilisateur

CREATE TABLE utilisateur (

    id_utilisateur INTEGER NOT NULL AUTO_INCREMENT PRIMARY KEY,

    uti_nom VARCHAR(100) NOT NULL,

    uti_prenom VARCHAR(100) NOT NULL,

    uti_email VARCHAR(150) NOT NULL UNIQUE,

    uti_motdepasse VARCHAR(50) NOT NULL, -- Ajouté (non présent sur l'image mais nécessaire au login)

    uti_role VARCHAR(50) NOT NULL

);


-- TABLE categorie

CREATE TABLE categorie (

    cat_id INTEGER NOT NULL AUTO_INCREMENT PRIMARY KEY,

    cat_nom VARCHAR(100) NOT NULL

);


-- TABLE lieu

CREATE TABLE lieu (

    lie_id INTEGER NOT NULL AUTO_INCREMENT PRIMARY KEY,

    lie_adresse VARCHAR(255) NOT NULL

);


-- TABLE bateau

CREATE TABLE bateau (

    bat_id INTEGER NOT NULL AUTO_INCREMENT PRIMARY KEY,

    bat_cat_id INTEGER NOT NULL,

    bat_lie_id INTEGER NOT NULL,

    bat_nom VARCHAR(100) NOT NULL,

    bat_description TEXT,

    bat_photo VARCHAR(255),

    bat_equipement TEXT,

    FOREIGN KEY (bat_cat_id) REFERENCES categorie(cat_id),

    FOREIGN KEY (bat_lie_id) REFERENCES lieu(lie_id)

);


-- TABLE location

CREATE TABLE location (

    loc_id INTEGER NOT NULL AUTO_INCREMENT PRIMARY KEY,

    loc_uti_id INTEGER NOT NULL,

    loc_bat_id INTEGER NOT NULL,

    loc_date DATE NOT NULL,

    FOREIGN KEY (loc_uti_id) REFERENCES utilisateur(id_utilisateur),

    FOREIGN KEY (loc_bat_id) REFERENCES bateau(bat_id)

);


---

-- INSERTIONS (Adaptées aux nouveaux noms de colonnes)

---


-- UTILISATEURS

INSERT INTO utilisateur (id_utilisateur, uti_nom, uti_prenom, uti_email, uti_motdepasse, uti_role) VALUES

(NULL,'Dupont','Jean','jean1@test.com','1111','user'),

(NULL,'Martin','Sophie','sophie2@test.com','2222','user'),

(NULL,'Leroy','Paul','paul3@test.com','3333','user'),

(NULL,'Moreau','Camille','camille4@test.com','4444','user'),

(NULL,'Bernard','Lucas','lucas5@test.com','5555','user'),

(NULL,'Petit','Emma','emma6@test.com','6666','user'),

(NULL,'Rousseau','Thomas','thomas7@test.com','7777','user'),

(NULL,'Faure','Claire','claire8@test.com','8888','user'),

(NULL,'Gauthier','Maxime','maxime9@test.com','9999','user'),

(NULL,'Michel','Julie','julie10@test.com','1234','user'),

(NULL,'Garcia','Hugo','hugo11@test.com','2345','user'),

(NULL,'David','Manon','manon12@test.com','3456','user'),

(NULL,'Robert','Nathan','nathan13@test.com','4567','user'),

(NULL,'Richard','Lina','lina14@test.com','5678','user'),

(NULL,'Durand','Leo','leo15@test.com','6789','user'),

(NULL,'Dubois','Sarah','sarah16@test.com','7890','user'),

(NULL,'Morel','Adam','adam17@test.com','1357','user'),

(NULL,'Fournier','Ines','ines18@test.com','2468','user'),

(NULL,'Girard','Noah','noah19@test.com','1122','user'),

(NULL,'Andre','Eva','eva20@test.com','3344','user');


-- CATEGORIES

INSERT INTO categorie (cat_id, cat_nom) VALUES

(NULL,'Voilier'), (NULL,'Bateau à moteur'), (NULL,'Péniche'), (NULL,'Semi-rigide'), (NULL,'Catamaran'),

(NULL,'Yacht'), (NULL,'Bateau de pêche'), (NULL,'Bateau fluvial'), (NULL,'Vedette'), (NULL,'Ferry');


-- LIEUX

INSERT INTO lieu (lie_id, lie_adresse) VALUES

(NULL,'Port de Marseille'), (NULL,'Port de Cannes'), (NULL,'Port de Nice'), (NULL,'Port de Brest'), (NULL,'Port de Toulon'),

(NULL,'Port de La Rochelle'), (NULL,'Port de Saint-Malo'), (NULL,'Port de Bordeaux'), (NULL,'Port du Havre'), (NULL,'Port de Calais'),

(NULL,'Canal du Midi'), (NULL,'Canal de Bourgogne'), (NULL,'Lac Léman'), (NULL,'Lac d Annecy'), (NULL,'Port de Sète'),

(NULL,'Port de Ajaccio'), (NULL,'Port de Bastia'), (NULL,'Port de Bonifacio'), (NULL,'Port de Dunkerque'), (NULL,'Port de Nantes');


-- BATEAUX

INSERT INTO bateau (bat_id, bat_cat_id, bat_lie_id, bat_nom, bat_description, bat_photo, bat_equipement) VALUES

(NULL,1,1,'Oceanis 38','Voilier de croisière','b1.jpg','GPS, VHF'),

(NULL,1,2,'Sun Odyssey','Voilier hauturier','b2.jpg','Pilote auto'),

(NULL,2,3,'Cap Camarat','Bateau rapide','b3.jpg','Radar'),

(NULL,2,4,'Flyer 8','Bateau à moteur','b4.jpg','Sondeur'),

(NULL,3,5,'Peniche Alpha','Peniche habitable','b5.jpg','Cuisine'),

(NULL,3,6,'Peniche Beta','Peniche fluvial','b6.jpg','Chauffage'),

(NULL,4,7,'Zodiac Pro','Semi-rigide','b7.jpg','Gilets'),

(NULL,4,8,'Zodiac Fast','Semi-rigide rapide','b8.jpg','GPS'),

(NULL,5,9,'Lagoon 40','Catamaran','b9.jpg','Double moteur'),

(NULL,5,10,'Excess 11','Catamaran sport','b10.jpg','Winchs'),

(NULL,6,11,'Sunseeker','Yacht luxe','b11.jpg','Clim'),

(NULL,6,12,'Azimut 50','Yacht','b12.jpg','Cuisine'),

(NULL,7,13,'Armor','Bateau de pêche','b13.jpg','Treuil'),

(NULL,7,14,'Pecheur Pro','Pêche côtière','b14.jpg','Filets'),

(NULL,8,15,'Fluvial 200','Bateau fluvial','b15.jpg','Radar'),

(NULL,8,16,'Canal Boat','Navigation canal','b16.jpg','Panneaux solaires'),

(NULL,9,17,'Vedette X','Vedette rapide','b17.jpg','GPS'),

(NULL,9,18,'Vedette Y','Vedette promenade','b18.jpg','Micro'),

(NULL,10,19,'Ferry Go','Transport passagers','b19.jpg','Rampes'),

(NULL,10,20,'Sea Ferry','Ferry côtier','b20.jpg','Sécurité');


-- LOCATIONS

INSERT INTO location (loc_id, loc_uti_id, loc_bat_id, loc_date) VALUES

(NULL,1,1,'2025-01-01'), (NULL,2,2,'2025-01-02'), (NULL,3,3,'2025-01-03'), (NULL,4,4,'2025-01-04'), (NULL,5,5,'2025-01-05'),

(NULL,6,6,'2025-01-06'), (NULL,7,7,'2025-01-07'), (NULL,8,8,'2025-01-08'), (NULL,9,9,'2025-01-09'), (NULL,10,10,'2025-01-10'),

(NULL,11,11,'2025-01-11'), (NULL,12,12,'2025-01-12'), (NULL,13,13,'2025-01-13'), (NULL,14,14,'2025-01-14'), (NULL,15,15,'2025-01-15'),

(NULL,16,16,'2025-01-16'), (NULL,17,17,'2025-01-17'), (NULL,18,18,'2025-01-18'), (NULL,19,19,'2025-01-19'), (NULL,20,20,'2025-01-20');
```