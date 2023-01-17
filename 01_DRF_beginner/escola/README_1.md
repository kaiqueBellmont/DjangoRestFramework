# Basic Django rest Framework

- APIViews (class based views do drf ):

```python
class CursoAPIView(APIView):
    def get(self, request):
        cursos = Curso.objects.all()
        serilizer = CursoSerializer(cursos, many=True)
        return Response(serilizer.data)
```

- Simple serializers:

```python
class AvaliacaoSerializer(serializers.ModelSerializer):
    class Meta:
        extra_kwargs = {
            'email': {'write_only': True}
        }
        model = Avaliacao
        fields = (
            'id',
            'curso',
            'nome',
            'email',
            'comentario',
            'avaliacao',
            'criacao',
            'ativo'
        )

```

- [Simple models](01_DRF_beginner/escola/cursos/models.py):

```python
class Base(models.Model):
    criacao = models.DateTimeField(auto_now_add=True)
    atualizacao = models.DateTimeField(auto_now=True)
    ativo = models.BooleanField(default=True)

    class Meta:
        abstract = True


class Curso(Base):
    titulo = models.CharField(max_length=255)
    url = models.URLField(unique=True)

    class Meta:
        verbose_name = 'Curso'
        verbose_name_plural = 'Cursos'

    def __str__(self):
        return self.titulo
```
- Simple URLs:
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('api/v1/', include('cursos.urls')),
    path('admin/', admin.site.urls),
    path('auth/', include('rest_framework.urls')),
]

```
- DRF settings.py config:
```python
# DRF
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework.authentication.SessionAuthentication',
    ),
    'DEFAULT_PERMISSION_CLASSES': (
        'rest_framework.permissions.IsAuthenticatedOrReadOnly',
    )
}
```
#### This only makes sure that only logged users (with session) can make requests other than get
