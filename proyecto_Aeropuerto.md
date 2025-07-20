Aquí tienes el archivo `models.py` con **todos los nombres de las clases y campos traducidos al español**, incluyendo las relaciones entre ellos. Se ha mantenido la funcionalidad intacta, pero ahora los nombres son más comprensibles para hablantes hispanohablantes.

---

### ✅ `models.py` en español

```python
from django.db import models


class Aeropuerto(models.Model):
    codigo = models.CharField(max_length=3, verbose_name="Código")
    ciudad = models.CharField(max_length=64, verbose_name="Ciudad")

    class Meta:
        verbose_name = "Aeropuerto"
        verbose_name_plural = "Aeropuertos"

    def __str__(self):
        return f"{self.ciudad} ({self.codigo})"


class Vuelo(models.Model):
    origen = models.ForeignKey(Aeropuerto, on_delete=models.CASCADE, related_name="salidas", verbose_name="Origen")
    destino = models.ForeignKey(Aeropuerto, on_delete=models.CASCADE, related_name="llegadas", verbose_name="Destino")
    duracion = models.IntegerField(verbose_name="Duración (minutos)")

    class Meta:
        verbose_name = "Vuelo"
        verbose_name_plural = "Vuelos"

    def __str__(self):
        return f"{self.id}: {self.origen} a {self.destino}"


class Pasajero(models.Model):
    nombre = models.CharField(max_length=64, verbose_name="Nombre")
    apellido = models.CharField(max_length=64, verbose_name="Apellido")
    vuelos = models.ManyToManyField(Vuelo, blank=True, related_name="pasajeros", verbose_name="Vuelos")

    class Meta:
        verbose_name = "Pasajero"
        verbose_name_plural = "Pasajeros"

    def __str__(self):
        return f"{self.nombre} {self.apellido}"
```

---

### 📌 Detalles de la traducción:

| Original (inglés)   | Traducido (español)   |
|---------------------|------------------------|
| Airport             | Aeropuerto             |
| code                | codigo                 |
| city                | ciudad                 |
| Flight              | Vuelo                  |
| origin              | origen                 |
| destination         | destino                |
| duration            | duracion               |
| Passenger           | Pasajero               |
| first (name)        | nombre                 |
| last (name)         | apellido               |
| flights             | vuelos                 |
| related_name: departures → salidas, arrivals → llegadas, passengers → pasajeros |

---

### ✅ ¿Qué más puedes hacer?

Si planeas usar esto en el panel de administración de Django, esta traducción ya se mostrará correctamente gracias a los `verbose_name` y a la configuración de `Meta`.

Si además quieres **soportar múltiples idiomas** en tu aplicación (i18n), puedo ayudarte a configurar Django para que cambie automáticamente el idioma de la interfaz según el usuario.

¿Te gustaría agregar esa funcionalidad también?
