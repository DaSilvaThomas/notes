# Notes Django

## Table des matières
1. [Django: Framework Web](#django-framework-web)
   - [Installation et configuration](#installation-et-configuration)
   - [Structure d'un projet](#structure-dun-projet)
   - [Création de pages web](#création-de-pages-web)
   - [Modèles et base de données](#modèles-et-base-de-données)
   - [Architecture MVT](#architecture-mvt)
   - [Opérations CRUD](#opérations-crud)

---

## Django: Framework Web

Django est un framework Python complet et open source conçu pour rendre le développement web efficace.

### Installation et configuration

Avant de commencer un projet avec Django, il est conseillé de mettre en place un environnement virtuel Python.

```bash
# Vérifier la version de Python (3 ou supérieur)
python --version

# Créer et activer un environnement virtuel
python -m venv env
env\Scripts\activate.bat

# Installer Django
pip install django

# Enregistrer les dépendances
pip freeze > requirements.txt
```

### Structure d'un projet

**Créer un nouveau projet Django :**
```bash
django-admin startproject <nom_du_projet>
```
Cela va générer le code de base pour un projet Django fonctionnel.

Note : Une fois le projet créé, utilisez `manage.py` au lieu de `django-admin` pour les commandes suivantes. `manage.py` est conçu pour fonctionner spécifiquement avec votre projet.

**Exécuter le serveur de développement :**
```bash
python manage.py runserver
```
Le serveur de développement démarre à l'adresse http://127.0.0.1:8000/

**Créer la base de données du projet :**
```bash
python manage.py migrate
```
Cela crée un fichier de base de données appelé db.sqlite3.

**Générer le code de base pour une application Django :**
```bash
python manage.py startapp <nom_application>
```
Une application représente une partie ou un module spécifique de votre projet global.

Après avoir créé une application, il faut l'ajouter dans le fichier `settings.py` du projet :
```python
INSTALLED_APPS = [
    # ...
    '<nom_application>',
]
```

### Création de pages web

**Ajouter une nouvelle page web au projet :**

1. Créer un nouveau modèle d'URL dans le fichier `urls.py` du projet :
   ```python
   from django.urls import path
   from <nom_application> import views
   
   urlpatterns = [
       # ...
       path('<nom_url>/', views.<nom_vue>),
   ]
   ```
   Exemple : `path('about-us/', views.about)`

2. Créer une vue dans le fichier `<nom_application>/views.py` :
   ```python
   from django.http import HttpResponse
   
   def <nom_vue>(request):
       return HttpResponse('<contenu_page>')
   ```
   Exemple :
   ```python
   def about(request):
       return HttpResponse('<h1>À propos</h1> <p>Nous adorons merch !</p>')
   ```
   
   Note : Mettre du code HTML directement dans une vue est une mauvaise pratique. Il est préférable d'utiliser les templates Django.

### Modèles et base de données

Django fournit un système appelé ORM (Object-Relational Mapping) qui permet de travailler avec des objets Python (les modèles) plutôt que d'écrire des requêtes SQL manuelles.

**Création d'un nouveau modèle :**

Dans le fichier `<nom_application>/models.py` :
```python
from django.db import models

class Band(models.Model):
    name = models.fields.CharField(max_length=100)
```

**Connexion du modèle avec la base de données à l'aide des migrations :**

Créer une migration :
```bash
python manage.py makemigrations
```

Appliquer la migration :
```bash
python manage.py migrate
```

Afficher toutes les migrations :
```bash
python manage.py showmigrations
```

Revenir à une migration précédente :
```bash
python manage.py migrate <nom_application> <nom_migration_précédente>

# Exemple
python manage.py migrate listings 0005_annonce_band
```

**Remplissage de la base de données :**

Ouvrir le shell Django :
```bash
python manage.py shell
```

Dans le shell Django :
```python
# Importer le modèle
from listings.models import Band

# Méthode 1 : Créer et sauvegarder en une étape
band = Band.objects.create(name='Foo Fighters')
band = Band.objects.create(name='Cut Copy')

# Méthode 2 : Créer puis sauvegarder
band = Band()
band.name = 'Radiohead'
band.save()

# Commandes utiles
band  # Afficher le dernier objet créé
band.name  # Afficher un attribut spécifique
Band.objects.count()  # Afficher le nombre d'objets
Band.objects.all()  # Afficher tous les objets
band.delete()  # Supprimer le dernier objet créé
exit()  # Quitter le shell
```

**Affichage des données sur une page web :**

Dans le fichier `<nom_application>/views.py` :
```python
from listings.models import Band
from django.http import HttpResponse

def hello(request):
    bands = Band.objects.all()
    return HttpResponse(f"""
        <ul>
            <li>{bands[0].name}</li>
            <li>{bands[1].name}</li>
            <li>{bands[2].name}</li>
        </ul>
    """)
```

### Architecture MVT

Django suit l'architecture Modèle-Vue-Template (MVT) :
- **Modèle** : Stocke les données
- **Vue** : Récupère les données
- **Template** : Affiche les données

**Créer un template HTML :**

1. Créer un fichier dans le chemin `<nom_application>/templates/<nom_application>/<nom_fichier>.html`
   
   Exemple : `listings/templates/listings/hello.html`

2. Créer une vue qui utilise ce template :
   ```python
   from django.shortcuts import render
   from listings.models import Band
   
   def hello(request):
       bands = Band.objects.all()
       return render(request, 'listings/hello.html')
   ```

**Passer des données au template :**

Pour un seul objet :
```python
return render(request, 'listings/hello.html', {'first_band': bands[0]})
```

Dans le template :
```html
<p>{{ first_band.name }}</p>
```

Pour plusieurs objets :
```python
return render(request, 'listings/hello.html', {'bands': bands})
```

Dans le template :
```html
<!-- Méthode 1 : Accès direct -->
<p>{{ bands.0.name }}</p>
<p>{{ bands.1.name }}</p>

<!-- Méthode 2 : Boucle (préférable) -->
<ul>
    {% for band in bands %}
    <li>{{ band.name }}</li>
    {% endfor %}
</ul>
```

**Utiliser des filtres de template :**
```html
<li>{{ band.name|upper }}</li>  <!-- Majuscules -->
<li>{{ band.name|lower }}</li>  <!-- Minuscules -->
<p>J'ai {{ bands|length }} groupes préférés.</p>  <!-- Longueur -->
```

**Utiliser des conditions dans les templates :**
```html
<p>
    J'ai..
    {% if bands|length < 5 %}
        peu de
    {% elif bands|length < 10 %}
        quelques
    {% else %}
        beaucoup de
    {% endif %}
    groupes préférés.
</p>
```

**Template de base :**

Créer un fichier `<nom_application>/templates/<nom_application>/base.html` :
```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Nom_site</title>
</head>
<body>
    {% block content %}{% endblock %}
</body>
</html>
```

Utiliser ce template de base dans d'autres templates :
```html
{% extends 'listings/base.html' %}

{% block content %}
<h1>Hello Django !</h1>
<p>Mes groupes préférés sont :</p>
<ul>
    {% for band in bands %}
        <li>{{ band.name }}</li>
    {% endfor %}
</ul>
{% endblock %}
```

**Ajouter des fichiers CSS statiques :**

1. Créer un fichier CSS dans `<nom_application>/static/<nom_application>/styles.css`

2. Dans le template de base :
   ```html
   {% load static %}
   
   <head>
       <link rel="stylesheet" href="{% static 'listings/styles.css' %}" />
   </head>
   ```

### Opérations CRUD

Les opérations CRUD (Create, Read, Update, Delete) peuvent être effectuées avec le site d'administration de Django.

**Créer un superutilisateur :**
```bash
python manage.py createsuperuser
```

**Enregistrer les modèles dans l'administration :**

Dans le fichier `<nom_application>/admin.py` :
```python
from django.contrib import admin
from listings.models import Band

admin.site.register(Band)
```

Accéder au site d'administration : http://127.0.0.1:8000/admin/

**Améliorer l'affichage des objets dans l'administration :**

Dans le fichier `<nom_application>/models.py` :
```python
class Band(models.Model):
    name = models.fields.CharField(max_length=100)
    year_formed = models.fields.IntegerField(null=True)
    genre = models.fields.CharField(max_length=50)
    
    def __str__(self):
        return f'{self.name}'
```

Dans le fichier `<nom_application>/admin.py` :
```python
class BandAdmin(admin.ModelAdmin):
    list_display = ('name', 'year_formed', 'genre')

admin.site.register(Band, BandAdmin)
```
