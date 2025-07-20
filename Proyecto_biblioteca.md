1. <img width="1022" height="605" alt="image" src="https://github.com/user-attachments/assets/6209f1cd-f4e9-4b81-a8f6-56aba4096095" />
2. ### Implementación de los modelos en Django basados en el diagrama de clases proporcionado

A continuación, se presenta la implementación de los modelos en `models.py` para el sistema bibliotecario descrito en el diagrama. Se han agregado 5 campos adicionales para las clases `Estudiante` y `Docente`, como se solicitó.

#### Análisis del Diagrama
1. **Lector**:
   - Atributos: `Nombre`, `Dirección`, `Teléfono`
   - Métodos: `Reservar(material)`, `Entregar(material)`
   - Relaciones:
     - Muchos a uno con `Pedido` (un lector puede hacer muchos pedidos).
     - Uno a uno con `Estudiante` o `Docente`.

2. **Pedido**:
   - Atributos: `ID del usuario`, `Título del material`, `Código del material`, `Fecha de reserva`, `Fecha de entrega`.
   - Métodos: `Reservar(fecha)`, `Entregar(fecha)`
   - Relaciones:
     - Uno a uno con `Libro` o `Revista` (un pedido necesita un libro o una revista).
     - Muchos a uno con `Bibliotecario` (un bibliotecario puede manejar muchos pedidos).

3. **Bibliotecario**:
   - Atributos: `ID del personal`
   - Métodos: `Reservar(fecha)`, `Entregar(fecha)`
   - Relaciones:
     - Uno a muchos con `Pedido` (un bibliotecario maneja muchos pedidos).

4. **Libro**:
   - Atributos: `Título del libro`, `Tipo de libro`, `Autor`, `Editorial`
   - Métodos: `Reservar(fecha)`, `Entregar(fecha)`
   - Relaciones:
     - Uno a uno con `Pedido` (un pedido necesita un libro).

5. **Revista**:
   - Atributos: `Título de la revista`, `Tipo de revista`, `Autor`, `Edición`
   - Métodos: `Reservar(fecha)`, `Entregar(fecha)`
   - Relaciones:
     - Uno a uno con `Pedido` (un pedido necesita una revista).

6. **Estudiante**:
   - Atributos: `Código del estudiante`
   - Métodos: `Reservar(libro)`, `Entregar(libro)`
   - Relaciones:
     - Herencia de `Lector` (es un tipo de lector).
     - Agregar 5 campos adicionales.

7. **Docente**:
   - Atributos: `Código del docente`
   - Métodos: `Reservar(libro)`, `Entregar(libro)`
   - Relaciones:
     - Herencia de `Lector` (es un tipo de lector).
     - Agregar 5 campos adicionales.

---

### Implementación en Django

Aquí está la implementación de los modelos en `models.py`:

```python
from django.db import models
from django.utils import timezone

# Clase abstracta Lector
class Lector(models.Model):
    nombre = models.CharField(max_length=100)
    direccion = models.CharField(max_length=200)
    telefono = models.CharField(max_length=20)

    class Meta:
        abstract = True

    def __str__(self):
        return self.nombre


# Clase Estudiante (hereda de Lector)
class Estudiante(Lector):
    codigo_estudiante = models.CharField(max_length=20, unique=True)
    carrera = models.CharField(max_length=100)
    año_ingreso = models.PositiveIntegerField()
    email = models.EmailField()
    fecha_nacimiento = models.DateField()

    def __str__(self):
        return f"Estudiante: {self.nombre} (Código: {self.codigo_estudiante})"


# Clase Docente (hereda de Lector)
class Docente(Lector):
    codigo_docente = models.CharField(max_length=20, unique=True)
    departamento = models.CharField(max_length=100)
    titulo_academico = models.CharField(max_length=100)
    fecha_contratacion = models.DateField()
    especialidad = models.CharField(max_length=100)

    def __str__(self):
        return f"Docente: {self.nombre} (Código: {self.codigo_docente})"


# Clase Bibliotecario
class Bibliotecario(models.Model):
    id_personal = models.CharField(max_length=20, unique=True)
    nombre = models.CharField(max_length=100)
    direccion = models.CharField(max_length=200)
    telefono = models.CharField(max_length=20)

    def __str__(self):
        return f"Bibliotecario: {self.nombre} (ID: {self.id_personal})"


# Clase Libro
class Libro(models.Model):
    titulo = models.CharField(max_length=200)
    tipo_libro = models.CharField(max_length=50)
    autor = models.CharField(max_length=100)
    editorial = models.CharField(max_length=100)
    fecha_reserva = models.DateField(null=True, blank=True)
    fecha_entrega = models.DateField(null=True, blank=True)

    def __str__(self):
        return f"Libro: {self.titulo}"


# Clase Revista
class Revista(models.Model):
    titulo = models.CharField(max_length=200)
    tipo_revista = models.CharField(max_length=50)
    autor = models.CharField(max_length=100)
    edicion = models.CharField(max_length=50)
    fecha_reserva = models.DateField(null=True, blank=True)
    fecha_entrega = models.DateField(null=True, blank=True)

    def __str__(self):
        return f"Revista: {self.titulo}"


# Clase Pedido
class Pedido(models.Model):
    ID_usuario = models.ForeignKey(Lector, on_delete=models.CASCADE, related_name='pedidos')
    material = models.CharField(max_length=100)  # Puede ser 'libro' o 'revista'
    codigo_material = models.CharField(max_length=50)
    fecha_reserva = models.DateField(default=timezone.now)
    fecha_entrega = models.DateField(null=True, blank=True)
    bibliotecario = models.ForeignKey(Bibliotecario, on_delete=models.CASCADE, related_name='pedidos')

    def __str__(self):
        return f"Pedido de {self.ID_usuario.nombre} (Material: {self.material})"
```

