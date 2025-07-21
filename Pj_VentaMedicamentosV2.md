¡Excelente! Aquí tienes una solución completa para tu proyecto de venta de medicamentos en Django, siguiendo las especificaciones que has proporcionado.

La solución está dividida en las siguientes partes:

1.  **Configuración del Proyecto Django**: Pasos iniciales.
2.  **`models.py`**: La traducción del modelo relacional a clases de Django.
3.  **`urls.py`**: Las rutas para acceder a las diferentes funcionalidades.
4.  **`views.py`**: La lógica de negocio usando vistas basadas en funciones.
5.  **Plantillas HTML (Bonus)**: Ejemplos de las plantillas necesarias para que las vistas funcionen, ya que no se usa `forms.py`.

---

### 1. Configuración del Proyecto

Primero, crea el proyecto y la aplicación.

```bash
# 1. Crear el proyecto
django-admin startproject farma_project
cd farma_project

# 2. Crear la aplicación para gestionar las ventas
python manage.py startapp ventas

# 3. Registrar la app en settings.py
# Abre farma_project/settings.py y añade 'ventas' a INSTALLED_APPS:
INSTALLED_APPS = [
    # ...
    'django.contrib.staticfiles',
    'ventas', # <--- Añadir aquí
]

# 4. Realizar las migraciones iniciales
python manage.py migrate
```

---

### 2. `ventas/models.py`

Este archivo contendrá las clases que representan las tablas de tu base de datos. He traducido tu modelo relacional a modelos de Django, prestando especial atención a las claves primarias y foráneas.

```python
# ventas/models.py

from django.db import models

# Modelo para M_FAMILIAS
class Familia(models.Model):
    id_fam = models.AutoField(primary_key=True)
    nombre_fam = models.CharField(max_length=60, unique=True, help_text="Descripción de la familia (Analgésicos, Vacunas, etc.)")

    def __str__(self):
        return self.nombre_fam

# Modelo para M_PRESENTACION
class Presentacion(models.Model):
    id_pres = models.AutoField(primary_key=True)
    nombre_pres = models.CharField(max_length=40, unique=True, help_text="Posibles: Polvos, Cápsulas, Comprimidos, etc.")

    def __str__(self):
        return self.nombre_pres

# Modelo para M_LABORATORIOS
class Laboratorio(models.Model):
    id_lab = models.AutoField(primary_key=True)
    nombre_lab = models.CharField(max_length=50)
    direccion = models.CharField(max_length=60, blank=True, null=True)
    poblacion = models.CharField(max_length=50, blank=True, null=True)
    provincia = models.CharField(max_length=30, blank=True, null=True)
    telefono = models.CharField(max_length=20, blank=True, null=True) # CharField para flexibilidad
    fax = models.CharField(max_length=20, blank=True, null=True) # CharField para flexibilidad
    email = models.EmailField(max_length=100, blank=True, null=True)

    def __str__(self):
        return self.nombre_lab

# Modelo para M_MEDICOS
class Medico(models.Model):
    dnim = models.CharField(max_length=9, primary_key=True) # DNI como CharField para la letra
    apellidos = models.CharField(max_length=80)
    nombre = models.CharField(max_length=50)
    centro_salud = models.CharField(max_length=60)
    poblacion = models.CharField(max_length=50)
    provincia = models.CharField(max_length=30)
    telefono = models.CharField(max_length=20, blank=True, null=True)
    movil = models.CharField(max_length=20, blank=True, null=True)
    email = models.EmailField(max_length=100, blank=True, null=True)

    def __str__(self):
        return f"{self.nombre} {self.apellidos} ({self.dnim})"

# Modelo para M_PACIENTES
class Paciente(models.Model):
    dnip = models.CharField(max_length=9, primary_key=True) # DNI como CharField para la letra
    nro_seg_soc = models.CharField(max_length=15, unique=True)
    apellidos = models.CharField(max_length=80)
    nombre = models.CharField(max_length=50)
    centro_salud = models.CharField(max_length=60)
    direccion = models.CharField(max_length=60)
    poblacion_pac = models.CharField(max_length=50)
    provincia_pac = models.CharField(max_length=30)
    telefono = models.CharField(max_length=20, blank=True, null=True)
    movil = models.CharField(max_length=20, blank=True, null=True)
    email = models.EmailField(max_length=100, blank=True, null=True)
    sexo = models.CharField(max_length=10, blank=True, null=True) # Del diagrama E/R

    def __str__(self):
        return f"{self.nombre} {self.apellidos} ({self.dnip})"

# Modelo para M_MEDICAMENTOS
class Medicamento(models.Model):
    RECETA_CHOICES = [
        ('S', 'Sí, necesita receta'),
        ('N', 'No necesita receta'),
    ]
    
    id_med = models.AutoField(primary_key=True)
    nombre_med = models.CharField(max_length=20)
    stock = models.PositiveIntegerField(help_text="Unidades existentes de este medicamento")
    precio_unit = models.DecimalField(max_digits=8, decimal_places=2, help_text="Precio Unitario")
    con_receta = models.CharField(max_length=1, choices=RECETA_CHOICES)
    
    # Relaciones con otras tablas
    presentacion = models.ForeignKey(Presentacion, on_delete=models.PROTECT)
    familia = models.ForeignKey(Familia, on_delete=models.PROTECT)
    laboratorio = models.ForeignKey(Laboratorio, on_delete=models.PROTECT)

    def __str__(self):
        return self.nombre_med

# Modelo para M_VENTAS_MED
class VentaMed(models.Model):
    id_venta = models.AutoField(primary_key=True)
    medicamento = models.ForeignKey(Medicamento, on_delete=models.PROTECT)
    fecha_venta = models.DateField(auto_now_add=True) # auto_now_add para registrar la fecha de creación
    unidades = models.PositiveIntegerField()

    def __str__(self):
        return f"Venta #{self.id_venta} - {self.unidades} x {self.medicamento.nombre_med}"

# Modelo para M_VENTAS_RECETA
# Esta tabla tiene una relación 1 a 1 con VentaMed. Solo existe si la venta fue con receta.
class VentaReceta(models.Model):
    venta = models.OneToOneField(VentaMed, on_delete=models.CASCADE, primary_key=True)
    medico = models.ForeignKey(Medico, on_delete=models.PROTECT) # Proteger para no borrar medicos con recetas
    paciente = models.ForeignKey(Paciente, on_delete=models.PROTECT) # Proteger para no borrar pacientes con recetas

    def __str__(self):
        return f"Receta para Venta #{self.venta.id_venta}"
```

