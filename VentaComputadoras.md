1. Diagrama de clase
2. <img width="946" height="722" alt="image" src="https://github.com/user-attachments/assets/33761d78-03a8-477c-846b-5b7a8f071df1" />
3. el proyecto
4. ### Proyecto de Venta de Computadoras

Basándonos en el diagrama UML proporcionado, vamos a implementar los modelos (`models.py`), vistas (`views.py`), y rutas (`urls.py`) para un proyecto Django que gestione la venta de computadoras. El diagrama muestra las entidades `Persona`, `Cliente`, `Empleado`, `Orden`, `Producto`, `Categoria`, y `OrdenDetalle`. A continuación, se detalla la implementación paso a paso.

---

#### **1. `models.py`**
Primero, definimos los modelos basados en el diagrama UML. Cada entidad será representada como una clase en Django, y las relaciones entre ellas se implementarán utilizando `ForeignKey`, `OneToOneField`, etc.

```python
from django.db import models
from django.core.validators import MinValueValidator

# Clase base Persona
class Persona(models.Model):
    cedula = models.CharField(max_length=20, unique=True)
    nombre = models.CharField(max_length=100)
    apellido = models.CharField(max_length=100)
    fecha_nacimiento = models.DateField()
    telefono = models.CharField(max_length=20)
    correo = models.EmailField()

    def __str__(self):
        return f"{self.nombre} {self.apellido}"

# Clase Cliente (hereda de Persona)
class Cliente(Persona):
    fecha_registro = models.DateField()
    numero_tarjeta = models.CharField(max_length=20)
    monto_credito = models.DecimalField(max_digits=10, decimal_places=2, validators=[MinValueValidator(0)])
    estado = models.BooleanField(default=True)

    def __str__(self):
        return f"Cliente: {self.nombre} {self.apellido}"

# Clase Empleado (hereda de Persona)
class Empleado(Persona):
    fecha_ingreso = models.DateField()
    departamento = models.CharField(max_length=100)
    cargo = models.CharField(max_length=100)
    salario = models.DecimalField(max_digits=10, decimal_places=2, validators=[MinValueValidator(0)])
    estado = models.BooleanField(default=True)

    def __str__(self):
        return f"Empleado: {self.nombre} {self.apellido}"

# Clase Categoria
class Categoria(models.Model):
    nombre = models.CharField(max_length=100)
    descripcion = models.TextField()

    def __str__(self):
        return self.nombre

# Clase Producto
class Producto(models.Model):
    nombre = models.CharField(max_length=100)
    descripcion = models.TextField()
    categoria = models.ForeignKey(Categoria, on_delete=models.CASCADE, related_name='productos')
    precio = models.DecimalField(max_digits=10, decimal_places=2, validators=[MinValueValidator(0)])

    def __str__(self):
        return self.nombre

# Clase Orden
class Orden(models.Model):
    orden_id = models.AutoField(primary_key=True)
    cliente = models.ForeignKey(Cliente, on_delete=models.CASCADE, related_name='ordenes')
    empleado = models.ForeignKey(Empleado, on_delete=models.CASCADE, related_name='ordenes')
    fecha = models.DateTimeField(auto_now_add=True)
    observacion = models.TextField(blank=True, null=True)

    def __str__(self):
        return f"Orden #{self.orden_id} - Cliente: {self.cliente.nombre}"

# Clase OrdenDetalle
class OrdenDetalle(models.Model):
    orden = models.ForeignKey(Orden, on_delete=models.CASCADE, related_name='detalles')
    producto = models.ForeignKey(Producto, on_delete=models.CASCADE)
    cantidad = models.PositiveIntegerField(validators=[MinValueValidator(1)])
    precio_unitario = models.DecimalField(max_digits=10, decimal_places=2, validators=[MinValueValidator(0)])

    def __str__(self):
        return f"Orden #{self.orden.orden_id} - Producto: {self.producto.nombre}"
```

---

#### **2. `views.py`**
A continuación, implementamos las vistas para manejar las operaciones CRUD (Crear, Leer, Actualizar, Eliminar) de las entidades.