---

### Explicación de la Implementación

1. **Clase `Lector`**:
   - Es una clase abstracta que define los atributos comunes para `Estudiante` y `Docente`: `nombre`, `direccion`, y `telefono`.
   - Las clases `Estudiante` y `Docente` heredan de `Lector`.

2. **Clase `Estudiante`**:
   - Hereda de `Lector` y agrega 5 campos adicionales:
     - `codigo_estudiante`: Código único del estudiante.
     - `carrera`: Carrera del estudiante.
     - `año_ingreso`: Año de ingreso al programa académico.
     - `email`: Correo electrónico del estudiante.
     - `fecha_nacimiento`: Fecha de nacimiento del estudiante.

3. **Clase `Docente`**:
   - Hereda de `Lector` y agrega 5 campos adicionales:
     - `codigo_docente`: Código único del docente.
     - `departamento`: Departamento al que pertenece el docente.
     - `titulo_academico`: Título académico del docente.
     - `fecha_contratacion`: Fecha de contratación del docente.
     - `especialidad`: Especialidad del docente.

4. **Clase `Bibliotecario`**:
   - Define los atributos básicos de un bibliotecario: `id_personal`, `nombre`, `direccion`, y `telefono`.

5. **Clase `Libro`**:
   - Define los atributos de un libro: `titulo`, `tipo_libro`, `autor`, `editorial`, `fecha_reserva`, y `fecha_entrega`.

6. **Clase `Revista`**:
   - Define los atributos de una revista: `titulo`, `tipo_revista`, `autor`, `edicion`, `fecha_reserva`, y `fecha_entrega`.

7. **Clase `Pedido`**:
   - Representa un pedido realizado por un lector (`Estudiante` o `Docente`).
   - Relaciones:
     - `ID_usuario`: FK a `Lector` (puede ser `Estudiante` o `Docente`).
     - `bibliotecario`: FK a `Bibliotecario`.
     - `material`: Indica si el pedido es para un libro o una revista.
     - `codigo_material`: Código único del material reservado.

---

### Relaciones entre Modelos

1. **Relación entre `Lector`, `Estudiante`, y `Docente`**:
   - `Estudiante` y `Docente` heredan de `Lector`. Esto permite que ambos sean considerados lectores.

2. **Relación entre `Pedido` y `Lector`**:
   - Un `Pedido` está asociado a un `Lector` (ya sea `Estudiante` o `Docente`) mediante la clave foránea `ID_usuario`.

3. **Relación entre `Pedido` y `Bibliotecario`**:
   - Un `Pedido` está asociado a un `Bibliotecario` mediante la clave foránea `bibliotecario`.

4. **Relación entre `Pedido` y `Libro`/`Revista`**:
   - El campo `material` indica si el pedido es para un libro o una revista. Los detalles del material se manejan mediante los campos `codigo_material`, `fecha_reserva`, y `fecha_entrega`.

---

### Pasos Adicionales

1. **Migraciones**:
   Ejecuta los siguientes comandos para crear las migraciones y aplicarlas a la base de datos:
   ```bash
   python manage.py makemigrations
   python manage.py migrate
   ```

2. **Panel de Administración**:
   Registra los modelos en `admin.py` para poder gestionarlos desde el panel de administración de Django:
   ```python
   from django.contrib import admin
   from .models import Estudiante, Docente, Bibliotecario, Libro, Revista, Pedido

   admin.site.register(Estudiante)
   admin.site.register(Docente)
   admin.site.register(Bibliotecario)
   admin.site.register(Libro)
   admin.site.register(Revista)
   admin.site.register(Pedido)
   ```

Con esta implementación, tendrás un sistema bibliotecario funcional en Django que refleja exactamente el diagrama de clases proporcionado.
