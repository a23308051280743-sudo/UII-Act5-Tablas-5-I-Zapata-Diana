# Proyecto Hotel - Django - Guía Completa

Te proporciono una guía completa paso a paso para crear el proyecto Hotel con Django:

## 1. Estructura inicial del proyecto

### 1.1 Crear carpeta del proyecto
```bash
mkdir UIII_Hotel_0743
cd UIII_Hotel_0743
```

### 1.2 Abrir VS Code en la carpeta
```bash
code .
```

### 1.3 Abrir terminal en VS Code
- `Ctrl + Ñ` (Windows/Linux) o `Cmd + Ñ` (Mac)
- O menú: View → Terminal

### 1.4 Crear entorno virtual
```bash
python -m venv .venv
```

### 1.5 Activar entorno virtual
**Windows:**
```bash
.venv\Scripts\activate
```

**Mac/Linux:**
```bash
source .venv/bin/activate
```

### 1.6 Activar intérprete de Python
- `Ctrl + Shift + P` → "Python: Select Interpreter"
- Elegir el del entorno virtual: `./.venv/Scripts/python.exe`

### 1.7 Instalar Django
```bash
pip install django
```

### 1.8 Crear proyecto Django
```bash
django-admin startproject backend_Hotel .
```

### 1.9 Crear aplicación
```bash
python manage.py startapp app_Hotel
```

## 2. Configuración del proyecto

### 2.1 settings.py - Agregar aplicación
```python
# backend_Hotel/settings.py

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'app_Hotel',  # Agregar esta línea
]
```

### 2.2 models.py - Modelos completos
```python
# app_Hotel/models.py
from django.db import models

class Huesped(models.Model):
    nombre = models.CharField(max_length=100)
    apellido = models.CharField(max_length=100)
    email = models.EmailField(unique=True)
    telefono = models.CharField(max_length=20)
    fecha_nacimiento = models.DateField()
    direccion = models.CharField(max_length=200)
    
    def __str__(self):
        return f"{self.nombre} {self.apellido}"

class Habitacion(models.Model):
    numero = models.IntegerField(unique=True)
    tipo = models.CharField(max_length=50)
    precio = models.DecimalField(max_digits=8, decimal_places=2)
    descripcion = models.TextField()
    estado = models.CharField(max_length=50)
    piso = models.IntegerField()
    
    def __str__(self):
        return f"Habitación {self.numero}"

class Reserva(models.Model):
    fecha_inicio = models.DateField()
    fecha_fin = models.DateField()
    fecha_reserva = models.DateField(auto_now_add=True)
    estado = models.CharField(max_length=50)
    huesped = models.ForeignKey(Huesped, on_delete=models.CASCADE)
    habitaciones = models.ManyToManyField(Habitacion, through='ReservaHabitacion')
    
    def __str__(self):
        return f"Reserva {self.id} - {self.huesped}"

class ReservaHabitacion(models.Model):
    reserva = models.ForeignKey(Reserva, on_delete=models.CASCADE)
    habitacion = models.ForeignKey(Habitacion, on_delete=models.CASCADE)
    precio = models.DecimalField(max_digits=8, decimal_places=2)
    fecha_checkin = models.DateField()
    fecha_checkout = models.DateField()
    estatus = models.CharField(max_length=50)
    
    def __str__(self):
        return f"Reserva {self.reserva.id} - Habitación {self.habitacion.numero}"
```

### 2.3 admin.py - Registrar modelos
```python
# app_Hotel/admin.py
from django.contrib import admin
from .models import Huesped, Habitacion, Reserva, ReservaHabitacion

admin.site.register(Huesped)
admin.site.register(Habitacion)
admin.site.register(Reserva)
admin.site.register(ReservaHabitacion)
```

### 2.4 Migraciones
```bash
python manage.py makemigrations
python manage.py migrate
```

## 3. Views para Huésped

