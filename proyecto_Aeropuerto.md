Aquí tienes el archivo `models.py` con **todos los nombres de las clases y campos traducidos al español**, incluyendo las relaciones entre ellos. Se ha mantenido la funcionalidad intacta, pero ahora los nombres son más comprensibles para hablantes hispanohablantes.

---

### ✅ `models.py` en español

```python
from django.db import models


class Aeropuerto(models.Model):
    codigo = models.CharField(max_length=3, verbose_name="Código")
    ciudad = models.CharField(max_length=64, verbose_name="Ciudad")

    class Meta:
        verbose_name = "Aeropuerto"
        verbose_name_plural = "Aeropuertos"

    def __str__(self):
        return f"{self.ciudad} ({self.codigo})"


class Vuelo(models.Model):
    origen = models.ForeignKey(Aeropuerto, on_delete=models.CASCADE, related_name="salidas", verbose_name="Origen")
    destino = models.ForeignKey(Aeropuerto, on_delete=models.CASCADE, related_name="llegadas", verbose_name="Destino")
    duracion = models.IntegerField(verbose_name="Duración (minutos)")

    class Meta:
        verbose_name = "Vuelo"
        verbose_name_plural = "Vuelos"

    def __str__(self):
        return f"{self.id}: {self.origen} a {self.destino}"


class Pasajero(models.Model):
    nombre = models.CharField(max_length=64, verbose_name="Nombre")
    apellido = models.CharField(max_length=64, verbose_name="Apellido")
    vuelos = models.ManyToManyField(Vuelo, blank=True, related_name="pasajeros", verbose_name="Vuelos")

    class Meta:
        verbose_name = "Pasajero"
        verbose_name_plural = "Pasajeros"

    def __str__(self):
        return f"{self.nombre} {self.apellido}"
```

---

### 📌 Detalles de la traducción:

| Original (inglés)   | Traducido (español)   |
|---------------------|------------------------|
| Airport             | Aeropuerto             |
| code                | codigo                 |
| city                | ciudad                 |
| Flight              | Vuelo                  |
| origin              | origen                 |
| destination         | destino                |
| duration            | duracion               |
| Passenger           | Pasajero               |
| first (name)        | nombre                 |
| last (name)         | apellido               |
| flights             | vuelos                 |
| related_name: departures → salidas, arrivals → llegadas, passengers → pasajeros |

---

### ✅ ¿Qué más puedes hacer?

Si planeas usar esto en el panel de administración de Django, esta traducción ya se mostrará correctamente gracias a los `verbose_name` y a la configuración de `Meta`.

Si además quieres **soportar múltiples idiomas** en tu aplicación (i18n), puedo ayudarte a configurar Django para que cambie automáticamente el idioma de la interfaz según el usuario.

¿Te gustaría agregar esa funcionalidad también?
Perfecto, Eliseo. Vamos a crear un **proyecto Django funcional y moderno** para un sistema de **aerolíneas** con Bootstrap y CRUD para los modelos: `Aeropuerto`, `Vuelo` y `Pasajero`. A continuación te doy la estructura completa y el contenido de los archivos necesarios.

---

## 🧱 Estructura de carpetas

```
aerolinea_django/
│
├── aerolinea_django/          ← Configuración principal
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
│
├── gestion_vuelos/            ← Aplicación principal
│   ├── migrations/
│   ├── templates/
│   │   └── gestion_vuelos/
│   │       ├── base.html
│   │       ├── aeropuerto/
│   │       │   ├── list.html
│   │       │   ├── form.html
│   │       │   └── confirm_delete.html
│   │       ├── vuelo/
│   │       ├── pasajero/
│   ├── static/
│   │   └── gestion_vuelos/
│   │       └── css/
│   │           └── styles.css
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── urls.py
│   └── views.py
│
├── manage.py
└── db.sqlite3
```

---

## 🔧 1. `models.py`

Tu archivo ya está perfecto. Lo mantendremos igual.

---

## 🌐 2. `urls.py`

### Proyecto: `aerolinea_django/urls.py`

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('gestion_vuelos.urls')),  # Incluye rutas de la app
]
```

### App: `gestion_vuelos/urls.py`

```python
from django.urls import path
from . import views

urlpatterns = [
    # Aeropuertos
    path('aeropuertos/', views.AeropuertoList.as_view(), name='aeropuerto_list'),
    path('aeropuertos/crear/', views.AeropuertoCreate.as_view(), name='aeropuerto_create'),
    path('aeropuertos/editar/<int:pk>/', views.AeropuertoUpdate.as_view(), name='aeropuerto_update'),
    path('aeropuertos/eliminar/<int:pk>/', views.AeropuertoDelete.as_view(), name='aeropuerto_delete'),

    # Vuelos
    path('vuelos/', views.VueloList.as_view(), name='vuelo_list'),
    path('vuelos/crear/', views.VueloCreate.as_view(), name='vuelo_create'),
    path('vuelos/editar/<int:pk>/', views.VueloUpdate.as_view(), name='vuelo_update'),
    path('vuelos/eliminar/<int:pk>/', views.VueloDelete.as_view(), name='vuelo_delete'),

    # Pasajeros
    path('pasajeros/', views.PasajeroList.as_view(), name='pasajero_list'),
    path('pasajeros/crear/', views.PasajeroCreate.as_view(), name='pasajero_create'),
    path('pasajeros/editar/<int:pk>/', views.PasajeroUpdate.as_view(), name='pasajero_update'),
    path('pasajeros/eliminar/<int:pk>/', views.PasajeroDelete.as_view(), name='pasajero_delete'),
]
```

---

## 👁️ 3. `views.py` con CRUD

```python
from django.views.generic import ListView, CreateView, UpdateView, DeleteView
from django.urls import reverse_lazy
from .models import Aeropuerto, Vuelo, Pasajero

