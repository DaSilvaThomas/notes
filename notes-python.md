# Notes Python

## Table des matières
1. [Les bases de Python](#les-bases-de-python)
   - [Boucles](#boucles)
   - [Conditions](#conditions)
2. [Gestion des exceptions](#gestion-des-exceptions)
   - [Structure try/except](#structure-tryexcept)
   - [Exemples d'utilisation](#exemples-dutilisation)
3. [Gestion des paquets Python](#gestion-des-paquets-python)
   - [Différence entre logiciel et paquet](#différence-entre-logiciel-et-paquet)
   - [Commandes pip essentielles](#commandes-pip-essentielles)
   - [Création d'un paquet Python](#création-dun-paquet-python)
4. [Pytest: Tests unitaires](#pytest-tests-unitaires)
   - [Installation](#installation)
   - [Organisation des tests](#organisation-des-tests)
   - [Exemples de tests](#exemples-de-tests)

---

## Les bases de Python

### Boucles

```python
# Boucle for avec une liste
for mot in liste:
    # Do something

# Boucle for avec range
for i in range(1, 11, 2):
    # Do something

# Boucle while
x = 0
while x < 10:
    # Do something
    x += 1  # Note: x++ n'est pas valide en Python, utilisez x += 1
```

### Conditions

```python
# Structure if-elif-else
if condition:
    # Do something
elif autre_condition:
    # Do something
else:
    # do something
    
# Vérifier si un élément est dans une liste
if "element" in liste:
    # do something    

if "element" not in liste:
    # do something

# Structure match-case (Python 3.10+)
match symbole:
    case "+":
        # Do something
    case "-":
        # Do something
    case "*":
        # Do something
    case "/":
        # Do something
    case _:
        # Do something (cas par défaut)
```

---

## Gestion des exceptions

En Python, les exceptions sont utilisées pour gérer les erreurs qui peuvent se produire lors de l'exécution d'un programme. L'utilisation de try et except permet de gérer ces erreurs de manière élégante sans que le programme ne plante.

### Structure try/except

```python
try:
    # Code qui pourrait provoquer une erreur
except <Nom_Erreur>:
    # Code exécuté en cas d'erreur
else:
    # Code exécuté si aucune exception n'a été levée dans le bloc try
finally:
    # Code exécuté quoi qu'il arrive
```

### Exemples d'utilisation

**Exemple simple**
```python
try:
    x = 10 / 0
except ZeroDivisionError:
    print("Erreur : Division par zéro !")
```

**Exemple avec une gestion globale des erreurs**
```python
try:
    x = 10 / 0
except Exception as e:
    print(f"Une erreur est survenue : {e}")   # Affichage de l'erreur
```

**Exemple complet**
```python
try:
    nombre = int(input("Entrez un nombre : "))   # Demander à l'utilisateur un nombre
    result = 10 / nombre
except ValueError:
    print("Erreur : Ce n'est pas un nombre valide !")
except ZeroDivisionError:
    print("Erreur : Division par zéro !")
else:
    print(f"Le résultat est : {result}")
finally:
    print("Merci d'avoir utilisé le programme.")
```

---

## Gestion des paquets Python

### Différence entre logiciel et paquet

Un logiciel (exécutable) est différent d'un paquet Python. Par exemple, Python est un logiciel exécutable qu'on peut utiliser en tapant son nom dans la ligne de commande.

Pour vérifier si un logiciel est installé :
```bash
<nom_logiciel> --version

# Exemple
pip --version
```

Un paquet Python est une bibliothèque ou un module. Ces paquets s'installent dans l'environnement Python (généralement via pip) et permettent d'étendre les fonctionnalités de base de Python. L'ensemble des paquets Python sont consultables sur le site : https://pypi.org/

### Commandes pip essentielles

```bash
# Savoir si un paquet est installé
pip show <nom_paquet>
    
# Lister tous les paquets installés
pip list
pip freeze

# Installer un paquet
pip install <nom_paquet>

# Installer des paquets contenus dans un fichier "requirements.txt"
pip install -r requirements.txt

# Stocker les paquets dans un fichier "requirements.txt"
pip freeze > requirements.txt

# Désinstaller un paquet
pip uninstall <nom_paquet>
```

### Création d'un paquet Python

Un dossier contenant un fichier `__init__.py` est considéré comme un package Python. Cela permet à Python de gérer ce dossier comme un module. Le fichier `__init__.py` peut être vide ou contenir du code pour initialiser le package.

**Structure d'un paquet Python :**
```
mon_package/
├── __init__.py   # Fichier de package
├── module1.py    # Un module dans le package
└── module2.py    # Un autre module dans le package
```

**Importation avec un fichier __init__.py vide :**
```python
# Méthode 1
import mon_package.module1   # Importe le module1 du package mon_package 
mon_package.module1.fonction1()   # Pour appeler une fonction ou accéder à une variable dans module1

# Méthode 2
from mon_package import module2   # Importe le module2 du package mon_package
module2.fonction2()   # Pour appeler une fonction ou accéder à une variable dans module2
```

**Importation avec un fichier __init__.py contenant du code :**

Fichier `__init__.py` :
```python
from .module1 import fonction1   # fonction1 sera directement accessible lorsque le paquet sera importé
from .module2 import fonction2   # idem pour fonction2
print("Le package a été initialisé.")   # Message lorsque le paquet est importé
```

Fichier dans lequel on souhaite importer le paquet :
```python
import mon_package   # Ceci déclenche l'exécution du code dans __init__.py
mon_package.fonction1()   # Utilisation de la fonction1 du module1
mon_package.fonction2()   # Utilisation de la fonction2 du module2
```

---

## Pytest: Tests unitaires

Pytest est un framework Python qui permet de créer et d'exécuter toute sorte de tests.

### Installation

```bash
pip install -U pytest
```

### Organisation des tests

1. Créer un dossier `/tests` à la racine du projet
2. Créer un fichier `__init__.py` dans le dossier `/tests`
3. Créer une arborescence de tests calquée sur le répertoire des fichiers sources
4. Nommer les fichiers de tests avec le même nom que le fichier source précédé de "test_"
   Note : Pytest va lancer les tests de tous les fichiers qui commencent par "test_" ou qui finissent par "_test"
5. Exécuter les tests avec la commande `pytest` à la racine du projet

### Exemples de tests

Fichier source `code_source.py` :
```python
def reverse_str(initial_string):
    final_string = ''
    index = len(initial_string)
    while index > 0:
        final_string += initial_string[index - 1]
        index = index - 1
    return final_string
```

Fichier test `test_code_source.py` :
```python
from source import reverse_str

def test_should_reverse_string():
    assert reverse_str('abc') == 'cba'
```
