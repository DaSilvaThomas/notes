# Notes sur le déploiement d'applications Django

## 1. Configuration d'un espace serveur

### Choix du serveur
- **VPS (Virtual Private Server)** : Services comme DigitalOcean, AWS, Linode, OVH
- **Configuration recommandée** : Au moins 1GB RAM, 1 CPU, 25GB d'espace disque

### Configuration initiale du serveur
```bash
# Mise à jour du système
sudo apt update
sudo apt upgrade -y

# Installation des dépendances
sudo apt install python3-pip python3-dev libpq-dev postgresql postgresql-contrib nginx curl

# Création d'un environnement virtuel
sudo apt install python3-venv
mkdir -p ~/myapp
cd ~/myapp
python3 -m venv env
source env/bin/activate
```

### Configuration du pare-feu
```bash
# Installation et configuration UFW
sudo apt install ufw
sudo ufw allow 22
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
```

### Configuration d'un utilisateur non-root
```bash
# Créer un nouvel utilisateur
sudo adduser appuser
sudo usermod -aG sudo appuser

# Configurer SSH pour le nouvel utilisateur
sudo su - appuser
mkdir ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys
# Coller votre clé publique SSH
chmod 600 ~/.ssh/authorized_keys
```

## 2. Téléchargement de l'application sur le serveur distant

### Utilisation de Git
```bash
# Installation de Git
sudo apt install git

# Clonage du dépôt
git clone https://github.com/username/myapp.git
cd myapp

# Installation des dépendances
pip install -r requirements.txt
```

### Alternative : transfert via SCP
```bash
# Sur votre machine locale
scp -r /chemin/vers/votreapp appuser@votreserveur:/home/appuser/myapp

# Sur le serveur
cd ~/myapp
pip install -r requirements.txt
```

### Configuration de l'application Django
```bash
# Création d'un fichier .env pour les variables d'environnement
nano .env
# Ajouter les variables d'environnement nécessaires
# SECRET_KEY=votre_cle_secrete
# DEBUG=False
# ALLOWED_HOSTS=votredomaine.com,www.votredomaine.com,IP_du_serveur

# Collecte des fichiers statiques
python manage.py collectstatic --no-input

# Migrations de la base de données
python manage.py migrate
```

## 3. Utilisation du serveur HTTP Nginx

### Installation et configuration de base
```bash
# Installation de Nginx (si ce n'est pas déjà fait)
sudo apt install nginx

# Création d'un fichier de configuration pour votre site
sudo nano /etc/nginx/sites-available/myapp
```

### Configuration de Nginx pour Django
```nginx
server {
    listen 80;
    server_name votredomaine.com www.votredomaine.com;

    location = /favicon.ico { access_log off; log_not_found off; }
    
    location /static/ {
        root /home/appuser/myapp;
    }
    
    location /media/ {
        root /home/appuser/myapp;
    }
    
    location / {
        include proxy_params;
        proxy_pass http://unix:/home/appuser/myapp/myapp.sock;
    }
}
```

### Activation de la configuration
```bash
# Créer un lien symbolique
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/

# Tester la configuration
sudo nginx -t

# Redémarrer Nginx
sudo systemctl restart nginx
```

### Configuration HTTPS avec Let's Encrypt
```bash
# Installation de Certbot
sudo apt install certbot python3-certbot-nginx

# Obtention d'un certificat
sudo certbot --nginx -d votredomaine.com -d www.votredomaine.com

# Renouvellement automatique
sudo certbot renew --dry-run
```

## 4. Configuration de Gunicorn et Supervisor

### Installation et configuration de Gunicorn
```bash
# Installation de Gunicorn
pip install gunicorn

# Test de Gunicorn
cd ~/myapp
gunicorn --bind 0.0.0.0:8000 myapp.wsgi:application
```

### Création d'un fichier socket pour Gunicorn
```bash
# Arrêter le test Gunicorn (Ctrl+C)
# Création d'un fichier de service systemd
sudo nano /etc/systemd/system/gunicorn.socket
```

```ini
[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/home/appuser/myapp/myapp.sock

[Install]
WantedBy=sockets.target
```

### Création d'un service Gunicorn
```bash
sudo nano /etc/systemd/system/gunicorn.service
```

```ini
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=appuser
Group=www-data
WorkingDirectory=/home/appuser/myapp
ExecStart=/home/appuser/myapp/env/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/home/appuser/myapp/myapp.sock \
          myapp.wsgi:application

[Install]
WantedBy=multi-user.target
```

### Activation et démarrage des services Gunicorn
```bash
sudo systemctl start gunicorn.socket
sudo systemctl enable gunicorn.socket
sudo systemctl status gunicorn.socket

# Test du socket
sudo systemctl status gunicorn

# Si nécessaire, redémarrage de Gunicorn
sudo systemctl daemon-reload
sudo systemctl restart gunicorn
```

### Installation et configuration de Supervisor
```bash
# Installation de Supervisor
sudo apt install supervisor

# Création d'un fichier de configuration pour votre application
sudo nano /etc/supervisor/conf.d/myapp.conf
```

```ini
[program:myapp]
command=/home/appuser/myapp/env/bin/gunicorn myapp.wsgi:application --bind unix:/home/appuser/myapp/myapp.sock
directory=/home/appuser/myapp
user=appuser
autostart=true
autorestart=true
redirect_stderr=true
stdout_logfile=/home/appuser/myapp/logs/gunicorn.log
```

```bash
# Création du dossier de logs
mkdir -p ~/myapp/logs

# Rechargement de la configuration de Supervisor
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl status myapp
```

## 5. Vérification du déploiement

### Tests de base
1. Vérifier que le site est accessible via le navigateur
2. Vérifier que les fichiers statiques sont correctement servis
3. Tester les fonctionnalités principales de l'application

### Surveillance et journaux
```bash
# Vérification des journaux Nginx
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log

# Vérification des journaux Gunicorn
tail -f ~/myapp/logs/gunicorn.log

# Statut des services
sudo systemctl status nginx
sudo systemctl status gunicorn
sudo supervisorctl status
```

### Maintenance courante
```bash
# Mise à jour du code (avec Git)
cd ~/myapp
git pull origin main

# Mise à jour des dépendances
source env/bin/activate
pip install -r requirements.txt

# Migrations et fichiers statiques
python manage.py migrate
python manage.py collectstatic --no-input

# Redémarrage des services
sudo supervisorctl restart myapp
# ou
sudo systemctl restart gunicorn
sudo systemctl restart nginx
```