```python
from django.shortcuts import render, redirect, get_object_or_404
from .models import Cliente, Empleado, Categoria, Producto, Orden, OrdenDetalle
from .forms import ClienteForm, EmpleadoForm, CategoriaForm, ProductoForm, OrdenForm, OrdenDetalleForm

# Vistas para Cliente
def cliente_list(request):
    clientes = Cliente.objects.all()
    return render(request, 'clientes/list.html', {'clientes': clientes})

def cliente_create(request):
    if request.method == 'POST':
        form = ClienteForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('cliente_list')
    else:
        form = ClienteForm()
    return render(request, 'clientes/create.html', {'form': form})

def cliente_update(request, pk):
    cliente = get_object_or_404(Cliente, pk=pk)
    if request.method == 'POST':
        form = ClienteForm(request.POST, instance=cliente)
        if form.is_valid():
            form.save()
            return redirect('cliente_list')
    else:
        form = ClienteForm(instance=cliente)
    return render(request, 'clientes/update.html', {'form': form, 'cliente': cliente})

def cliente_delete(request, pk):
    cliente = get_object_or_404(Cliente, pk=pk)
    if request.method == 'POST':
        cliente.delete()
        return redirect('cliente_list')
    return render(request, 'clientes/delete.html', {'cliente': cliente})

# Vistas para Empleado
def empleado_list(request):
    empleados = Empleado.objects.all()
    return render(request, 'empleados/list.html', {'empleados': empleados})

def empleado_create(request):
    if request.method == 'POST':
        form = EmpleadoForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('empleado_list')
    else:
        form = EmpleadoForm()
    return render(request, 'empleados/create.html', {'form': form})

def empleado_update(request, pk):
    empleado = get_object_or_404(Empleado, pk=pk)
    if request.method == 'POST':
        form = EmpleadoForm(request.POST, instance=empleado)
        if form.is_valid():
            form.save()
            return redirect('empleado_list')
    else:
        form = EmpleadoForm(instance=empleado)
    return render(request, 'empleados/update.html', {'form': form, 'empleado': empleado})

def empleado_delete(request, pk):
    empleado = get_object_or_404(Empleado, pk=pk)
    if request.method == 'POST':
        empleado.delete()
        return redirect('empleado_list')
    return render(request, 'empleados/delete.html', {'empleado': empleado})

# Vistas para Categoria
def categoria_list(request):
    categorias = Categoria.objects.all()
    return render(request, 'categorias/list.html', {'categorias': categorias})

def categoria_create(request):
    if request.method == 'POST':
        form = CategoriaForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('categoria_list')
    else:
        form = CategoriaForm()
    return render(request, 'categorias/create.html', {'form': form})

def categoria_update(request, pk):
    categoria = get_object_or_404(Categoria, pk=pk)
    if request.method == 'POST':
        form = CategoriaForm(request.POST, instance=categoria)
        if form.is_valid():
            form.save()
            return redirect('categoria_list')
    else:
        form = CategoriaForm(instance=categoria)
    return render(request, 'categorias/update.html', {'form': form, 'categoria': categoria})

def categoria_delete(request, pk):
    categoria = get_object_or_404(Categoria, pk=pk)
    if request.method == 'POST':
        categoria.delete()
        return redirect('categoria_list')
    return render(request, 'categorias/delete.html', {'categoria': categoria})

# Vistas para Producto
def producto_list(request):
    productos = Producto.objects.all()
    return render(request, 'productos/list.html', {'productos': productos})

def producto_create(request):
    if request.method == 'POST':
        form = ProductoForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('producto_list')
    else:
        form = ProductoForm()
    return render(request, 'productos/create.html', {'form': form})

def producto_update(request, pk):
    producto = get_object_or_404(Producto, pk=pk)
    if request.method == 'POST':
        form = ProductoForm(request.POST, instance=producto)
        if form.is_valid():
            form.save()
            return redirect('producto_list')
    else:
        form = ProductoForm(instance=producto)
    return render(request, 'productos/update.html', {'form': form, 'producto': producto})

def producto_delete(request, pk):
    producto = get_object_or_404(Producto, pk=pk)
    if request.method == 'POST':
        producto.delete()
        return redirect('producto_list')
    return render(request, 'productos/delete.html', {'producto': producto})

# Vistas para Orden
def orden_list(request):
    ordenes = Orden.objects.all()
    return render(request, 'ordenes/list.html', {'ordenes': ordenes})

def orden_create(request):
    if request.method == 'POST':
        form = OrdenForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('orden_list')
    else:
        form = OrdenForm()
    return render(request, 'ordenes/create.html', {'form': form})

def orden_update(request, pk):
    orden = get_object_or_404(Orden, pk=pk)
    if request.method == 'POST':
        form = OrdenForm(request.POST, instance=orden)
        if form.is_valid():
            form.save()
            return redirect('orden_list')
    else:
        form = OrdenForm(instance=orden)
    return render(request, 'ordenes/update.html', {'form': form, 'orden': orden})

def orden_delete(request, pk):
    orden = get_object_or_404(Orden, pk=pk)
    if request.method == 'POST':
        orden.delete()
        return redirect('orden_list')
    return render(request, 'ordenes/delete.html', {'orden': orden})

# Vistas para OrdenDetalle
def ordendetalle_list(request):
    detalles = OrdenDetalle.objects.all()
    return render(request, 'ordendetalles/list.html', {'detalles': detalles})

def ordendetalle_create(request):
    if request.method == 'POST':
        form = OrdenDetalleForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('ordendetalle_list')
    else:
        form = OrdenDetalleForm()
    return render(request, 'ordendetalles/create.html', {'form': form})

def ordendetalle_update(request, pk):
    detalle = get_object_or_404(OrdenDetalle, pk=pk)
    if request.method == 'POST':
        form = OrdenDetalleForm(request.POST, instance=detalle)
        if form.is_valid():
            form.save()
            return redirect('ordendetalle_list')
    else:
        form = OrdenDetalleForm(instance=detalle)
    return render(request, 'ordendetalles/update.html', {'form': form, 'detalle': detalle})

def ordendetalle_delete(request, pk):
    detalle = get_object_or_404(OrdenDetalle, pk=pk)
    if request.method == 'POST':
        detalle.delete()
        return redirect('ordendetalle_list')
    return render(request, 'ordendetalles/delete.html', {'detalle': detalle})
```

