# DRF Advanced

## token authentication

--------

### tip:

#### - Auth session used in  *browser*

#### - Token used in *another any device*

-----

#### start by putting the line of code in "INSTALLD_APPS": (dont need install)

#### settings.py

```python
...
'rest_framework.authtoken',
...
```

#### then, put the follow config in "REST_FRAMEWORK":

```python

# DRF
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        # THIS        
        'rest_framework.authentication.TokenAuthentication',
    ),
    ...
}
```

#### Note:

#### when you save this configuration django will create a new table, called: *authtoken*, where DRF keeps the tokens.

## Using JWT:

### Follow steps:

```
1- Install the package by running pip install djangorestframework-jwt
2 - Add 'rest_framework_jwt' to your INSTALLED_APPS setting in your Django project's settings.py file
3 - Add 'rest_framework_jwt.authentication.JSONWebTokenAuthentication' to your REST_FRAMEWORK setting in your
Django project's settings.py file.
4 - In your views, include authentication_classes and specify JSONWebTokenAuthentication
as one of the authentication classes, e.g.


    authentication_classes = (JSONWebTokenAuthentication,)
    permission_classes = (IsAuthenticated,)
    def post(self, request):
        ...

5 - To get the user associated with a token, use request.user in your views.
Please note that this setup would work as long as your JWT's are correctly encoded and signed, if not then you
need to configure the settings for encoding and decoding of JWT.
```

### how is the config in settings.py:

```python

import datetime

JWT_AUTH = {
    'JWT_PAYLOAD_HANDLER':
        'rest_framework_jwt.utils.jwt_payload_handler',

    'JWT_PAYLOAD_GET_USERNAME_HANDLER':
        'rest_framework_jwt.utils.jwt_get_username_from_payload_handler',

    'JWT_RESPONSE_PAYLOAD_HANDLER':
        'rest_framework_jwt.utils.jwt_response_payload_handler',

    'JWT_SECRET_KEY': 'secret_key',
    'JWT_ALGORITHM': 'HS256',
    'JWT_VERIFY': True,
    'JWT_VERIFY_EXPIRATION': True,
    'JWT_LEEWAY': 0,
    'JWT_EXPIRATION_DELTA': datetime.timedelta(days=7),
    'JWT_AUDIENCE': None,
    'JWT_ISSUER': None,

    'JWT_ALLOW_REFRESH': True,
    'JWT_REFRESH_EXPIRATION_DELTA': datetime.timedelta(days=7),

    'JWT_AUTH_HEADER_PREFIX': 'Bearer',
}

REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': (
        'rest_framework.permissions.IsAuthenticated',
    ),
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_jwt.authentication.JSONWebTokenAuthentication',
    ),
}

```

-----------

```
This is a basic configuration, but you can adjust the settings as per your need. For example, you can change the
 algorithm used to sign the token, the expiration time, and other details.

JWT_SECRET_KEY: This is the secret key that will be used to sign the JWT.
 It must be kept secret, as anyone with this key can create valid JWTs.

JWT_ALGORITHM: This is the algorithm used to sign the JWT. By default it is HS256 which is a symmetric algorithm using HMAC.

JWT_EXPIRATION_DELTA: This is the amount of time before the token expires. It is a datetime.timedelta object

JWT_AUTH_HEADER_PREFIX: This is the prefix of the Authorization header. The default is 'Bearer'

Also you can define the payload and response handlers, and other details in the JWT_AUTH dictionary.

You also need to add 'rest_framework_jwt' to your INSTALLED_APPS setting in your Django project's settings.py file.
```

### it will set automatically the token to the user models?

```
The Django REST framework JWT library does not automatically set the token to the user models. The token is created
and returned to the client when the client successfully authenticates with a username and password, and the token
is then sent with each subsequent request to the server in the Authorization header, the server will validate the
token and set the user to the request object.

You can either generate the token by yourself or you can use the provided views and serializers in the package
to generate the token.

For example, you can use the ObtainJSONWebToken view to handle the login process and return the token to the client.
And then use the jwt_response_payload_handler to customize the response payload.

You can also use the JSONWebTokenSerializer to validate the credentials and return the token, like this:

```
```python
from rest_framework_jwt.serializers import JSONWebTokenSerializer

class LoginView(APIView):
    def post(self, request):
        serializer = JSONWebTokenSerializer(data=request.data)
        if serializer.is_valid():
            user = serializer.object.get('user') or request.user
            token = serializer.object.get('token')
            response_data = jwt_response_payload_handler(token, user, request)
            return Response(response_data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

```

