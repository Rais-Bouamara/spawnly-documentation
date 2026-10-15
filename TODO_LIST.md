# Plan d’amélioration de Pix.Undo

## Objectif et périmètre

Ce plan propose des améliorations progressives pour le site pédagogique Pix.Undo. Il s’appuie sur le code décrit dans `DOCUMENTATION.md`. L’environnement reste **local sous WAMP (Apache et PHP sous Windows)**. Le site conserve son stockage actuel en fichiers JSON. Aucune installation logicielle supplémentaire, aucun serveur distant, conteneur, déploiement automatisé ni paiement réel n’est prévu.

La présence de JSON plutôt que d’une base de données n’est pas considérée comme une erreur. Une évolution vers SQLite est proposée uniquement comme option pédagogique, à valider avant de la commencer.

**Priorités :** P0 = bloquant/critique; P1 = important; P2 = amélioration facultative. Aucun P0 n’est retenu : l’analyse du code disponible ne confirme pas de défaut critique pour l’usage local prévu. La priorité P1 indique des sujets à traiter avant toute exposition du site à d’autres utilisateurs.

**Complexité :** simple = modification ciblée; moyenne = plusieurs fichiers ou vérifications coordonnées; complexe = changement transversal nécessitant une conception et des essais plus larges.

## Actions proposées

### Sécurité et exploitation locale

| Action | Priorité | Complexité | Critère d’acceptation vérifiable | Fichiers concernés |
|---|---|---|---|---|
| Clarifier la protection des identifiants SMTP sur le poste local | P1 | Simple | Le mot de passe SMTP ne figure dans aucun fichier suivi par Git; `admin/private/smtp.json` reste inaccessible par HTTP; la documentation rappelle qu’il est enregistré en clair et recommande de ne pas publier le dossier WAMP. | `admin/core.php`, `admin/private/.htaccess`, `.gitignore`, `README.md`, `DOCUMENTATION.md` |
| Vérifier les protections HTTP des fichiers privés avec Apache | P1 | Simple | Depuis le navigateur local, une requête vers `admin/private/` et `storage/` est refusée; noter le code HTTP observé et vérifier que le site continue de fonctionner. | `admin/private/.htaccess`, `storage/.htaccess`, `README.md` |
| Rendre les limites de tentatives plus faciles à contrôler | P2 | Moyenne | Des essais locaux documentés confirment le délai croissant, la limite de quatre heures et la remise à zéro après une connexion réussie; aucune adresse IP en clair n’est ajoutée aux fichiers de limitation. | `admin/core.php`, `admin/login.php`, `admin/maintenance.php` |

### Stockage et données personnelles

| Action | Priorité | Complexité | Critère d’acceptation vérifiable | Fichiers concernés |
|---|---|---|---|---|
| Décrire les données de contact et leur durée de conservation | P1 | Simple | La page indique quelles coordonnées sont enregistrées, pourquoi elles le sont et comment demander leur suppression; la documentation distingue ce comportement local d’une validation juridique. | `index.html`, `api/contact.php`, `admin/messages.php`, `README.md` |
| Tester la sauvegarde et les opérations de suppression sans données réelles | P1 | Moyenne | Sur des données de test, confirmer que l’export est un JSON lisible, qu’il exclut les secrets annoncés et que les actions de suppression demandent une confirmation; vérifier qu’aucune donnée en dehors de la cible n’est effacée. | `admin/backup.php`, `admin/maintenance.php`, `admin/admin.js` |
| Étudier SQLite comme évolution facultative du stockage JSON | P2 | Complexe | Ne commencer qu’après accord de l’enseignant et vérification de l’extension SQLite du PHP WAMP; définir une migration réversible des produits, catégories et messages, puis démontrer l’export et le fonctionnement du site avec les données migrées. Le site JSON actuel reste intact tant que cette validation n’est pas faite. | `admin/core.php`, `admin/products.php`, `admin/categories.php`, `admin/messages.php`, `api/products.php`, `api/contact.php` |

### Qualité du code et vérifications

