1. <img width="782" height="565" alt="image" src="https://github.com/user-attachments/assets/3add5bdb-1c61-4935-9175-bbfb89f881e3" />
Para crear las clases en Django basadas en el diagrama de la base de datos proporcionado, necesitamos traducir cada tabla del diagrama en un modelo de Django. Los modelos se definirán en el archivo `models.py` de la aplicación correspondiente.

### Análisis del Diagrama
1. **tNacionalidad**:
   - Atributos: `idNacionalidad`, `pais`, `nacionalidad`
   - PK: `idNacionalidad`

2. **tHabitacion**:
   - Atributos: `idHabitacion`, `numero`, `estado`, `costo`, `descripcion`, `fkTipo`
   - PK: `idHabitacion`
   - FK: `fkTipo` (referencia a `tTipoHabitacion`)

3. **tTipoHabitacion**:
   - Atributos: `idTipo`, `nombre`, `descripcion`
   - PK: `idTipo`

4. **tEstado**:
   - Atributos: `idEstado`, `nombre`
   - PK: `idEstado`

5. **tCliente**:
   - Atributos: `idCliente`, `nombre`, `direccion`, `documento`, `telefono`, `fkNacionalidad`
   - PK: `idCliente`
   - FK: `fkNacionalidad` (referencia a `tNacionalidad`)

6. **tAlquiler**:
   - Atributos: `idAlquiler`, `fechaHoraEntrada`, `fechaHoraSalida`, `costoTotal`, `observacion`, `fkHabitacion`, `fkCliente`, `fkRegistrador`, `fkEstado`
   - PK: `idAlquiler`
   - FK: 
     - `fkHabitacion` (referencia a `tHabitacion`)
     - `fkCliente` (referencia a `tCliente`)
     - `fkRegistrador` (referencia a `tRegistrador`)
     - `fkEstado` (referencia a `tEstado`)

7. **tRegistrador**:
   - Atributos: `idRegistrador`, `nombre`, `direccion`, `documento`, `telefono`, `estado`, `observacion`
   - PK: `idRegistrador`

### Implementación en Django

A continuación, se presenta la implementación de los modelos en `models.py`:

```python
from django.db import models

# Modelo para tNacionalidad
class Nacionalidad(models.Model):
    idNacionalidad = models.AutoField(primary_key=True)
    pais = models.CharField(max_length=100)
    nacionalidad = models.CharField(max_length=100)

    def __str__(self):
        return f"{self.nacionalidad} ({self.pais})"


# Modelo para tTipoHabitacion
class TipoHabitacion(models.Model):
    idTipo = models.AutoField(primary_key=True)
    nombre = models.CharField(max_length=100)
    descripcion = models.TextField()

    def __str__(self):
        return self.nombre


# Modelo para tHabitacion
class Habitacion(models.Model):
    idHabitacion = models.AutoField(primary_key=True)
    numero = models.CharField(max_length=10)
    estado = models.CharField(max_length=50)  # Puede ser 'disponible', 'ocupada', etc.
    costo = models.DecimalField(max_digits=10, decimal_places=2)
    descripcion = models.TextField()
    fkTipo = models.ForeignKey(TipoHabitacion, on_delete=models.CASCADE, related_name='habitaciones')

    def __str__(self):
        return f"Habitación {self.numero}"


# Modelo para tEstado
class Estado(models.Model):
    idEstado = models.AutoField(primary_key=True)
    nombre = models.CharField(max_length=100)

    def __str__(self):
        return self.nombre


# Modelo para tCliente
class Cliente(models.Model):
    idCliente = models.AutoField(primary_key=True)
    nombre = models.CharField(max_length=200)
    direccion = models.CharField(max_length=200)
    documento = models.CharField(max_length=50)
    telefono = models.CharField(max_length=20)
    fkNacionalidad = models.ForeignKey(Nacionalidad, on_delete=models.CASCADE, related_name='clientes')

    def __str__(self):
        return self.nombre


# Modelo para tRegistrador
class Registrador(models.Model):
    idRegistrador = models.AutoField(primary_key=True)
    nombre = models.CharField(max_length=200)
    direccion = models.CharField(max_length=200)
    documento = models.CharField(max_length=50)
    telefono = models.CharField(max_length=20)
    estado = models.CharField(max_length=50)
    observacion = models.TextField(blank=True, null=True)

    def __str__(self):
        return self.nombre


# Modelo para tAlquiler
class Alquiler(models.Model):
    idAlquiler = models.AutoField(primary_key=True)
    fechaHoraEntrada = models.DateTimeField()
    fechaHoraSalida = models.DateTimeField()
    costoTotal = models.DecimalField(max_digits=10, decimal_places=2)
    observacion = models.TextField(blank=True, null=True)
    fkHabitacion = models.ForeignKey(Habitacion, on_delete=models.CASCADE, related_name='alquileres')
    fkCliente = models.ForeignKey(Cliente, on_delete=models.CASCADE, related_name='alquileres')
    fkRegistrador = models.ForeignKey(Registrador, on_delete=models.CASCADE, related_name='alquileres')
    fkEstado = models.ForeignKey(Estado, on_delete=models.CASCADE, related_name='alquileres')

    def __str__(self):
        return f"Alquiler #{self.idAlquiler} - Cliente: {self.fkCliente.nombre}"
```

