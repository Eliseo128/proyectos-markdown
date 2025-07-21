1. me ayudas a crear el modelo models.py  python django de acuerdo a lo siguiente: Atención médica: registros médicos electrónicos

Las organizaciones de atención médica, como hospitales, clínicas y compañías de seguros, dependen de registros médicos electrónicos (EMR) precisos y seguros para brindar una atención eficiente al paciente y agilizar las operaciones. Las bases de datos relacionales son la base de los sistemas EMR y permiten a los proveedores de atención médica almacenar, administrar y analizar una gran cantidad de datos de pacientes. Los sistemas EMR pueden almacenar una amplia gama de información del paciente, como diagnósticos, medicamentos, resultados de laboratorio, imágenes y planes de tratamiento. Estos datos deben estar organizados y ser fácilmente accesibles para respaldar una atención eficaz al paciente y el cumplimiento normativo. Una base de datos relacional de atención médica típica podría incluir tablas como:

Pacientes: ID del paciente (clave principal), nombre, fecha de nacimiento, dirección, número de teléfono y otros datos demográficos.
Diagnósticos: ID de diagnóstico (clave principal), ID de paciente (clave externa), condición médica y fecha
Medicamentos: ID del medicamento (clave principal), ID del paciente (clave externa), nombre del medicamento, dosis e instrucciones de administración.
Resultados de laboratorio: ID del resultado (clave principal), ID del paciente (clave externa), prueba, valor y fecha
Planes de tratamiento: ID del plan de tratamiento (clave principal), ID del paciente (clave externa), ID del diagnóstico (clave externa), tratamiento recomendado y fecha de inicio.

Los proveedores de atención pueden utilizar las relaciones entre estas tablas para crear registros completos de pacientes, coordinar la atención y evaluar los resultados del tratamiento. Además, las bases de datos relacionales ayudan a garantizar la integridad y seguridad de los datos en los sistemas EMR, cumpliendo con las normas de protección de datos como HIPAA. La gestión de inventario en la industria minorista y los registros médicos electrónicos en la atención médica ilustran el poder y la versatilidad de las bases de datos relacionales para respaldar las operaciones comerciales y el éxito.


2. ¡Con gusto! A continuación, te presento un archivo `models.py` para **Django** que representa un sistema de **Registros Médicos Electrónicos (EMR)**, basado en tu descripción. Este modelo está diseñado para ser seguro, escalable y respetuoso con buenas prácticas de salud digital (como la integridad de datos y relaciones claras).

---

### ✅ Archivo: `models.py` – Registros Médicos Electrónicos (EMR)

```python
from django.db import models
from django.contrib.auth.models import User  # Para médicos o personal autorizado

class Paciente(models.Model):
    id_paciente = models.AutoField(primary_key=True)
    nombre = models.CharField(max_length=100)
    apellido = models.CharField(max_length=100)
    fecha_nacimiento = models.DateField()
    SEXO_CHOICES = [
        ('M', 'Masculino'),
        ('F', 'Femenino'),
        ('O', 'Otro'),
        ('NS', 'No especificado')
    ]
    sexo = models.CharField(max_length=2, choices=SEXO_CHOICES, default='NS')
    direccion = models.CharField(max_length=200, blank=True, null=True)
    telefono = models.CharField(max_length=20, blank=True, null=True)
    email = models.EmailField(blank=True, null=True)
    numero_identificacion = models.CharField(max_length=50, unique=True, help_text="Ej. DNI, SSN, etc.")
    fecha_registro = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return f"{self.nombre} {self.apellido} ({self.numero_identificacion})"

    class Meta:
        verbose_name = "Paciente"
        verbose_name_plural = "Pacientes"


class Diagnostico(models.Model):
    id_diagnostico = models.AutoField(primary_key=True)
    paciente = models.ForeignKey(Paciente, on_delete=models.CASCADE, related_name='diagnosticos')
    condicion_medica = models.CharField(max_length=200, help_text="Nombre de la enfermedad o diagnóstico")
    descripcion = models.TextField(blank=True, null=True)
    fecha_diagnostico = models.DateField()
    activo = models.BooleanField(default=True, help_text="Indica si el diagnóstico sigue vigente")

    def __str__(self):
        return f"{self.condicion_medica} - {self.paciente}"

    class Meta:
        verbose_name = "Diagnóstico"
        verbose_name_plural = "Diagnósticos"


class Medicamento(models.Model):
    id_medicamento = models.AutoField(primary_key=True)
    paciente = models.ForeignKey(Paciente, on_delete=models.CASCADE, related_name='medicamentos')
    nombre = models.CharField(max_length=100)
    dosis = models.CharField(max_length=100)  # Ej: "500 mg", "10 ml"
    frecuencia = models.CharField(max_length=100, help_text="Ej: cada 8 horas, diario")  # Nuevo campo útil
    instrucciones = models.TextField(blank=True, null=True, help_text="Tomar con comida, evitar alcohol, etc.")
    fecha_inicio = models.DateField()
    fecha_fin = models.DateField(blank=True, null=True)
    activo = models.BooleanField(default=True)

    def __str__(self):
        return f"{self.nombre} - {self.paciente}"

    class Meta:
        verbose_name = "Medicamento"
        verbose_name_plural = "Medicamentos"


class ResultadoLaboratorio(models.Model):
    id_resultado = models.AutoField(primary_key=True)
    paciente = models.ForeignKey(Paciente, on_delete=models.CASCADE, related_name='resultados_laboratorio')
    prueba = models.CharField(max_length=150, help_text="Tipo de prueba: hemograma, glucosa, etc.")
    valor = models.CharField(max_length=100)  # Puede ser numérico o textual (ej: "positivo")
    unidad = models.CharField(max_length=30, blank=True, null=True)  # Ej: mg/dL, mmol/L
    rango_normal = models.CharField(max_length=100, blank=True, null=True)  # Referencia normal
    fecha_toma = models.DateField()
    fecha_resultado = models.DateField()
    archivo_adjunto = models.FileField(upload_to='laboratorio/', blank=True, null=True, help_text="PDF o imagen del informe")

    def __str__(self):
        return f"{self.prueba} - {self.paciente} ({self.fecha_resultado})"

    class Meta:
        verbose_name = "Resultado de Laboratorio"
        verbose_name_plural = "Resultados de Laboratorio"


class PlanTratamiento(models.Model):
    id_plan = models.AutoField(primary_key=True)
    paciente = models.ForeignKey(Paciente, on_delete=models.CASCADE, related_name='planes_tratamiento')
    diagnostico = models.ForeignKey(Diagnostico, on_delete=models.SET_NULL, null=True, blank=True, related_name='planes')
    tratamiento_recomendado = models.TextField(help_text="Descripción del plan: cirugía, fisioterapia, medicación, etc.")
    fecha_inicio = models.DateField()
    fecha_fin_estimada = models.DateField(blank=True, null=True)
    estado = models.CharField(
        max_length=20,
        choices=[
            ('activo', 'Activo'),
            ('en_espera', 'En espera'),
            ('completado', 'Completado'),
            ('cancelado', 'Cancelado')
        ],
        default='activo'
    )
    observaciones = models.TextField(blank=True, null=True)

    def __str__(self):
        return f"Plan para {self.paciente} - {self.tratamiento_recomendado[:50]}..."

    class Meta:
        verbose_name = "Plan de Tratamiento"
        verbose_name_plural = "Planes de Tratamiento"
```

