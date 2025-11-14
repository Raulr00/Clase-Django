# Clase-Django
**Django (Python) — Framework web de alto nivel**

Este repositorio acompaña a la clase práctica para crear una **aplicación web funcional con Django**.


---

## Índice
1. [Requisitos](#requisitos)  
2. [Entorno virtual (recomendado)](#entorno-virtual-recomendado)  
3. [Inicialización del proyecto](#inicialización-del-proyecto)  
4. [Estructura básica (archivos clave)](#estructura-básica-archivos-clave)  
5. [Añadir una app](#añadir-una-app)  
6. [Vistas y URLs](#vistas-y-urls)  
7. [Modelos y base de datos](#modelos-y-base-de-datos)  
8. [Plantillas (Templates)](#plantillas-templates)  
9. [Archivos estáticos (CSS/JS/imagenes)](#archivos-estáticos-cssjsimagenes)  
10. [Panel de administración](#panel-de-administración)  
11. [Ejecutar el servidor de desarrollo](#ejecutar-el-servidor-de-desarrollo)  
12. [Patrón MVT en contexto](#patrón-mvt-en-contexto)  
13. [Problemas comunes y soluciones](#problemas-comunes-y-soluciones)  
14. [Checklist de evaluación / Rúbrica](#checklist-de-evaluación--rúbrica)  
15. [Créditos y licencia](#créditos-y-licencia)

---

## Requisitos
- **Python 3.10+** (recomendado 3.11/3.12). Comprueba con:
  ```bash
  python --version
  # o
  python3 --version
  ```
- **Pip** (gestor de paquetes de Python).
- **Django** (se instalará en el entorno virtual).

---

## Entorno virtual (recomendado)
Un **entorno virtual** aísla las dependencias por proyecto.

**Windows (PowerShell)**
```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install django
```

**macOS / Linux (bash/zsh)**
```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install django
```

Para **desactivar** el entorno: `deactivate`

---

## Inicialización del proyecto


```bash
# dentro del entorno virtual
django-admin startproject clase_django .
# El punto final crea el proyecto en la carpeta actual
```

Crea la base de datos SQLite inicial y aplica migraciones:
```bash
python manage.py migrate
```

Crea un superusuario (para el admin):
```bash
python manage.py createsuperuser
# sigue las instrucciones en consola (usuario, email opcional, contraseña)
```

---

## Estructura básica (archivos clave)
Después de `startproject`, tendrás algo así:

```
manage.py                # Punto de entrada a comandos del proyecto
clase_django/
  __init__.py            # Indica que el directorio es un paquete
  settings.py            # Configuración del proyecto (DB, apps, templates, static, auth, etc.)
  urls.py                # Enrutador principal: asigna URLs a vistas
  asgi.py                # Configuración para servidores ASGI (Uvicorn/Daphne)
  wsgi.py                # Configuración para servidores WSGI (Gunicorn/uWSGI)
```

- **manage.py**: ejecutar servidores, migraciones, crear apps, etc.  
- **settings.py**: corazón de la configuración del proyecto.  
- **urls.py**: “mapa de rutas” del proyecto.  
- **asgi.py / wsgi.py**: puntos de entrada para desplegar en producción.

---

## Añadir una app
Crea tu primera app (p. ej., `encuestas`):
```bash
python manage.py startapp encuestas
```

Añádela a `INSTALLED_APPS` en `clase_django/settings.py`:
```python
INSTALLED_APPS = [
    # ...
    'encuestas',
]
```

Estructura mínima de una app:
```
encuestas/
  __init__.py
  admin.py
  apps.py
  migrations/
    __init__.py
  models.py
  tests.py
  views.py
  urls.py
```

Crea `encuestas/urls.py`:
```python
from django.urls import path
from . import views

urlpatterns = [
    path('hello/', views.hello, name='hello'),
]
```

Incluye las URLs de la app en `clase_django/urls.py`:
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('blog.urls')),
]
```

---

## Vistas y URLs
Ejemplo de **vista** simple en `blog/views.py`:
```python
from django.http import HttpResponse

def hello(request):
    return HttpResponse("¡Hola Django!")
```

Ve a `http://127.0.0.1:8000/hello/` y deberías ver el mensaje.

---

## Modelos y base de datos
Define un modelo en `blog/models.py`:
```python
from django.db import models

class Post(models.Model):
    title = models.CharField(max_length=200)
    body = models.TextField(blank=True)
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.title
```

Crea y aplica migraciones:
```bash
python manage.py makemigrations
python manage.py migrate
```

Registra el modelo en el admin (`blog/admin.py`):
```python
from django.contrib import admin
from .models import Post

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_display = ('id', 'title', 'created_at')
    search_fields = ('title',)
```
Ahora entra en `http://127.0.0.1:8000/admin/` con el superusuario.

---

## Plantillas (Templates)
Crea la carpeta de plantillas y configura `settings.py`:

**Estructura recomendada**
```
templates/
  base.html
  blog/
    index.html
```

```python
import os
from pathlib import Path
BASE_DIR = Path(__file__).resolve().parent.parent

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'templates'],  # carpeta global de templates
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
```

**`templates/index.html`** :
```html
<!doctype html>
<html lang="es">
  <head>
    <meta charset="utf-8">
    <title>{% block title %}Clase Django{% endblock %}</title>
  </head>
  <body>
    <header>
      <h1>Django — Clase práctica</h1>
      <nav><a href="/">Inicio</a> · <a href="/hello/">Hello</a></nav>
    </header>
    <main>
      {% block content %}{% endblock %}
    </main>
  </body>
</html>
```

**`templates/encuestas/index.html`**:
```html
{% extends "base.html" %}
{% block title %}Listado de posts{% endblock %}
{% block content %}
  <h2>Posts</h2>
  <ul>
    {% for post in posts %}
      <li><strong>{{ post.title }}</strong> — {{ post.created_at }}</li>
    {% empty %}
      <li>No hay posts todavía.</li>
    {% endfor %}
  </ul>
{% endblock %}
```

**Vista que usa la plantilla** (`encuestas/views.py`):
```python
from django.shortcuts import render
from .models import Post

def index(request):
    posts = Post.objects.order_by('-created_at')
    return render(request, "encuestas/index.html", {"posts": posts})
```

**Ruta para la vista index** (`encuestas/urls.py`):
```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='home'),
    path('hello/', views.hello, name='hello'),
]
```

## Panel de administración
- URL por defecto: `http://127.0.0.1:8000/admin/`
- Usuario/contraseña: los creados con `createsuperuser`.
- Registra tus modelos en `admin.py` para gestionarlos visualmente.

---
