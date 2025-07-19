¡Claro! A continuación te presento un archivo **`README.md`** completo que puedes usar como documentación principal del proyecto **Recetas de Cocina en Django**. Este archivo está estructurado en formato **Markdown** y es ideal para subirlo a un repositorio de GitHub o cualquier plataforma de control de versiones.

---

# 🧑‍🍳 Recetas de Cocina con Django

Este proyecto es una aplicación web desarrollada con **Django** que permite **crear, visualizar, actualizar y eliminar recetas de cocina**, incluyendo imágenes.  
La interfaz utiliza **Bootstrap** para un diseño moderno y responsivo, y permite subir imágenes para cada receta.

---

## 📌 Características Principales

- ✅ Formulario para agregar nuevas recetas con imagen.
- 🔍 Búsqueda de recetas por nombre.
- ✏️ Actualización de recetas, incluyendo cambio de imagen.
- 📋 Visualización de todas las recetas en una tabla.
- ❌ Eliminación de recetas.
- 🖼️ Soporte para archivos multimedia (`ImageField`) con configuración en `MEDIA_URL`.

---

## 🧰 Tecnologías Utilizadas

| Tecnología | Descripción |
|-----------|-------------|
| Python    | Lenguaje de programación principal |
| Django    | Framework web para Python |
| Bootstrap | Framework CSS para diseño responsive |
| Pillow    | Para procesamiento de imágenes |
| SQLite    | Base de datos por defecto de Django |
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

## 🛠️ Pasos para Instalar y Ejecutar el Proyecto

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

- **Windows:**
  ```bash
  venv\Scripts\activate
  ```
- **Linux/macOS:**
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

---

## 📄 Plantillas HTML

### `base.html`

Plantilla base con Bootstrap.

### `receta.html`

Formulario para agregar recetas y tabla con listado de recetas.

### `actualizar_receta.html`

Formulario para editar una receta existente.

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

Contiene las vistas para:

- Agregar recetas
- Buscar recetas
- Actualizar recetas
- Eliminar recetas

---

## 🗂️ Modelos

```python
class Receta(models.Model):
    user = models.ForeignKey(User, on_delete=models.SET_NULL, null=True, blank=True)
    nombre_receta = models.CharField(max_length=100)
    descripcion_receta = models.TextField()
    imagen_receta = models.ImageField(upload_to="recetas", null=True, blank=True)
    ver_contar_receta = models.PositiveIntegerField(default=1)

    def __str__(self):
        return self.nombre_receta
```

---

## 🚀 Ejecutar el Servidor

```bash
python manage.py runserver
```

Visita `http://127.0.0.1:8000/` en tu navegador.

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

¿Te gustaría que te lo entregue en un archivo comprimido `.zip` o como repositorio listo para GitHub?