### Explicación de la Implementación

1. **Claves Primarias (`primary_key=True`)**:
   - Cada modelo tiene un campo autoincremental como clave primaria (`AutoField`).

2. **Relaciones (`ForeignKey`)**:
   - Las relaciones entre tablas se representan usando el campo `ForeignKey`. Por ejemplo:
     - `Habitacion.fkTipo` referencia a `TipoHabitacion`.
     - `Cliente.fkNacionalidad` referencia a `Nacionalidad`.
     - `Alquiler.fkHabitacion`, `Alquiler.fkCliente`, `Alquiler.fkRegistrador`, y `Alquiler.fkEstado` tienen referencias a sus respectivos modelos.

3. **On Delete (`on_delete=models.CASCADE`)**:
   - Se utiliza `CASCADE` para que cuando se elimine un registro padre, también se eliminen los registros hijos relacionados.

4. **Related Name**:
   - El parámetro `related_name` permite acceder a los objetos relacionados desde el modelo padre. Por ejemplo:
     - `TipoHabitacion.habitaciones` para acceder a todas las habitaciones de un tipo específico.
     - `Cliente.alquileres` para acceder a todos los alquileres de un cliente.

5. **Método `__str__`**:
   - Se define el método `__str__` para proporcionar una representación legible de cada objeto en el panel de administración de Django.

### Resumen de Relaciones
- **tNacionalidad**:
  - Es referenciada por `tCliente` mediante `fkNacionalidad`.

- **tTipoHabitacion**:
  - Es referenciada por `tHabitacion` mediante `fkTipo`.

- **tHabitacion**:
  - Referencia a `tTipoHabitacion` mediante `fkTipo`.
  - Es referenciada por `tAlquiler` mediante `fkHabitacion`.

- **tEstado**:
  - Es referenciada por `tAlquiler` mediante `fkEstado`.

- **tCliente**:
  - Referencia a `tNacionalidad` mediante `fkNacionalidad`.
  - Es referenciada por `tAlquiler` mediante `fkCliente`.

- **tRegistrador**:
  - Es referenciada por `tAlquiler` mediante `fkRegistrador`.

- **tAlquiler**:
  - Referencia a `tHabitacion`, `tCliente`, `tRegistrador`, y `tEstado` mediante `fkHabitacion`, `fkCliente`, `fkRegistrador`, y `fkEstado`, respectivamente.

### Pasos Adicionales
1. Después de definir los modelos, ejecuta los siguientes comandos para migrarlos a la base de datos:
   ```bash
   python manage.py makemigrations
   python manage.py migrate
   ```

2. Si deseas usar el panel de administración de Django, regístralos en `admin.py`:
   ```python
   from django.contrib import admin
   from .models import Nacionalidad, TipoHabitacion, Habitacion, Estado, Cliente, Registrador, Alquiler

   admin.site.register(Nacionalidad)
   admin.site.register(TipoHabitacion)
   admin.site.register(Habitacion)
   admin.site.register(Estado)
   admin.site.register(Cliente)
   admin.site.register(Registrador)
   admin.site.register(Alquiler)
   ```

Con esta implementación, tendrás una estructura de modelos en Django que refleja exactamente el diagrama de base de datos proporcionado.