### 3.1 views.py
```python
# app_Hotel/views.py
from django.shortcuts import render, redirect, get_object_or_404
from .models import Huesped
from django.http import HttpResponse

def inicio_hotel(request):
    return render(request, 'inicio.html')

# HUÉSPED - CRUD
def agregar_huesped(request):
    if request.method == 'POST':
        huesped = Huesped(
            nombre=request.POST['nombre'],
            apellido=request.POST['apellido'],
            email=request.POST['email'],
            telefono=request.POST['telefono'],
            fecha_nacimiento=request.POST['fecha_nacimiento'],
            direccion=request.POST['direccion']
        )
        huesped.save()
        return redirect('ver_huespedes')
    
    return render(request, 'huesped/agregar_huesped.html')

def ver_huespedes(request):
    huespedes = Huesped.objects.all()
    return render(request, 'huesped/ver_huesped.html', {'huespedes': huespedes})

def actualizar_huesped(request, id):
    huesped = get_object_or_404(Huesped, id=id)
    
    if request.method == 'POST':
        return realizar_actualizacion_huesped(request, id)
    
    return render(request, 'huesped/actualizar_huesped.html', {'huesped': huesped})

def realizar_actualizacion_huesped(request, id):
    huesped = get_object_or_404(Huesped, id=id)
    
    huesped.nombre = request.POST['nombre']
    huesped.apellido = request.POST['apellido']
    huesped.email = request.POST['email']
    huesped.telefono = request.POST['telefono']
    huesped.fecha_nacimiento = request.POST['fecha_nacimiento']
    huesped.direccion = request.POST['direccion']
    
    huesped.save()
    return redirect('ver_huespedes')

def borrar_huesped(request, id):
    huesped = get_object_or_404(Huesped, id=id)
    
    if request.method == 'POST':
        huesped.delete()
        return redirect('ver_huespedes')
    
    return render(request, 'huesped/borrar_huesped.html', {'huesped': huesped})
```

## 4. URLs

### 4.1 app_Hotel/urls.py
```python
# app_Hotel/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('', views.inicio_hotel, name='inicio_hotel'),
    
    # Huésped URLs
    path('huesped/agregar/', views.agregar_huesped, name='agregar_huesped'),
    path('huesped/ver/', views.ver_huespedes, name='ver_huespedes'),
    path('huesped/actualizar/<int:id>/', views.actualizar_huesped, name='actualizar_huesped'),
    path('huesped/borrar/<int:id>/', views.borrar_huesped, name='borrar_huesped'),
]
```

### 4.2 backend_Hotel/urls.py
```python
# backend_Hotel/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('app_Hotel.urls')),
]
```

## 5. Templates

### 5.1 Estructura de carpetas
```
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
```

### 5.2 base.html
```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sistema Hotel</title>
    <!-- Bootstrap CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <style>
        body {
            background-color: #f8f9fa;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        .navbar-brand {
            font-weight: bold;
            color: #2c3e50 !important;
        }
        .main-content {
            min-height: calc(100vh - 120px);
            padding: 20px 0;
        }
        .card {
            border: none;
            border-radius: 15px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
            margin-bottom: 20px;
        }
        .btn-primary {
            background-color: #3498db;
            border-color: #3498db;
        }
        .btn-primary:hover {
            background-color: #2980b9;
            border-color: #2980b9;
        }
    </style>
</head>
<body>
    {% include 'header.html' %}
    {% include 'navbar.html' %}
    
    <div class="container main-content">
        {% block content %}
        {% endblock %}
    </div>
    
    {% include 'footer.html' %}
    
    <!-- Bootstrap JS -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

### 5.3 header.html
```html
<header class="bg-light py-3">
    <div class="container">
        <h1 class="text-center mb-0">🏨 Sistema de Administración Hotel</h1>
    </div>
