1. curso
2. <img width="552" height="456" alt="image" src="https://github.com/user-attachments/assets/c388c5b0-fa7a-4ab8-99c5-02b4ea7caca5" />
3. ¡Claro! A continuación, te ayudo a crear los archivos `models.py`, `views.py`, y `urls.py` para el sistema de gestión académica basado en la estructura de tu diagrama ER. El modelo incluirá las entidades `Personas`, `Alumnos`, `Docentes`, `Cursos`, `Matriculas`, y `Cursos_Alumnos`.

---

### 1. **`models.py`**

El archivo `models.py` define los modelos que representan las tablas del diagrama ER.

```python
# models.py

from django.db import models

class Persona(models.Model):
    idpersona = models.AutoField(primary_key=True)
    nombre = models.CharField(max_length=30)
    apellido = models.CharField(max_length=35)
    rut = models.CharField(max_length=10, unique=True)

    def __str__(self):
        return f"{self.nombre} {self.apellido}"

    class Meta:
        verbose_name = "Persona"
        verbose_name_plural = "Personas"


class Alumno(models.Model):
    idalumno = models.AutoField(primary_key=True)
    persona = models.OneToOneField(Persona, on_delete=models.CASCADE, related_name='alumno')
    carrera = models.CharField(max_length=35)
    facultad = models.CharField(max_length=35)

    def __str__(self):
        return f"Alumno: {self.persona.nombre} {self.persona.apellido}"

    class Meta:
        verbose_name = "Alumno"
        verbose_name_plural = "Alumnos"


class Docente(models.Model):
    iddocente = models.AutoField(primary_key=True)
    persona = models.OneToOneField(Persona, on_delete=models.CASCADE, related_name='docente')
    tipocontrato = models.CharField(max_length=20)

    def __str__(self):
        return f"Docente: {self.persona.nombre} {self.persona.apellido}"

    class Meta:
        verbose_name = "Docente"
        verbose_name_plural = "Docentes"


class Curso(models.Model):
    idcurso = models.AutoField(primary_key=True)
    docente = models.ForeignKey(Docente, on_delete=models.SET_NULL, null=True, related_name='cursos_dictados')
    nombre_curso = models.CharField(max_length=50)
    horas = models.IntegerField()

    def __str__(self):
        return f"{self.nombre_curso} - Dictado por {self.docente.persona.nombre} {self.docente.persona.apellido}"

    class Meta:
        verbose_name = "Curso"
        verbose_name_plural = "Cursos"


class Matricula(models.Model):
    idmatricula = models.AutoField(primary_key=True)
    alumno = models.ForeignKey(Alumno, on_delete=models.CASCADE, related_name='matriculas')
    curso = models.ForeignKey(Curso, on_delete=models.CASCADE, related_name='matriculas')
    grado = models.CharField(max_length=30)
    fecha = models.DateField()

    def __str__(self):
        return f"Matrícula de {self.alumno.persona.nombre} en {self.curso.nombre_curso}"

    class Meta:
        verbose_name = "Matrícula"
        verbose_name_plural = "Matrículas"


class CursosAlumnos(models.Model):
    idcurso_alumno = models.AutoField(primary_key=True)
    curso = models.ForeignKey(Curso, on_delete=models.CASCADE, related_name='notas_alumnos')
    alumno = models.ForeignKey(Alumno, on_delete=models.CASCADE, related_name='notas_cursos')
    nota = models.CharField(max_length=60)

    def __str__(self):
        return f"Nota de {self.alumno.persona.nombre} en {self.curso.nombre_curso}: {self.nota}"

    class Meta:
        verbose_name = "Nota de Curso-Alumno"
        verbose_name_plural = "Notas de Cursos-Alumnos"
```

---

### 2. **`views.py`**

El archivo `views.py` contiene las vistas para manejar las solicitudes HTTP y renderizar las páginas.

```python
# views.py

from django.shortcuts import render, get_object_or_404
from .models import Persona, Alumno, Docente, Curso, Matricula, CursosAlumnos

def index(request):
    alumnos = Alumno.objects.all()
    docentes = Docente.objects.all()
    cursos = Curso.objects.all()
    matriculas = Matricula.objects.all()
    notas = CursosAlumnos.objects.all()
    context = {
        'alumnos': alumnos,
        'docentes': docentes,
        'cursos': cursos,
        'matriculas': matriculas,
        'notas': notas,
    }
    return render(request, 'academico/index.html', context)


def alumno_detail(request, idalumno):
    alumno = get_object_or_404(Alumno, idalumno=idalumno)
    matriculas = Matricula.objects.filter(alumno=alumno)
    notas = CursosAlumnos.objects.filter(alumno=alumno)
    context = {
        'alumno': alumno,
        'matriculas': matriculas,
        'notas': notas,
    }
    return render(request, 'academico/alumno_detail.html', context)


def docente_detail(request, iddocente):
    docente = get_object_or_404(Docente, iddocente=iddocente)
    cursos = Curso.objects.filter(docente=docente)
    context = {
        'docente': docente,
        'cursos': cursos,
    }
    return render(request, 'academico/docente_detail.html', context)


def curso_detail(request, idcurso):
    curso = get_object_or_404(Curso, idcurso=idcurso)
    matriculas = Matricula.objects.filter(curso=curso)
    notas = CursosAlumnos.objects.filter(curso=curso)
    context = {
        'curso': curso,
        'matriculas': matriculas,
        'notas': notas,
    }
    return render(request, 'academico/curso_detail.html', context)
```

---

### 3. **`urls.py`**

El archivo `urls.py` define las rutas URL para acceder a las vistas.

