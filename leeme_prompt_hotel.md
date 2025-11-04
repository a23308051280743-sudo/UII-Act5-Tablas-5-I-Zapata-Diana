Crear carpeta del proyecto UIII_Hotel_0743

Windows / macOS / Linux (terminal):

# desde la carpeta donde quieras crear el proyecto
mkdir UIII_Hotel_0743
cd UIII_Hotel_0743

2. Abrir VS Code sobre la carpeta UIII_Hotel_0743

Desde la misma carpeta en terminal:

code .


(Esto abre VS Code en la carpeta actual.)

También desde VS Code: File > Open Folder... y seleccionar UIII_Hotel_0743.

3. Abrir terminal integrado en VS Code

En VS Code: menú Terminal > New Terminal
o atajo: `Ctrl + `` (control + backtick) en Windows/macOS.

4. Crear carpeta entorno virtual .venv desde terminal de VS Code

En la terminal integrada, ejecutar:

Windows (PowerShell):

python -m venv .venv


macOS / Linux:

python3 -m venv .venv


Esto crea la carpeta .venv dentro de UIII_Hotel_0743.

5. Activar el entorno virtual

Windows (PowerShell):

# Si PowerShell bloquea la ejecución, ejecutar: Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser (solo si sabes lo que haces)
.venv\Scripts\Activate.ps1
# O con cmd:
.venv\Scripts\activate


macOS / Linux:

source .venv/bin/activate


Al activarse verás (.venv) al inicio del prompt.

6. Activar intérprete de Python en VS Code

En VS Code: Ctrl+Shift+P → escribir Python: Select Interpreter → elegir el intérprete que apunta a .../UIII_Hotel_0743/.venv/bin/python (o Scripts\python.exe en Windows).
Esto asegura que VS Code use el .venv.

7. Instalar Django

Con el entorno activado:

pip install --upgrade pip
pip install django


Comprobar versión:

python -m django --version

8. Crear proyecto backend_Hotel sin duplicar carpeta

Para que manage.py quede en la raíz UIII_Hotel_0743 y la carpeta del proyecto se llame backend_Hotel, ejecuta:

django-admin startproject backend_Hotel .


Nota: el punto (.) al final crea los archivos en la carpeta actual sin crear una carpeta extra.

Estructura resultante mínima:

UIII_Hotel_0743/
  .venv/
  manage.py
  backend_Hotel/
    __init__.py
    settings.py
    urls.py
    wsgi.py
    asgi.py

9. Ejecutar servidor en el puerto 8036

Para ejecución local:

python manage.py runserver 8036


o para permitir acceso desde otras máquinas:

python manage.py runserver 0.0.0.0:8036

10. Copiar y pegar el link en el navegador

Abre tu navegador y pega:

http://127.0.0.1:8036/


ó

http://localhost:8036/

11. Crear aplicación app_Hotel

Con el entorno activado y en la raíz:

python manage.py startapp app_Hotel


Estructura ahora:

UIII_Hotel_0743/
  app_Hotel/
    migrations/
    __init__.py
    admin.py
    apps.py
    models.py
    views.py
    tests.py
    apps.py
    ...

12. Aquí el modelo models.py

Ya diste el código; colócalo en app_Hotel/models.py. Te lo repito tal cual (pegalo en ese archivo):

from django.db import models

# ==========================================
# MODELO: HUESPED
# ==========================================
class Huesped(models.Model):
    nombre = models.CharField(max_length=100)
    apellido = models.CharField(max_length=100)
    email = models.EmailField(unique=True)
    telefono = models.CharField(max_length=20)
    fecha_nacimiento = models.DateField()
    direccion = models.CharField(max_length=200)
    def __str__(self):
        return self.nombre

# ==========================================
# MODELO: HABITACION
# ==========================================
class Habitacion(models.Model):
    numero = models.IntegerField(unique=True)
    tipo = models.CharField(max_length=50)
    precio = models.DecimalField(max_digits=8, decimal_places=2)
    descripcion = models.TextField()
    estado = models.CharField(max_length=50)
    piso = models.IntegerField()
    def __str__(self):
        return f"Habitación {self.numero}"

# ==========================================
# MODELO: RESERVA
# ==========================================
class Reserva(models.Model):
    fecha_inicio = models.DateField()
    fecha_fin = models.DateField()
    fecha_reserva = models.DateField(auto_now_add=True)
    estado = models.CharField(max_length=50)
    huesped = models.ForeignKey(Huesped, on_delete=models.CASCADE)
    habitaciones = models.ManyToManyField(Habitacion, through='ReservaHabitacion')
    def __str__(self):
        return f"Reserva {self.id}"

# ==========================================
# TABLA INTERMEDIA (7 CAMPOS)
# ==========================================
class ReservaHabitacion(models.Model):
    reserva = models.ForeignKey(Reserva, on_delete=models.CASCADE)
    habitacion = models.ForeignKey(Habitacion, on_delete=models.CASCADE)
    precio = models.DecimalField(max_digits=8, decimal_places=2)
    fecha_checkin = models.DateField()
    fecha_checkout = models.DateField()
    estatus = models.CharField(max_length=50)
    def __str__(self):
        return f"Reserva {self.reserva.id} - Habitación {self.habitacion.numero}"


IMPORTANTE: Por ahora trabajaremos sólo con Huesped (según tu punto 27). Los otros modelos quedan pendientes.

12.5 Procedimiento para realizar migraciones (makemigrations y migrate)

Agrega la app a INSTALLED_APPS (ver punto 25).

Ejecuta:

python manage.py makemigrations app_Hotel
python manage.py migrate

13. Primero trabajamos con el MODELO: HUESPED

Hecho: models.py tiene Huesped. Nos centraremos en vistas, templates y CRUD para Huesped.

14. En views.py de app_Hotel crear las funciones

Coloca el siguiente código en app_Hotel/views.py (funciones: inicio_hotel, agregar_huesped, actualizar_huesped, realizar_actualizacion_huesped, borrar_huesped):

from django.shortcuts import render, redirect, get_object_or_404
from .models import Huesped
from django.urls import reverse

def inicio_hotel(request):
    # muestra página de inicio con información general
    return render(request, 'inicio.html')

def agregar_huesped(request):
    if request.method == 'POST':
        # sin validación, guardar directamente
        nombre = request.POST.get('nombre', '')
        apellido = request.POST.get('apellido', '')
        email = request.POST.get('email', '')
        telefono = request.POST.get('telefono', '')
        fecha_nacimiento = request.POST.get('fecha_nacimiento', None)
        direccion = request.POST.get('direccion', '')
        Huesped.objects.create(
            nombre=nombre,
            apellido=apellido,
            email=email,
            telefono=telefono,
            fecha_nacimiento=fecha_nacimiento,
            direccion=direccion
        )
        return redirect('ver_huesped')
    return render(request, 'huesped/agregar_huesped.html')

def ver_huesped(request):
    lista = Huesped.objects.all().order_by('id')
    return render(request, 'huesped/ver_huesped.html', {'huespedes': lista})

def actualizar_huesped(request, pk):
    huesped = get_object_or_404(Huesped, pk=pk)
    return render(request, 'huesped/actualizar_huesped.html', {'huesped': huesped})

def realizar_actualizacion_huesped(request, pk):
    huesped = get_object_or_404(Huesped, pk=pk)
    if request.method == 'POST':
        huesped.nombre = request.POST.get('nombre', huesped.nombre)
        huesped.apellido = request.POST.get('apellido', huesped.apellido)
        huesped.email = request.POST.get('email', huesped.email)
        huesped.telefono = request.POST.get('telefono', huesped.telefono)
        huesped.fecha_nacimiento = request.POST.get('fecha_nacimiento', huesped.fecha_nacimiento)
        huesped.direccion = request.POST.get('direccion', huesped.direccion)
        huesped.save()
        return redirect('ver_huesped')
    # si no es POST, redirigir a formulario de actualización
    return redirect('actualizar_huesped', pk=pk)

def borrar_huesped(request, pk):
    huesped = get_object_or_404(Huesped, pk=pk)
    if request.method == 'POST':
        huesped.delete()
        return redirect('ver_huesped')
    return render(request, 'huesped/borrar_huesped.html', {'huesped': huesped})


Observación: separé ver_huesped porque lo necesitas para listar.

15. Crear la carpeta templates dentro de app_Hotel

Estructura recomendada:

app_Hotel/
  templates/
    base.html
    header.html
    navbar.html
    footer.html
    inicio.html
    huesped/
      agregar_huesped.html
      ver_huesped.html
      actualizar_huesped.html
      borrar_huesped.html


Crea la carpeta app_Hotel/templates y la subcarpeta huesped.

16. Crear archivos HTML (base, header, navbar, footer, inicio)

A continuación el contenido mínimo moderno y sencillo. Usa Bootstrap desde CDN.

app_Hotel/templates/base.html
<!doctype html>
<html lang="es">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>{% block title %}Sistema Hotel{% endblock %}</title>
    <!-- Bootstrap CSS (CDN) -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
    <style>
      /* Colores suaves y moderno */
      body { background-color: #f7fafc; color: #0f172a; }
      .header-brand { font-weight:700; color:#0b5ed7; }
      footer { position: fixed; bottom: 0; left:0; right:0; background:#ffffffcc; padding:10px 0; border-top:1px solid #e6edf3; }
      .content { padding-bottom:80px; } /* espacio para footer fijo */
    </style>
    {% block extra_head %}{% endblock %}
  </head>
  <body>
    {% include 'navbar.html' %}
    <div class="container content mt-4">
      {% block content %}{% endblock %}
    </div>

    {% include 'footer.html' %}

    <!-- Bootstrap JS (CDN) -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>
    {% block extra_js %}{% endblock %}
  </body>
</html>

app_Hotel/templates/navbar.html
<nav class="navbar navbar-expand-lg navbar-light bg-white shadow-sm">
  <div class="container">
    <a class="navbar-brand header-brand" href="{% url 'inicio' %}">Sistema de Administración Hotel</a>
    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navMenu">
      <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse" id="navMenu">
      <ul class="navbar-nav ms-auto">
        <li class="nav-item"><a class="nav-link" href="{% url 'inicio' %}">Inicio</a></li>

        <!-- Huésped -->
        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" id="huespedMenu" role="button" data-bs-toggle="dropdown">
            <i class="bi bi-people-fill"></i> Huésped
          </a>
          <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="{% url 'agregar_huesped' %}">Agregar huésped</a></li>
            <li><a class="dropdown-item" href="{% url 'ver_huesped' %}">Ver huéspedes</a></li>
          </ul>
        </li>

        <!-- Habitación -->
        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" id="habMenu" role="button" data-bs-toggle="dropdown">
            <i class="bi bi-door-closed-fill"></i> Habitación
          </a>
          <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="#">Agregar habitación</a></li>
            <li><a class="dropdown-item" href="#">Ver habitaciones</a></li>
          </ul>
        </li>

        <!-- Reservas -->
        <li class="nav-item dropdown">
          <a class="nav-link dropdown-toggle" href="#" id="resMenu" role="button" data-bs-toggle="dropdown">
            <i class="bi bi-calendar-check-fill"></i> Reservas
          </a>
          <ul class="dropdown-menu">
            <li><a class="dropdown-item" href="#">Agregar reserva</a></li>
            <li><a class="dropdown-item" href="#">Ver reservas</a></li>
          </ul>
        </li>
      </ul>
    </div>
  </div>
</nav>

<!-- Agregamos iconos Bootstrap Icons CDN -->
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.5/font/bootstrap-icons.css">


Nota: pediste iconos solo en opciones principales (los puse junto a los textos).

app_Hotel/templates/footer.html
<footer class="text-center">
  <div class="container">
    <small>&copy; {{ now|"date:""%Y"" }} Sistema Hotel. Creado por Ing. Eliseo Nava, Cbtis 128. Todos los derechos reservados.</small>
  </div>
</footer>


Para que now funcione en template, puedes activar el contexto o usar {% now "Y" %}. Si quieres que muestre fecha del sistema:

<small>&copy; {% now "Y" %} Sistema Hotel. Creado por Ing. Eliseo Nava, Cbtis 128.</small>

app_Hotel/templates/inicio.html
{% extends 'base.html' %}
{% block title %}Inicio - Sistema Hotel{% endblock %}
{% block content %}
<div class="row">
  <div class="col-md-7">
    <h1>Bienvenido al Sistema de Administración Hotel</h1>
    <p class="lead">Gestión sencilla de huéspedes, habitaciones y reservas.</p>
    <p>Usa el menú para navegar por las operaciones del sistema.</p>
  </div>
  <div class="col-md-5">
    <img src="https://images.unsplash.com/photo-1542314831-068cd1dbfeeb?q=80&w=1200&auto=format&fit=crop&ixlib=rb-4.0.3&s=..." alt="Hotel" class="img-fluid rounded shadow-sm">
  </div>
</div>
{% endblock %}


La imagen la tomé de la red (Unsplash). Puedes reemplazar la URL por otra.

21. Crear subcarpeta huesped dentro de app_Hotel/templates

Ya incluida en la estructura: app_Hotel/templates/huesped/

22. Crear archivos HTML para huesped (agregar, ver, actualizar, borrar)
app_Hotel/templates/huesped/agregar_huesped.html
{% extends 'base.html' %}
{% block title %}Agregar Huésped{% endblock %}
{% block content %}
<div class="card p-3 shadow-sm">
  <h4>Agregar Huésped</h4>
  <form method="post">
    {% csrf_token %}
    <div class="row">
      <div class="col-md-6 mb-2"><input name="nombre" placeholder="Nombre" class="form-control"></div>
      <div class="col-md-6 mb-2"><input name="apellido" placeholder="Apellido" class="form-control"></div>
      <div class="col-md-6 mb-2"><input name="email" placeholder="Email" class="form-control" type="email"></div>
      <div class="col-md-6 mb-2"><input name="telefono" placeholder="Teléfono" class="form-control"></div>
      <div class="col-md-6 mb-2"><input name="fecha_nacimiento" placeholder="Fecha de nacimiento" class="form-control" type="date"></div>
      <div class="col-md-6 mb-2"><input name="direccion" placeholder="Dirección" class="form-control"></div>
    </div>
    <button class="btn btn-primary mt-2">Guardar</button>
    <a href="{% url 'ver_huesped' %}" class="btn btn-secondary mt-2">Cancelar</a>
  </form>
</div>
{% endblock %}

app_Hotel/templates/huesped/ver_huesped.html
{% extends 'base.html' %}
{% block title %}Ver Huéspedes{% endblock %}
{% block content %}
<div class="d-flex justify-content-between align-items-center mb-3">
  <h4>Lista de Huéspedes</h4>
  <a href="{% url 'agregar_huesped' %}" class="btn btn-success">+ Agregar</a>
</div>

<table class="table table-striped table-hover bg-white shadow-sm">
  <thead class="table-light">
    <tr>
      <th>#</th><th>Nombre</th><th>Apellido</th><th>Email</th><th>Teléfono</th><th>Acciones</th>
    </tr>
  </thead>
  <tbody>
    {% for h in huespedes %}
    <tr>
      <td>{{ forloop.counter }}</td>
      <td>{{ h.nombre }}</td>
      <td>{{ h.apellido }}</td>
      <td>{{ h.email }}</td>
      <td>{{ h.telefono }}</td>
      <td>
        <a href="{% url 'actualizar_huesped' h.id %}" class="btn btn-sm btn-warning">Editar</a>
        <a href="{% url 'borrar_huesped' h.id %}" class="btn btn-sm btn-danger">Borrar</a>
      </td>
    </tr>
    {% empty %}
    <tr><td colspan="6">No hay huéspedes.</td></tr>
    {% endfor %}
  </tbody>
</table>
{% endblock %}

app_Hotel/templates/huesped/actualizar_huesped.html
{% extends 'base.html' %}
{% block title %}Actualizar Huésped{% endblock %}
{% block content %}
<div class="card p-3 shadow-sm">
  <h4>Actualizar Huésped</h4>
  <form method="post" action="{% url 'realizar_actualizacion_huesped' huesped.id %}">
    {% csrf_token %}
    <div class="row">
      <div class="col-md-6 mb-2"><input name="nombre" value="{{ huesped.nombre }}" class="form-control"></div>
      <div class="col-md-6 mb-2"><input name="apellido" value="{{ huesped.apellido }}" class="form-control"></div>
      <div class="col-md-6 mb-2"><input name="email" value="{{ huesped.email }}" class="form-control" type="email"></div>
      <div class="col-md-6 mb-2"><input name="telefono" value="{{ huesped.telefono }}" class="form-control"></div>
      <div class="col-md-6 mb-2"><input name="fecha_nacimiento" value="{{ huesped.fecha_nacimiento }}" class="form-control" type="date"></div>
      <div class="col-md-6 mb-2"><input name="direccion" value="{{ huesped.direccion }}" class="form-control"></div>
    </div>
    <button class="btn btn-primary mt-2">Guardar cambios</button>
    <a href="{% url 'ver_huesped' %}" class="btn btn-secondary mt-2">Cancelar</a>
  </form>
</div>
{% endblock %}

app_Hotel/templates/huesped/borrar_huesped.html
{% extends 'base.html' %}
{% block title %}Borrar Huésped{% endblock %}
{% block content %}
<div class="card p-3 shadow-sm">
  <h4>¿Eliminar huésped?</h4>
  <p>Confirma eliminar a <strong>{{ huesped.nombre }} {{ huesped.apellido }}</strong></p>
  <form method="post">
    {% csrf_token %}
    <button type="submit" class="btn btn-danger">Sí, eliminar</button>
    <a href="{% url 'ver_huesped' %}" class="btn btn-secondary">Cancelar</a>
  </form>
</div>
{% endblock %}


Punto 23: no usamos forms.py, utilizamos formularios HTML directos.

24. Procedimiento para crear urls.py en app_Hotel

Crea app_Hotel/urls.py con este contenido:

from django.urls import path
from . import views

urlpatterns = [
    path('', views.inicio_hotel, name='inicio'),
    # Huesped CRUD
    path('huesped/agregar/', views.agregar_huesped, name='agregar_huesped'),
    path('huesped/ver/', views.ver_huesped, name='ver_huesped'),
    path('huesped/actualizar/<int:pk>/', views.actualizar_huesped, name='actualizar_huesped'),
    path('huesped/actualizar/realizar/<int:pk>/', views.realizar_actualizacion_huesped, name='realizar_actualizacion_huesped'),
    path('huesped/borrar/<int:pk>/', views.borrar_huesped, name='borrar_huesped'),
]

25. Agregar app_Hotel en settings.py de backend_Hotel

Editar backend_Hotel/settings.py y en INSTALLED_APPS agregar 'app_Hotel', por ejemplo:

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    'app_Hotel',  # <-- agrega aquí
]


No es necesario modificar TEMPLATES si usas templates dentro de la app; Django detecta app/templates.

26. Configurar urls.py de backend_Hotel para enlazar con app_Hotel

Editar backend_Hotel/urls.py para incluir las urls de la app:

from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('app_Hotel.urls')),  # la app maneja la raíz
]

27. Registrar modelos en admin.py y volver a realizar migraciones

En app_Hotel/admin.py:

from django.contrib import admin
from .models import Huesped  # sólo Huesped por ahora

@admin.register(Huesped)
class HuespedAdmin(admin.ModelAdmin):
    list_display = ('id', 'nombre', 'apellido', 'email', 'telefono')
    search_fields = ('nombre', 'apellido', 'email')


Luego:

python manage.py makemigrations app_Hotel
python manage.py migrate


Crea un superusuario si quieres acceder al admin:

python manage.py createsuperuser

28. Diseño y colores

Usé Bootstrap + estilos en base.html para colores suaves y moderno. El HTML es sencillo y limpio tal como pediste (páginas sencillas). No realicé validación de datos (punto 28).

29. Al inicio crear la estructura completa de carpetas y archivos

Resumen de la estructura final (recomendada):

UIII_Hotel_0743/
├─ .venv/
├─ manage.py
├─ backend_Hotel/
│  ├─ __init__.py
│  ├─ settings.py
│  ├─ urls.py
│  ├─ wsgi.py
│  └─ asgi.py
├─ app_Hotel/
│  ├─ migrations/
│  ├─ templates/
│  │  ├─ base.html
│  │  ├─ navbar.html
│  │  ├─ footer.html
│  │  ├─ inicio.html
│  │  └─ huesped/
│  │     ├─ agregar_huesped.html
│  │     ├─ ver_huesped.html
│  │     ├─ actualizar_huesped.html
│  │     └─ borrar_huesped.html
│  ├─ admin.py
│  ├─ apps.py
│  ├─ models.py
│  ├─ urls.py
│  └─ views.py

30. Proyecto totalmente funcional (con HUESPED)

Con los pasos anteriores deberías tener un sistema funcional para CRUD de huéspedes:

http://127.0.0.1:8036/ → inicio

http://127.0.0.1:8036/huesped/ver/ → lista de huéspedes

http://127.0.0.1:8036/huesped/agregar/ → formulario para agregar

editar y borrar funcionan vía botones en la tabla

31. Finalmente ejecutar servidor en el puerto 8036

Si no está ya corriendo:

python manage.py runserver 8036


Abrir: http://127.0.0.1:8036/ en el navegador. ¡Listo!

Resumen rápido de comandos clave (copiar/pegar)
mkdir UIII_Hotel_0743
cd UIII_Hotel_0743
python -m venv .venv
# activar:
# Windows PowerShell:
.venv\Scripts\Activate.ps1
# mac/linux:
source .venv/bin/activate

pip install --upgrade pip
pip install django

django-admin startproject backend_Hotel .
python manage.py startapp app_Hotel

# editar archivos (models.py, views.py, templates, urls.py, admin.py, settings.py)
python manage.py makemigrations app_Hotel
python manage.py migrate
python manage.py createsuperuser   # opcional
python manage.py runserver 8036
