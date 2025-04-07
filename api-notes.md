# Notes de cours OpenClassrooms sur les API

## Table des matières
- [Chapitre 1 : Introduction aux API](#chapitre-1--introduction-aux-api)
  - [Définition et concepts de base](#définition-et-concepts-de-base)
  - [Différence entre Service Web et API](#différence-entre-service-web-et-api)
  - [Types d'API](#types-dapi)
  - [Standards et protocoles](#standards-et-protocoles)
  - [Concepts clés](#concepts-clés)
  - [Formats de données](#formats-de-données)
- [Chapitre 2 : Formuler des requêtes API avec Postman](#chapitre-2--formuler-des-requêtes-api-avec-postman)
  - [Structure des requêtes](#structure-des-requêtes)
  - [Structure des réponses](#structure-des-réponses)
  - [CRUD et verbes HTTP](#crud-et-verbes-http)
  - [Utilisation des API avec documentation](#utilisation-des-api-avec-documentation)
- [Chapitre 3 : Authentification et sécurité](#chapitre-3--authentification-et-sécurité)
  - [Authentification](#authentification)
  - [Tokens](#tokens)
  - [Sécurité des API](#sécurité-des-api)
- [Chapitre 4 : Opérations CRUD avec les verbes HTTP](#chapitre-4--opérations-crud-avec-les-verbes-http)
  - [POST (Create)](#post-create)
  - [PUT/PATCH (Update)](#putpatch-update)
  - [DELETE](#delete)
- [Chapitre 5 : Conception d'une API REST](#chapitre-5--conception-dune-api-rest)
  - [Raisons de développer sa propre API](#raisons-de-développer-sa-propre-api)
  - [Bonnes pratiques de conception](#bonnes-pratiques-de-conception)
- [Chapitre 6 : Frameworks pour construire une API](#chapitre-6--frameworks-pour-construire-une-api)
  - [Frameworks populaires](#frameworks-populaires)
- [Chapitre 7 : Exemples d'utilisation d'API](#chapitre-7--exemples-dutilisation-dapi)
  - [Exemple en Python avec Requests](#exemple-en-python-avec-requests)
  - [Exemple en JavaScript avec fetch](#exemple-en-javascript-avec-fetch)

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

---

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

---

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

---

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

---

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

---

## Chapitre 6 : Frameworks pour construire une API

### Frameworks populaires

- **JavaScript** : Express.js
- **Ruby** : Ruby on Rails
- **Python** : Django, Flask
- **Java** : Spring
- **Cloud** : AWS API Gateway

---

## Chapitre 7 : Exemples d'utilisation d'API

### Exemple en Python avec Requests

```python
# Installation préalable : pip install requests
import requests
import json

# Exemple 1 : Obtenir des informations sur un utilisateur GitHub
def get_github_user(username):
    # Construction de l'URL de l'endpoint
    url = f"https://api.github.com/users/{username}"
    
    # Envoi de la requête GET
    response = requests.get(url)
    
    # Vérification du code de statut
    if response.status_code == 200:
        # Conversion de la réponse JSON en dictionnaire Python
        user_data = response.json()
        print(f"Nom: {user_data.get('name', 'Non disponible')}")
        print(f"Bio: {user_data.get('bio', 'Non disponible')}")
        print(f"Followers: {user_data.get('followers', 0)}")
        print(f"Repositories publics: {user_data.get('public_repos', 0)}")
        return user_data
    else:
        print(f"Erreur: {response.status_code}")
        print(response.text)
        return None

# Exemple 2 : Créer un nouveau repository (nécessite authentification)
def create_github_repo(token, repo_name, description="", is_private=False):
    # Construction de l'URL de l'endpoint
    url = "https://api.github.com/user/repos"
    
    # Préparation des headers avec token d'authentification
    headers = {
        "Authorization": f"token {token}",
        "Accept": "application/vnd.github.v3+json"
    }
    
    # Préparation du body de la requête
    data = {
        "name": repo_name,
        "description": description,
        "private": is_private
    }
    
    # Envoi de la requête POST avec le body en JSON
    response = requests.post(url, headers=headers, data=json.dumps(data))
    
    # Vérification du code de statut
    if response.status_code in [201, 200]:  # 201 Created ou 200 OK
        repo_data = response.json()
        print(f"Repo créé: {repo_data['full_name']}")
        print(f"URL: {repo_data['html_url']}")
        return repo_data
    else:
        print(f"Erreur: {response.status_code}")
        print(response.text)
        return None

# Utilisation des fonctions
if __name__ == "__main__":
    # Exemple d'utilisation sans authentification
    user_info = get_github_user("octocat")
    
    # Exemple d'utilisation avec authentification (remplacer par votre token)
    # my_token = "votre_token_github"
    # new_repo = create_github_repo(my_token, "mon-nouveau-repo", "Un repo créé via l'API GitHub")
```

### Exemple en JavaScript avec fetch

```javascript
// Exemple 1 : Obtenir des informations sur un utilisateur GitHub

function getGithubUser(username) {
  // Construction de l'URL de l'endpoint
  const url = `https://api.github.com/users/${username}`;
  
  // Envoi de la requête GET avec fetch
  fetch(url)
    .then(response => {
      // Vérification du code de statut
      if (!response.ok) {
        throw new Error(`Erreur HTTP: ${response.status}`);
      }
      // Conversion de la réponse en JSON
      return response.json();
    })
    .then(userData => {
      // Traitement des données reçues
      console.log(`Nom: ${userData.name || 'Non disponible'}`);
      console.log(`Bio: ${userData.bio || 'Non disponible'}`);
      console.log(`Followers: ${userData.followers}`);
      console.log(`Repositories publics: ${userData.public_repos}`);
      return userData;
    })
    .catch(error => {
      console.error('Problème avec la requête fetch:', error);
    });
}

// Exemple 2 : Créer un nouveau repository (nécessite authentification)

function createGithubRepo(token, repoName, description = "", isPrivate = false) {
  // Construction de l'URL de l'endpoint
  const url = "https://api.github.com/user/repos";
  
  // Configuration de la requête
  const options = {
    method: 'POST',
    headers: {
      'Authorization': `token ${token}`,
      'Accept': 'application/vnd.github.v3+json',
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      name: repoName,
      description: description,
      private: isPrivate
    })
  };
  
  // Envoi de la requête POST avec fetch
  fetch(url, options)
    .then(response => {
      if (!response.ok) {
        throw new Error(`Erreur HTTP: ${response.status}`);
      }
      return response.json();
    })
    .then(repoData => {
      console.log(`Repo créé: ${repoData.full_name}`);
      console.log(`URL: ${repoData.html_url}`);
      return repoData;
    })
    .catch(error => {
      console.error('Problème avec la requête fetch:', error);
    });
}

// Utilisation des fonctions
// Exemple d'utilisation sans authentification
getGithubUser("octocat");

// Exemple d'utilisation avec authentification (remplacer par votre token)
// const myToken = "votre_token_github";
// createGithubRepo(myToken, "mon-nouveau-repo", "Un repo créé via l'API GitHub");

// Version avec async/await (plus moderne)
async function getGithubUserAsync(username) {
  try {
    const response = await fetch(`https://api.github.com/users/${username}`);
    if (!response.ok) {
      throw new Error(`Erreur HTTP: ${response.status}`);
    }
    const userData = await response.json();
    console.log(`Nom: ${userData.name || 'Non disponible'}`);
    console.log(`Followers: ${userData.followers}`);
    return userData;
  } catch (error) {
    console.error('Erreur:', error);
  }
}
```