```python
# urls.py

from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
    path('alumno/<int:idalumno>/', views.alumno_detail, name='alumno_detail'),
    path('docente/<int:iddocente>/', views.docente_detail, name='docente_detail'),
    path('curso/<int:idcurso>/', views.curso_detail, name='curso_detail'),
]
```

---

### 4. **Estructura de Templates (HTML)**

Para completar la aplicación, necesitarás crear los templates HTML correspondientes en la carpeta `templates/academico/`. Aquí tienes ejemplos básicos:

#### `templates/academico/index.html`
```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Gestión Académica</title>
</head>
<body>
    <h1>Gestión Académica</h1>
    <h2>Alumnos</h2>
    <ul>
        {% for alumno in alumnos %}
            <li><a href="{% url 'alumno_detail' idalumno=alumno.idalumno %}">{{ alumno.persona.nombre }} {{ alumno.persona.apellido }}</a></li>
        {% endfor %}
    </ul>

    <h2>Docentes</h2>
    <ul>
        {% for docente in docentes %}
            <li><a href="{% url 'docente_detail' iddocente=docente.iddocente %}">{{ docente.persona.nombre }} {{ docente.persona.apellido }}</a></li>
        {% endfor %}
    </ul>

    <h2>Cursos</h2>
    <ul>
        {% for curso in cursos %}
            <li><a href="{% url 'curso_detail' idcurso=curso.idcurso %}">{{ curso.nombre_curso }}</a></li>
        {% endfor %}
    </ul>
</body>
</html>
```

#### `templates/academico/alumno_detail.html`
```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Detalles del Alumno</title>
</head>
<body>
    <h1>Detalles del Alumno</h1>
    <p><strong>Nombre:</strong> {{ alumno.persona.nombre }} {{ alumno.persona.apellido }}</p>
    <p><strong>Carrera:</strong> {{ alumno.carrera }}</p>
    <p><strong>Facultad:</strong> {{ alumno.facultad }}</p>

    <h2>Matrículas</h2>
    <ul>
        {% for matricula in matriculas %}
            <li>{{ matricula.curso.nombre_curso }} - Grado: {{ matricula.grado }} - Fecha: {{ matricula.fecha }}</li>
        {% endfor %}
    </ul>

    <h2>Notas</h2>
    <ul>
        {% for nota in notas %}
            <li>{{ nota.curso.nombre_curso }} - Nota: {{ nota.nota }}</li>
        {% endfor %}
    </ul>
</body>
</html>
```

#### `templates/academico/docente_detail.html`
```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Detalles del Docente</title>
</head>
<body>
    <h1>Detalles del Docente</h1>
    <p><strong>Nombre:</strong> {{ docente.persona.nombre }} {{ docente.persona.apellido }}</p>
    <p><strong>Tipo de Contrato:</strong> {{ docente.tipocontrato }}</p>

    <h2>Cursos Dictados</h2>
    <ul>
        {% for curso in cursos %}
            <li><a href="{% url 'curso_detail' idcurso=curso.idcurso %}">{{ curso.nombre_curso }} - Horas: {{ curso.horas }}</a></li>
        {% endfor %}
    </ul>
</body>
</html>
```

#### `templates/academico/curso_detail.html`
```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Detalles del Curso</title>
</head>
<body>
    <h1>Detalles del Curso</h1>
    <p><strong>Nombre:</strong> {{ curso.nombre_curso }}</p>
    <p><strong>Horas:</strong> {{ curso.horas }}</p>
    <p><strong>Docente:</strong> {{ curso.docente.persona.nombre }} {{ curso.docente.persona.apellido }}</p>

    <h2>Matrículas</h2>
    <ul>
        {% for matricula in matriculas %}
            <li>{{ matricula.alumno.persona.nombre }} {{ matricula.alumno.persona.apellido }} - Grado: {{ matricula.grado }} - Fecha: {{ matricula.fecha }}</li>
        {% endfor %}
    </ul>

    <h2>Notas</h2>
    <ul>
        {% for nota in notas %}
            <li>{{ nota.alumno.persona.nombre }} {{ nota.alumno.persona.apellido }} - Nota: {{ nota.nota }}</li>
        {% endfor %}
    </ul>
</body>
</html>
```

---

### 5. **Pasos Adicionales**

1. **Crear Migraciones**:
   ```bash
   python manage.py makemigrations
   python manage.py migrate
   ```

2. **Registrar Modelos en Admin**:
   Agrega los modelos al archivo `admin.py` para gestionarlos desde el panel de administración.
   ```python
   # admin.py
   from django.contrib import admin
   from .models import Persona, Alumno, Docente, Curso, Matricula, CursosAlumnos

   admin.site.register(Persona)
   admin.site.register(Alumno)
   admin.site.register(Docente)
   admin.site.register(Curso)
   admin.site.register(Matricula)
   admin.site.register(CursosAlumnos)
   ```

3. **Configurar URLs Globales**:
   Asegúrate de incluir las URLs de esta app en el archivo `urls.py` principal del proyecto.
   ```python
   # mi_proyecto/urls.py
   from django.contrib import admin
   from django.urls import path, include

   urlpatterns = [
       path('admin/', admin.site.urls),
       path('academico/', include('academico.urls')),
   ]
   ```

---

### 📌 Resumen
- **`models.py`**: Define los modelos para las entidades del diagrama ER.
- **`views.py`**: Maneja las vistas para mostrar información sobre alumnos, docentes, cursos, matrículas y notas.
- **`urls.py`**: Define las rutas URL para acceder a las vistas.
- **Templates**: Renderiza la información en formatos HTML legibles.

Este sistema permite ver detalles de alumnos, docentes, cursos, matrículas y notas. Puedes expandirlo con formularios para agregar o editar registros si es necesario. ¡Espero que sea útil! 😊
