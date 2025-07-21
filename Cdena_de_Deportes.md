[media pointer="file-service://file-ENrc4PRBG1d4H3QAFk1A18"]
Crear el models.py , urls.py  las views, con sus respectivos códigos en python django. para el siguiente proyecto.  agrego también una imagen  del modelo relacional. Cadena de Deportes 
Una cadena de casas de deportes desea realizar una base de datos para manejar sus sucursales, empleados,productos y clientes.
De las sucursales se sabe el número único que la identifica dentro de la cadena, el domicilio y la ciudad.
De los empleados el legajo, el nombre, el dni, el domicilio (calle, número y ciudad) y los números de teléfono en los cuales puede ser contactado.
Los empleados trabajan en diferentes sucursales en diferentes días de la semana y en cada sucursal tiene asignado un horario en particular, que puede no ser el mismo en diferentes sucursales. Por ejemplo, el empleado GBA trabaja los lunes de 9hs. a 18hs. en la Sucursal 1, y los martes de 10hs. a 20hs. en la Sucursal 2. 
En cada sucursal trabajan varios empleados.
De los productos se conoce un código, una descripción, un color y un costo fijo de fabricación. A su vez, existen también las fábricas que son identificadas con CUIT, nombre, país de origen, cantidad de empleados y nombre de gerente. Cada producto es fabricado en una sola fábrica, y cada fábrica solamente realiza un tipo de producto. El costo fijo de fabricación no depende de la fábrica.
Cada sucursal puede vender varios productos, y a su vez, cada producto puede ser vendido por varias sucursales. Cada sucursal establece cuál es el precio de venta del producto que ofrece. Es decir, un mismo producto podría tener diferentes precios en diferentes sucursales.
De los clientes se conoce el código de cliente, el dni, el nombre, la fecha de nacimiento y la ciudad en la que vive. A su vez, también sabemos que cada cliente puede utilizar varias tarjetas de crédito, que son identificadas por el nombre de la tarjeta, el número, el código de seguridad y la fecha de vencimiento.
Cada cliente solamente realiza compras en una sola sucursal, y en una sucursal pueden comprar varios clientes. Cada sucursal le ofrece a sus clientes un descuento fijo por su fidelidad en las compras.

2. <img width="1160" height="857" alt="image" src="https://github.com/user-attachments/assets/a470d56e-c992-4e1a-ad90-5c02b0dddbf9" />
3. Perfecto, Eliseo. A continuación te presento una implementación en Django basada en el modelo entidad-relación de la imagen y la descripción del proyecto **Cadena de Deportes**.

---

## 📁 `models.py`

