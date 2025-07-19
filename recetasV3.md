¡Perfecto! A continuación, te presento un **proyecto completo de Django** para una aplicación web llamada **"Recetas de Cocina"**. Este proyecto permite gestionar recetas con imágenes, incluyendo **CRUD completo**, **búsqueda**, y una interfaz moderna con Bootstrap. Todo esto sin usar `forms.py`, siguiendo las buenas prácticas de Django y usando solo lo necesario.

---

# 🧑‍💻 Proyecto: Recetas de Cocina con Django

## 📌 Descripción General

Este es un sistema web desarrollado con **Django** que permite a los usuarios crear, visualizar, actualizar y eliminar recetas de cocina. Cada receta puede incluir nombre, descripción e imagen. La aplicación utiliza **Bootstrap** para el diseño y tiene funcionalidad de búsqueda y edición de recetas.

---

## 🧰 Tecnologías Utilizadas

| Tecnología | Descripción |
|-----------|-------------|
| Python | Lenguaje de programación principal |
| Django | Framework web para Python |
| Bootstrap | Framework CSS para diseño responsive |
| Pillow | Para procesamiento de imágenes |
| SQLite | Base de datos por defecto de Django |
| HTML5 / CSS3 | Plantillas y estilos web |

---

## 🧱 Estructura del Proyecto

```
Pj_recetas/
├── manage.py
├── media/
├── static/
│   └── css/
│       └── style.css
├── app_receta/
│   ├── migrations/
│   ├── templates/
│   │   ├── base.html
│   │   ├── receta.html
│   │   └── actualizar_receta.html
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── tests.py
│   └── views.py
├── Pj_recetas/
│   ├── __init__.py
│   ├── asgi.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── requirements.txt
```

---

## 🛠️ Requisitos Previos

Asegúrate de tener instalado:

- Python 3.x
- pip
- Un editor de código (como VSCode)

---

## 📦 Pasos para Instalar y Ejecutar el Proyecto

### 1. Verificar instalación de Python y pip

```bash
python --version
pip --version
```

### 2. Crear entorno virtual

```bash
python -m venv venv
```

### 3. Activar el entorno virtual

- En Windows:
  ```bash
  venv\Scripts\activate
  ```
- En Linux/macOS:
  ```bash
  source venv/bin/activate
  ```

### 4. Seleccionar intérprete de Python (en VSCode)

1. Abre VSCode.
2. Presiona `Ctrl + Shift + P`.
3. Escribe `Python: Select Interpreter`.
4. Elige el que apunta a `venv`.

### 5. Instalar Django y Pillow

```bash
pip install django pillow
```

### 6. Crear proyecto Django

```bash
django-admin startproject Pj_recetas
```

### 7. Crear aplicación

```bash
cd Pj_recetas
python manage.py startapp app_receta
```

### 8. Agregar `app_receta` al `INSTALLED_APPS` en `settings.py`

```python
INSTALLED_APPS = [
    ...
    'app_receta',
]
```

### 9. Configurar `MEDIA_URL` y `STATIC_URL` en `settings.py`

```python
import os

# Media files
MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')

# Static files
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'static')
```

### 10. Configurar en `urls.py` del proyecto

```python
from django.conf import settings
from django.conf.urls.static import static
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('app_receta.urls')),
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

### 11. Crear carpetas necesarias

Dentro de la carpeta del proyecto (`Pj_recetas`):

- `media/` (para imágenes)
- `static/css/` (para estilos)
- `templates/` dentro de `app_receta`

### 12. Crear modelos y migraciones

#### `models.py`

```python
from django.db import models
from django.contrib.auth.models import User

class Receta(models.Model):
    user = models.ForeignKey(User, on_delete=models.SET_NULL, null=True, blank=True)
    nombre_receta = models.CharField(max_length=100)
    descripcion_receta = models.TextField()
    imagen_receta = models.ImageField(upload_to="recetas", null=True, blank=True)
    ver_contar_receta = models.PositiveIntegerField(default=1)

    def __str__(self):
        return self.nombre_receta
```

Ejecutar migraciones:

```bash
python manage.py makemigrations
python manage.py migrate
```

### 13. Crear superusuario

```bash
python manage.py createsuperuser
```

### 14. Registrar modelo en `admin.py`

```python
from django.contrib import admin
from .models import Receta

admin.site.register(Receta)
```

---

## 🌐 Configuración de URLs

#### `urls.py` (en `app_receta`)

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.recetas, name='recetas'),
    path('borrar_receta/<int:id>', views.borrar_receta, name='borrar_receta'),
    path('actualizar_receta/<int:id>', views.actualizar_receta, name='actualizar_receta'),
]
```

---

## 📄 Plantillas HTML

### 1. `base.html`

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Recetas de Cocina</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="/static/css/style.css">
</head>
<body>
    <div class="container mt-4">
        {% block content %}{% endblock %}
    </div>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

---

### 2. `receta.html`

