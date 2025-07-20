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

### Beneficios de esta versión:
- **Interfaz de admin en español**: Al incluir `verbose_name` y `verbose_name_plural`, el panel de administración mostrará los nombres de los modelos y campos en español.
- **Mejor usabilidad para hispanohablantes**: Facilita la lectura y mantenimiento del código por parte de desarrolladores hispanohablantes.

Si también deseas que te genere la traducción de los mensajes de error o formularios, puedo ayudarte a configurar la internacionalización (i18n) de Django para que la aplicación soporte múltiples idiomas. ¿Te interesa hacer eso también?