```python
from django.db import models

# === MODELOS BÁSICOS ===

class Sucursal(models.Model):
    numero = models.IntegerField(primary_key=True)
    domicilio = models.CharField(max_length=200)
    ciudad = models.CharField(max_length=100)
    descuento = models.DecimalField(max_digits=5, decimal_places=2, default=0.00)

    def __str__(self):
        return f"Sucursal {self.numero} - {self.ciudad}"

class Empleado(models.Model):
    legajo = models.AutoField(primary_key=True)
    nombre = models.CharField(max_length=100)
    dni = models.CharField(max_length=15)
    calle = models.CharField(max_length=100)
    numero = models.CharField(max_length=10)
    ciudad = models.CharField(max_length=50)
    telefonos = models.CharField(max_length=200)  # múltiples separados por coma

    def __str__(self):
        return self.nombre

class Cliente(models.Model):
    codigo = models.AutoField(primary_key=True)
    dni = models.CharField(max_length=15)
    nombre = models.CharField(max_length=100)
    fecha_nac = models.DateField()
    ciudad = models.CharField(max_length=100)
    sucursal = models.ForeignKey(Sucursal, on_delete=models.CASCADE, related_name="clientes")

    def __str__(self):
        return self.nombre

class Tarjeta(models.Model):
    numero = models.CharField(max_length=20, primary_key=True)
    nombre = models.CharField(max_length=100)
    codigo = models.CharField(max_length=10)
    fecha_venc = models.DateField()
    cliente = models.ForeignKey(Cliente, on_delete=models.CASCADE, related_name="tarjetas")

    def __str__(self):
        return f"{self.nombre} - {self.numero}"

# === PRODUCTOS Y FÁBRICAS ===

class Fabrica(models.Model):
    cuit = models.CharField(max_length=15, primary_key=True)
    nombre = models.CharField(max_length=100)
    pais = models.CharField(max_length=100)
    cant_emp = models.IntegerField()
    gerente = models.CharField(max_length=100)

    def __str__(self):
        return self.nombre

class Producto(models.Model):
    codigo = models.AutoField(primary_key=True)
    descripcion = models.TextField()
    color = models.CharField(max_length=50)
    costo = models.DecimalField(max_digits=10, decimal_places=2)
    fabrica = models.OneToOneField(Fabrica, on_delete=models.CASCADE, related_name="producto")

    def __str__(self):
        return self.descripcion

class VentaProducto(models.Model):
    producto = models.ForeignKey(Producto, on_delete=models.CASCADE)
    sucursal = models.ForeignKey(Sucursal, on_delete=models.CASCADE)
    precio_venta = models.DecimalField(max_digits=10, decimal_places=2)

    class Meta:
        unique_together = ('producto', 'sucursal')

    def __str__(self):
        return f"{self.producto} vendido en {self.sucursal} a {self.precio_venta}"

# === RELACIONES COMPLEJAS ===

class Trabajo(models.Model):
    empleado = models.ForeignKey(Empleado, on_delete=models.CASCADE)
    sucursal = models.ForeignKey(Sucursal, on_delete=models.CASCADE)
    dia = models.CharField(max_length=20)
    horario = models.CharField(max_length=50)

    class Meta:
        unique_together = ('empleado', 'sucursal', 'dia')

    def __str__(self):
        return f"{self.empleado} en {self.sucursal} - {self.dia}"
```

---

## 🧭 `urls.py`

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.inicio, name='inicio'),
    path('clientes/', views.ClienteListView.as_view(), name='clientes'),
    path('sucursales/', views.SucursalListView.as_view(), name='sucursales'),
    path('empleados/', views.EmpleadoListView.as_view(), name='empleados'),
    path('productos/', views.ProductoListView.as_view(), name='productos'),
    path('fabricas/', views.FabricaListView.as_view(), name='fabricas'),
    path('ventas/', views.VentaProductoListView.as_view(), name='ventas'),
]
```

---

## 🎯 `views.py`

```python
from django.shortcuts import render
from django.views.generic import ListView
from .models import Cliente, Sucursal, Empleado, Producto, Fabrica, VentaProducto

def inicio(request):
    return render(request, 'inicio.html')

class ClienteListView(ListView):
    model = Cliente
    template_name = 'clientes.html'

class SucursalListView(ListView):
    model = Sucursal
    template_name = 'sucursales.html'

class EmpleadoListView(ListView):
    model = Empleado
    template_name = 'empleados.html'

class ProductoListView(ListView):
    model = Producto
    template_name = 'productos.html'

class FabricaListView(ListView):
    model = Fabrica
    template_name = 'fabricas.html'

class VentaProductoListView(ListView):
    model = VentaProducto
    template_name = 'ventas.html'
```

---

¿Deseas que te genere las plantillas HTML (`templates`) para cada vista como `clientes.html`, `productos.html`, etc., con Bootstrap integrado?

También puedo preparar el `admin.py`, los formularios (`forms.py`) y los CRUD completos. ¿Te gustaría continuar con eso?

4. ¡Perfecto, Eliseo! Vamos a reestructurar tu proyecto para que el archivo `views.py` use **funciones en lugar de clases**. También actualizaremos `urls.py` para que coincida con esta nueva estructura.

---

### ✅ views.py (con funciones)

```python
from django.shortcuts import render
from .models import Cliente, Sucursal, Empleado, Producto, Fabrica, VentaProducto

def inicio(request):
    return render(request, 'inicio.html')

def listar_clientes(request):
    clientes = Cliente.objects.all()
    return render(request, 'clientes.html', {'clientes': clientes})

def listar_sucursales(request):
    sucursales = Sucursal.objects.all()
    return render(request, 'sucursales.html', {'sucursales': sucursales})

def listar_empleados(request):
    empleados = Empleado.objects.all()
    return render(request, 'empleados.html', {'empleados': empleados})

def listar_productos(request):
    productos = Producto.objects.all()
    return render(request, 'productos.html', {'productos': productos})