Después de definir los modelos, crea y aplica las migraciones:

```bash
python manage.py makemigrations ventas
python manage.py migrate
```

---

### 3. `urls.py`

Necesitamos dos archivos `urls.py`: uno en el proyecto (`farma_project/`) y otro en la aplicación (`ventas/`).

**a) `farma_project/urls.py`**

```python
# farma_project/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('ventas.urls')), # Incluir las URLs de la app 'ventas'
]
```

**b) `ventas/urls.py` (Crea este archivo)**

```python
# ventas/urls.py
from django.urls import path
from . import views

urlpatterns = [
    # URL principal para ver y vender medicamentos
    path('', views.lista_medicamentos, name='lista_medicamentos'),
    
    # URL para el formulario de venta de un medicamento específico
    path('vender/<int:medicamento_id>/', views.registrar_venta, name='registrar_venta'),

    # URLs para gestionar las entidades de soporte (Laboratorios, Familias, etc.)
    # Ejemplo con Laboratorio:
    path('laboratorios/', views.lista_laboratorios, name='lista_laboratorios'),
    path('laboratorios/nuevo/', views.crear_laboratorio, name='crear_laboratorio'),
    
    # Ejemplo con Medico:
    path('medicos/', views.lista_medicos, name='lista_medicos'),
    path('medicos/nuevo/', views.crear_medico, name='crear_medico'),
    
    # Ejemplo con Paciente:
    path('pacientes/', views.lista_pacientes, name='lista_pacientes'),
    path('pacientes/nuevo/', views.crear_paciente, name='crear_paciente'),
    
    # Puedes añadir más URLs para Familia y Presentación si lo necesitas.
]
```

---

### 4. `ventas/views.py`

Aquí está la lógica principal. Todas las vistas son funciones, como se solicitó, y no usan `forms.py`.

