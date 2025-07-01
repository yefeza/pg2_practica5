# Práctica 5

## Patrones de diseño en componentes propios

### Pasos para la práctica

1. Configurar e instalar dependencias en el entorno virtual de python.

   ```bash
   python -m venv env
   .\env\Scripts\activate  # En Windows
   # source env/bin/activate  # En Linux/Mac
   pip install -r requirements.txt
   ```

2. Crear un proyecto Django y una aplicación llamada `blog`.

   ```bash
   django-admin startproject api_patrones .
   python manage.py startapp pedidos_cafe
   ```

3. Configurar la base de datos en `settings.py` para usar SQLite.

   ```python
   INSTALLED_APPS = [
       ...
       'django_extensions',
       'rest_framework',
       'pedidos_cafe',
   ]
   ```

4. Crear el modelos en `pedidos_cafe/models.py`.

   ```python
   from django.db import models

   class PedidoCafe(models.Model):
        cliente = models.CharField(max_length=100)
        tipo_base = models.CharField(
            max_length=20,
            choices=[
                ("espresso", "Espresso"),
                ("americano", "Americano"),
                ("latte", "Latte"),
            ],
        )
        ingredientes = models.JSONField(default=list)
        tamanio = models.CharField(
            max_length=10,
            choices=[
                ("pequenio", "Pequeño"),
                ("mediano", "Mediano"),
                ("grande", "Grande"),
            ],
        )
        fecha = models.DateTimeField(auto_now_add=True)

   ```

5. Crear el serializer en `pedidos_cafe/serializers.py`.

   ```python
    from rest_framework import serializers
    from pedidos_cafe.models import PedidoCafe
    from pedidos_cafe.factory import CafeFactory
    from pedidos_cafe.builder import CafePersonalizadoBuilder, CafeDirector
    from api_patrones.logger import Logger


    class PedidoCafeSerializer(serializers.ModelSerializer):
        precio_total = serializers.SerializerMethodField()
        ingredientes_finales = serializers.SerializerMethodField()

        class Meta:
            model = PedidoCafe
            fields = [
                "id",
                "cliente",
                "tipo_base",
                "ingredientes",
                "tamanio",
                "fecha",
                "precio_total",
                "ingredientes_finales",
            ]

        def get_precio_total(self, obj):
            # Ver en `pedidos_cafe/factory.py`

        def get_ingredientes_finales(self, obj):
            # Ver en `pedidos_cafe/factory.py`
   ```

6. Crear las vistas en `pedidos_cafe/views.py`.

   ```python
    # Python native imports
    from datetime import timedelta

    # External imports
    from rest_framework import viewsets
    from django.utils import timezone as tz

    # Local imports
    from pedidos_cafe.serializers import *
    from pedidos_cafe.models import *
    from django.conf import settings


    class PedidoCafeViewSet(viewsets.ModelViewSet):
        queryset = PedidoCafe.objects.all().order_by("-fecha")
        serializer_class = PedidoCafeSerializer
   ```

7. Configurar las URLs en `pedidos_cafe/urls.py`.

   ```python
   # Django imports
   from django.contrib import admin
   from django.urls import path, include

   # Rest framework imports
   from rest_framework import routers

   # Local imports
   from pedidos_cafe.views import PedidoCafeViewSet

   # Create a router and register our viewset with it.
   router = routers.DefaultRouter()

   router.register(r"pedidos_cafe", PedidoCafeViewSet, basename="pedidos_cafe")


   urlpatterns = [
       path("admin/", admin.site.urls),
       path("api/", include(router.urls)),
   ]
   ```

8. Hacer la validación ingredientes extra en el registro de pedidos