def listar_fabricas(request):
    fabricas = Fabrica.objects.all()
    return render(request, 'fabricas.html', {'fabricas': fabricas})

def listar_ventas(request):
    ventas = VentaProducto.objects.select_related('producto', 'sucursal').all()
    return render(request, 'ventas.html', {'ventas': ventas})
```

---

### ✅ urls.py (actualizado para usar funciones)

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.inicio, name='inicio'),
    path('clientes/', views.listar_clientes, name='listar_clientes'),
    path('sucursales/', views.listar_sucursales, name='listar_sucursales'),
    path('empleados/', views.listar_empleados, name='listar_empleados'),
    path('productos/', views.listar_productos, name='listar_productos'),
    path('fabricas/', views.listar_fabricas, name='listar_fabricas'),
    path('ventas/', views.listar_ventas, name='listar_ventas'),
]
```

---

### ✅ Siguiente paso sugerido

¿Deseas que te genere las plantillas HTML correspondientes como `clientes.html`, `productos.html`, etc., con diseño Bootstrap moderno y tabla para los listados?

Puedo hacer eso ahora si lo deseas.
Perfecto, Eliseo. Aquí tienes las plantillas HTML para cada vista: listas Bootstrap limpias, responsivas, con estilo moderno y tabla hover. Todas heredan de una base común (`base.html`) para mantener consistencia visual.

---

### ✅ `templates/base.html`

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Cadena de Deportes</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body class="bg-light">
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark mb-4">
        <div class="container-fluid">
            <a class="navbar-brand" href="{% url 'inicio' %}">Cadena Deportes</a>
            <div class="collapse navbar-collapse">
                <ul class="navbar-nav">
                    <li class="nav-item"><a class="nav-link" href="{% url 'listar_clientes' %}">Clientes</a></li>
                    <li class="nav-item"><a class="nav-link" href="{% url 'listar_empleados' %}">Empleados</a></li>
                    <li class="nav-item"><a class="nav-link" href="{% url 'listar_sucursales' %}">Sucursales</a></li>
                    <li class="nav-item"><a class="nav-link" href="{% url 'listar_productos' %}">Productos</a></li>
                    <li class="nav-item"><a class="nav-link" href="{% url 'listar_fabricas' %}">Fábricas</a></li>
                    <li class="nav-item"><a class="nav-link" href="{% url 'listar_ventas' %}">Ventas</a></li>
                </ul>
            </div>
        </div>
    </nav>
    <div class="container">
        {% block content %}{% endblock %}
    </div>
</body>
</html>
```

---

### ✅ `inicio.html`

```html
{% extends 'base.html' %}
{% block content %}
    <div class="text-center mt-5">
        <h1>Bienvenido a Cadena de Deportes</h1>
        <p class="lead">Sistema de gestión de clientes, empleados, productos y sucursales.</p>
    </div>
{% endblock %}
```

---

### ✅ `clientes.html`

```html
{% extends 'base.html' %}
{% block content %}
<h2>Clientes</h2>
<table class="table table-hover shadow-sm">
    <thead class="table-dark">
        <tr>
            <th>Código</th>
            <th>Nombre</th>
            <th>DNI</th>
            <th>Fecha Nac.</th>
            <th>Ciudad</th>
            <th>Sucursal</th>
        </tr>
    </thead>
    <tbody>
        {% for cliente in clientes %}
        <tr>
            <td>{{ cliente.codigo }}</td>
            <td>{{ cliente.nombre }}</td>
            <td>{{ cliente.dni }}</td>
            <td>{{ cliente.fecha_nac }}</td>
            <td>{{ cliente.ciudad }}</td>
            <td>{{ cliente.sucursal.numero }}</td>
        </tr>
        {% endfor %}
    </tbody>
</table>
{% endblock %}
```

---

### ✅ `empleados.html`

```html
{% extends 'base.html' %}
{% block content %}
<h2>Empleados</h2>
<table class="table table-hover shadow-sm">
    <thead class="table-dark">
        <tr>
            <th>Legajo</th>
            <th>Nombre</th>
            <th>DNI</th>
            <th>Teléfonos</th>
            <th>Dirección</th>
        </tr>
    </thead>
    <tbody>
        {% for empleado in empleados %}
        <tr>
            <td>{{ empleado.legajo }}</td>
            <td>{{ empleado.nombre }}</td>
            <td>{{ empleado.dni }}</td>
            <td>{{ empleado.telefonos }}</td>
            <td>{{ empleado.calle }} {{ empleado.numero }}, {{ empleado.ciudad }}</td>
        </tr>
        {% endfor %}
    </tbody>