</header>
```

### 5.4 navbar.html
```html
<nav class="navbar navbar-expand-lg navbar-dark bg-dark">
    <div class="container">
        <a class="navbar-brand" href="{% url 'inicio_hotel' %}">
            🏨 Hotel CBTIS 128
        </a>
        
        <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav">
            <span class="navbar-toggler-icon"></span>
        </button>
        
        <div class="collapse navbar-collapse" id="navbarNav">
            <ul class="navbar-nav me-auto">
                <li class="nav-item">
                    <a class="nav-link" href="{% url 'inicio_hotel' %}">
                        🏠 Inicio
                    </a>
                </li>
                
                <li class="nav-item dropdown">
                    <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown">
                        👥 Huésped
                    </a>
                    <ul class="dropdown-menu">
                        <li><a class="dropdown-item" href="{% url 'agregar_huesped' %}">Agregar huésped</a></li>
                        <li><a class="dropdown-item" href="{% url 'ver_huespedes' %}">Ver huéspedes</a></li>
                    </ul>
                </li>
                
                <li class="nav-item dropdown">
                    <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown">
                        🛏️ Habitación
                    </a>
                    <ul class="dropdown-menu">
                        <li><a class="dropdown-item" href="#">Agregar habitación</a></li>
                        <li><a class="dropdown-item" href="#">Ver habitaciones</a></li>
                    </ul>
                </li>
                
                <li class="nav-item dropdown">
                    <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown">
                        📅 Reservas
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
```

### 5.5 footer.html
```html
<footer class="bg-dark text-white text-center py-3 mt-5">
    <div class="container">
        <p class="mb-1">&copy; 2024 Sistema de Administración Hotel - Todos los derechos reservados</p>
        <p class="mb-1">Fecha del sistema: {% now "d/m/Y" %}</p>
        <p class="mb-0">Creado por Ing. Eliseo Nava, Cbtis 128</p>
    </div>
</footer>
```

### 5.6 inicio.html
```html
{% extends 'base.html' %}

{% block content %}
<div class="row">
    <div class="col-md-6">
        <div class="card">
            <div class="card-body">
                <h2 class="card-title">Bienvenido al Sistema Hotel</h2>
                <p class="card-text">
                    Sistema de administración hotelera desarrollado con Django para la gestión 
                    eficiente de huéspedes, habitaciones y reservas.
                </p>
                <p class="card-text">
                    Características principales:
                </p>
                <ul>
                    <li>Gestión completa de huéspedes</li>
                    <li>Control de habitaciones</li>
                    <li>Sistema de reservas</li>
                    <li>Interfaz moderna y responsive</li>
                </ul>
            </div>
        </div>
    </div>
    <div class="col-md-6">
        <div class="card">
            <div class="card-body text-center">
                <img src="https://images.unsplash.com/photo-1566073771259-6a8506099945?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=800&q=80" 
                     alt="Hotel" class="img-fluid rounded" style="max-height: 400px;">
                <p class="mt-3 text-muted">Imagen representativa de instalaciones hoteleras</p>
            </div>
        </div>
    </div>
</div>
{% endblock %}
```

## 6. Templates de Huésped

### 6.1 agregar_huesped.html
```html
{% extends 'base.html' %}

{% block content %}
<div class="row justify-content-center">
    <div class="col-md-8">
        <div class="card">
            <div class="card-header bg-primary text-white">
                <h4 class="mb-0">➕ Agregar Nuevo Huésped</h4>
            </div>
            <div class="card-body">
                <form method="POST">
                    {% csrf_token %}
                    <div class="row">
                        <div class="col-md-6 mb-3">
                            <label class="form-label">Nombre</label>
                            <input type="text" class="form-control" name="nombre" required>
                        </div>
                        <div class="col-md-6 mb-3">
                            <label class="form-label">Apellido</label>
                            <input type="text" class="form-control" name="apellido" required>
                        </div>
                    </div>
                    
                    <div class="mb-3">
                        <label class="form-label">Email</label>
                        <input type="email" class="form-control" name="email" required>
                    </div>
                    
                    <div class="mb-3">
                        <label class="form-label">Teléfono</label>
                        <input type="text" class="form-control" name="telefono" required>
                    </div>
                    
                    <div class="mb-3">
                        <label class="form-label">Fecha de Nacimiento</label>
                        <input type="date" class="form-control" name="fecha_nacimiento" required>
                    </div>
                    
                    <div class="mb-3">
                        <label class="form-label">Dirección</label>
                        <textarea class="form-control" name="direccion" rows="3" required></textarea>
                    </div>
                    
                    <div class="d-grid gap-2">
                        <button type="submit" class="btn btn-primary">Guardar Huésped</button>
                        <a href="{% url 'ver_huespedes' %}" class="btn btn-secondary">Cancelar</a>
                    </div>
                </form>
            </div>
        </div>
    </div>
