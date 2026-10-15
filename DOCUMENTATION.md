# Pix.Undo — documentation technique

Documentation établie à partir des fichiers présents dans ce projet. Les réglages propres à l’installation WAMP et les données créées à l’exécution ne sont pas inclus dans le dépôt.

## Table des matières

1. [Présentation et fonctions](#1-présentation-et-fonctions)
2. [Technologies et versions](#2-technologies-et-versions)
3. [Arborescence](#3-arborescence)
4. [Entrées, routage et URL](#4-entrées-routage-et-url)
5. [Stockage des données](#5-stockage-des-données)
6. [Authentification, sessions et sécurité](#6-authentification-sessions-et-sécurité)
7. [Interface front-end](#7-interface-front-end)
8. [API, services externes et courriels](#8-api-services-externes-et-courriels)
9. [Configuration et constantes](#9-configuration-et-constantes)
10. [Limites, risques et points forts](#10-limites-risques-et-points-forts)

## 1. Présentation et fonctions

Pix.Undo est un site vitrine fictif de vente de périphériques informatiques, réalisé à des fins pédagogiques. La page publique présente l’entreprise, un catalogue filtrable, un panier de démonstration et un formulaire de contact. L’administration permet de gérer les produits et catégories, consulter les messages, configurer les courriels, modifier les identifiants et effectuer certaines opérations de maintenance.

Le panier ne passe aucune commande. Le formulaire enregistre les messages dans les données locales du site et précise qu’aucun courriel n’est envoyé par cette fonction.

## 2. Technologies et versions

| Élément | Constat vérifié dans le projet |
|---|---|
| Site public | HTML, CSS et JavaScript natifs, sans framework. |
| Administration et API | PHP natif. `composer.json` déclare PHP `>= 8.0`. |
| Dépendance d’envoi de courriels | PHPMailer `6.12.0`, version inscrite dans `composer.lock` et présente sous `vendor/`. |
| Serveur prévu | Apache avec PHP, notamment WAMP selon `README.md`. Le projet ne fixe pas de version Apache ou de version PHP réellement installée. |
| Persistance | Fichiers JSON et JSONL locaux; aucune base de données ni couche PDO n’est utilisée dans le code applicatif. |
| Ressources graphiques | Illustrations SVG locales dans `assets/illustrations/`. Aucune police ou bibliothèque front-end distante n’est requise. |

La version effective d’Apache et de PHP dépend donc de l’installation WAMP qui exécute le projet; elle ne peut pas être déduite des seuls fichiers du dépôt.

## 3. Arborescence

```text
.
├── index.html                   Page publique
├── css/
│   └── styles.css               Styles du site public
├── js/
│   └── app.js                   Catalogue, filtres, panier et formulaire
├── assets/illustrations/        Illustrations SVG locales
├── admin/
│   ├── core.php                 Fonctions partagées, stockage et sécurité
│   ├── setup.php                Création initiale du compte administrateur
│   ├── login.php, logout.php    Connexion et déconnexion
│   ├── index.php                Tableau de bord
│   ├── products.php             Gestion du catalogue
│   ├── categories.php            Gestion des catégories
│   ├── messages.php              Consultation des demandes de contact
│   ├── email.php                 Configuration et test SMTP
│   ├── security.php              Modification des identifiants
│   ├── reset-start.php           Demande de code de récupération
│   ├── reset-finish.php          Vérification du code et nouveau mot de passe
│   ├── maintenance.php           Diagnostic, journaux, export et purge
│   ├── backup.php                Génération de l’export JSON
│   ├── admin.css, admin.js       Présentation et interactions de l’administration
│   └── private/.htaccess         Interdiction d’accès HTTP direct au dossier privé
├── api/
│   ├── products.php              Catalogue public au format JSON
│   ├── csrf.php                  Jeton CSRF pour le formulaire public
│   ├── contact.php               Validation et enregistrement des messages
│   └── .htaccess                 Désactivation de l’indexation du dossier
├── storage/
│   └── .htaccess                 Interdiction d’accès HTTP direct au stockage
├── vendor/                       Dépendances Composer, dont PHPMailer
├── composer.json                 Dépendances et PHP minimal déclaré
├── composer.lock                 Versions verrouillées
├── .gitignore                    Exclusion des dépendances et données privées locales
└── README.md                     Instructions de lancement et d’utilisation
```

Les fichiers de données et `storage/tmp/` sont créés ou utilisés à l’exécution. Dans le dossier examiné, `admin/private/` et `storage/` ne contiennent que leur fichier `.htaccess` : aucun compte, message ou identifiant SMTP réel n’a été trouvé dans cette copie.

## 4. Entrées, routage et URL

Il n’y a pas de routeur applicatif ni de règles de réécriture dans les fichiers du projet. La page publique est `index.html` et sa navigation utilise des ancres (`#accueil`, `#entreprise`, `#catalogue`, `#panier`, `#contact`). Les fonctions serveur sont des pages et endpoints PHP distincts.

| Chemin relatif | Rôle et méthode constatée |
|---|---|
| `/` ou `/index.html` | Site vitrine; le document par défaut exact dépend de la configuration Apache. |
| `/admin/` | `index.php` est le point d’entrée du panneau et redirige vers la configuration initiale ou la connexion selon l’existence du compte. |
| `/admin/setup.php`, `/admin/login.php`, `/admin/logout.php` | Création initiale, connexion et déconnexion; la déconnexion exige une requête POST protégée par CSRF. |
| `/admin/index.php`, `/admin/products.php`, `/admin/categories.php`, `/admin/messages.php`, `/admin/email.php`, `/admin/security.php`, `/admin/maintenance.php` | Pages du panneau réservées à une session administrateur. |
| `/admin/reset-start.php`, `/admin/reset-finish.php` | Endpoints POST JSON de récupération du compte. |
| `/admin/backup.php` | Endpoint POST qui fournit le fichier de sauvegarde JSON après contrôle d’accès et de CSRF. |
| `/api/products.php`, `/api/csrf.php` | Endpoints GET JSON appelés par le JavaScript public. |
| `/api/contact.php` | Endpoint POST JSON de validation et d’enregistrement des demandes de contact. |

`maintenance.php` utilise le paramètre `tool` (`system`, `logs`, `backup`, `tmp`) et le paramètre facultatif `filter` pour filtrer le journal. Les URL publiques en WAMP dépendent du nom du dossier et de la configuration Apache; `README.md` donne `http://localhost/InfoLeader/` comme exemple.

## 5. Stockage des données

Le code applicatif ne crée pas de schéma SQL. Il persiste les informations dans les fichiers suivants, sous `admin/private/` :

| Fichier | Contenu |
|---|---|
| `store.json` | Données principales : `admin`, `email`, `categories`, `products` et `messages`. Créé à partir de `il_seed_store()` lorsqu’il manque. |
| `smtp.json` | Identifiant et mot de passe SMTP séparés des données générales. |
| `security-log.jsonl` | Journal local, une entrée JSON par ligne : date, événement, détail et empreinte SHA-256 de l’adresse IP. |
| `attempts-<empreinte>.json` | État du limiteur de tentatives par adresse source : compteur d’échecs et instant de la prochaine tentative. |

Dans `store.json`, `admin` est initialement `null`, puis contient notamment le nom, l’adresse de connexion, le hash du mot de passe et la version d’authentification. `email` contient l’adresse de récupération, l’adresse expéditeur, l’hôte et le port SMTP. Les produits ont notamment un identifiant, un nom, une catégorie, une description, un prix et les références d’illustration; les messages contiennent l’expéditeur, le sujet, le texte, la date et l’état lu/non lu.

`storage/tmp/` est réservé aux fichiers temporaires et peut être purgé par l’administration. Les sessions sont gérées par le mécanisme de sessions PHP; leur emplacement et leur stockage effectif dépendent de la configuration PHP/WAMP et ne sont pas définis dans le projet.

## 6. Authentification, sessions et sécurité

- Le premier compte est créé par `setup.php` uniquement lorsque `admin` est encore `null`. Le mot de passe est stocké avec `password_hash()` et contrôlé par `password_verify()`. La règle serveur impose au moins 20 caractères, une majuscule, une minuscule, un chiffre et une ponctuation ou un symbole.
- La connexion régénère l’identifiant de session. Le cookie de session est configuré `HttpOnly` et `SameSite=Strict`; l’attribut `Secure` est activé lorsque PHP détecte HTTPS. Une modification du mot de passe ou de l’adresse augmente `auth_version`, ce qui invalide les autres sessions à leur prochaine vérification.
- Les formulaires administratifs et les changements d’état vérifient un jeton CSRF. La déconnexion est également une action POST avec ce contrôle.
- Le limiteur de connexion persiste les échecs dans un fichier associé au hash de `REMOTE_ADDR`. Le délai commence à 3 secondes, augmente de façon exponentielle et est plafonné à 14 400 secondes (4 heures). Une connexion réussie remet l’état à zéro.
- La récupération génère un code à 8 chiffres valide 5 minutes, conservé sous forme de hash dans la session; le serveur limite les saisies incorrectes à cinq et une nouvelle demande est limitée à une par minute et par session. Quand le SMTP n’est pas configuré et que la requête vient de `127.0.0.1` ou `::1`, le code est affiché comme code de démonstration au lieu d’être envoyé.
- Les sorties HTML sont échappées avec `htmlspecialchars()`. Le JavaScript échappe les contenus dynamiques et limite les noms d’illustrations aux fichiers SVG prévus.
- `admin/private/.htaccess` et `storage/.htaccess` interdisent les accès HTTP directs; la purge vérifie que le chemin réel est bien `storage/tmp` dans le projet et refuse les liens symboliques de dossier. Ces protections supposent qu’Apache autorise les règles `.htaccess`.

Ces protections sont adaptées à un exercice local, mais ne constituent pas une garantie de sécurité pour un service exposé sur Internet.

## 7. Interface front-end

`index.html`, `css/styles.css` et `js/app.js` forment le site public. Le script affiche un catalogue de secours puis tente de charger les produits et catégories depuis `/api/products.php`. Il gère la recherche, le filtre par catégorie, le tri par prix ou nom, le panier temporaire et l’envoi du formulaire avec son jeton CSRF.

Les styles utilisent une palette bleu nuit/cyan et des mises en page adaptatives (`@media` à 900, 680 et 370 px). L’administration utilise `admin/admin.css` avec des adaptations à 850 et 620 px. Des éléments d’accessibilité sont présents : langue française déclarée, lien d’évitement, libellés de formulaire, attributs ARIA, états de focus visibles et prise en compte de `prefers-reduced-motion`. Les illustrations produit sont chargées localement avec chargement différé.

L’interface d’administration est rendue côté serveur en PHP; `admin/admin.js` gère les confirmations, la fermeture différée des messages et les formulaires de récupération du mot de passe.

## 8. API, services externes et courriels

Les seuls appels réseau intégrés au JavaScript visent les endpoints PHP locaux du projet. Le catalogue public lit les produits et catégories du stockage local. Le formulaire de contact valide côté serveur le nom, l’adresse, le sujet et le message, puis enregistre celui-ci; il ne déclenche aucun courriel.

Les seuls services externes identifiés sont liés à l’envoi SMTP :

- PHPMailer envoie via le relais Brevo `smtp-relay.brevo.com`, avec authentification et port 587 (STARTTLS) ou 465 (TLS implicite).
- Le test SMTP et le courriel de récupération utilisent les adresses configurées dans l’administration. L’envoi nécessite des identifiants SMTP valides, OpenSSL actif dans le PHP d’Apache, PHPMailer présent et une connexion sortante autorisée.
- Les clés SMTP ne sont pas incluses dans le dépôt examiné. Le code enregistre cependant le mot de passe fourni en clair dans `admin/private/smtp.json`; l’accès HTTP direct à ce dossier est bloqué par `.htaccess`.

Le code source ne fournit pas de configuration Brevo effective, d’identifiant SMTP ni de preuve qu’un envoi réel aboutit. La configuration réseau et les règles de pare-feu sont propres à la machine.

## 9. Configuration et constantes

- `composer.json` déclare `php >= 8.0` et `phpmailer/phpmailer ^6.9`; `composer.lock` fixe la version PHPMailer présente à `6.12.0`. Aucun script de test applicatif n’est défini dans `composer.json`.
- `admin/core.php` définit les chemins `IL_PRIVATE_DIR`, `IL_STORE_FILE`, `IL_SMTP_SECRET_FILE`, `IL_SECURITY_LOG_FILE`, `IL_TEMP_DIR` et la limite `IL_LOCK_LIMIT_SECONDS` à 14 400 secondes.
- Les valeurs initiales du catalogue, des catégories et des paramètres SMTP sont définies par `il_seed_store()` dans `admin/core.php`. L’hôte SMTP est limité dans l’interface serveur à `smtp-relay.brevo.com`; seuls les ports 465 et 587 sont acceptés.
- Les données administrateur et les réglages généraux sont stockés dans `admin/private/store.json`; les identifiants SMTP sont enregistrés séparément dans `admin/private/smtp.json`. Ils ne sont pas des variables d’environnement dans le code livré.
- `.gitignore` exclut `vendor/`, le contenu de `admin/private/` sauf son `.htaccess`, et `storage/tmp/`.
- Le `php.ini`, les versions Apache/PHP effectivement actives, les hôtes virtuels, les ports ouverts et les identifiants SMTP sont configurés hors du dépôt; aucune valeur propre à ces installations ne peut être confirmée ici.

## 10. Limites, risques et points forts

### Limites et risques connus

- Aucun dossier ni script de tests applicatifs n’a été trouvé dans cette copie; `composer.json` ne déclare pas de suite de tests. Les tests livrés avec PHPMailer ne testent pas l’application Pix.Undo.
- Le stockage JSON est simple et lisible, mais ne fournit ni transactions de base de données ni gestion robuste des écritures concurrentes. Il convient à une démonstration mono-instance, pas à une boutique à fort trafic ou multi-utilisateur.
- Le mot de passe SMTP est stocké en clair sur le disque. `.htaccess` empêche l’accès web direct si Apache applique bien cette configuration, mais ne protège pas le fichier contre un utilisateur ayant accès au disque ou au compte Windows.
- Une sauvegarde téléchargée contient des données du compte et les messages de contact, même si les secrets d’authentification sont exclus. Le fichier exporté doit donc être conservé avec précaution.
- Le code ne définit pas à lui seul HTTPS, les en-têtes de sécurité du serveur, les permissions du système de fichiers ou une sauvegarde automatique. Leur état dépend de la configuration de déploiement.
- Le code de démonstration de récupération est intentionnellement affiché lors d’une requête locale sans SMTP; cette voie ne doit pas être exposée comme mécanisme de production.
- Les produits, prix et coordonnées présentés sont fictifs. Le panier n’est pas persistant et n’effectue ni commande, ni réservation, ni paiement.
- `README.md` recommande de ne pas exposer ce WAMP local à Internet. Pour un usage public, il faudrait notamment revoir le stockage des secrets et des données, les protections serveur et l’exploitation.

### Points forts vérifiables

- Séparation lisible entre vitrine, styles, interactions, administration, API, illustrations et stockage.
- Pas de dépendance réseau front-end obligatoire; les illustrations sont locales et les ressources de base ne reposent pas sur un CDN.
- Validations des formulaires côté navigateur et côté serveur, encodage des sorties HTML, protection CSRF et contrôles d’accès sur les pages administratives.
- Politique de mot de passe exigeante, invalidation des autres sessions après changement d’identifiants et ralentissement progressif des tentatives répétées.
- Limites de démonstration affichées explicitement : aucun paiement, aucun achat et aucun envoi de courriel par le formulaire public.
- Fonctions pédagogiques d’administration : catalogue, catégories, messages, diagnostic, journaux, export expurgé et purge temporaire circonscrite.
