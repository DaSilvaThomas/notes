# Notes Environnements Virtuels

## Table des matières
1. [Environnements virtuels](#environnements-virtuels)
   - [Création et activation](#création-et-activation)
   - [Gestion des environnements](#gestion-des-environnements)

---

## Environnements virtuels

### Création et activation

**Créer un environnement virtuel :**
```bash
python -m venv <environment_name>

# Exemple
python -m venv env
```
Cela crée un répertoire "env" à l'emplacement où la commande a été entrée.

**Activer l'environnement virtuel :**

Sur Windows :
```bash
<environment_name>\Scripts\activate.bat

# Exemple
env\Scripts\activate.bat
```

Sur Linux :
```bash
source <environment_name>/bin/activate

# Exemple
source env/bin/activate
```

Attention : il faut exécuter cette commande à la racine du répertoire de l'environnement. 

L'indication `(env)` au début de la ligne de commande montre que vous êtes dans l'environnement virtuel :
```
(env) D:\DEV\Python\demo-app>
```

### Gestion des environnements

**Désactiver l'environnement virtuel :**
```bash
deactivate
```

**Supprimer un environnement virtuel :**

Sur Windows :
```bash
rmdir /S /Q env
```
- /S : Supprime tous les fichiers et sous-dossiers dans le dossier
- /Q : Supprime le dossier sans demander de confirmation

Sur Linux :
```bash
rm -r env/
```
