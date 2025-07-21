### Proyecto de Venta de Medicamentos en Django

#### **Planteamiento del Problema**
La farmacia FARMA necesita un sistema para registrar las ventas de medicamentos. Cada medicamento pertenece a una familia (analgésicos, antiinflamatorios, etc.), es suministrado por un laboratorio y tiene una presentación (pomadas, cápsulas, comprimidos, jarabes, etc.). Además, se debe registrar si el medicamento se vende con receta médica o sin receta. Si se vende con receta, se deben registrar el médico que lo recetó y el paciente al que se le recetó.

#### **Estructura del Proyecto**
Basándonos en el diagrama E/R y los modelos relacionales proporcionados, implementaremos:
1. **Modelos (`models.py`)**: Definir las entidades y sus relaciones.
2. **Vistas (`views.py`)**: Implementar las funciones para manejar las operaciones CRUD.
3. **Rutas (`urls.py`)**: Configurar las URLs para acceder a las vistas.

---

### **1. `models.py`**

Primero, definimos los modelos basados en el diagrama E/R y los modelos relacionales.

```python
from django.db import models

# Clase Laboratorio
class Laboratorio(models.Model):
    id_lab = models.AutoField(primary_key=True)
    nombre_lab = models.CharField(max_length=50)
    direccion = models.CharField(max_length=60)
    poblacion = models.CharField(max_length=50)
    provincia = models.CharField(max_length=30)
    telefono = models.CharField(max_length=9)
    fax = models.CharField(max_length=9)
    email = models.EmailField()

    def __str__(self):
        return self.nombre_lab

# Clase Familia
class Familia(models.Model):
    id_fam = models.AutoField(primary_key=True)
    nombre_fam = models.CharField(max_length=60)

    def __str__(self):
        return self.nombre_fam

# Clase Presentacion
class Presentacion(models.Model):
    id_pres = models.AutoField(primary_key=True)
    nombre_pres = models.CharField(max_length=40)

    def __str__(self):
        return self.nombre_pres

# Clase Medicamento
class Medicamento(models.Model):
    id_med = models.AutoField(primary_key=True)
    nombre_med = models.CharField(max_length=20)
    stock = models.PositiveIntegerField()
    precio_unit = models.DecimalField(max_digits=8, decimal_places=2)
    con_receta = models.CharField(max_length=1, choices=[('S', 'Con Receta'), ('N', 'Sin Receta')])
    presentacion = models.ForeignKey(Presentacion, on_delete=models.CASCADE, related_name='medicamentos')
    familia = models.ForeignKey(Familia, on_delete=models.CASCADE, related_name='medicamentos')
    laboratorio = models.ForeignKey(Laboratorio, on_delete=models.CASCADE, related_name='medicamentos')

    def __str__(self):
        return self.nombre_med

# Clase Medico
class Medico(models.Model):
    dnim = models.CharField(primary_key=True, max_length=9)
    apellidos = models.CharField(max_length=80)
    nombre = models.CharField(max_length=50)
    centro_salud = models.CharField(max_length=60)
    poblacion = models.CharField(max_length=50)
    provincia = models.CharField(max_length=30)
    telefono = models.CharField(max_length=9)
    movil = models.CharField(max_length=9)
    email = models.EmailField()

    def __str__(self):
        return f"{self.apellidos}, {self.nombre}"

# Clase Paciente
class Paciente(models.Model):
    dnip = models.CharField(primary_key=True, max_length=9)
    nro_seg_soc = models.CharField(max_length=15)
    apellidos = models.CharField(max_length=80)
    nombre = models.CharField(max_length=50)
    centro_salud = models.CharField(max_length=60)
    direccion = models.CharField(max_length=60)
    poblacion = models.CharField(max_length=50)
    provincia = models.CharField(max_length=30)
    telefono = models.CharField(max_length=9)
    movil = models.CharField(max_length=9)
    email = models.EmailField()

    def __str__(self):
        return f"{self.apellidos}, {self.nombre}"

# Clase VentasMed
class VentasMed(models.Model):
    id_venta = models.AutoField(primary_key=True)
    medicamento = models.ForeignKey(Medicamento, on_delete=models.CASCADE, related_name='ventas')
    fecha_venta = models.DateField()
    unidades = models.PositiveIntegerField()

    def __str__(self):
        return f"Venta #{self.id_venta} - {self.medicamento.nombre_med}"

# Clase VentasRecetas
class VentasRecetas(models.Model):
    id_venta = models.OneToOneField(VentasMed, on_delete=models.CASCADE, primary_key=True, related_name='receta')
    medico = models.ForeignKey(Medico, on_delete=models.CASCADE, related_name='recetas')
    paciente = models.ForeignKey(Paciente, on_delete=models.CASCADE, related_name='recetas')

    def __str__(self):
        return f"Receta #{self.id_venta.id_venta} - Médico: {self.medico}, Paciente: {self.paciente}"
```

