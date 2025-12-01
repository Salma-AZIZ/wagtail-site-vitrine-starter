# 🐍 **Guide de déploiement Wagtail sur PythonAnywhere**

## ⚠️ Informations à personnaliser

Remplace dans toutes les commandes :

* **TON_COMPTE** → ton nom de compte GitHub
* **TON_REPO** → nom du dépôt GitHub
* **TON_PROJET** → nom du projet Django/Wagtail (dossier contenant `settings/`)
* **USERNAME** → nom de ton compte PythonAnywhere


## 🔧 **0. Préparer le projet pour le déploiement**

### 1) Projet sur GitHub

Ton projet Wagtail doit être sur GitHub, avec un `.gitignore` qui exclut :

```
venv/
__pycache__/
db.sqlite3
media/
```

### 2) requirements.txt

Exemple recommandé :

```
Django>=5.2,<5.3
wagtail>=7.1,<7.2
gunicorn
whitenoise
psycopg[binary]
Pillow
```



# 1. **Créer un compte PythonAnywhere**

1. Aller sur : [https://www.pythonanywhere.com/registration/register/beginner/](https://www.pythonanywhere.com/registration/register/beginner/)
2. Choisir un `USERNAME`
3. Confirmer votre email

➡️ Votre site sera visible ici :
`https://USERNAME.pythonanywhere.com/`



# 2. **Créer une Web App**

1. Menu **Web**
2. **Add a new web app**
3. Domaine proposé : `USERNAME.pythonanywhere.com`
4. Choisir :

   * **Manual configuration**
   * **Python 3.13** (ou version récente)



# 3. **Cloner votre projet GitHub**

Menu **Files** → **Open Bash console**

```bash
git clone https://github.com/TON_COMPTE/TON_REPO.git
```

Le projet sera dans :
`/home/USERNAME/TON_REPO/`



# 4. **Créer un environnement virtuel + installer les dépendances**

```bash
cd /home/USERNAME
python3 -m venv venv
source venv/bin/activate
pip install -r TON_REPO/requirements.txt
```



# 5. **Configurer les settings de production**

Dans `TON_PROJET/settings/`, vous devez avoir :

* `base.py`
* `dev.py`
* `production.py`

### a) Copier la SECRET_KEY

Dans `dev.py` ou `base.py` :

```python
SECRET_KEY = "votre_cle_secrete"
```

### b) Modifier `production.py`

Exemple recommandé :

```python
from .base import *

DEBUG = False

SECRET_KEY = "COLLER_ICI_LA_SECRET_KEY"

ALLOWED_HOSTS = [
    "USERNAME.pythonanywhere.com",
    "localhost",
    "127.0.0.1",
]

try:
    from .local import *
except ImportError:
    pass
```

➡️ Remplacer `USERNAME` par votre vrai compte.



# 6. **Créer la base de données en production**

```bash
cd /home/USERNAME/TON_REPO/
source ../venv/bin/activate
python manage.py migrate
```


# 7. **Créer le superuser**

```bash
python manage.py createsuperuser
```

Si Django affiche :

> Bypass password validation and create user anyway? (y/N)

Répondez **y**.



# 8. **Collecter les fichiers statiques**

```bash
python manage.py collectstatic
```

Répondre **yes**.



# 9. **Configurer la Web App (menu Web)**

### A) Associer le virtualenv

Web → **Virtualenv** →

```
/home/USERNAME/venv
```

### B) Modifier le fichier WSGI

Menu Web → **WSGI configuration file** → ouvrir → remplacer tout par :

```python
import os
import sys

project_path = "/home/USERNAME/TON_REPO"
if project_path not in sys.path:
    sys.path.append(project_path)

os.environ.setdefault(
    "DJANGO_SETTINGS_MODULE",
    "TON_PROJET.settings.production"
)

from django.core.wsgi import get_wsgi_application
application = get_wsgi_application()
```

➡️ Remplacer :

* USERNAME
* TON_REPO
* TON_PROJET



# 10. **Configurer Static & Media**

Dans `base.py` :

```python
STATIC_URL = "/static/"
STATIC_ROOT = os.path.join(BASE_DIR, "static")

MEDIA_URL = "/media/"
MEDIA_ROOT = os.path.join(BASE_DIR, "media")
```

### Dans Web → Static files :

#### ✔ Fichiers statiques

* URL : `/static/`
* Directory : `/home/USERNAME/TON_REPO/static/`

#### ✔ Fichiers media

* URL : `/media/`
* Directory : `/home/USERNAME/TON_REPO/media/`

Enregistrer.



# 11. **Redémarrer la Web App**

Bouton **Reload** en haut.



# 12. **Tester votre site**

### Site public

`https://USERNAME.pythonanywhere.com/`

### Admin

`https://USERNAME.pythonanywhere.com/admin/`



# 13. **Créer les pages en production (IMPORTANT)**

La base de production est **vide**.

Dans `/admin/` :

1. Pages → Home
2. Add child page
3. Choisir un type (Home, About, Contact, etc.)
4. Publier

### Ajouter les images

Admin → Images → **Add image**


# 🎉 **Votre site Wagtail est maintenant en production !**

