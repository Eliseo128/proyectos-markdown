¡Perfecto! Sí, puedes cambiar `ItemPedido` por `ArticuloPedido`, y los nombres `item` por `articulo`, `items` por `articulos`, sin ningún problema. Solo hay que hacerlo de manera coherente y uniforme en todo el modelo.

Aquí tienes **todo el archivo `models.py` corregido**, con esos cambios aplicados:

---

### ✅ `models.py` COMPLETO Y CORREGIDO CON NOMBRES EN ESPAÑOL

```python
from django.db import models
from django.conf import settings
from decimal import Decimal


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
    precio = models.DecimalField(max_digits=8, decimal_places=2)
    descripcion = models.TextField(blank=True)
    stock = models.PositiveIntegerField(default=0)
    imagen_portada = models.ImageField(upload_to='portadas_libros/', blank=True, null=True)
    fecha_publicacion = models.DateField(null=True, blank=True)
    autor = models.ForeignKey(Autor, on_delete=models.CASCADE)
    editorial = models.ForeignKey(Editorial, on_delete=models.CASCADE)
    categorias = models.ManyToManyField(Categoria, related_name='libros')

    def __str__(self):
        return self.titulo


class Pedido(models.Model):
    ESTADO_CHOICES = [
        ('pendiente', 'Pendiente'),
        ('procesando', 'Procesando'),
        ('enviando', 'Enviando'),
        ('completado', 'Completado'),
        ('cancelado', 'Cancelado'),
    ]

    METODO_PAGO_CHOICES = [
        ('efectivo', 'Efectivo'),
        ('transferencia_bancaria', 'Transferencia bancaria'),
        ('tarjeta', 'Tarjeta de crédito'),
    ]

    cliente = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE, verbose_name="Cliente")
    creado_en = models.DateTimeField(auto_now_add=True, verbose_name="Fecha de creación")
    estado = models.CharField(max_length=20, choices=ESTADO_CHOICES, default='pendiente', verbose_name="Estado")
    nota = models.TextField(blank=True, verbose_name="Nota")
    metodo_pago = models.CharField(max_length=30, choices=METODO_PAGO_CHOICES, default='efectivo', verbose_name="Método de pago")
    monto_total = models.DecimalField(max_digits=10, decimal_places=2, default=0, verbose_name="Monto total")

    class Meta:
        verbose_name = "Pedido"
        verbose_name_plural = "Pedidos"

    def __str__(self):
        return f"Pedido #{self.pk} por {self.cliente.username if self.cliente else 'Invitado'}"

    def get_articulos_total(self):
        """Calcula el total de los artículos en el pedido."""
        return sum(articulo.subtotal() for articulo in self.articulos.all())


class ArticuloPedido(models.Model):
    pedido = models.ForeignKey(Pedido, related_name='articulos', on_delete=models.CASCADE, verbose_name="Pedido")
    libro = models.ForeignKey(Libro, on_delete=models.CASCADE, verbose_name="Libro")
    cantidad = models.PositiveIntegerField(default=1, verbose_name="Cantidad")
    precio = models.DecimalField(max_digits=8, decimal_places=2, verbose_name="Precio")

    class Meta:
        verbose_name = "Artículo de pedido"
        verbose_name_plural = "Artículos de pedido"

    def __str__(self):
        return f"{self.cantidad} x {self.libro.titulo}"

    def subtotal(self):
        return self.cantidad * self.precio if self.precio else Decimal('0.00')
```

---

### ✅ ¿Qué cambió?

| Antes                       | Ahora                      |
| --------------------------- | -------------------------- |
| `ItemPedido`                | `ArticuloPedido`           |
| `item` en funciones y loops | `articulo`                 |
| `related_name='items'`      | `related_name='articulos'` |
| `get_items_total()`         | `get_articulos_total()`    |

---

¿Deseas también que te genere el `admin.py` para registrar todo con nombres personalizados? ¿O una imagen del diagrama ER ya actualizado?