```html
{% extends "base.html" %}
{% block content %}
<h2>Agregar Receta</h2>

<form method="post" enctype="multipart/form-data">
    {% csrf_token %}
    <div class="mb-3">
        <label for="nombre" class="form-label">Nombre de la Receta</label>
        <input type="text" name="nombre_receta" class="form-control" required>
    </div>
    <div class="mb-3">
        <label for="descripcion" class="form-label">Descripción</label>
        <textarea name="descripcion_receta" class="form-control" required></textarea>
    </div>
    <div class="mb-3">
        <label for="imagen" class="form-label">Imagen</label>
        <input type="file" name="imagen_receta" class="form-control">
    </div>
    <button type="submit" class="btn btn-primary">Guardar</button>
</form>

<hr>
<h3>Buscar Receta</h3>
<form method="get" action="">
    <input type="text" name="q" placeholder="Buscar por nombre..." class="form-control mb-3">
</form>

<h3>Todas las Recetas</h3>
<table class="table table-striped">
    <thead>
        <tr>
            <th>No.</th>
            <th>Nombre</th>
            <th>Descripción</th>
            <th>Imagen</th>
            <th>Acciones</th>
        </tr>
    </thead>
    <tbody>
        {% for r in recetas %}
        <tr>
            <td>{{ forloop.counter }}</td>
            <td>{{ r.nombre_receta }}</td>
            <td>{{ r.descripcion_receta|truncatewords:10 }}</td>
            <td><img src="{{ r.imagen_receta.url }}" width="100" alt=""></td>
            <td>
                <a href="{% url 'actualizar_receta' r.id %}" class="btn btn-warning btn-sm">Actualizar</a>
                <a href="{% url 'borrar_receta' r.id %}" class="btn btn-danger btn-sm">Borrar</a>
            </td>
        </tr>
        {% endfor %}
    </tbody>
</table>
{% endblock %}
```

---

### 3. `actualizar_receta.html`

```html
{% extends "base.html" %}
{% block content %}
<h2>Actualizar Receta</h2>
<form method="post" enctype="multipart/form-data">
    {% csrf_token %}
    <div class="mb-3">
        <label for="nombre" class="form-label">Nombre</label>
        <input type="text" name="nombre_receta" value="{{ receta.nombre_receta }}" class="form-control" required>
    </div>
    <div class="mb-3">
        <label for="descripcion" class="form-label">Descripción</label>
        <textarea name="descripcion_receta" class="form-control">{{ receta.descripcion_receta }}</textarea>
    </div>
    <div class="mb-3">
        <label for="imagen" class="form-label">Imagen (opcional)</label>
        <input type="file" name="imagen_receta" class="form-control">
    </div>
    <button type="submit" class="btn btn-success">Actualizar</button>
</form>
<a href="{% url 'recetas' %}" class="btn btn-secondary mt-3">Volver</a>
{% endblock %}
```

---

## 🎨 Estilos CSS (`style.css`)

```css
body {
    background-color: #f8f9fa;
}
img {
    max-width: 100px;
    border-radius: 5px;
}
```

---

## 🧠 Lógica del Proyecto en `views.py`

```python
from django.shortcuts import render, redirect, get_object_or_404
from .models import Receta
from django.db.models import Q

def recetas(request):
    if request.method == 'POST':
        nombre = request.POST['nombre_receta']
        descripcion = request.POST['descripcion_receta']
        imagen = request.FILES.get('imagen_receta')
        Receta.objects.create(
            nombre_receta=nombre,
            descripcion_receta=descripcion,
            imagen_receta=imagen
        )
        return redirect('recetas')

    q = request.GET.get('q', '')
    recetas = Receta.objects.all()
    if q:
        recetas = recetas.filter(nombre_receta__icontains=q)

    return render(request, 'receta.html', {'recetas': recetas, 'query': q})

def borrar_receta(request, id):
    receta = get_object_or_404(Receta, id=id)
    receta.delete()
    return redirect('recetas')

def actualizar_receta(request, id):
    receta = get_object_or_404(Receta, id=id)
    if request.method == 'POST':
        receta.nombre_receta = request.POST['nombre_receta']
        receta.descripcion_receta = request.POST['descripcion_receta']
        imagen = request.FILES.get('imagen_receta')
        if imagen:
            receta.imagen_receta = imagen
        receta.save()
        return redirect('recetas')
    return render(request, 'actualizar_receta.html', {'receta': receta})
```

---

## 📦 Generar archivo `requirements.txt`

```bash
pip freeze > requirements.txt
```

Contenido típico:

```
Django==4.2
Pillow==10.0.0
```

---

## 🚀 Ejecutar el servidor

```bash
python manage.py runserver
```

---

## 📌 Resumen del Proyecto

Este proyecto es una aplicación web sencilla pero funcional que permite gestionar recetas de cocina con imágenes. Tiene todas las funcionalidades CRUD, búsqueda de recetas, interfaz moderna con Bootstrap y soporte para imágenes.

---

## 🔑 Palabras Clave y Explicación

| Palabra Clave | Explicación |
|---------------|-------------|
| Django        | Framework web de alto nivel para Python |
| CRUD          | Crear, Leer, Actualizar, Eliminar datos |
| ImageField    | Campo de modelo para subir imágenes |
| Bootstrap     | Framework CSS para diseño responsivo |
| MEDIA_URL     | Configuración para servir archivos multimedia |
| Template      | Archivo HTML que Django puede renderizar |
| Static Files  | Archivos como CSS, JS, imágenes estáticas |
| Migraciones   | Mecanismo de Django para aplicar cambios a la base de datos |

---

## 📚 Referencias Oficiales

- [Django Documentation](https://docs.djangoproject.com/)
- [Bootstrap](https://getbootstrap.com/)
- [Pillow Documentation](https://pillow.readthedocs.io/en/stable/)

---

## 📄 Licencia

Este proyecto está desarrollado por **Eliseo Nava**.  
Licencia: **MIT License**  
Puedes usarlo, modificarlo y distribuirlo libremente.

---

¿Quieres que te genere el proyecto completo como archivo ZIP o estructura comprimida para descargar?