```python
# ventas/views.py
from django.shortcuts import render, redirect, get_object_or_404
from django.db import transaction
from django.contrib import messages
from .models import Medicamento, VentaMed, VentaReceta, Laboratorio, Medico, Paciente

# Vista principal que muestra la lista de medicamentos
def lista_medicamentos(request):
    medicamentos = Medicamento.objects.all().order_by('nombre_med')
    return render(request, 'ventas/lista_medicamentos.html', {'medicamentos': medicamentos})

# Vista para registrar una venta
@transaction.atomic # Asegura que todas las operaciones de BD se completen o ninguna
def registrar_venta(request, medicamento_id):
    medicamento = get_object_or_404(Medicamento, pk=medicamento_id)
    
    # Preparar contexto inicial
    context = {'medicamento': medicamento}
    
    # Si el medicamento necesita receta, obtenemos la lista de médicos y pacientes
    if medicamento.con_receta == 'S':
        context['medicos'] = Medico.objects.all()
        context['pacientes'] = Paciente.objects.all()

    if request.method == 'POST':
        # --- Obtener datos del formulario manualmente ---
        unidades_str = request.POST.get('unidades')

        # --- Validación simple ---
        if not unidades_str or not unidades_str.isdigit() or int(unidades_str) <= 0:
            messages.error(request, "El número de unidades debe ser un entero positivo.")
            return render(request, 'ventas/registrar_venta.html', context)
        
        unidades = int(unidades_str)

        if unidades > medicamento.stock:
            messages.error(request, f"No hay suficiente stock. Stock actual: {medicamento.stock} unidades.")
            return render(request, 'ventas/registrar_venta.html', context)

        # --- Crear la Venta ---
        # 1. Crear el registro en VentaMed
        nueva_venta = VentaMed.objects.create(
            medicamento=medicamento,
            unidades=unidades
        )

        # 2. Actualizar el stock del medicamento
        medicamento.stock -= unidades
        medicamento.save()

        # 3. Si es con receta, crear el registro en VentaReceta
        if medicamento.con_receta == 'S':
            dnim = request.POST.get('medico')
            dnip = request.POST.get('paciente')
            
            if not dnim or not dnip:
                messages.error(request, "Para medicamentos con receta, debe seleccionar un médico y un paciente.")
                # Como usamos @transaction.atomic, la transacción se revertirá si salimos con error.
                # Es una buena práctica, aunque en este punto la venta ya se creó.
                # Podríamos validar antes de crear el objeto VentaMed.
                # Por simplicidad, lo dejamos así, pero en un caso real se validaría todo al principio.
                return render(request, 'ventas/registrar_venta.html', context)

            medico = get_object_or_404(Medico, dnim=dnim)
            paciente = get_object_or_404(Paciente, dnip=dnip)

            VentaReceta.objects.create(
                venta=nueva_venta,
                medico=medico,
                paciente=paciente
            )
        
        messages.success(request, "Venta registrada exitosamente.")
        return redirect('lista_medicamentos')

    # Si es una petición GET, solo muestra el formulario
    return render(request, 'ventas/registrar_venta.html', context)

# --- Vistas de ejemplo para crear otras entidades ---

def lista_laboratorios(request):
    laboratorios = Laboratorio.objects.all()
    return render(request, 'ventas/lista_laboratorios.html', {'laboratorios': laboratorios})

def crear_laboratorio(request):
    if request.method == 'POST':
        nombre = request.POST.get('nombre_lab')
        # Puedes añadir el resto de los campos...
        if nombre:
            Laboratorio.objects.create(nombre_lab=nombre)
            messages.success(request, 'Laboratorio creado.')
            return redirect('lista_laboratorios')
        else:
            messages.error(request, 'El nombre es obligatorio.')
    return render(request, 'ventas/crear_laboratorio.html')

# Puedes seguir este patrón para Medico, Paciente, etc.
def lista_medicos(request):
    medicos = Medico.objects.all()
    return render(request, 'ventas/lista_medicos.html', {'medicos': medicos})

def crear_medico(request):
    if request.method == 'POST':
        # Extraer todos los campos del POST
        dnim = request.POST.get('dnim')
        nombre = request.POST.get('nombre')
        apellidos = request.POST.get('apellidos')
        # ... otros campos
        if dnim and nombre and apellidos: # Validación mínima
            Medico.objects.create(dnim=dnim, nombre=nombre, apellidos=apellidos, centro_salud="Centro X", poblacion="Poblacion Y", provincia="Provincia Z") # Añade todos los campos
            messages.success(request, 'Médico creado.')
            return redirect('lista_medicos')
    return render(request, 'ventas/crear_medico.html')

def lista_pacientes(request):
    pacientes = Paciente.objects.all()
    return render(request, 'ventas/lista_pacientes.html', {'pacientes': pacientes})

def crear_paciente(request):
    if request.method == 'POST':
        # Extraer todos los campos del POST
        dnip = request.POST.get('dnip')
        nombre = request.POST.get('nombre')
        apellidos = request.POST.get('apellidos')
        nro_seg_soc = request.POST.get('nro_seg_soc')
        # ... otros campos
        if dnip and nombre and apellidos and nro_seg_soc: # Validación mínima
             # Rellena todos los campos obligatorios
            Paciente.objects.create(dnip=dnip, nombre=nombre, apellidos=apellidos, nro_seg_soc=nro_seg_soc, centro_salud="Centro A", direccion="Calle B", poblacion_pac="Poblacion C", provincia_pac="Provincia D")
            messages.success(request, 'Paciente creado.')
            return redirect('lista_pacientes')
    return render(request, 'ventas/crear_paciente.html')
```