</table>
{% endblock %}
```

---

### ✅ `sucursales.html`

```html
{% extends 'base.html' %}
{% block content %}
<h2>Sucursales</h2>
<table class="table table-hover shadow-sm">
    <thead class="table-dark">
        <tr>
            <th>Número</th>
            <th>Domicilio</th>
            <th>Ciudad</th>
            <th>Descuento (%)</th>
        </tr>
    </thead>
    <tbody>
        {% for sucursal in sucursales %}
        <tr>
            <td>{{ sucursal.numero }}</td>
            <td>{{ sucursal.domicilio }}</td>
            <td>{{ sucursal.ciudad }}</td>
            <td>{{ sucursal.descuento }}</td>
        </tr>
        {% endfor %}
    </tbody>
</table>
{% endblock %}
```

---

### ✅ `productos.html`

```html
{% extends 'base.html' %}
{% block content %}
<h2>Productos</h2>
<table class="table table-hover shadow-sm">
    <thead class="table-dark">
        <tr>
            <th>Código</th>
            <th>Descripción</th>
            <th>Color</th>
            <th>Costo</th>
            <th>Fábrica</th>
        </tr>
    </thead>
    <tbody>
        {% for producto in productos %}
        <tr>
            <td>{{ producto.codigo }}</td>
            <td>{{ producto.descripcion }}</td>
            <td>{{ producto.color }}</td>
            <td>{{ producto.costo }}</td>
            <td>{{ producto.fabrica.nombre }}</td>
        </tr>
        {% endfor %}
    </tbody>
</table>
{% endblock %}
```

---

### ✅ `fabricas.html`

```html
{% extends 'base.html' %}
{% block content %}
<h2>Fábricas</h2>
<table class="table table-hover shadow-sm">
    <thead class="table-dark">
        <tr>
            <th>CUIT</th>
            <th>Nombre</th>
            <th>País</th>
            <th>Cantidad Empleados</th>
            <th>Gerente</th>
        </tr>
    </thead>
    <tbody>
        {% for fabrica in fabricas %}
        <tr>
            <td>{{ fabrica.cuit }}</td>
            <td>{{ fabrica.nombre }}</td>
            <td>{{ fabrica.pais }}</td>
            <td>{{ fabrica.cant_emp }}</td>
            <td>{{ fabrica.gerente }}</td>
        </tr>
        {% endfor %}
    </tbody>
</table>
{% endblock %}
```

---

### ✅ `ventas.html`

```html
{% extends 'base.html' %}
{% block content %}
<h2>Ventas por Sucursal</h2>
<table class="table table-hover shadow-sm">
    <thead class="table-dark">
        <tr>
            <th>Producto</th>
            <th>Sucursal</th>
            <th>Precio Venta</th>
        </tr>
    </thead>
    <tbody>
        {% for venta in ventas %}
        <tr>
            <td>{{ venta.producto.descripcion }}</td>
            <td>{{ venta.sucursal.numero }}</td>
            <td>{{ venta.precio_venta }}</td>
        </tr>
        {% endfor %}
    </tbody>
</table>
{% endblock %}
```

---

¿Deseas que estos archivos te los empaquete como proyecto `.zip` o que genere también los formularios para crear, editar y eliminar datos (CRUD completo)?
¡Perfecto, Eliseo! Vamos a implementar el CRUD completo con **formularios** para crear, editar y eliminar datos para el proyecto **Cadena de Deportes**, utilizando funciones basadas en vistas y formularios con Bootstrap.

📁 Incluiremos:

1. `forms.py`
2. Vistas en `views.py` para **crear, editar y eliminar**
3. Nuevas rutas en `urls.py`
4. Plantillas HTML para formularios (`form.html`, `confirm_delete.html`)

---

## ✅ `forms.py`

```python
from django import forms
from .models import Cliente, Empleado, Sucursal, Producto, Fabrica, VentaProducto