```
You can also use the jwt_response_payload_handler to customize the response payload.

It's important to note that the token represents the user's authentication state and should be kept secret at all times.
It should not be stored in the user model or in the client-side storage.
The token should only be sent over a secure connection (HTTPS) and should be stored in the client's memory (e.g.
in a cookie or local storage) and sent with each request to the server for authentication and authorization.
```

### Final notes:
#### You can use JWT easily in DRF, just set the right configurations and create a user with token, an abstract example:

```python
from rest_framework import serializers
from django.contrib.auth.models import User
from rest_framework_jwt.settings import api_settings

class UserSerializer(serializers.ModelSerializer):
    token = serializers.SerializerMethodField()
    password = serializers.CharField(write_only=True)

    def get_token(self, obj):
        jwt_payload_handler = api_settings.JWT_PAYLOAD_HANDLER
        jwt_encode_handler = api_settings.JWT_ENCODE_HANDLER

        payload = jwt_payload_handler(obj)
        token = jwt_encode_handler(payload)
        return token

    def create(self, validated_data):
        user = User.objects.create(
            username=validated_data['username']
        )
        user.set_password(validated_data['password'])
        user.save()
        return user

    class Meta:
        model = User
        fields = ('id', 'username', 'password', 'token')
```

--------
## PERMISSION_CLASSES:
### Notes:
#### You can make this comand in the shell:
```python
from rest_framework import permissions
dir(permissions)
```
#### and you will see all permissions available in *django permission classes*

-------
### settings.py:

```python

# DRF
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        # 'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.TokenAuthentication',
    ),
    'DEFAULT_PERMISSION_CLASSES': (
        'rest_framework.permissions.IsAuthenticatedOrReadOnly',
    )
    ...
```

### views.py:

```python
from rest_framework import permissions

...
...
class CursoViewSet(viewsets.ModelViewSet):
    # HERE
    permission_classes = (permissions.DjangoModelPermissions, )
    
    queryset = Curso.objects.all()
    serializer_class = CursoSerializer
    
...

```
--------
## Notes:
### Its a Tuple, so put the (permissions.DjangoModelPermissions, ) [for a less a comma in case of one value]
*if you put a custom permission on your view, it will only count that permission.
django stops following global permissions on this view.*
<br>
*(You can see the user's permission level in django's admin area.)*
-------

## Bonus:
## Creating a custom permission class to your project:
### 1- Start creating a .py file called  *permissions.py*
### one simple ecample:
```python
from rest_framework import permissions


class EhSuperUser(permissions.BasePermission):

    def has_permission(self, request, view):
        if request.method == 'DELETE':
            if request.user.is_superuser:
                return True
            return False
        return True
```
 
*You just create a class and extend it from the permissions, manipulating according to the model group, request view, etc...*

### Full example:
```python
from rest_framework import permissions
from .permissions import EhSuperUser
...
...


class CursoViewSet(viewsets.ModelViewSet):
    permission_classes = (
        EhSuperUser,
        permissions.DjangoModelPermissions,
    )
    queryset = Curso.objects.all()
    serializer_class = CursoSerializer
    ...
```
### Notes:
*With more than 1 permission in the class, there is a hierarchy, where if the first can supply,
it does not check the second and so on.*

#### the order makes a difference.

--------

## Limiting the number of requests with Throttling
### Definition:
*API throttling is the process of limiting the number of API requests a user can make in a certain period.
An application programming interface (API) functions as a gateway between a user and a software application.
For example, when a user clicks the post button on social media, the button click triggers an API call.*

### Start in the settings.py.
### In the DRF config:

```python
# DRF
REST_FRAMEWORK = {
    ...
    'DEFAULT_THROTTLE_CLASSES': (
        'rest_framework.throttling.AnonRateThrottle',
        'rest_framework.throttling.UserRateThrottle',
    ),
    'DEFAULT_THROTTLE_RATES': {
        'anon': '5/minute',  # second, day, month, year
        'user': '10/minute'
    },
    ...
}
```
**in this example, we limit the use of requests from the two user classes we have.
(anonymous users and logged in users)**

### Notes:
#### The settings are global, just use this setting.
## curiosity:
### django counts this through a mechanism called cache-back (for local memory) extremely performant in development,
### but for production it is strongly recommended to use databases for caching, such as redis, for example.