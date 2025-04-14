# Notes sur le déploiement d'applications Django

## Table des matières
1. [Chapitre 1 : Configuration d'un espace serveur](#chapitre-1--configuration-dun-espace-serveur)
   - [Qu'est-ce qu'un SysAdmin ?](#quest-ce-quun-sysadmin-)
   - [Types de services Cloud](#types-de-services-cloud)
   - [1. Préparation avant connexion au serveur](#1-préparation-avant-connexion-au-serveur)
   - [2. Création d'un serveur virtuel](#2-création-dun-serveur-virtuel)
   - [3. Connexion SSH au serveur](#3-connexion-ssh-au-serveur)
   - [4. Création d'un utilisateur avec privilèges limités](#4-création-dun-utilisateur-avec-privilèges-limités)
   - [Récapitulatif des étapes clés](#récapitulatif-des-étapes-clés)

2. [Chapitre 2 : Téléchargez l'application sur le serveur distant](#chapitre-2--téléchargez-lapplication-sur-le-serveur-distant)
   - [Objectifs du chapitre](#objectifs-du-chapitre)
   - [1. Installation des librairies nécessaires](#1-installation-des-librairies-nécessaires)
   - [2. Téléchargement de l'application](#t2-éléchargement-de-lapplication)
   - [3. Configuration de l'environnement virtuel](#3-configuration-de-lenvironnement-virtuel)
   - [4. Configuration de la base de données PostgreSQL](#4-configuration-de-la-base-de-données-postgresql)
   - [5. Modification des paramètres Django](#5-modification-des-paramètres-django)
   - [6. Gestion des fichiers statiques](#6-gestion-des-fichiers-statiques)
   - [7. Migration et chargement des données](#7-migration-et-chargement-des-données)
   - [8. Vérification de l'installation](#8-vérification-de-linstallation)

3. [Chapitre 3 : Utilisez le serveur HTTP Nginx](#chapitre-3--utilisez-le-serveur-http-nginx)
   - [Introduction aux serveurs HTTP et adresses IP](#introduction-aux-serveurs-HTTP-et-adresses-ip)
   - [1. Présentation de Nginx](#1-présentation-de-nginx)
   - [2. Structure et syntaxe des fichiers de configuration](#2-structure-et-syntaxe-des-fichiers-de-configuration)
   - [3. Syntaxe et structure d'un fichier de configuration Nginx](#3-syntaxe-et-structure-dun-fichier-de-configuration-nginx)
   - [4. Directives clés et leur fonction](#4-directives-clés-et-leur-fonction)
   - [5. Configuration pour une application Django](#5-configuration-pour-une-application-django)
   - [6. Variables et expressions conditionnelles dans Nginx](#6-variables-et-expressions-conditionnelles-dans-nginx)
   - [7. Commandes de gestion Nginx](#7-commandes-de-gestion-nginx)
   - [Bonnes pratiques](#bonnes-pratiques)

---

## Chapitre 1 : Configuration d'un espace serveur

## Qu'est-ce qu'un SysAdmin ?

Un SysAdmin (Administrateur Système) est responsable de :
- Prendre le code d'un développeur et le mettre en œuvre sur un serveur
- S'occuper de toute la mise en production d'un projet terminé

## Types de services Cloud

Il existe plusieurs types de Cloud selon le niveau de configuration souhaité :

1. **SAAS (Software as a Service)**
   - Plateforme web accessible via un site internet
   - Aucune visibilité sur l'état des serveurs
   - Exemples : Google Drive, OneDrive, Dropbox

2. **PAAS (Platform as a Service)**
   - Plateforme à laquelle vous confiez votre code
   - Configuration automatique des serveurs
   - Maîtrise partielle sur les serveurs
   - Exemples : Heroku, Clever Cloud

3. **IAAS (Infrastructure as a Service)**
   - Location d'un espace sur un serveur
   - Installation personnalisée selon vos besoins
   - Aucune configuration automatique (sauf demande)
   - Exemples : Amazon Web Services (AWS), Openstack, DigitalOcean

## 1. Préparation avant connexion au serveur

### Sous Linux/Mac
#### Génération de clés SSH
```bash
# Générer une nouvelle paire de clés SSH
ssh-keygen -t rsa -b 4096 -C "votre_email@exemple.com"

# Appuyez sur Entrée pour accepter l'emplacement par défaut (~/.ssh/id_rsa)
# Définissez une phrase de passe si vous le souhaitez (recommandé)
```

#### Vérification des clés SSH
```bash
# Vérifier la présence de la clé publique
cd ~/.ssh
cat id_rsa.pub

# Le résultat doit ressembler à :
# ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC3... votre_email@exemple.com
```

### Sous Windows
#### Installation de PuTTY
1. Téléchargez PuTTY depuis le [site officiel](https://www.putty.org/)
2. Installez PuTTY en suivant les instructions

#### Génération de clés SSH avec PuTTYgen
1. Lancez PuTTYgen (installé avec PuTTY)
2. Cliquez sur "Generate" et déplacez la souris pour générer de l'aléatoire
3. Une fois la clé générée :
   - Entrez une phrase de passe (Key passphrase) - recommandé
   - Cliquez sur "Save private key" pour sauvegarder votre clé privée (.ppk)
   - Copiez la clé publique affichée dans la zone de texte en haut

## 2. Création d'un serveur virtuel

### Services de serveurs web courants
- **DigitalOcean** : Propose des "Droplets" (serveurs virtuels)
- **Amazon Web Services (AWS)** : EC2 pour les serveurs virtuels
- **Google Cloud Platform** : Compute Engine
- **Microsoft Azure** : Virtual Machines
- **OVH** : VPS ou serveurs dédiés
- **Linode** : Serveurs virtuels Linux

### Configuration d'un serveur sur DigitalOcean (exemple)
1. Créez un compte sur DigitalOcean
2. Cliquez sur "Create Droplet"
3. Sélectionnez les options :
   - **Image** : Choisissez le système d'exploitation (Ubuntu recommandé)
   - **Taille** : Sélectionnez selon vos besoins (pour test, option la moins chère)
   - **Région** : Choisissez un datacenter proche de vos utilisateurs
   - **Options supplémentaires** : Activez les sauvegardes si nécessaire
   - **Authentication** : Ajoutez votre clé SSH publique

### Activation du pare-feu (Firewall)
Un pare-feu détermine les connexions autorisées sur votre serveur.

Configuration recommandée pour une application Django :
- SSH (port 22) : Pour l'administration
- HTTP (port 80) : Pour accéder à l'application web
- HTTPS (port 443) : Pour les connexions sécurisées

Sur DigitalOcean :
1. Allez dans Networking > Firewalls
2. Créez un nouveau firewall
3. Ajoutez les règles entrantes (Inbound Rules) pour SSH, HTTP et HTTPS
4. Appliquez le firewall à votre Droplet

Sur d'autres fournisseurs :
- AWS : Security Groups
- Google Cloud : Règles de pare-feu
- Azure : Groupes de sécurité réseau

## 3. Connexion SSH au serveur

### Depuis Linux/Mac

#### Connexion avec l'utilisateur root
```bash
# Syntaxe de base
ssh root@ADRESSE_IP_SERVEUR

# Exemple
ssh root@178.62.117.192

# À la première connexion, acceptez l'empreinte du serveur en tapant "yes"
```

#### Avec un port personnalisé (si différent du port 22)
```bash
ssh -p NUMERO_PORT root@ADRESSE_IP_SERVEUR
```

### Depuis Windows avec PuTTY

1. Lancez PuTTY
2. Configuration de base :
   - Dans "Host Name (or IP address)" : Entrez l'adresse IP du serveur
   - Port : 22 (par défaut)
   
3. Configuration de l'authentification SSH :
   - Dans le menu de gauche, allez à Connection > SSH > Auth > Credentials
   - Sous "Private key file for authentication", sélectionnez votre fichier .ppk
   
4. Sauvegardez la session :
   - Revenez à "Session"
   - Entrez un nom dans "Saved Sessions"
   - Cliquez sur "Save"
   
5. Cliquez sur "Open" pour démarrer la connexion
   - Acceptez l'alerte de sécurité lors de la première connexion
   - Entrez "root" comme nom d'utilisateur

## 4. Création d'un utilisateur avec privilèges limités

### Pourquoi créer un autre utilisateur ?
- Sécurité : évite les actions dangereuses sans confirmation
- Protection : limite les risques en cas de compromission de l'application
- Bonnes pratiques : ne jamais utiliser root pour les opérations quotidiennes

### Création d'un nouvel utilisateur
```bash
# Connecté en tant que root
adduser nom_utilisateur

# Exemple
adduser celinems

# Suivez les instructions pour définir le mot de passe et les informations
```

### Ajout des privilèges sudo
```bash
# Ajouter l'utilisateur au groupe sudo
gpasswd -a nom_utilisateur sudo

# Exemple
gpasswd -a celinems sudo
```

### Configuration de l'accès SSH pour le nouvel utilisateur

```bash
# Passer à l'utilisateur créé
su - nom_utilisateur

# Créer le dossier SSH et définir les permissions
mkdir .ssh
chmod 700 .ssh

# Créer le fichier pour les clés autorisées
vi .ssh/authorized_keys
# (Collez votre clé publique et sauvegardez avec :wq)

# Définir les permissions du fichier
chmod 600 .ssh/authorized_keys

# Quitter la session utilisateur
exit
```

#### Méthode alternative depuis votre machine locale
```bash
# Pour Linux/Mac
scp ~/.ssh/id_rsa.pub nom_utilisateur@ADRESSE_IP:.ssh/authorized_keys

# Pour Windows (depuis PowerShell ou cmd avec PuTTY installé)
pscp C:\chemin\vers\votre_cle.pub nom_utilisateur@ADRESSE_IP:.ssh/authorized_keys
```

### Désactivation de la connexion root via SSH

```bash
# Éditer le fichier de configuration SSH
sudo vi /etc/ssh/sshd_config

# Chercher la ligne "PermitRootLogin yes" et remplacer par
PermitRootLogin no

# Sauvegarder et quitter (:wq dans vi)

# Redémarrer le service SSH
sudo service ssh reload
```

### Test de la configuration

**Important** : Ne fermez pas la session root actuelle tant que vous n'avez pas vérifié que la nouvelle configuration fonctionne !

1. Ouvrez un nouveau terminal ou une nouvelle fenêtre PuTTY
2. Tentez de vous connecter avec le nouvel utilisateur :
   ```bash
   # Linux/Mac
   ssh nom_utilisateur@ADRESSE_IP
   
   # Windows : Configurez une nouvelle session PuTTY avec le nouvel utilisateur
   ```
3. Vérifiez que vous pouvez exécuter des commandes sudo :
   ```bash
   sudo apt update
   ```

## Récapitulatif des étapes clés

1. Générer les clés SSH sur votre machine locale
2. Créer un serveur chez un fournisseur de services cloud (DigitalOcean, AWS, etc.)
3. Configurer le pare-feu pour autoriser SSH, HTTP et HTTPS
4. Se connecter en SSH avec l'utilisateur root
5. Créer un nouvel utilisateur avec les privilèges sudo
6. Configurer l'accès SSH pour ce nouvel utilisateur
7. Désactiver la connexion SSH pour root
8. Tester la nouvelle configuration

---

# Chapitre 2 : Téléchargez l'application sur le serveur distant

## Objectifs du chapitre
- Installer les paquets nécessaires sur Ubuntu
- Télécharger une application Django sur un serveur
- Configurer l'environnement de développement
- Migrer les données

## 1. Installation des librairies nécessaires

### Paquets essentiels
```bash
sudo apt-get update
sudo apt-get install python3-pip python3-dev libpq-dev postgresql postgresql-contrib
```

**Important :** Toujours mettre à jour les programmes du système avec `apt-get update` avant d'installer de nouveaux logiciels.

### Paquets installés
- Python 3
- PostgreSQL et ses extensions

## 2. Téléchargement de l'application

### Clonage depuis GitHub
```bash
git clone votredepot.git
```

**Note :** Il n'est pas nécessaire de configurer SSH si on clone simplement un dépôt public.

**Astuce :** On peut renommer le dossier cloné pour plus de simplicité (ex: "disquaire").

## 3. Configuration de l'environnement virtuel

### Installation et activation
```bash
sudo apt install virtualenv
virtualenv env -p python3
source env/bin/activate
```

### Installation des dépendances
```bash
pip install -r disquaire/requirements.txt
```

## 4. Configuration de la base de données PostgreSQL

### Connexion à PostgreSQL
```bash
sudo -u postgres psql
```

### Création de la base de données
```sql
CREATE DATABASE disquaire;
```

### Création d'un utilisateur
```sql
CREATE USER celinems WITH PASSWORD '0+0=LaTeteaT0t0';
```

### Configuration des paramètres pour optimiser les performances
```sql
ALTER ROLE celinems SET client_encoding TO 'utf8';
ALTER ROLE celinems SET default_transaction_isolation TO 'read committed';
ALTER ROLE celinems SET timezone TO 'Europe/Paris';
```

### Attribution des privilèges
```sql
GRANT ALL PRIVILEGES ON DATABASE disquaire TO celinems;
```

**Note :** Pour quitter la console PostgreSQL, tapez `\q`.

**Remarque :** Le serveur PostgreSQL est déjà en cours d'exécution après l'installation (tâche de fond). C'est un comportement standard pour les services installés via apt-get.

## 5. Modification des paramètres Django

### Configuration de la base de données dans `settings.py`
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'disquaire',
        'USER': 'celinems',
        'PASSWORD': '0+0=LaTeteaT0t0',
        'HOST': 'localhost',
        'PORT': '5432',
    }
}
```

## 6. Gestion des fichiers statiques

### Définition de la variable d'environnement
```bash
export ENV=PRODUCTION
```

**Attention :** Cette variable n'est valable que dans le terminal actuel. Une solution plus pérenne sera abordée plus tard dans le cours.

### Collecte des fichiers statiques
```bash
./disquaire/manage.py collectstatic
```

## 7. Migration et chargement des données

### Application des migrations
```bash
./disquaire/manage.py migrate
```

### Importation des données existantes
```bash
./disquaire/manage.py loaddata disquaire/store/dumps/store.json
```

### Création d'un super utilisateur
```bash
./disquaire/manage.py createsuperuser
```

## 8. Vérification de l'installation

Pour tester le serveur de développement :
```bash
./disquaire/manage.py runserver
```

**Note :** Le serveur de développement fonctionne à l'adresse http://127.0.0.1:8000/, mais il faudra configurer Gunicorn pour gérer les requêtes en production (sujet du prochain chapitre).

---

Voici une version plus détaillée de la note sur Nginx au format Markdown, avec une attention particulière à l'explication du fonctionnement des fichiers de configuration Nginx :

## Chapitre 3 : Utilisez le serveur HTTP Nginx

### Introduction aux serveurs HTTP et adresses IP

- **Serveur HTTP** : Logiciel qui reçoit des requêtes HTTP et renvoie des réponses (pages web, fichiers, etc.)
- **Types d'adresses IP** :
  - **Adresse IP privée** (ex: 127.0.0.1) : Identifie un ordinateur sur un réseau local, non accessible depuis Internet
  - **Adresse IP publique** : Identifiant unique visible sur Internet, fourni par votre hébergeur (ex: Digital Ocean)
- **Rôle du serveur HTTP** : Transférer les requêtes arrivant à l'adresse IP publique vers l'application locale (sur l'adresse IP privée)

### 1. Présentation de Nginx

- **Définition** : Serveur web open source créé par Igor Sysoev en 2002 pour Rambler (site russe à fort trafic)
- **Avantages** : Syntaxe simple, haute performance, faible consommation de ressources
- **Adoption** : 20,75% des sites actifs dans le monde en septembre 2017 selon Netcraft
- **Installation** : `sudo apt-get install nginx`
- **Emplacement** : `/etc/nginx/`

### 2. Structure et syntaxe des fichiers de configuration

- **Dossiers principaux** :
  - `/etc/nginx/` : Dossier racine de configuration
  - `/etc/nginx/nginx.conf` : Configuration globale
  - `/etc/nginx/sites-available/` : Fichiers de configuration des sites disponibles
  - `/etc/nginx/sites-enabled/` : Liens symboliques vers les configurations actives
  - `/var/www/html/` : Emplacement par défaut des fichiers web

- **Liens symboliques** :
  - Permettent d'activer/désactiver des sites sans dupliquer les fichiers
  - Création : `sudo ln -s /etc/nginx/sites-available/mon_site /etc/nginx/sites-enabled/`
  - Un `@` dans le listing indique un lien symbolique

### 3. Syntaxe et structure d'un fichier de configuration Nginx

- **Commentaires** : Lignes commençant par `#`, ignorées par l'interpréteur
  ```nginx
  # Ceci est un commentaire
  ```

- **Directives** : Instructions simples composées d'un mot-clé, d'arguments et d'un point-virgule
  ```nginx
  listen 80;          # Mot-clé + argument + point-virgule
  root /var/www/html; # Indique le dossier racine des fichiers
  ```

- **Blocs** : Regroupent des directives entre accolades `{}`
  ```nginx
  server {
      listen 80;
      root /var/www/html;
  }
  ```

- **Hiérarchie des blocs** : Les blocs peuvent être imbriqués
  ```nginx
  server {
      location / {
          try_files $uri $uri/ =404;
      }
  }
  ```

- **Indentation** : 4 espaces recommandés pour la lisibilité

### 4. Directives clés et leur fonction

- **`server`** : Définit un serveur virtuel (un site ou une application)
  ```nginx
  server {
      # Configuration du serveur virtuel
  }
  ```

- **`listen`** : Spécifie le port et éventuellement l'interface réseau
  ```nginx
  listen 80;                # Port HTTP standard
  listen 443 ssl;           # Port HTTPS avec SSL
  listen [::]:80;           # IPv6
  listen 80 default_server; # Serveur par défaut pour ce port
  ```

- **`server_name`** : Définit les noms d'hôte (domaines) auxquels ce bloc s'applique
  ```nginx
  server_name example.com www.example.com; # Plusieurs domaines
  server_name *.example.com;               # Sous-domaines
  server_name _;                           # Tous les domaines (configuration par défaut)
  server_name 178.62.117.192;              # Adresse IP directe
  ```

- **`root`** : Définit le répertoire racine pour chercher les fichiers
  ```nginx
  root /var/www/mon_site;  # Tous les chemins sont relatifs à ce dossier
  ```

- **`index`** : Liste des fichiers à rechercher comme page d'index par ordre de priorité
  ```nginx
  index index.html index.htm index.php;
  ```

- **`location`** : Définit comment traiter les requêtes selon l'URI
  ```nginx
  location / {
      # Configuration pour toutes les URIs commençant par /
  }
  
  location /static/ {
      # Configuration spécifique pour les URIs commençant par /static/
  }
  
  location ~ \.php$ {
      # URIs correspondant à l'expression régulière (fichiers .php)
  }
  ```

- **`try_files`** : Teste l'existence de fichiers dans un ordre spécifié
  ```nginx
  try_files $uri $uri/ =404;
  # Cherche le fichier demandé ($uri)
  # Puis cherche le même chemin comme dossier ($uri/)
  # Sinon renvoie une erreur 404
  ```

- **`proxy_pass`** : Redirige les requêtes vers un autre serveur (ex: application Django)
  ```nginx
  proxy_pass http://127.0.0.1:8000;
  ```

- **`alias`** : Définit un chemin alternatif pour un emplacement spécifique
  ```nginx
  location /static/ {
      alias /home/user/project/staticfiles/;
      # Une requête pour /static/css/style.css cherchera
      # /home/user/project/staticfiles/css/style.css
  }
  ```

### 5. Configuration pour une application Django

#### 1. Création et activation du fichier de configuration

```bash
sudo touch /etc/nginx/sites-available/django_project
sudo ln -s /etc/nginx/sites-available/django_project /etc/nginx/sites-enabled/
```

#### 2. Configuration complète pour une application Django

```nginx
server {
    # Écoute sur le port HTTP standard
    listen 80;
    
    # Nom de domaine ou adresse IP
    server_name 178.62.117.192 example.com;
    
    # Dossier racine du projet
    root /home/user/django_project/;
    
    # Servir les fichiers statiques directement
    location /static/ {
        alias /home/user/django_project/staticfiles/;
        expires 30d;           # Cache des fichiers statiques pendant 30 jours
        access_log off;        # Désactive les logs pour les fichiers statiques
        add_header Cache-Control "public";
    }
    
    # Servir les fichiers media (téléchargés par les utilisateurs)
    location /media/ {
        alias /home/user/django_project/media/;
    }
    
    # Pour toutes les autres requêtes
    location / {
        # Headers pour le proxy
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Désactive la redirection
        proxy_redirect off;
        
        # Si le fichier n'existe pas directement
        if (!-f $request_filename) {
            # Passe la requête à l'application Django
            proxy_pass http://127.0.0.1:8000;
            break;
        }
    }

    # Options de sécurité supplémentaires
    # Empêcher l'accès aux fichiers cachés
    location ~ /\. {
        deny all;
        access_log off;
        log_not_found off;
    }
}
```

#### 3. Configuration Django associée

Dans le fichier `settings.py` de votre projet Django :

```python
# Autoriser l'adresse IP du serveur et le nom de domaine
ALLOWED_HOSTS = ['178.62.117.192', 'example.com']

# Si vous utilisez WhiteNoise pour les fichiers statiques en développement,
# désactivez-le en production puisque Nginx s'en charge
MIDDLEWARE = [
    # ...
    # Supprimer ou commenter: 'whitenoise.middleware.WhiteNoiseMiddleware',
]

# Supprimer également cette configuration si elle existe
# STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'
```

### 6. Variables et expressions conditionnelles dans Nginx

- **Variables** : Nginx fournit de nombreuses variables intégrées
  ```nginx
  $uri            # URI demandée
  $http_host      # Valeur de l'en-tête Host
  $remote_addr    # Adresse IP du client
  $scheme         # Protocole (http ou https)
  ```

- **Conditions** : Utilisez `if` pour une logique conditionnelle
  ```nginx
  if (!-f $request_filename) {
      # Exécuté si le fichier n'existe pas
      proxy_pass http://127.0.0.1:8000;
  }
  
  if ($request_method = POST) {
      # Actions spécifiques pour les requêtes POST
  }
  ```

### 7. Commandes de gestion Nginx

- **Tester la configuration** : `sudo nginx -t`
- **Recharger la configuration** : `sudo service nginx reload` ou `sudo nginx -s reload`
- **Redémarrer le service** : `sudo service nginx restart`
- **Arrêter le service** : `sudo service nginx stop`
- **Démarrer le service** : `sudo service nginx start`
- **Vérifier le statut** : `sudo service nginx status`

### Bonnes pratiques

1. **Ne pas utiliser le serveur de développement Django en production** : 
   - `./manage.py runserver` est uniquement pour le développement
   - Utiliser WSGI avec Gunicorn ou uWSGI en production

2. **Logs et débogage** :
   - Logs d'erreur : `/var/log/nginx/error.log`
   - Logs d'accès : `/var/log/nginx/access.log`

3. **Sécurité** :
   - Toujours maintenir Nginx à jour
   - Configurer SSL/TLS pour HTTPS
   - Limiter les méthodes HTTP autorisées si nécessaire
   - Protéger contre les attaques courantes (XSS, injections)

4. **Performance** :
   - Activer la compression gzip
   - Configurer le cache pour les fichiers statiques
   - Optimiser les buffers et les timeouts

---