| Action | Priorité | Complexité | Critère d’acceptation vérifiable | Fichiers concernés |
|---|---|---|---|---|
| Préparer une liste de tests manuels reproductibles | P1 | Simple | Un élève peut suivre les étapes et noter le résultat attendu pour la connexion, le refus d’un mot de passe incorrect, les modifications du catalogue, le formulaire de contact, la récupération locale et les refus d’accès HTTP. Les tests n’utilisent pas le compte ou les messages réels. | `README.md`, `admin/login.php`, `admin/security.php`, `admin/reset-start.php`, `admin/reset-finish.php`, `api/contact.php` |
| Vérifier les validations des données à chaque entrée serveur | P1 | Moyenne | Les essais avec champs manquants, trop longs ou invalides sont refusés côté PHP sans modifier les fichiers JSON; les entrées valides continuent de fonctionner. | `api/contact.php`, `admin/products.php`, `admin/categories.php`, `admin/security.php`, `admin/email.php`, `admin/core.php` |
| Afficher des erreurs compréhensibles en cas de fichier JSON illisible ou non inscriptible | P2 | Moyenne | En simulant ces problèmes uniquement sur une copie de test, l’utilisateur voit une explication non technique et le journal PHP conserve les détails utiles; aucune donnée existante n’est remplacée silencieusement. | `admin/core.php`, `admin/maintenance.php`, `admin/products.php`, `admin/categories.php`, `admin/messages.php` |

### Interface et accessibilité

| Action | Priorité | Complexité | Critère d’acceptation vérifiable | Fichiers concernés |
|---|---|---|---|---|
| Vérifier les parcours essentiels sur petit écran et au clavier | P2 | Moyenne | Sur une largeur mobile, navigation, filtres, panier et formulaire restent utilisables; au clavier, le focus est visible et les fenêtres de dialogue peuvent être ouvertes et fermées sans souris. | `index.html`, `css/styles.css`, `js/app.js`, `admin/admin.css`, `admin/admin.js` |
| Améliorer les messages d’état du catalogue et du formulaire | P2 | Simple | Les états de chargement, catalogue indisponible, absence de résultat et envoi réussi/échoué sont compréhensibles et annoncés aux lecteurs d’écran. | `index.html`, `js/app.js` |

### Documentation et usage pédagogique

| Action | Priorité | Complexité | Critère d’acceptation vérifiable | Fichiers concernés |
|---|---|---|---|---|
| Garder les instructions locales et la documentation alignées sur le code | P2 | Simple | Les instructions de démarrage, les limites du formulaire, les données enregistrées et les dépendances mentionnées correspondent aux fichiers effectivement présents; les versions inconnues restent indiquées comme telles. | `README.md`, `DOCUMENTATION.md`, `composer.json`, `composer.lock` |

## Jalons

1. **Sécuriser et comprendre les données (P1)** : vérifier les refus d’accès HTTP, relire le stockage SMTP, préciser le traitement des messages et tester les suppressions uniquement avec des données fictives.
2. **Vérifier les parcours fonctionnels (P1)** : rédiger puis exécuter la liste de tests manuels et contrôler les validations serveur sur les opérations principales.
3. **Améliorer l’expérience locale (P2)** : tester l’interface au clavier et sur mobile, puis améliorer les retours d’état si un défaut est observé.
4. **Choisir les évolutions facultatives (P2)** : décider avec l’enseignant si une étude SQLite ou d’autres améliorations sont utiles; conserver le fonctionnement JSON si ce n’est pas nécessaire.

## Définition de Terminé

Une action est considérée terminée quand :

- le critère d’acceptation de sa ligne a été vérifié sur une copie ou avec des données de test, sans perte des données existantes;
- les fichiers concernés ont été relus et la documentation est mise à jour si le comportement change;
- le résultat et les éventuelles limites sont expliqués simplement;
- aucun service distant, paiement réel ou logiciel supplémentaire n’a été introduit sans accord préalable.

## Petit glossaire

- **JSON** : format texte structuré utilisé ici pour conserver localement les produits, les paramètres et les messages.
- **SQLite** : base de données stockée dans un fichier; elle est une piste facultative, pas un prérequis du projet actuel.
- **Validation côté serveur** : contrôle effectué en PHP; il reste nécessaire même si le navigateur vérifie déjà les champs.
- **HTTP 403** : réponse indiquant que l’accès à la ressource est interdit.
- **Donnée personnelle** : information se rapportant à une personne, par exemple son nom ou son adresse e-mail. Ce plan ne constitue pas un avis juridique.