</div>
{% endblock %}
```

### 6.2 ver_huesped.html
```html
{% extends 'base.html' %}

{% block content %}
<div class="card">
    <div class="card-header bg-success text-white d-flex justify-content-between align-items-center">
        <h4 class="mb-0">👥 Lista de Huéspedes</h4>
        <a href="{% url 'agregar_huesped' %}" class="btn btn-light">➕ Agregar Huésped</a>
    </div>
    <div class="card-body">
        {% if huespedes %}
        <div class="table-responsive">
            <table class="table table-striped table-hover">
                <thead class="table-dark">
                    <tr>
                        <th>ID</th>
                        <th>Nombre</th>
                        <th>Apellido</th>
                        <th>Email</th>
                        <th>Teléfono</th>
                        <th>Acciones</th>
                    </tr>
                </thead>
                <tbody>
                    {% for huesped in huespedes %}
                    <tr>
                        <td>{{ huesped.id }}</td>
                        <td>{{ huesped.nombre }}</td>
                        <td>{{ huesped.apellido }}</td>
                        <td>{{ huesped.email }}</td>
                        <td>{{ huesped.telefono }}</td>
                        <td>
                            <a href="{% url 'actualizar_huesped' huesped.id %}" class="btn btn-warning btn-sm">✏️ Editar</a>
                            <a href="{% url 'borrar_huesped' huesped.id %}" class="btn btn-danger btn-sm">🗑️ Borrar</a>
                        </td>
                    </tr>
                    {% endfor %}
                </tbody>
            </table>
        </div>
        {% else %}
        <div class="alert alert-info text-center">
            <h5>No hay huéspedes registrados</h5>
            <p>Comienza agregando el primer huésped al sistema.</p>
            <a href="{% url 'agregar_huesped' %}" class="btn btn-primary">Agregar Primer Huésped</a>
        </div>
        {% endif %}
    </div>
</div>
{% endblock %}
```

### 6.3 actualizar_huesped.html
```html
{% extends 'base.html' %}

{% block content %}
<div class="row justify-content-center">
    <div class="col-md-8">
        <div class="card">
            <div class="card-header bg-warning text-dark">
                <h4 class="mb-0">✏️ Actualizar Huésped</h4>
            </div>
            <div class="card-body">
                <form method="POST">
                    {% csrf_token %}
                    <div class="row">
                        <div class="col-md-6 mb-3">
                            <label class="form-label">Nombre</label>
                            <input type="text" class="form-control" name="nombre" value="{{ huesped.nombre }}" required>
                        </div>
                        <div class="col-md-6 mb-3">
                            <label class="form-label">Apellido</label>
                            <input type="text" class="form-control" name="apellido" value="{{ huesped.apellido }}" required>
                        </div>
                    </div>
                    
                    <div class="mb-3">
                        <label class="form-label">Email</label>
                        <input type="email" class="form-control" name="email" value="{{ huesped.email }}" required>
                    </div>
                    
                    <div class="mb-3">
                        <label class="form-label">Teléfono</label>
                        <input type="text" class="form-control" name="telefono" value="{{ huesped.telefono }}" required>
                    </div>
                    
                    <div class="mb-3">
                        <label class="form-label">Fecha de Nacimiento</label>
                        <input type="date" class="form-control" name="fecha_nacimiento" value="{{ huesped.fecha_nacimiento|date:'Y-m-d' }}" required>
                    </div>
                    
                    <div class="mb-3">
                        <label class="form-label">Dirección</label>
                        <textarea class="form-control" name="direccion" rows="3" required>{{ huesped.direccion }}</textarea>
                    </div>
                    
                    <div class="d-grid gap-2">
                        <button type="submit" class="btn btn-warning">Actualizar Huésped</button>
                        <a href="{% url 'ver_huespedes' %}" class="btn btn-secondary">Cancelar</a>
                    </div>
                </form>
            </div>
        </div>
    </div>