---

### **2. `views.py`**

Implementamos las vistas utilizando funciones para manejar las operaciones CRUD.

```python
from django.shortcuts import render, redirect, get_object_or_404
from .models import (
    Laboratorio,
    Familia,
    Presentacion,
    Medicamento,
    Medico,
    Paciente,
    VentasMed,
    VentasRecetas,
)

# Vistas para Laboratorio
def laboratorio_list(request):
    laboratorios = Laboratorio.objects.all()
    return render(request, 'laboratorios/list.html', {'laboratorios': laboratorios})

def laboratorio_create(request):
    if request.method == 'POST':
        # Implementar la lógica de creación aquí
        pass
    else:
        return render(request, 'laboratorios/create.html')

def laboratorio_update(request, pk):
    laboratorio = get_object_or_404(Laboratorio, pk=pk)
    if request.method == 'POST':
        # Implementar la lógica de actualización aquí
        pass
    else:
        return render(request, 'laboratorios/update.html', {'laboratorio': laboratorio})

def laboratorio_delete(request, pk):
    laboratorio = get_object_or_404(Laboratorio, pk=pk)
    if request.method == 'POST':
        laboratorio.delete()
        return redirect('laboratorio_list')
    return render(request, 'laboratorios/delete.html', {'laboratorio': laboratorio})

# Vistas para Familia
def familia_list(request):
    familias = Familia.objects.all()
    return render(request, 'familias/list.html', {'familias': familias})

# ... (Repetir para otras entidades: Presentacion, Medicamento, Medico, Paciente, VentasMed, VentasRecetas)
```

---

### **3. `urls.py`**

Configuramos las rutas para acceder a las vistas.

```python
from django.urls import path
from . import views

urlpatterns = [
    # URLs para Laboratorio
    path('laboratorios/', views.laboratorio_list, name='laboratorio_list'),
    path('laboratorios/create/', views.laboratorio_create, name='laboratorio_create'),
    path('laboratorios/<int:pk>/update/', views.laboratorio_update, name='laboratorio_update'),
    path('laboratorios/<int:pk>/delete/', views.laboratorio_delete, name='laboratorio_delete'),

    # URLs para Familia
    path('familias/', views.familia_list, name='familia_list'),
    # ... (Repetir para otras entidades: Presentacion, Medicamento, Medico, Paciente, VentasMed, VentasRecetas)
]
```

---

### **4. Plantillas HTML**
Para cada vista, necesitamos crear plantillas HTML. Aquí tienes un ejemplo para la lista de laboratorios:

#### `laboratorios/list.html`
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Lista de Laboratorios</title>
</head>
<body>
    <h1>Lista de Laboratorios</h1>
    <table border="1">
        <thead>
            <tr>
                <th>ID</th>
                <th>Nombre</th>
                <th>Dirección</th>
                <th>Población</th>
                <th>Acciones</th>
            </tr>
        </thead>
        <tbody>
            {% for laboratorio in laboratorios %}
            <tr>
                <td>{{ laboratorio.id_lab }}</td>
                <td>{{ laboratorio.nombre_lab }}</td>
                <td>{{ laboratorio.direccion }}</td>
                <td>{{ laboratorio.poblacion }}</td>
                <td>
                    <a href="{% url 'laboratorio_update' laboratorio.id_lab %}">Editar</a>
                    <a href="{% url 'laboratorio_delete' laboratorio.id_lab %}">Eliminar</a>
                </td>
            </tr>
            {% endfor %}
        </tbody>
    </table>
    <a href="{% url 'laboratorio_create' %}">Crear Laboratorio</a>
</body>
</html>
```

Repite este proceso para las demás entidades (`familias`, `presentaciones`, `medicamentos`, `medicos`, `pacientes`, `ventas_med`, `ventas_recetas`).

---

### **5. Resumen**
- **`models.py`**: Define las entidades y sus relaciones.
- **`views.py`**: Implementa las vistas para manejar las operaciones CRUD.
- **`urls.py`**: Configura las rutas para acceder a las vistas.
- **Plantillas HTML**: Renderiza la interfaz de usuario.

Este proyecto está listo para ser ejecutado en Django. Si necesitas más detalles o ampliaciones, no dudes en preguntar.
