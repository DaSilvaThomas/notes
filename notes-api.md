# Notes de cours OpenClassrooms sur les API

## Chapitre 1 : Introduction aux API

### Définition et concepts de base

- **API** : Application Programming Interface (interface de programmation d'application)
- **Rôle** : Moyen de communication entre deux logiciels ou composants d'applications
- **Principe** : Communication client-serveur
  - Le client formule une requête pour obtenir une information
  - Le serveur envoie une réponse contenant les données demandées

### Différence entre Service Web et API

- **Service Web** : Facilite la communication entre deux machines via un réseau
- **API** : Sert d'intermédiaire entre deux applications pour qu'elles puissent communiquer
  - Le client demande une information à l'API
  - L'API récupère cette information dans la base de données
  - L'API renvoie l'information au client

### Types d'API

- **API publique** : Utilisable sans autorisation stricte pour obtenir des données générales (météo, films, musique)
- **API privée** : Accessible uniquement aux utilisateurs autorisés au sein d'une entreprise ou application

### Standards et protocoles

- **REST** : Representational State Transfer
  - Ensemble de normes architecturales structurant la communication des données
  - Basé sur le protocole HTTP
  - Six principes architecturaux :
    1. Séparation du client et du serveur
    2. Stateless (sans état)
    3. Sauvegardable
    4. Interface uniforme
    5. Système de couches
    6. Code à la demande

- **SOAP** : Simple Object Access Protocol
  - Protocole d'accès aux objets
  - Était le standard le plus courant avant REST

### Concepts clés

- **Ressource** : Objet nominal représentant les données dans l'application
- **Collection** : Ensemble regroupant les ressources
- **URI et Endpoints** :
  - URI (Uniform Resource Identifier) : Identifiant de ressource
  - Endpoint : URL complète = nom de domaine + URI
  - Toutes les URL sont des URI, mais toutes les URI ne sont pas des URL
  - L'URI identifie une ressource, l'URL la localise

### Formats de données

- **Données vs Ressources** :
  - Données : Terme général pour l'information envoyée ou reçue
  - Ressource : Éléments précis contenus dans cette information

- **Formats pour les API REST** :
  - XML : Utilise des balises ouvrantes et fermantes (peut contenir des balises imbriquées)
  - JSON : Format clé-valeur, données entourées d'accolades `{ }`, paires clé-valeur envoyées comme chaînes de caractères avec guillemets `""`

## Chapitre 2 : Formuler des requêtes API avec Postman

### Structure des requêtes

- **Verbes HTTP** :
  - GET : Obtenir des données
  - PUT : Mettre à jour des données
  - POST : Créer des données
  - DELETE : Supprimer des données

- **URI** : Identifie les ressources
  - Exemple général : `/users`
  - Exemple avec ID : `/users/145`
  - Les placeholders sont indiqués par `:` (ex: `/users/:user_id`)

- **Headers** (en-têtes) :
  - Informations supplémentaires sur le message
  - Format clé-valeur
  - Exemples :
    - Date
    - Agent utilisateur
    - Clé d'authentification

- **Body** (corps) :
  - Utilisé principalement avec PUT et POST
  - Contient les données de la ressource à créer ou mettre à jour
  - Format généralement JSON

### Structure des réponses

- **Format** : Version HTTP + Code de réponse HTTP + Headers + Body
- **Codes de réponse HTTP** :
  - 100+ : Information
  - 200+ : Succès (ex: 200 OK)
  - 300+ : Redirection
  - 400+ : Erreur client (ex: 404 Not Found)
  - 500+ : Erreur serveur

### CRUD et verbes HTTP

| Action CRUD | Verbe HTTP associé |
|-------------|-------------------|
| Create (Créer) | POST (Publier) |
| Read (Lire) | GET (Obtenir) |
| Update (Mettre à jour) | PUT (Mettre) |
| Delete (Supprimer) | DELETE (Supprimer) |

### Utilisation des API avec documentation

- Importance de consulter la documentation avant d'utiliser une API
- Exemple avec l'API GitHub :
  - Accès aux informations d'utilisateur via `GET /users/:username`
  - Les requêtes GET peuvent être effectuées directement depuis le navigateur

## Chapitre 3 : Authentification et sécurité

### Authentification

- **Rôle** : Garantir que le client a les autorisations nécessaires pour accéder et manipuler les données
- **Méthode courante** : Obtention d'un token (clé) via inscription sur le site web de l'API

### Tokens

- **Définition** : Chaîne unique de lettres et chiffres aléatoires assignée à un utilisateur
- **Fonction** : Identifie l'utilisateur et détermine son niveau d'autorisation
- **Transmission** : Via les paramètres du header ou dans l'endpoint

### Sécurité des API

- **Responsabilité du développeur** : Assurer la sécurité des données des utilisateurs
- **Critères pour API fiables** :
  - Mesures de sécurité (authentification, autorisation, cryptage)
  - Mises à jour récentes selon les derniers standards de sécurité

## Chapitre 4 : Opérations CRUD avec les verbes HTTP

### POST (Create)

- **Fonction** : Créer une nouvelle ressource
- **Exemples d'utilisation** : Création d'un compte, publication d'un tweet, ajout d'une photo
- **Particularité** : Utilise le body pour contenir les informations à créer (format JSON)
- **Exemple pratique** : Création d'un repository GitHub
  - Requête POST vers `/user/repos`
  - Ajout du token d'authentification dans les headers
  - Inclusion des paramètres nécessaires dans le body (nom, description)
  - Code de réponse attendu : 201 Created

### PUT/PATCH (Update)

- **PUT** : Met à jour la ressource complète (remplacement total)
- **PATCH** : Met à jour uniquement la partie modifiée de la ressource
- **Exemple pratique** : Mise à jour d'un repository GitHub
  - Requête PATCH vers `/repos/:Owner/:repo`
  - Inclusion de la nouvelle description dans le body
  - Code de réponse attendu : 200 OK

### DELETE

- **Fonction** : Supprimer une ressource existante
- **Exemple** : Suppression d'un repository GitHub

## Chapitre 5 : Conception d'une API REST

### Raisons de développer sa propre API

1. **API interne** :
   - Pour applications nécessitant stockage et manipulation de données
   - Pour séparer la base de données de l'interface utilisateur
   - Pour assurer sécurité et confidentialité des données
   - Pour faciliter la recherche et l'interrogation des données

2. **API pour développeurs externes** :
   - Pour permettre à d'autres développeurs d'interagir avec vos données
   - Pour créer une plateforme collaborative

### Bonnes pratiques de conception

- **Documentation détaillée** incluant :
  - Descriptions des ressources API
  - URI et verbes HTTP disponibles avec leur fonction
  - Paramètres nécessaires
  - Exemples de requêtes et réponses

- **Paramètres de requête** :
  - Utilisation du `?` pour indiquer un paramètre
  - Exemple : `GET /photos?location={locationId}`

- **Recherche** :
  - Permet de filtrer les ressources selon critères
  - Exemple : `GET /photos?search=snowboard`

- **Chaînage de requêtes** :
  - Utilisation du `&` pour combiner plusieurs paramètres
  - Exemple : `GET /users/{userId}/followers?sort=lastName&order=asc`
  - Exemple complexe : `GET /photos?location=NYC&created_at=2018-12-31&likes_greater_than=5000`

- **Pagination** :
  - Limite le nombre de résultats par requête
  - Exemple : `GET /photos?page=23`

- **Versioning** :
  - Facilite le suivi des changements et assure la compatibilité
  - Deux méthodes :
    1. Champ version dans les headers : `"accept-version": "1.3"`
    2. Version dans l'URI : `GET /v1/photos`

## Chapitre 6 : Frameworks pour construire une API

### Frameworks populaires

- **JavaScript** : Express.js
- **Ruby** : Ruby on Rails
- **Python** : Django, Flask
- **Java** : Spring
- **Cloud** : AWS API Gateway
