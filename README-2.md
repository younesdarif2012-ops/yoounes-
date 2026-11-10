# Volkswagen × L’argus — Émulateur de demande d’essai (V2)

**État : prototype fonctionnel de préproduction, PC + mobile.** La maquette ne publie rien sur largus.fr, ne transmet aucun lead à Data On Demand et n'envoie aucun SMS. Le flux de production ne doit pas être activé sans accord de l'équipe L’argus, des équipes juridiques/RGPD et du partenaire.

## Démarrage

Prérequis : Node.js 18 ou supérieur, aucune dépendance npm externe.

1. Décompresser le ZIP.
2. Ouvrir un terminal dans le dossier `volkswagen-largus-v2`.
3. Exécuter `npm start` (ou `node server.js`).
4. Ouvrir **http://localhost:4173** dans un navigateur récent.
5. Pour un aperçu mobile, utiliser le mode responsive des outils de développement du navigateur ou ouvrir la même URL depuis un téléphone connecté à un serveur accessible.

`npm test` exécute la suite de vérifications serveur. Le port peut être changé avec `PORT=8080 npm start`.

## Parcours et pages

Le bandeau « APERÇUS D’INTÉGRATION » permet de naviguer entre : landing d'essai, accueil, page marque, article / encart latéral, fiche modèle, annonces et Cote Argus. Tous les appels à l'action ouvrent le même formulaire :

1. **Modèle** : 4 modèles proposés en premier, catalogue étendu via « Voir les autres modèles ».
2. **Concession** : code postal, résultats de concessionnaires actifs, nom, adresse et code `FRAVxxxxx`.
3. **Coordonnées** : prénom, nom, e-mail, téléphone mobile, consentement pour la demande; opt-in marketing séparé et facultatif.
4. **Confirmation** : référence `DEMO-xxxx`, résumé de la concession et mention explicite qu'aucun SMS/lead réel n'a été envoyé.

L’affichage est responsive, y compris pour le parcours en panneau mobile. Les boutons Cote Argus qui ne correspondent pas au service de cotation réel sont explicitement présentés comme maquettes.

## Données fournies et images

Les 2 fichiers texte transmis contiennent le même export JSON (empreintes identiques), de **315 concessionnaires Volkswagen**. Le fichier local `data/dealers-snapshot.json` est une copie de cet export. Le service tente de récupérer la liste en direct depuis `https://www.insightondemand.fr/api/volkswagen/dealers.php`, puis utilise la copie locale en cas d'indisponibilité.

La recherche est géographique **si** le géocodage du code postal par `geo.api.gouv.fr` répond : distances à vol d'oiseau depuis le centre de la commune (pas des trajets routiers). À défaut, le filtrage se fait par département et l'interface l'indique sans inventer de distance.

**16 photos originales PNG de l'utilisateur** sont disponibles dans `assets/`. 19 modèles ont un code technique dans `data/models.json`; 4 n'ont pas de photo, et **ID.3 (visuel fourni)** n'est pas activable car son code API distinct est à confirmer. Une référence du catalogue n'indique pas forcément une éligibilité effective à un essai.

## API interne

- `GET /api/health` : état du serveur.
- `GET /api/catalog` : catalogue et état de livraison.
- `GET /api/dealers?zip=69003` : concessionnaires, code partenaire, adresse, source et éventuelle distance estimée.
- `POST /api/leads` : **validation et simulation uniquement**. Retourne une référence fictive. Les données personnelles ne sont ni persistées ni transmises au partenaire. Validation côté serveur, requêtes limitées par IP, refus des origines incohérentes.

Le fichier `server.js` expose uniquement les assets statiques et l'API interne ; les exports concessionnaires et scripts de test ne sont pas publiés. La vraie mise en production exigera une plateforme d'hébergement, HTTPS, un cadre RGPD validé, une politique de conservation et des journaux sécurisés.

## Spécifications Data On Demand fournies

- Documentation : `https://www.insightondemand.fr/webservice/rest/volkswagen/leadgen_emulateurs.php?_documentation=1`
- Réseau concessionnaires : `https://www.insightondemand.fr/api/volkswagen/dealers.php`
- Origine : `testerunevoiture`
- Campagne : `EMULATEUR_LEADGEN_2026_VOLKSWAGEN`
- `td_concession` : utiliser exactement la propriété `code` (ex. `FRAV07981`).
- `td_modele` : utiliser exactement le code modèle autorisé dans la documentation et le catalogue.
- Nom partenaire SMS : **« L’argus »**, sous réserve de confirmation explicite de Sophiann.

Un **script CLI séparé**, `scripts/partner-test.js`, prépare l'appel de test `testmode=y` avec des **données fictives**. Il ne s'exécute jamais au chargement du site et est verrouillé par défaut. **Ne pas l'exécuter sans validation de l'interlocuteur Data On Demand et d'un créneau de test.** Son exécution éventuelle requiert la variable `I_HAVE_AUTHORIZED_TESTS=YES` et ne prouve pas, à elle seule, que le partenaire a accepté le lead.

## À valider avant une campagne réelle

- Validation exacte des champs/méthodes HTTP par Data On Demand, gestion des réponses/erreurs, secrets, retries, idempotence et journalisation technique sans exposer de données sensibles.
- Correspondance des modèles et éligibilité à l'essai, concessions ouvertes/éligibles, liste mise à jour, politique de proximité en cas de géocodage absent.
- Consentement RGPD, mentions de confidentialité, finalités, durées de conservation, prestataires, collecte et preuve des consentements; contrat de sous-traitance.
- Autorisation d'usage du branding et des photos, intégration éditoriale sur largus.fr, UX validée pour tous les emplacements publicitaires.
- Confirmation du nom SMS « L’argus », du véritable dispositif SMS et de la confirmation attendue.
- Recette conjointe avec la plateforme, puis livraison d'un test fictif au webservice avec confirmation écrite de réception avant activation des leads.
- Mise en production, domaine public et lien de recette communiqués au client : **non réalisés dans ce livrable local**.

## Tests effectués

`npm test` : 7 tests automatiques de disponibilité, validation et protection de l'API exécutés et réussis lors de la livraison. La navigation Chrome automatisée de bout en bout n'a pas pu être exécutée dans cet environnement en raison d'une restriction du navigateur sur l'accès localhost. Des captures desktop et mobile antérieures sont présentes dans `previews/`, mais ne doivent pas être considérées comme une recette navigateur complète du build final.