</div>
{% endblock %}
```

### 6.4 borrar_huesped.html
```html
{% extends 'base.html' %}

{% block content %}
<div class="row justify-content-center">
    <div class="col-md-6">
        <div class="card">
            <div class="card-header bg-danger text-white">
                <h4 class="mb-0">🗑️ Confirmar Eliminación</h4>
            </div>
            <div class="card-body text-center">
                <h5>¿Estás seguro de que deseas eliminar al huésped?</h5>
                <p class="lead">{{ huesped.nombre }} {{ huesped.apellido }}</p>
                <p><strong>Email:</strong> {{ huesped.email }}</p>
                <p><strong>Teléfono:</strong> {{ huesped.telefono }}</p>
                
                <form method="POST" class="d-inline">
                    {% csrf_token %}
                    <button type="submit" class="btn btn-danger">Sí, Eliminar</button>
                </form>
                <a href="{% url 'ver_huespedes' %}" class="btn btn-secondary">Cancelar</a>
            </div>
        </div>
    </div>
</div>
{% endblock %}
```

## 7. Ejecutar el proyecto

### 7.1 Crear superusuario (opcional)
```bash
python manage.py createsuperuser
```

### 7.2 Ejecutar servidor en puerto 8036
```bash
python manage.py runserver 8036
```

### 7.3 Acceder al sistema
- **Sistema principal:** http://localhost:8036
- **Admin Django:** http://localhost:8036/admin

## Comandos finales de verificación

```bash
# Verificar que todo esté correcto
python manage.py check
python manage.py makemigrations
python manage.py migrate
python manage.py runserver 8036
```

El sistema estará completamente funcional para la gestión de huéspedes con una interfaz moderna y atractiva. Los modelos para habitaciones y reservas están definidos pero pendientes de implementar en las vistas.
UIII_Hotel_0743/
│
├── .venv/                          # Entorno virtual (se crea con: python -m venv .venv)
│
├── backend_Hotel/                  # Proyecto principal Django
│   ├── __init__.py
│   ├── settings.py                 # Configuración del proyecto
│   ├── urls.py                     # URLs principales
│   ├── wsgi.py
│   └── asgi.py
│
├── app_Hotel/                      # Aplicación principal
│   ├── __init__.py
│   ├── admin.py                    # Registro de modelos en admin
│   ├── apps.py
│   ├── models.py                   # Modelos: Huesped, Habitacion, Reserva, ReservaHabitacion
│   ├── tests.py
│   ├── views.py                    # Vistas para CRUD de huéspedes
│   ├── urls.py                     # URLs de la aplicación
│   │
│   └── templates/                  # Carpeta de templates
│       ├── base.html               # Template base con Bootstrap
│       ├── header.html             # Encabezado del sistema
│       ├── navbar.html             # Menú de navegación
│       ├── footer.html             # Pie de página fijo
│       ├── inicio.html             # Página de inicio
│       │
│       └── huesped/                # Subcarpeta para templates de huésped
│           ├── agregar_huesped.html
│           ├── ver_huesped.html    # Tabla con botones ver, editar, borrar
│           ├── actualizar_huesped.html
│           └── borrar_huesped.html
│
├── db.sqlite3                      # Base de datos (se crea automáticamente)
│
├── manage.py                       # Script de administración de Django
│
└── requirements.txt                # Dependencias del proyecto (opcional)