---

#### **3. `forms.py`**
Creamos los formularios para cada modelo para facilitar la validación y renderizado en las vistas.

```python
from django import forms
from .models import Cliente, Empleado, Categoria, Producto, Orden, OrdenDetalle

# Formulario para Cliente
class ClienteForm(forms.ModelForm):
    class Meta:
        model = Cliente
        fields = ['cedula', 'nombre', 'apellido', 'fecha_nacimiento', 'telefono', 'correo', 'fecha_registro', 'numero_tarjeta', 'monto_credito', 'estado']

# Formulario para Empleado
class EmpleadoForm(forms.ModelForm):
    class Meta:
        model = Empleado
        fields = ['cedula', 'nombre', 'apellido', 'fecha_nacimiento', 'telefono', 'correo', 'fecha_ingreso', 'departamento', 'cargo', 'salario', 'estado']

# Formulario para Categoria
class CategoriaForm(forms.ModelForm):
    class Meta:
        model = Categoria
        fields = ['nombre', 'descripcion']

# Formulario para Producto
class ProductoForm(forms.ModelForm):
    class Meta:
        model = Producto
        fields = ['nombre', 'descripcion', 'categoria', 'precio']

# Formulario para Orden
class OrdenForm(forms.ModelForm):
    class Meta:
        model = Orden
        fields = ['cliente', 'empleado', 'observacion']

# Formulario para OrdenDetalle
class OrdenDetalleForm(forms.ModelForm):
    class Meta:
        model = OrdenDetalle
        fields = ['orden', 'producto', 'cantidad', 'precio_unitario']
```

---

#### **4. `urls.py`**
Finalmente, configuramos las rutas para acceder a las vistas.