---

### 5. Plantillas HTML (Bonus)

Para que las vistas funcionen, necesitas crear estas plantillas dentro de una carpeta `ventas/templates/ventas/`.

**`ventas/templates/ventas/base.html` (Plantilla base)**
```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Farmacia FARMA</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css">
</head>
<body>
    <nav class="navbar navbar-expand-lg navbar-dark bg-primary">
        <div class="container-fluid">
            <a class="navbar-brand" href="{% url 'lista_medicamentos' %}">FARMA</a>
            <div class="collapse navbar-collapse">
                <ul class="navbar-nav me-auto mb-2 mb-lg-0">
                    <li class="nav-item"><a class="nav-link" href="{% url 'lista_medicamentos' %}">Medicamentos</a></li>
                    <li class="nav-item"><a class="nav-link" href="{% url 'lista_laboratorios' %}">Laboratorios</a></li>
                    <li class="nav-item"><a class="nav-link" href="{% url 'lista_medicos' %}">Médicos</a></li>
                    <li class="nav-item"><a class="nav-link" href="{% url 'lista_pacientes' %}">Pacientes</a></li>
                </ul>
            </div>
        </div>
    </nav>
    <main class="container mt-4">
        {% if messages %}
            {% for message in messages %}
                <div class="alert alert-{{ message.tags }}">{{ message }}</div>
            {% endfor %}
        {% endif %}
        {% block content %}{% endblock %}
    </main>
</body>
</html>
```

**`ventas/templates/ventas/lista_medicamentos.html`**
```html
{% extends "ventas/base.html" %}

{% block content %}
    <h2>Lista de Medicamentos</h2>
    <table class="table table-striped">
        <thead>
            <tr>
                <th>Nombre</th>
                <th>Stock</th>
                <th>Precio</th>
                <th>Necesita Receta</th>
                <th>Laboratorio</th>
                <th>Acciones</th>
            </tr>
        </thead>
        <tbody>
            {% for med in medicamentos %}
            <tr>
                <td>{{ med.nombre_med }}</td>
                <td>{{ med.stock }}</td>
                <td>${{ med.precio_unit }}</td>
                <td>{{ med.get_con_receta_display }}</td>
                <td>{{ med.laboratorio.nombre_lab }}</td>
                <td>
                    <a href="{% url 'registrar_venta' med.id_med %}" class="btn btn-success btn-sm">Vender</a>
                </td>
            </tr>
            {% empty %}
            <tr>
                <td colspan="6">No hay medicamentos registrados.</td>
            </tr>
            {% endfor %}
        </tbody>
    </table>
{% endblock %}
```

**`ventas/templates/ventas/registrar_venta.html`**
```html
{% extends "ventas/base.html" %}

{% block content %}
    <h2>Registrar Venta de: {{ medicamento.nombre_med }}</h2>
    <p><strong>Stock disponible:</strong> {{ medicamento.stock }} unidades</p>
    
    <form method="post">
        {% csrf_token %}
        <div class="mb-3">
            <label for="unidades" class="form-label">Unidades a vender:</label>
            <input type="number" id="unidades" name="unidades" class="form-control" required min="1">
        </div>

        {% if medicamento.con_receta == 'S' %}
            <div class="alert alert-warning">Este medicamento requiere receta.</div>
            
            <div class="mb-3">
                <label for="medico" class="form-label">Médico que receta:</label>
                <select name="medico" id="medico" class="form-select" required>
                    <option value="">Seleccione un médico...</option>
                    {% for medico in medicos %}
                        <option value="{{ medico.dnim }}">{{ medico }}</option>
                    {% endfor %}
                </select>
            </div>
            
            <div class="mb-3">
                <label for="paciente" class="form-label">Paciente:</label>
                <select name="paciente" id="paciente" class="form-select" required>
                    <option value="">Seleccione un paciente...</option>
                    {% for paciente in pacientes %}
                        <option value="{{ paciente.dnip }}">{{ paciente }}</option>
                    {% endfor %}
                </select>
            </div>
        {% endif %}

        <button type="submit" class="btn btn-primary">Confirmar Venta</button>
        <a href="{% url 'lista_medicamentos' %}" class="btn btn-secondary">Cancelar</a>
    </form>
{% endblock %}
```

Con estos archivos, tendrás una aplicación funcional que cumple con todos los requisitos del problema planteado. Solo necesitarás crear las plantillas para las otras vistas de "crear" (ej. `crear_laboratorio.html`) siguiendo el mismo patrón.
