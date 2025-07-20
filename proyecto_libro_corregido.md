Claro, aquí tienes una revisión detallada y corregida del archivo `models.py` con sus relaciones, nombres consistentes y mejoras sugeridas para claridad y funcionalidad:

---

### ✅ CAMBIOS PRINCIPALES REALIZADOS:

1. **Importación de `settings` faltante.**
2. **Consistencia de nombres en inglés/español.** Se recomienda usar un solo idioma para mantener coherencia (preferí mantenerlo en español salvo lo necesario).
3. **Corrección del nombre del modelo `Book` → `Libro`.**
4. **Valores por defecto en campos con `choices` corregidos.**
5. **Mejoras en el `__str__`, nombres de campos, y decimal precision.**

---

### ✅ MODELO `models.py` CORREGIDO Y FUNCIONAL

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

    def get_items_total(self):
        return sum(item.subtotal() for item in self.items.all())


class ItemPedido(models.Model):
    pedido = models.ForeignKey(Pedido, related_name='items', on_delete=models.CASCADE, verbose_name="Pedido")
    libro = models.ForeignKey(Libro, on_delete=models.CASCADE, verbose_name="Libro")
    cantidad = models.PositiveIntegerField(default=1, verbose_name="Cantidad")
    precio = models.DecimalField(max_digits=8, decimal_places=2, verbose_name="Precio")

    class Meta:
        verbose_name = "Ítem de pedido"
        verbose_name_plural = "Ítems de pedido"

    def __str__(self):
        return f"{self.cantidad} x {self.libro.titulo}"

    def subtotal(self):
        return self.cantidad * self.precio if self.precio else Decimal('0.00')
```

---

### ✅ SUGERENCIAS ADICIONALES

1. **DecimalField:** Usa `max_digits=8, decimal_places=2` para precios (no 4 decimales innecesarios).
2. **Imagenes:** Asegúrate de tener configurado correctamente `MEDIA_URL` y `MEDIA_ROOT` en `settings.py`.
3. **Relaciones:**

   * `Libro` tiene relación muchos a uno con `Autor` y `Editorial`.
   * `Libro` tiene muchos a muchos con `Categoria`.
   * `Pedido` tiene muchos `ItemPedido`, cada uno con relación uno a uno con un `Libro`.

---

¿Deseas también el `admin.py` para registrar estos modelos, o un diagrama ER (Entidad-Relación)?