```python
from django.urls import path
from . import views

urlpatterns = [
    # URLs para Cliente
    path('clientes/', views.cliente_list, name='cliente_list'),
    path('clientes/create/', views.cliente_create, name='cliente_create'),
    path('clientes/<int:pk>/update/', views.cliente_update, name='cliente_update'),
    path('clientes/<int:pk>/delete/', views.cliente_delete, name='cliente_delete'),

    # URLs para Empleado
    path('empleados/', views.empleado_list, name='empleado_list'),
    path('empleados/create/', views.empleado_create, name='empleado_create'),
    path('empleados/<int:pk>/update/', views.empleado_update, name='empleado_update'),
    path('empleados/<int:pk>/delete/', views.empleado_delete, name='empleado_delete'),

    # URLs para Categoria
    path('categorias/', views.categoria_list, name='categoria_list'),
    path('categorias/create/', views.categoria_create, name='categoria_create'),
    path('categorias/<int:pk>/update/', views.categoria_update, name='categoria_update'),
    path('categorias/<int:pk>/delete/', views.categoria_delete, name='categoria_delete'),

    # URLs para Producto
    path('productos/', views.producto_list, name='producto_list'),
    path('productos/create/', views.producto_create, name='producto_create'),
    path('productos/<int:pk>/update/', views.producto_update, name='producto_update'),
    path('productos/<int:pk>/delete/', views.producto_delete, name='producto_delete'),

    # URLs para Orden
    path('ordenes/', views.orden_list, name='orden_list'),
    path('ordenes/create/', views.orden_create, name='orden_create'),
    path('ordenes/<int:pk>/update/', views.orden_update, name='orden_update'),
    path('ordenes/<int:pk>/delete/', views.orden_delete, name='orden_delete'),

    # URLs para OrdenDetalle
    path('ordendetalles/', views.ordendetalle_list, name='ordendetalle_list'),
    path('ordendetalles/create/', views.ordendetalle_create, name='ordendetalle_create'),
    path('ordendetalles/<int:pk>/update/', views.ordendetalle_update, name='ordendetalle_update'),
    path('ordendetalles/<int:pk>/delete/', views.ordendetalle_delete, name='ordendetalle_delete'),
]
```

---

### **5. Plantillas HTML**
Para completar el proyecto, necesitarás crear plantillas HTML para cada vista. Aquí tienes un ejemplo básico para la lista de clientes:

#### `clientes/list.html`
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Lista de Clientes</title>
</head>
<body>
    <h1>Lista de Clientes</h1>
    <table border="1">
        <thead>
            <tr>
                <th>Cédula</th>
                <th>Nombre</th>
                <th>Apellido</th>
                <th>Teléfono</th>
                <th>Correo</th>
                <th>Acciones</th>
            </tr>
        </thead>
        <tbody>
            {% for cliente in clientes %}
            <tr>
                <td>{{ cliente.cedula }}</td>
                <td>{{ cliente.nombre }}</td>
                <td>{{ cliente.apellido }}</td>
                <td>{{ cliente.telefono }}</td>
                <td>{{ cliente.correo }}</td>
                <td>
                    <a href="{% url 'cliente_update' cliente.pk %}">Editar</a>
                    <a href="{% url 'cliente_delete' cliente.pk %}">Eliminar</a>
                </td>
            </tr>
            {% endfor %}
        </tbody>
    </table>
    <a href="{% url 'cliente_create' %}">Crear Cliente</a>
</body>
</html>
```

Repite este proceso para las demás entidades (`empleados`, `categorias`, `productos`, `ordenes`, `ordendetalles`).

---

### **6. Resumen**
- **`models.py`**: Define las entidades y sus relaciones.
- **`views.py`**: Implementa las vistas para manejar las operaciones CRUD.
- **`forms.py`**: Crea formularios para validar y renderizar datos.
- **`urls.py`**: Configura las rutas para acceder a las vistas.
- **Plantillas HTML**: Renderiza la interfaz de usuario.

Este proyecto está listo para ser ejecutado en Django. Si necesitas más detalles o ampliaciones, no dudes en preguntar.