---

### 📌 Explicación de los Modelos

| Modelo | Descripción |
|-------|------------|
| **`Paciente`** | Datos demográficos básicos, contacto y clave única (número de identificación). Incluye sexo, fecha de nacimiento y registro automático. |
| **`Diagnostico`** | Relaciona un paciente con una condición médica. Permite múltiples diagnósticos por paciente. |
| **`Medicamento`** | Controla qué medicamentos toma un paciente, con dosis, frecuencia e instrucciones. Útil para historiales farmacológicos. |
| **`ResultadoLaboratorio`** | Almacena pruebas, valores y archivos adjuntos. Ideal para seguimiento clínico. |
| **`PlanTratamiento`** | Vincula un diagnóstico con un plan terapéutico, incluyendo duración y estado actual. |

---

### 🔐 Consideraciones de Seguridad y Cumplimiento (HIPAA / LOPD / GDPR)

Aunque Django no aplica cifrado automático, puedes reforzar la seguridad:

- **Autenticación fuerte**: Usa `django.contrib.auth` y roles (médicos, enfermeros, administradores).
- **Registro de auditoría**: Agrega campos como `creado_por`, `fecha_creacion`, `modificado_en`.
- **Cifrado de datos sensibles**: Usa librerías como `django-cryptography` para campos críticos.
- **Control de acceso**: Usa `@login_required` y permisos personalizados.
- **Archivos seguros**: No almacenes exámenes médicos en carpetas públicas. Usa almacenamiento privado (AWS S3 con permisos, etc.).

---

### 🔧 Pasos siguientes

1. **Instalar Pillow y soporte para archivos (si usas imágenes o PDFs)**:
   ```bash
   pip install Pillow
   ```

2. **Crear migraciones**:
   ```bash
   python manage.py makemigrations
   python manage.py migrate
   ```

3. **Registrar modelos en el admin (`admin.py`)**:
   ```python
   from django.contrib import admin
   from .models import Paciente, Diagnostico, Medicamento, ResultadoLaboratorio, PlanTratamiento

   admin.site.register(Paciente)
   admin.site.register(Diagnostico)
   admin.site.register(Medicamento)
   admin.site.register(ResultadoLaboratorio)
   admin.site.register(PlanTratamiento)
   ```

4. **(Opcional)** Crear vistas API con Django REST Framework para integrar con apps móviles o sistemas externos.

---

### 💡 Ideas para mejorar (futuras funcionalidades)
- Historial clínico completo por paciente (resumen visual).
- Alertas de alergias o interacciones medicamentosas.
- Calendario de citas médicas.
- Notificaciones automáticas para seguimientos.
- Exportación de historial médico (PDF/JSON).

---

¿Te gustaría que agregue también un sistema de usuarios médicos, citas o integración con API de laboratorios? Puedo ayudarte a extenderlo paso a paso.