class ClienteForm(forms.ModelForm):
    class Meta:
        model = Cliente
        fields = '__all__'

class EmpleadoForm(forms.ModelForm):
    class Meta:
        model = Empleado
        fields = '__all__'

class SucursalForm(forms.ModelForm):
    class Meta:
        model = Sucursal
        fields = '__all__'

class ProductoForm(forms.ModelForm):
    class Meta:
        model = Producto
        fields = '__all__'

class FabricaForm(forms.ModelForm):
    class Meta:
        model = Fabrica
        fields = '__all__'

class VentaProductoForm(forms.ModelForm):
    class Meta:
        model = VentaProducto
        fields = '__all__'
```

---

## ✅ Actualización en `views.py` (ejemplo para Cliente, replicable a otros)

```python
from django.shortcuts import render, get_object_or_404, redirect
from .models import Cliente
from .forms import ClienteForm

# --- CRUD Cliente ---

def crear_cliente(request):
    form = ClienteForm(request.POST or None)
    if form.is_valid():
        form.save()
        return redirect('listar_clientes')
    return render(request, 'form.html', {'form': form, 'titulo': 'Nuevo Cliente'})

def editar_cliente(request, pk):
    cliente = get_object_or_404(Cliente, pk=pk)
    form = ClienteForm(request.POST or None, instance=cliente)
    if form.is_valid():
        form.save()
        return redirect('listar_clientes')
    return render(request, 'form.html', {'form': form, 'titulo': 'Editar Cliente'})

def eliminar_cliente(request, pk):
    cliente = get_object_or_404(Cliente, pk=pk)
    if request.method == "POST":
        cliente.delete()
        return redirect('listar_clientes')
    return render(request, 'confirm_delete.html', {'obj': cliente, 'titulo': 'Eliminar Cliente'})
```

---

## ✅ Actualización `urls.py` (solo para Cliente, similar para los demás)

```python
# Importar vistas
from . import views

urlpatterns = [
    path('', views.inicio, name='inicio'),

    # Clientes
    path('clientes/', views.listar_clientes, name='listar_clientes'),
    path('clientes/nuevo/', views.crear_cliente, name='crear_cliente'),
    path('clientes/editar/<int:pk>/', views.editar_cliente, name='editar_cliente'),
    path('clientes/eliminar/<int:pk>/', views.eliminar_cliente, name='eliminar_cliente'),

    # Repite para empleados, sucursales, productos, fabricas, ventas...
]
```

---

## ✅ Plantilla `form.html`

```html
{% extends 'base.html' %}
{% block content %}
<div class="card shadow">
    <div class="card-header bg-primary text-white">{{ titulo }}</div>
    <div class="card-body">
        <form method="post">
            {% csrf_token %}
            {{ form.as_p }}
            <button type="submit" class="btn btn-success">Guardar</button>
            <a href="{% url 'inicio' %}" class="btn btn-secondary">Cancelar</a>
        </form>
    </div>
</div>
{% endblock %}
```

---

## ✅ Plantilla `confirm_delete.html`

```html
{% extends 'base.html' %}
{% block content %}
<div class="alert alert-danger mt-4 shadow-sm">
    <h4>{{ titulo }}</h4>
    <p>¿Estás seguro que deseas eliminar <strong>{{ obj }}</strong>?</p>
    <form method="post">
        {% csrf_token %}
        <button type="submit" class="btn btn-danger">Sí, eliminar</button>
        <a href="{% url 'listar_clientes' %}" class="btn btn-secondary">Cancelar</a>
    </form>
</div>
{% endblock %}
```

---

## ✅ Actualizar tabla de `clientes.html` para botones

```html
...
<td>
    <a href="{% url 'editar_cliente' cliente.pk %}" class="btn btn-sm btn-warning">Editar</a>
    <a href="{% url 'eliminar_cliente' cliente.pk %}" class="btn btn-sm btn-danger">Eliminar</a>
</td>
...
```

---

## 🔁 ¿Qué sigue?

Puedo generar automáticamente el CRUD para el resto de modelos (`Empleado`, `Sucursal`, `Producto`, etc.) siguiendo la misma estructura. ¿Te gustaría que genere todo el CRUD completo para cada uno?

También puedo empaquetarte el proyecto base en `.zip` listo para correr. ¿Deseas eso también?
