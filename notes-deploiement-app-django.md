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
   - [2. Téléchargement de l'application](#2-téléchargement-de-lapplication)
   - [3. Configuration de l'environnement virtuel](#3-configuration-de-lenvironnement-virtuel)
   - [4. Configuration de la base de données PostgreSQL](#4-configuration-de-la-base-de-données-postgresql)
   - [5. Modification des paramètres Django](#5-modification-des-paramètres-django)
   - [6. Gestion des fichiers statiques](#6-gestion-des-fichiers-statiques)
   - [7. Migration et chargement des données](#7-migration-et-chargement-des-données)
   - [8. Vérification de l'installation](#8-vérification-de-linstallation)

3. [Chapitre 3 : Utilisez le serveur HTTP Nginx](#chapitre-3--utilisez-le-serveur-http-nginx)
   - [Qu'est-ce que Nginx ?](#quest-ce-que-nginx-)
   - [Configuration par défaut](#configuration-par-défaut)
   - [1. Mise en application](#1-mise-en-application)
      - [1.1. Création d'un fichier de configuration](#11-création-dun-fichier-de-configuration)
      - [1.2. Servir l'application Django](#12-servir-lapplication-django)
      - [1.3. Servir les fichiers statiques](#13-servir-les-fichiers-statiques)

4. [Chapitre 4 : Configurez Gunicorn et Supervisor](#chapitre-4--configurez-gunicorn-et-supervisor)
   - [Qu'est-ce que WSGI ?](#quest-ce-que-wsgi-)
   - [Pourquoi utiliser un serveur WSGI en production ?](#pourquoi-utiliser-un-serveur-wsgi-en-production-)
   - [1. Gunicorn](#1-gunicorn)
   - [2. Supervisor](#2-supervisor)

5. [Chapitre 5 : Séparez les environnements](#chapitre-5--séparez-les-environnements)
   - [Pourquoi séparer les environnements ?](#pourquoi-séparer-les-environnements-)
   - [Préparation](#préparation)
   - [1. Configuration de l'environnement local](#1-configuration-de-lenvironnement-local)
   - [2. Configuration de l'environnement de production](#2-configuration-de-lenvironnement-de-production)

6. [Chapitre 6 : Intégrez des changements automatiquement](#chapitre-6--intégrez-des-changements-automatiquement)
   - [Intégration continue (CI)](#pourquoi-séparer-les-environnements-)
   - [1. Travis CI](#préparation)
      - [1.1. Activer l'intégration continue avec Travis CI](#1-configuration-de-lenvironnement-local)
      - [1.2. Étapes d'un déploiement sur le serveur de test](#2-configuration-de-lenvironnement-de-production)
      - [1.3. Créer un fichier de configuration](#2-configuration-de-lenvironnement-de-production)
   - [Configuration spécifique pour Django avec PostgreSQL](#configuration-spécifique-pour-django-avec-postgresql)

7. [Chapitre 7 : Surveillez l'activité d'un serveur](#chapitre-7--surveillez-lactivité-dun-serveur)
   - [Monitorer un serveur, à quoi ça sert ?](#monitorer-un-serveur-à-quoi-ça-sert-)
   - [1. Activer le monitoring de Digital Ocean](#1-activer-le-monitoring-de-digital-ocean)
   - [2. Configurer Sentry](#2-configurer-sentry)

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

# Chapitre 3 : Utilisez le serveur HTTP Nginx

## Qu'est-ce que Nginx ?

Nginx (prononcé "engine-x") est un logiciel libre de serveur web développé par Igor Sysoev depuis 2002 pour les besoins d'un site russe à fort trafic (Rambler). En septembre 2017, Nginx était utilisé par 20,75% des sites actifs dans le monde selon Netcraft.

### Pourquoi utiliser un serveur HTTP ?
- Les adresses publiques et privées sont différentes
- L'adresse `127.0.0.1:8000` n'est pas accessible depuis l'extérieur
- Un serveur HTTP transfère les requêtes de l'IP publique vers l'IP privée

### Installation de Nginx
```bash
sudo apt-get install nginx
```

Une fois installé, Nginx gère automatiquement le trafic arrivant sur votre serveur. Pour le vérifier, entrez l'adresse IP publique de votre serveur dans un navigateur web, et vous devriez voir un message de bienvenue Nginx.

## Configuration par défaut

La configuration par défaut de Nginx se trouve dans `/etc/nginx/sites-enabled/`. Le fichier principal est `default`.

### Syntaxe de configuration Nginx

- **Commentaires** : Les lignes commençant par `#` sont des commentaires
- **Directives** : Commencent par un mot-clé, suivies d'arguments et se terminent par un point-virgule
- **Blocs** : Délimités par des accolades `{}`
- **Indentation** : Recommandée pour la lisibilité (4 espaces par niveau)

### Principales directives

- **server** : Regroupe les configurations pour un "serveur virtuel"
- **listen** : Spécifie le port à écouter (80 par défaut pour HTTP)
- **root** : Indique le dossier racine des fichiers à servir
- **index** : Définit les fichiers d'index à rechercher par ordre de priorité
- **server_name** : Spécifie le nom de domaine ou l'adresse IP ciblée
- **location** : Fait référence au chemin relatif dans l'URL (URI)
- **try_files** : Vérifie l'existence des fichiers par ordre de priorité

## 1. Mise en application

### 1.1. Création d'un fichier de configuration

Par convention, il est recommandé de créer un fichier de configuration distinct pour chaque application/site hébergé sur le serveur.

1. **Structure des dossiers de configuration Nginx**:
   - `/etc/nginx/sites-available/` : Contient tous les fichiers de configuration
   - `/etc/nginx/sites-enabled/` : Contient des liens symboliques vers les configurations actives

2. **Création d'un nouveau fichier de configuration**:
```bash
sudo touch /etc/nginx/sites-available/disquaire
```

3. **Création d'un lien symbolique**:
```bash
sudo ln -s /etc/nginx/sites-available/disquaire /etc/nginx/sites-enabled
```

### 1.2. Servir l'application Django

Pour rediriger le trafic vers l'application Django, voici la configuration à mettre dans `/etc/nginx/sites-available/disquaire` :

```nginx
server { 
    listen 80;
    server_name 178.62.117.192; 
    root /home/celinems/disquaire/;
        
    location / {
        proxy_set_header Host $http_host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_redirect off;
        proxy_pass http://127.0.0.1:8000;
    }
}
```

#### Explication des directives :
- **listen 80** : Le port par défaut pour les requêtes HTTP
- **server_name 178.62.117.192** : L'adresse IP du serveur (à remplacer par la vôtre ou un nom de domaine)
- **root /home/celinems/disquaire/** : Le dossier racine de l'application
- **location /** : Toutes les requêtes commençant par `/` seront traitées par ce bloc
- **proxy_pass** : Redirige les requêtes vers l'application Django sur le port 8000
- **proxy_set_header** : Réécrit les en-têtes HTTP pour transmettre les informations nécessaires à Django

### Activation de la configuration

Pour appliquer les modifications de configuration :

```bash
sudo service nginx reload
```

### Configuration de Django

Pour que Django accepte les requêtes provenant de votre adresse IP publique, modifiez le fichier `settings.py` :

```python
ALLOWED_HOSTS = ['178.62.117.192']  # Remplacez par votre adresse IP
```

> **Note :** Ne mettez pas de barre oblique à la fin de l'adresse IP !

### 1.3. Servir les fichiers statiques

L'application utilise WhiteNoise pour servir les fichiers statiques, mais Nginx est plus performant pour cette tâche.

1. **Désactivation de WhiteNoise** dans `settings.py` :
```python
MIDDLEWARE = [
    # ...
    # Supprimer la ligne suivante
    # 'whitenoise.middleware.WhiteNoiseMiddleware',
]

# ...

if os.environ.get('ENV') == 'PRODUCTION':
    # ...
    # Supprimer la ligne suivante
    # STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'
    # ...
```

2. **Configuration de Nginx** pour servir les fichiers statiques :
```nginx
server {
    # ... 

    location /static {
        alias /home/celinems/disquaire/disquaire_project/staticfiles/;
    }

    location / {
        proxy_set_header Host $http_host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_redirect off;
        if (!-f $request_filename) {
            proxy_pass http://127.0.0.1:8000;
            break;
        }
    }
}
```

La directive `location /static` indique à Nginx de servir les fichiers du dossier `staticfiles` directement, sans passer par Django.

> **Important :** Le serveur de développement Django (`./manage.py runserver`) n'est pas adapté pour la production. Il faudra configurer Gunicorn et Supervisor, qui seront abordés dans le prochain chapitre.

N'oubliez pas de recharger Nginx après chaque modification :
```bash
sudo service nginx reload
```

---

# Chapitre 4 : Configurez Gunicorn et Supervisor

## Qu'est-ce que WSGI ?

WSGI (Web Server Gateway Interface) est une spécification qui décrit comment un serveur web communique avec des applications web Python. Elle sert d'interface standardisée entre les serveurs HTTP et les applications Python.

## Pourquoi utiliser un serveur WSGI en production ?

- Le serveur de développement Django n'est pas conçu pour la production
- Les serveurs WSGI comme Gunicorn sont plus robustes et performants
- Ils permettent de gérer efficacement les requêtes HTTP et de les transférer au code Python

## 1. Gunicorn

Gunicorn (Green Unicorn) est un serveur HTTP Python pour Unix qui implémente les spécifications WSGI. Ses avantages principaux sont :

- Rapide et puissant
- Facile à configurer
- Très utilisé dans l'écosystème Python

### Installation de Gunicorn

Gunicorn est généralement installé via pip dans l'environnement virtuel du projet :

```bash
# Généralement inclus dans le fichier requirements.txt
pip install gunicorn
```

### Lancement de Gunicorn

```bash
# Syntaxe de base
(env) gunicorn [module_wsgi]:[application]

# Exemple pour un projet Django
(env) gunicorn disquaire_project.wsgi:application
```

**Note :** Django intègre par défaut un module WSGI (`project_name/wsgi.py`) prêt à l'emploi.

### Sortie typique de Gunicorn

```
[2017-09-11 23:03:46 +0000] [15335] [INFO] Starting gunicorn 19.7.1
[2017-09-11 23:03:46 +0000] [15335] [INFO] Listening at: http://127.0.0.1:8000 (15335)
[2017-09-11 23:03:46 +0000] [15335] [INFO] Using worker: sync
[2017-09-11 23:03:46 +0000] [15338] [INFO] Booting worker with pid: 15338
```

### Options de configuration

Gunicorn accepte de nombreux paramètres optionnels :
- Port d'écoute
- Nombre de workers
- Niveau de log
- Timeout

Pour plus d'informations, consultez la [documentation officielle de Gunicorn](https://docs.gunicorn.org/).

## 2. Supervisor

Supervisor est un système de contrôle de processus pour les environnements UNIX. Il permet de :

- Surveiller plusieurs processus
- Redémarrer automatiquement les processus en cas de défaillance
- Faciliter la gestion des services

### Pourquoi utiliser Supervisor ?

Dans un scénario de production, si Gunicorn tombe en panne (par exemple à cause d'une surcharge mémoire), l'application devient inaccessible. Supervisor peut :
- Surveiller l'état de Gunicorn
- Le redémarrer automatiquement en cas de défaillance
- Assurer une haute disponibilité de l'application

### Installation de Supervisor

```bash
sudo apt-get install supervisor
```

Après l'installation, Supervisor démarre automatiquement et s'exécute en arrière-plan.

### Configuration de Supervisor

Supervisor utilise des fichiers de configuration situés dans `/etc/supervisor/conf.d/`. Chaque fichier avec l'extension `.conf` représente un processus à surveiller.

#### Création d'un fichier de configuration pour Gunicorn

```bash
sudo vi /etc/supervisor/conf.d/disquaire-gunicorn.conf
```

#### Contenu du fichier de configuration

```ini
[program:disquaire-gunicorn]
command = /home/celinems/env/bin/gunicorn disquaire_project.wsgi:application
user = celinems
directory = /home/celinems/disquaire
autostart = true
autorestart = true
environment = ENV="PRODUCTION",SECRET_KEY="votre_clé_secrète"
```

#### Explication des directives

- **program:disquaire-gunicorn** : Nom du processus à surveiller
- **command** : Chemin absolu vers l'exécutable Gunicorn et ses paramètres
- **user** : L'utilisateur qui exécute le processus
- **directory** : Le répertoire de travail
- **autostart** : Démarrage automatique au lancement de Supervisor
- **autorestart** : Redémarrage automatique en cas d'échec
- **environment** : Variables d'environnement à définir

### Génération d'une clé secrète pour Django

```python
# Dans une console Python
import random, string
"".join([random.choice(string.printable) for _ in range(24)])
# Exemple de sortie : '-~aO;| F;rE[??/w^zcumh(9'
```

### Activation de la configuration

```bash
# Informer Supervisor de la disponibilité du nouveau processus
sudo supervisorctl reread
# disquaire-gunicorn: available

# Mettre à jour Supervisor avec la nouvelle configuration
sudo supervisorctl update
# disquaire-gunicorn: added process group

# Vérifier l'état du processus
sudo supervisorctl status
# disquaire-gunicorn               RUNNING   pid 16309, uptime 0:00:09
```

### Bonnes pratiques de sécurité

Il est préférable de ne pas stocker la clé secrète directement dans le fichier de configuration Supervisor. Une meilleure approche consiste à :

1. Stocker les informations sensibles dans les paramètres Django
2. Utiliser des variables d'environnement ou des fichiers de configuration séparés
3. Implémenter une gestion centralisée des secrets

## Récapitulatif

1. Installation de Gunicorn (généralement via requirements.txt)
2. Test du lancement manuel de Gunicorn
3. Installation de Supervisor
4. Configuration de Supervisor pour gérer Gunicorn
5. Activation de la configuration dans Supervisor
6. Vérification du bon fonctionnement

L'application est maintenant servie par Nginx, qui transmet les requêtes à Gunicorn, lui-même surveillé par Supervisor pour assurer une haute disponibilité.

---

# Chapitre 5 : Séparez les environnements

## Pourquoi séparer les environnements ?
- Les réglages en production sont différents des réglages en local
- À mesure que votre projet grandit, les réglages deviennent plus complexes
- Il est recommandé de séparer les fichiers de configuration selon l'environnement
- La méthode la plus simple est de créer un module avec un fichier par environnement

## Préparation
Il est préférable de travailler en local plutôt que directement sur le serveur de production pour éviter les problèmes lors des requêtes utilisateurs.

Si ce n'est pas déjà fait :
```bash
# Clonez en local le dépôt GitHub que vous avez forké précédemment
git clone URL_DE_VOTRE_DEPOT
```

## 1. Configuration de l'environnement local

### Création d'un module de paramètres
```bash
# Créer un répertoire pour les paramètres
mkdir disquaire_project/settings

# Créer le fichier d'initialisation
touch disquaire_project/settings/__init__.py
```

### Organisation des fichiers de configuration

Par commodité :
- Les réglages par défaut seront ceux de développement
- Chaque environnement aura son propre fichier qui écrasera les réglages par défaut si nécessaire

### Migration des paramètres

1. Copiez l'intégralité du contenu de `settings.py` dans `__init__.py`
2. Supprimez le fichier `settings.py` original
3. Dans `__init__.py`, supprimez les structures conditionnelles `if os.environ.get('ENV') == 'PRODUCTION'`
4. Gardez uniquement les réglages spécifiques à l'environnement de développement local

Voici le contenu du fichier `settings/__init__.py` :

```python
"""
Django settings for disquaire_project project.

Generated by 'django-admin startproject' using Django 1.11.3.

For more information on this file, see
https://docs.djangoproject.com/en/1.11/topics/settings/

For the full list of settings and their values, see
https://docs.djangoproject.com/en/1.11/ref/settings/
"""

import os


# Build paths inside the project like this: os.path.join(BASE_DIR, ...)
BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))


# Quick-start development settings - unsuitable for production
# See https://docs.djangoproject.com/en/1.11/howto/deployment/checklist/

# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = '4i&amp;u(!%shd*0-3$ls)fohsjsd48t(gu%1-ch_wyzk7@#n3bd8e'
# '-~aO;| F;rE[??/w^zcumh(9'


# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = True


ALLOWED_HOSTS = []


# Application definition

INSTALLED_APPS = [
    'store.apps.StoreConfig',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'debug_toolbar',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    'django.middleware.locale.LocaleMiddleware',
    'debug_toolbar.middleware.DebugToolbarMiddleware',
]

ROOT_URLCONF = 'disquaire_project.urls'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]


WSGI_APPLICATION = 'disquaire_project.wsgi.application'


# Database
# https://docs.djangoproject.com/en/1.11/ref/settings/#databases

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql', # on utilise l'adaptateur postgresql
        'NAME': 'disquaire', # le nom de notre base de données créée précédemment
        'USER': 'celinems', # attention : remplacez par votre nom d'utilisateur !!
        'PASSWORD': '',
        'HOST': '',
        'PORT': '5432',
    }
}



# Password validation
# https://docs.djangoproject.com/en/1.11/ref/settings/#auth-password-validators

AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]


# Internationalization
# https://docs.djangoproject.com/en/1.11/topics/i18n/

LANGUAGE_CODE = 'fr'

TIME_ZONE = 'Europe/Paris'

USE_I18N = True

USE_L10N = True

USE_TZ = True


# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/1.11/howto/static-files/

STATIC_URL = '/static/'

# Django debug toolbar
INTERNAL_IPS = ['127.0.0.1']

STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
```

> **Note :** Si l'application renvoie une erreur 400, vérifiez que la constante BASE_DIR est bien égale à `os.path.dirname(os.path.dirname(os.path.abspath(__file__)))`.

## 2. Configuration de l'environnement de production

Le fichier de configuration de production contient des informations sensibles (identifiants de base de données, clé secrète, etc.) qui ne doivent pas être versionnées sur GitHub.

### Stratégie de gestion

- Le fichier de production ne doit exister que sur le serveur
- Créer un fichier `.gitignore` pour exclure ce fichier du suivi Git
- Envoyer sur GitHub les modifications de structure
- Récupérer ces modifications sur le serveur
- Créer le fichier de production directement sur le serveur

### Mise en place

1. **Création du fichier .gitignore**:
```bash
touch .gitignore
```

2. **Contenu du fichier .gitignore**:
```
disquaire_project/settings/production.py
__pycache__
disquaire_project/staticfiles/
```

3. **Envoi des modifications sur GitHub**:
```bash
git add .gitignore disquaire_project/settings
git commit -m "new settings configuration"
git push origin master
```

4. **Sur le serveur, récupération des modifications**:
```bash
celinems@disquaire:~/disquaire$ git pull origin master
```

5. **Création du fichier de configuration de production sur le serveur**:
```bash
celinems@disquaire:~/disquaire$ touch disquaire_project/settings/production.py
```

6. **Contenu du fichier settings/production.py**:
```python
from . import *

SECRET_KEY = '-~aO;| F;rE[??/w^zcumh(9'
DEBUG = False
ALLOWED_HOSTS = ['178.62.117.192']  # Remplacez par votre adresse IP

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'disquaire',
        'USER': 'celinems',  # Remplacez par votre nom d'utilisateur
        'PASSWORD': '0+0=LaTeteaT0t0',
        'HOST': '',
        'PORT': '5432',
    }
}
```

### Configuration de l'application pour utiliser les nouveaux paramètres

Django cherche la variable d'environnement `DJANGO_SETTINGS_MODULE` pour identifier le module de configuration à utiliser. Par défaut, il s'agit de `votreprojet.settings`.

Pour modifier cette variable, nous utilisons Supervisor :

1. **Édition du fichier de configuration Supervisor**:
```bash
celinems@disquaire:~/disquaire$ sudo vi /etc/supervisor/conf.d/disquaire-gunicorn.conf
```

2. **Modification de la ligne environment**:
```
environment = DJANGO_SETTINGS_MODULE='disquaire_project.settings.production'
```

3. **Rechargement de la configuration Supervisor**:
```bash
(env) celinems@disquaire:~/disquaire$ sudo supervisorctl reread
disquaire-gunicorn: changed
(env) celinems@disquaire:~/disquaire$ sudo supervisorctl update
disquaire-gunicorn: stopped
disquaire-gunicorn: updated process group
(env) celinems@disquaire:~/disquaire$ sudo supervisorctl status
disquaire-gunicorn               RUNNING   pid 21764, uptime 0:00:09
```

## Récapitulatif

1. Création d'un module dédié aux configurations : `disquaire_project/settings/`
2. Transfert des paramètres par défaut (développement) dans `__init__.py`
3. Création d'un fichier non-versionné `production.py` sur le serveur
4. Configuration de Supervisor pour utiliser le bon module de paramètres

Cette approche permet une séparation claire entre les environnements de développement et de production, tout en gardant les informations sensibles uniquement sur le serveur concerné.

---

# Chapitre 6 : Intégrez des changements automatiquement

## Objectifs du chapitre
- Comprendre le concept d'intégration continue
- Mettre en place l'intégration continue avec Travis CI
- Automatiser les tests lors des pull requests

## Pourquoi l'intégration continue ?

Le workflow typique d'une équipe de développement avec Git ressemble à ceci :
1. Un développeur crée une branche pour une nouvelle fonctionnalité
2. Il envoie régulièrement son code sur GitHub
3. Il crée une pull request (PR) pour intégrer sa branche dans master
4. Un autre développeur examine la PR et lance les tests en local
5. Après éventuelles corrections, la branche est fusionnée dans master

Ce processus présente plusieurs problèmes :
- Perte de temps pour le développeur qui revoit le code
- Interruption du travail pour examiner les PR
- Tendance à regrouper trop de changements dans une seule PR
- Difficulté à se concentrer sur la qualité du code plutôt que sur l'exécution des tests

L'intégration continue permet d'automatiser ce processus en exécutant les tests automatiquement lorsqu'une PR est créée.

## Intégration continue (CI)

### Définition

L'intégration continue est une pratique des méthodologies Agile qui consiste à vérifier à chaque modification de code que celui-ci :
- Ne contient aucun bug
- N'a pas d'impact négatif sur le reste de l'application
- Respecte les bonnes pratiques de développement de l'équipe

### Objectifs de l'intégration continue

Une équipe qui pratique l'intégration continue vise à :
- Réduire presque à zéro la durée et l'effort nécessaires à chaque épisode d'intégration
- Pouvoir, à tout moment, fournir un produit exploitable

### Fonctionnement

Le processus d'intégration continue se déroule typiquement comme suit :
1. Une pull request est créée
2. Le service d'intégration continue lance l'application sur un serveur de test
3. Si les tests échouent, le service indique que la fonctionnalité n'est pas prête
4. Si les tests passent, le service affiche un voyant vert
5. Le développeur peut alors relire sereinement la PR et la valider si tout est correct

### Services d'intégration continue

Plusieurs services proposent l'intégration continue :
- Travis CI
- Jenkins
- Ansible
- GitLab CI/CD
- CircleCI

## 1. Travis CI

Travis CI est un service d'automatisation qui s'interface parfaitement avec GitHub. Il permet de :
- Créer rapidement un environnement de développement
- Faire tourner une application
- Lancer des tests automatiquement

### Avantages de Travis CI

- Configuration simple et directe
- Intégration native avec GitHub
- Configuration via un fichier YAML versionné avec le code
- Prise en charge de nombreux langages et frameworks

### Configuration de base

Pour utiliser Travis CI, deux étapes sont nécessaires :
1. Activer Travis CI pour surveiller un dépôt GitHub
2. Intégrer un fichier de configuration `.travis.yml` dans le projet GitHub

### Exemple de fichier `.travis.yml` pour Python

```yaml
language: python
python:
  - "3.5"
# command to install dependencies
install:
  - pip install -r requirements.txt
# command to run tests
script:
  - pytest # or py.test for Python versions 3.5 and below
```

Ce script simple :
- Spécifie Python 3.5 comme environnement
- Installe les dépendances listées dans `requirements.txt`
- Exécute les tests avec pytest

## 1.1. Activer l'intégration continue avec Travis CI

### Création d'un compte Travis CI

1. Rendez-vous sur https://www.travis-ci.com/
2. Cliquez sur "Sign in with GitHub"
3. Connectez-vous avec vos identifiants GitHub

### Activation d'un dépôt

1. Sur le tableau de bord Travis, cliquez sur la croix à droite de l'onglet "My repositories"
2. Travis affiche tous les dépôts publics de votre compte GitHub
3. Activez le dépôt souhaité en cliquant sur la croix grise correspondante

## 1.2. Étapes d'un déploiement sur le serveur de test

Travis CI suit un processus de déploiement en plusieurs phases :

1. **Installation** : Installation des logiciels et dépendances
   - Définie par le mot-clé `install` dans le fichier de configuration

2. **Exécution** : Lancement des tests et autres scripts
   - Définie par le mot-clé `script` dans le fichier de configuration

### Commandes additionnelles disponibles

Travis permet également d'exécuter des commandes à différentes étapes du processus :

- `before_install` : Commandes à exécuter avant la phase d'installation
- `before_script` : Commandes à exécuter après l'installation mais avant les scripts
- `after_success` ou `after_failure` : Commandes à exécuter selon le résultat des scripts
- `after_script` : Commandes à exécuter après l'exécution des scripts

### Application au projet Django

Pour un projet Django typique, les étapes peuvent être :
1. Installation des dépendances depuis `requirements.txt`
2. Configuration de la base de données (PostgreSQL)
3. Exécution des tests avec `./manage.py test`

## 1.3. Créer un fichier de configuration

Pour un projet Django avec PostgreSQL, voici un exemple de fichier `.travis.yml` :

```yaml
language: python
python:
  - '3.5'

before_script:
  - pip install -r requirements.txt

env: DJANGO_SETTINGS_MODULE="disquaire_project.settings"

services:
  - postgresql

script:
  - ./manage.py test
```

### Automatisation avec Travis

Travis est conçu pour simplifier la configuration en installant automatiquement certains outils essentiels :
- Pour Python : pip, virtualenv et activation automatique de l'environnement virtuel
- Pour les bases de données : création d'utilisateurs par défaut

### Configuration des déclencheurs de build

Vous pouvez configurer Travis pour déclencher des builds selon différents événements :

1. À chaque modification sur n'importe quelle branche
2. Uniquement pour certaines branches spécifiques
3. Lorsqu'une pull request est créée ou mise à jour

Pour limiter les builds à certaines branches, ajoutez la configuration suivante :

```yaml
# safelist
branches:
  only:
    - staging
```

Cette configuration n'exécutera les builds que lorsque des modifications sont apportées à la branche "staging".

### Envoi du fichier de configuration

Une fois le fichier `.travis.yml` créé, envoyez-le sur GitHub :

```bash
$ git add .travis.yml
$ git commit -m "Add travis file"
$ git push origin master
```

## Configuration spécifique pour Django avec PostgreSQL

### Problème courant : configuration de la base de données

Lorsque vous configurez Travis pour un projet Django avec PostgreSQL, vous pouvez rencontrer des erreurs liées aux différences entre l'environnement local et celui de Travis :

```
django.db.utils.OperationalError: FATAL: role "votre_nom_utilisateur" does not exist
```

### Solution : fichier de configuration spécifique à Travis

Pour résoudre ce problème, créez un fichier de configuration Django spécifique à Travis :

```python
# settings/travis.py
from . import *

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': '',
        'USER': 'postgres',
        'PASSWORD': '',
        'HOST': '',
        'PORT': '',
    },
}
```

Puis mettez à jour votre fichier `.travis.yml` pour utiliser cette configuration :

```yaml
language: python
python:
  - '3.5'

# safelist
branches:
  only:
    - staging

before_script:
  - pip install -r requirements.txt

services:
  - postgresql

env: DJANGO_SETTINGS_MODULE=disquaire_project.settings.travis

script:
  - ./manage.py test
```

### Déclenchement du build

Pour tester la configuration, poussez les modifications sur la branche "staging" :

```bash
$ git checkout -b staging
$ git push origin staging
```

Travis devrait alors déclencher un build automatiquement et exécuter les tests.

## Résumé

1. L'intégration continue permet d'automatiser l'exécution des tests lors des modifications de code
2. Travis CI est un service d'intégration continue facile à configurer avec GitHub
3. La configuration se fait via un fichier `.travis.yml` à la racine du projet
4. Pour Django avec PostgreSQL, il faut créer une configuration spécifique pour Travis
5. Les builds peuvent être configurés pour se déclencher uniquement sur certaines branches

Cette approche permet de gagner du temps et d'augmenter la qualité du code en assurant que les tests sont systématiquement exécutés avant l'intégration des modifications.

---

# Chapitre 7 : Surveillez l'activité d'un serveur

## Objectifs du chapitre
- Surveiller l'activité d'un serveur
- Déterminer les informations pertinentes à surveiller
- Utiliser Sentry pour afficher les informations

## Monitorer un serveur, à quoi ça sert ?

La surveillance d'un serveur ne se limite pas à vérifier que les composants fonctionnent. Elle permet de :
- Suivre l'usage de la mémoire vive et de la bande passante
- Anticiper les problèmes potentiels
- Être alerté en cas d'incident

Bien que vous puissiez suivre ces métriques en temps réel via la commande `top` dans la console, il est préférable d'utiliser un service tiers pour :
- Surveiller automatiquement votre serveur
- Vous alerter en cas d'incident
- Éviter une surveillance manuelle constante

Deux types d'activités principales sont à surveiller :
1. Les performances du serveur (matériel)
2. Le comportement de l'application Django (logs)

Les logs sont essentiels car ils permettent de :
- Remonter le temps pour comprendre ce qui s'est passé
- Tracer les erreurs, les paiements, les inscriptions, les recherches, etc.

## 1. Activer le monitoring de Digital Ocean

### Installation de l'agent

```bash
curl -sSL https://agent.digitalocean.com/install.sh | sh
```

### Métriques disponibles dans l'interface d'administration

- **CPU** : Pourcentage de puissance du processeur utilisé
- **Memory** : Quantité de mémoire vive utilisée
- **Disk I/O** : Quantité de données lues/écrites sur le disque dur par seconde
- **Disk usage** : Espace de stockage utilisé sur le disque dur
- **Bandwidth** : Quantité de données transférées entre le serveur et les ressources externes
- **Top processes** : Processus les plus exigeants en termes de mémoire et de CPU

### Configuration des alertes

Dans l'interface de Digital Ocean :
1. Cliquez sur "Monitoring"
2. Sélectionnez "Create alert policy"
3. Configurez les alertes selon vos besoins

### Métriques généralement surveillées

- Utilisation du CPU et de la RAM
- Bande passante disponible
- Nombre de requêtes par rapport à la capacité du serveur

> **Note :** La pertinence des alertes dépend des besoins spécifiques de chaque projet. Il n'existe pas de configuration universellement optimale.

## 2. Configurer Sentry

Sentry est un tableau de bord qui permet de visualiser le comportement de l'application Django avec des fonctionnalités avancées :
- Détails sur le navigateur de l'utilisateur
- Information sur les URLs visitées
- Intégration possible avec GitHub pour suivre les issues

### Mise en place de Sentry

1. **Création d'un compte**
   ```
   https://sentry.io/signup/
   ```

2. **Installation de la librairie Raven** (client Python pour Sentry)
   ```bash
   pip install raven
   ```

3. **Configuration dans les paramètres de production** (`settings/production.py`)
   ```python
   import raven

   INSTALLED_APPS += [
       'raven.contrib.django.raven_compat',
   ]

   RAVEN_CONFIG = {
       'dsn': 'https://somethingverylong@sentry.io/216272', # Remplacer par votre DSN !
       # Si vous utilisez git, vous pouvez configurer automatiquement
       # la version basée sur les informations git
       'release': raven.fetch_git_sha(os.path.dirname(os.pardir)),
   }

   LOGGING = {
       'version': 1,
       'disable_existing_loggers': True,
       'root': {
           'level': 'INFO', # WARNING par défaut. Changez pour capturer plus que les warnings.
           'handlers': ['sentry'],
       },
       'formatters': {
           'verbose': {
               'format': '%(levelname)s %(asctime)s %(module)s '
                         '%(process)d %(thread)d %(message)s'
           },
       },
       'handlers': {
           'sentry': {
               'level': 'INFO', # Pour capturer plus que ERROR, changez à WARNING, INFO, etc.
               'class': 'raven.contrib.django.raven_compat.handlers.SentryHandler',
               'tags': {'custom-tag': 'x'},
           },
           'console': {
               'level': 'DEBUG',
               'class': 'logging.StreamHandler',
               'formatter': 'verbose'
           }
       },
       'loggers': {
           'django.db.backends': {
               'level': 'ERROR',
               'handlers': ['console'],
               'propagate': False,
           },
           'raven': {
               'level': 'DEBUG',
               'handlers': ['console'],
               'propagate': False,
           },
           'sentry.errors': {
               'level': 'DEBUG',
               'handlers': ['console'],
               'propagate': False,
           },
       },
   }
   ```

### Test de la configuration

1. **Créer une erreur intentionnelle** pour vérifier que Sentry la capture
   ```python
   # store/views.py
   def index(request):
       MAIS_POURQUOI_EST_IL_SI_MECHANT ?  # Cette ligne provoquera une erreur
       # ...
   ```

2. **Redémarrer le serveur** pour appliquer les modifications
   ```bash
   sudo supervisorctl restart disquaire-gunicorn
   ```

3. **Vérifier dans Sentry** si l'erreur a été capturée en visitant `/store`

### Ajout de logs personnalisés

Vous pouvez envoyer des événements personnalisés à Sentry pour suivre des actions spécifiques :

```python
# views.py
import logging

# Obtenir une instance du logger
logger = logging.getLogger(__name__)

# ...

def search(request):
    # Enregistrer un événement de recherche
    logger.info('New search', exc_info=True, extra={
        # Passer optionnellement une requête pour obtenir plus d'informations
        'request': request,
    })
    return render(request, 'store/search.html', context)
```

Après avoir enregistré et redémarré le serveur, chaque recherche effectuée par les utilisateurs créera un événement "New search" dans le tableau de bord Sentry.

## Ressources complémentaires

Pour plus d'informations sur la gestion des logs :
- [Documentation officielle de Django sur le logging](https://docs.djangoproject.com/en/1.11/topics/logging/)
- [Documentation de Sentry pour l'intégration avec Django](https://docs.sentry.io/platforms/python/integrations/django/)