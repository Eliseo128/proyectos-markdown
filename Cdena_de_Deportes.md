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

