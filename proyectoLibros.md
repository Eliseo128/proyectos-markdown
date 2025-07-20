Aquí tienes el archivo `models.py` con **todos los nombres de las clases y campos traducidos al español**, manteniendo la funcionalidad intacta:

```python
from django.db import models


class Editorial(models.Model):
    nombre = models.CharField(max_length=50)
    direccion = models.TextField()
    correo_electronico = models.EmailField()

    def __str__(self):
        return self.nombre


class Autor(models.Model):
    nombre = models.CharField(max_length=100)
    biografia = models.TextField()
    imagen = models.ImageField(upload_to='autor/', blank=True, null=True)
    nacionalidad = models.CharField(max_length=40)
    fecha_nacimiento = models.DateField()

    def __str__(self):
        return self.nombre


class Categoria(models.Model):
    nombre = models.CharField(max_length=100)
    descripcion = models.TextField()

    def __str__(self):
        return self.nombre


class Libro(models.Model):
    titulo = models.CharField(max_length=200)
    precio = models.DecimalField(max_digits=8, decimal_places=4)
    descripcion = models.TextField(blank=True)
    stock = models.PositiveIntegerField(default=0)  # Inventario disponible
    imagen_portada = models.ImageField(upload_to='portadas_libros/', blank=True, null=True)
    fecha_publicacion = models.DateField(null=True, blank=True)
    autor = models.ForeignKey(Autor, on_delete=models.CASCADE)
    editorial = models.ForeignKey(Editorial, on_delete=models.CASCADE)
    categorias = models.ManyToManyField(Categoria, related_name='libros')

    def __str__(self):
        return self.titulo
```

### Notas:
- Todos los nombres de las clases (`Publisher` → `Editorial`, `Author` → `Autor`, etc.) han sido traducidos.
- Los campos también han sido traducidos al español (`name` → `nombre`, `address` → `direccion`, etc.).
- Se han mantenido las mismas funcionalidades, tipos de campos y relaciones.
- Las rutas de carga de imágenes (`upload_to`) también han sido adaptadas al español.

Si necesitas también traducir las etiquetas (`verbose_name`, `verbose_name_plural`) para que aparezcan en español en la interfaz de administración de Django, puedo ayudarte a agregarlas. ¿Te gustaría incluir eso también?
Perfecto, aquí tienes el archivo `models.py` **completamente traducido al español**, incluyendo:

- Nombres de clases y campos.
- Etiquetas descriptivas (`verbose_name` y `verbose_name_plural`) para que aparezcan en la interfaz de administración de Django en español.
- Comentarios también traducidos (opcionalmente).

```python
from django.db import models


class Editorial(models.Model):
    nombre = models.CharField("Nombre", max_length=50)
    direccion = models.TextField("Dirección")
    correo_electronico = models.EmailField("Correo electrónico")

    class Meta:
        verbose_name = "Editorial"
        verbose_name_plural = "Editoriales"

    def __str__(self):
        return self.nombre


class Autor(models.Model):
    nombre = models.CharField("Nombre", max_length=100)
    biografia = models.TextField("Biografía")
    imagen = models.ImageField("Imagen", upload_to='autor/', blank=True, null=True)
    nacionalidad = models.CharField("Nacionalidad", max_length=40)
    fecha_nacimiento = models.DateField("Fecha de nacimiento")

    class Meta:
        verbose_name = "Autor"
        verbose_name_plural = "Autores"

    def __str__(self):
        return self.nombre


class Categoria(models.Model):
    nombre = models.CharField("Nombre", max_length=100)
    descripcion = models.TextField("Descripción")

    class Meta:
        verbose_name = "Categoría"
        verbose_name_plural = "Categorías"

    def __str__(self):
        return self.nombre


class Libro(models.Model):
    titulo = models.CharField("Título", max_length=200)
    precio = models.DecimalField("Precio", max_digits=8, decimal_places=4)
    descripcion = models.TextField("Descripción", blank=True)
    stock = models.PositiveIntegerField("Stock", default=0)  # Inventario disponible
    imagen_portada = models.ImageField("Imagen de portada", upload_to='portadas_libros/', blank=True, null=True)
    fecha_publicacion = models.DateField("Fecha de publicación", null=True, blank=True)
    autor = models.ForeignKey(Autor, on_delete=models.CASCADE)
    editorial = models.ForeignKey(Editorial, on_delete=models.CASCADE)
    categorias = models.ManyToManyField(Categoria, related_name='libros', verbose_name="Categorías")

    class Meta:
        verbose_name = "Libro"
        verbose_name_plural = "Libros"

    def __str__(self):
        return self.titulo
```

Claro, aquí tienes el archivo `models.py` **traducido completamente al español**, incluyendo:

- Nombres de clases y campos.
- Opciones de campos (`choices`) en español.
- Comentarios y documentación (`docstrings`).
- El campo `verbose_name` también se traduce.

> **Nota**: Se asume que el modelo `Book` está en una aplicación llamada `books`, por lo que mantendré la importación como está, pero si también necesitas traducir esa parte, puedo ayudarte.

---

### ✅ `models.py` traducido al español

```python
from django.db import models
from django.conf import settings
from books.models import Book
from decimal import Decimal


class Pedido(models.Model):
    ESTADO_CHOICES = [
        ('pending', 'Pendiente'),
        ('processing', 'Procesando'),
        ('shipping', 'Enviando'),
        ('completed', 'Completado'),
        ('cancelled', 'Cancelado'),
    ]

    METODO_PAGO_CHOICES = [
        ('cash', 'Efectivo'),
        ('bank_transfer', 'Transferencia bancaria'),
        ('card', 'Tarjeta de crédito'),
        ('momo', 'Billetera MoMo'),
    ]

    cliente = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE, verbose_name="Cliente")
    creado_en = models.DateTimeField(auto_now_add=True, verbose_name="Fecha de creación")
    estado = models.CharField(max_length=20, choices=ESTADO_CHOICES, default='pending', verbose_name="Estado")
    nota = models.TextField(blank=True, verbose_name="Nota")
    metodo_pago = models.CharField(max_length=20, choices=METODO_PAGO_CHOICES, default='cash', verbose_name="Método de pago")
    monto_total = models.DecimalField(max_digits=10, decimal_places=2, default=0, verbose_name="Monto total")

    class Meta:
        verbose_name = "Pedido"
        verbose_name_plural = "Pedidos"

    def __str__(self):
        return f"Pedido #{self.pk} por {self.cliente.username if self.cliente else 'Invitado'}"

    def get_items_total(self):
        """Calcula el total de los artículos (sin incluir descuentos)"""
        return sum(item.subtotal() for item in self.items.all())


class ItemPedido(models.Model):
    pedido = models.ForeignKey(Pedido, related_name='items', on_delete=models.CASCADE, verbose_name="Pedido")
    libro = models.ForeignKey(Book, on_delete=models.CASCADE, verbose_name="Libro")
    cantidad = models.PositiveIntegerField(default=1, verbose_name="Cantidad")
    precio = models.DecimalField(max_digits=6, decimal_places=2, verbose_name="Precio")  # Precio en el momento de la compra

    class Meta:
        verbose_name = "Ítem de pedido"
        verbose_name_plural = "Ítems de pedido"

    def __str__(self):
        return f"{self.cantidad} x {self.libro.title}"

    def subtotal(self):
        """Calcula el subtotal del ítem"""
        if self.precio is None:
            return Decimal('0.00')  # Devuelve 0 si el precio es None
        return self.cantidad * self.precio
```