# Aeropuertos
class AeropuertoList(ListView):
    model = Aeropuerto
    template_name = 'gestion_vuelos/aeropuerto/list.html'

class AeropuertoCreate(CreateView):
    model = Aeropuerto
    fields = '__all__'
    success_url = reverse_lazy('aeropuerto_list')
    template_name = 'gestion_vuelos/aeropuerto/form.html'

class AeropuertoUpdate(UpdateView):
    model = Aeropuerto
    fields = '__all__'
    success_url = reverse_lazy('aeropuerto_list')
    template_name = 'gestion_vuelos/aeropuerto/form.html'

class AeropuertoDelete(DeleteView):
    model = Aeropuerto
    success_url = reverse_lazy('aeropuerto_list')
    template_name = 'gestion_vuelos/aeropuerto/confirm_delete.html'

# Vuelos
class VueloList(ListView):
    model = Vuelo
    template_name = 'gestion_vuelos/vuelo/list.html'

class VueloCreate(CreateView):
    model = Vuelo
    fields = '__all__'
    success_url = reverse_lazy('vuelo_list')
    template_name = 'gestion_vuelos/vuelo/form.html'

class VueloUpdate(UpdateView):
    model = Vuelo
    fields = '__all__'
    success_url = reverse_lazy('vuelo_list')
    template_name = 'gestion_vuelos/vuelo/form.html'

class VueloDelete(DeleteView):
    model = Vuelo
    success_url = reverse_lazy('vuelo_list')
    template_name = 'gestion_vuelos/vuelo/confirm_delete.html'

# Pasajeros
class PasajeroList(ListView):
    model = Pasajero
    template_name = 'gestion_vuelos/pasajero/list.html'

class PasajeroCreate(CreateView):
    model = Pasajero
    fields = '__all__'
    success_url = reverse_lazy('pasajero_list')
    template_name = 'gestion_vuelos/pasajero/form.html'

class PasajeroUpdate(UpdateView):
    model = Pasajero
    fields = '__all__'
    success_url = reverse_lazy('pasajero_list')
    template_name = 'gestion_vuelos/pasajero/form.html'

class PasajeroDelete(DeleteView):
    model = Pasajero
    success_url = reverse_lazy('pasajero_list')
    template_name = 'gestion_vuelos/pasajero/confirm_delete.html'
```

---

## 🎨 4. `templates`

### `base.html`

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Aerolínea</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css">
</head>
<body>
    <nav class="navbar navbar-expand-lg navbar-dark bg-primary">
        <div class="container-fluid">
            <a class="navbar-brand" href="#">Aerolínea</a>
            <div class="navbar-nav">
                <a class="nav-link" href="{% url 'aeropuerto_list' %}">Aeropuertos</a>
                <a class="nav-link" href="{% url 'vuelo_list' %}">Vuelos</a>
                <a class="nav-link" href="{% url 'pasajero_list' %}">Pasajeros</a>
            </div>
        </div>
    </nav>
    <div class="container mt-4">
        {% block content %}
        {% endblock %}
    </div>
</body>
</html>
```

### `aeropuerto/list.html`

```html
{% extends 'gestion_vuelos/base.html' %}
{% block content %}
<h2>Listado de Aeropuertos</h2>
<a class="btn btn-success mb-2" href="{% url 'aeropuerto_create' %}">Agregar</a>
<table class="table table-hover table-bordered">
    <thead class="table-dark">
        <tr>
            <th>Código</th>
            <th>Ciudad</th>
            <th>Acciones</th>
        </tr>
    </thead>
    <tbody>
        {% for aeropuerto in object_list %}
        <tr>
            <td>{{ aeropuerto.codigo }}</td>
            <td>{{ aeropuerto.ciudad }}</td>
            <td>
                <a class="btn btn-warning btn-sm" href="{% url 'aeropuerto_update' aeropuerto.id %}">Editar</a>
                <a class="btn btn-danger btn-sm" href="{% url 'aeropuerto_delete' aeropuerto.id %}">Eliminar</a>
            </td>
        </tr>
        {% endfor %}
    </tbody>
</table>
{% endblock %}
```

### Reutilizable: `form.html`

```html
{% extends 'gestion_vuelos/base.html' %}
{% block content %}
<h2>{{ view.__class__.__name__|cut:"View" }}</h2>
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit" class="btn btn-primary">Guardar</button>
    <a href="javascript:history.back()" class="btn btn-secondary">Cancelar</a>
</form>
{% endblock %}
```

### Reutilizable: `confirm_delete.html`

```html
{% extends 'gestion_vuelos/base.html' %}
{% block content %}
<h2>¿Confirmar eliminación?</h2>
<p>¿Estás seguro de que deseas eliminar: <strong>{{ object }}</strong>?</p>
<form method="post">
    {% csrf_token %}
    <button type="submit" class="btn btn-danger">Eliminar</button>
    <a href="javascript:history.back()" class="btn btn-secondary">Cancelar</a>
</form>
{% endblock %}
```

Crea las carpetas `vuelo/` y `pasajero/` con los mismos archivos que en `aeropuerto/`.

---

¿Quieres que te lo prepare como proyecto `.zip` listo para descargar o subirlo a GitHub?
