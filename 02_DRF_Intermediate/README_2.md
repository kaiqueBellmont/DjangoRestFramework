# Intermediate Django rest Framework

## api/v1

- Generics APIs Views (generics.ListCreateAPIView):

```python
from rest_framework import generics
```

- GET ALL / POST (collections (without ID)) (plural/verbose):

```python
class CursosAPIView(generics.ListCreateAPIView):
    queryset = Curso.objects.all()
    serializer_class = CursoSerializer
```

- GET/UPDATE/DELETE:

```python
class CursoAPIView(generics.RetrieveUpdateDestroyAPIView):
    queryset = Curso.objects.all()
    serializer_class = CursoSerializer
```

- URLs (urls.py):
- ps: (you need to set plural and singular routes)

- Overriding generic methods:
  (You can click on generics, mixins and go to the views source code and override the methods, to meet any need or
  customize your views.)

### urls.py: (Overriding to find nested route)

#### - **all reviews for a specific course**:

```python
path('cursos/<int:curso_pk>/avaliacoes/', AvaliacoesAPIView.as_view(), name='curso_avaliacoes'),
```

- overriding the **get_queryset** ( GET ALL METHOD ) to make a nested route:

### views.py

```python
class AvaliacoesAPIView(generics.ListCreateAPIView):
    queryset = Avaliacao.objects.all()
    serializer_class = AvaliacaoSerializer

    def get_queryset(self):
        if self.kwargs.get('curso_pk'):
            return self.queryset.filter(curso_id=self.kwargs.get('curso_pk'))
        return self.queryset.all()
```

-----
This checks if the queryset is getting any id of a course to make a filter and display only the evaluations of that
particular course.
-----

### urls.py: (Overriding to find nested route)

#### **specific evaluation of a specific course**:

```python
path('cursos/<int:curso_pk>/avaliacoes/<int:avaliacao_pk>/', AvaliacaoAPIView.as_view(), name='curso_avaliacao'),
```

- overriding the **get_object** ( GET/pk ) to make a nested route:

### views.py

```python
class AvaliacaoAPIView(generics.RetrieveUpdateDestroyAPIView):
    queryset = Avaliacao.objects.all()
    serializer_class = AvaliacaoSerializer

    def get_object(self):
        if self.kwargs.get('curso_pk'):
            return get_object_or_404(self.get_queryset(),
                                     curso_id=self.kwargs.get('curso_pk'),
                                     pk=self.kwargs.get('avaliacao_pk'))
        return get_object_or_404(self.get_queryset(), pk=self.kwargs.get('avaliacao_pk'))
```

-----
this method does something similar to a "join"/filter.
it passes the queryset, filters by foreign key and pk.
If it does not find the FK, it returns the evaluation with the last ID.
-----

### the above overriding methods become useful for creating nested routes.

-----

## api/v2

## viewsets and routers

**Routers serve to "automate" the creation of endpoints.
With routers, we can do the same thing as **generics**, but without having to distinguish between plural and singular.
(it's a much more advanced feature)**

### views.py

```python
from rest_framework import viewsets


class CursoViewSet(viewsets.ModelViewSet):
    queryset = Curso.objects.all()
    serializer_class = CursoSerializer


class AvaliacaoViewSet(viewsets.ModelViewSet):
    queryset = Avaliacao.objects.all()
    serializer_class = AvaliacaoSerializer
```

-------

**this is already enough for the creation of the api's CRUD.**

------

#### Router:

##### app/urls.py:

```python
from rest_framework.routers import SimpleRouter

# this registers/links the models with the views
router = SimpleRouter()
router.register('cursos', CursoViewSet)
router.register('avaliacoes', AvaliacaoViewSet)
```

##### project/urls.py:

```python
from django.contrib import admin
from django.urls import path, include

from cursos.urls import router

urlpatterns = [
    path('api/v1/', include('cursos.urls')),

    # import the router from app/urls.py and include the routes in the project.
    path('api/v2/', include(router.urls)),

    path('admin/', admin.site.urls),
    path('auth/', include('rest_framework.urls')),
]
```

### important: the modelViewset, generates only CRUD operations of a single model.

### nested routes wont works good.

## solving the nested routes with router problem:

### views.py:

#### # with action, we can change actions inside our modelviewset

```python
from rest_framework.decorators import action
from rest_framework.response import Response
```

----

With the "action" decorator, we can create a new route inside the modelviewset.
In the case of the example above, we can do this easily because in the models there is a field [related_name]

detail=True -> Create a new route. <br>
detail=False -> dont create. <br>
methods= crud method you need.

----

### in this way, we can bring all the evaluations of a certain course, thus solving the "modelviewset curse"

----

### You can also create a custom ViewSet, just inheriting the mixins you want to use.

### In this way:

```python
class AvaliacaoViewSet(
    mixins.ListModelMixin,
    mixins.CreateModelMixin,
    mixins.RetrieveModelMixin,
    mixins.UpdateModelMixin,
    mixins.DestroyModelMixin,
    # Agrupa todos os outros
    viewsets.GenericViewSet
):
    queryset = Avaliacao.objects.all()
    serializer_class = AvaliacaoSerializer
```

#### If you want to create a restriction on the view, just comment some mixinn, which the method is already deprived of.

----------------

## relations with the DRF:

### Nested relationship:

#### to add nested relationships in django, just add the field in serielizers,
#### respecting the relationship that is in models.

### e.g:
```python
class CursoSerializer(serializers.ModelSerializer):
    # Nested Relationship
    avaliacoes = serializers.PrimaryKeyRelatedField(many=True, read_only=True)

    class Meta:
        model = Curso
        fields = (
            'id',
            'titulo',
            'url',
            'criacao',
            'ativo',
            
            # here
            'avaliacoes'
        )
```

-----
## django pagination:
### settings.py
```python
# DRF
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework.authentication.SessionAuthentication',
    ),
    'DEFAULT_PERMISSION_CLASSES': (
        'rest_framework.permissions.IsAuthenticatedOrReadOnly',
    ),
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 2
}

```

### models.py
#### for consistency, add to the models how the data should be ordered.

```python
class Meta:
  verbose_name = 'Curso'
  verbose_name_plural = 'Cursos'
  # this
  ordering = ['id']
```
#### PS: the global Pagination does not influence the extra methods you create in modelsViewsets.
#### We must create our own pagination.

```python
class CursoViewSet(viewsets.ModelViewSet):
    queryset = Curso.objects.all()
    serializer_class = CursoSerializer

    @action(detail=True, methods=['get'])
    def avaliacoes(self, request, pk=None):
        curso = self.get_object()
        serializer = AvaliacoesSerilizer(curso.avaliacoes.all(), many=True)
        return Response(serializer.data)
```
### With pagination:
```python
class CursoViewSet(viewsets.ModelViewSet):
    queryset = Curso.objects.all()
    serializer_class = CursoSerializer

    @action(detail=True, methods=['get'])
    def avaliacoes(self, request, pk=None):
        self.pagination_class.page_size = 2
        avaliacoes = Avaliacao.objects.filter(curso_id=pk)
        page = self.paginate_queryset(avaliacoes)

        if page is not None:
            serializer = AvaliacaoSerializer(page, many=True)
            return self.get_paginated_response(serializer.data)

        serializer = AvaliacaoSerializer(avaliacoes, many=True)
        return Response(serializer.data)
```