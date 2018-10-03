Стандартный url 

    /api

json formating

    /api/?format=json


Добавление permission 

    from rest_framework.permissions import IsAdminUser
Внутри вьюхи
	

    permission_classes = (IsAdminUser,)



## Secure issue
> There’s one last step we need to do and that’s deal with [Cross-Origin
> Resource Sharing
> (CORS)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS).
> Whenever a client interacts with an API hosted on a **different
> domain** there are potential security issues. CORS requires the server
> to include specific HTTP headers that allow the browser to determine
> if and when cross-domain requests should be allowed.
The easiest way to handle this–and the one [recommended by DRF](http://www.django-rest-framework.org/topics/ajax-csrf-cors/)–is to use **middleware that will automatically include the appropriate HTTP headers based on our settings**.
> 
> 
> The recommended package is
> [django-cors-headers](https://github.com/ottoyiu/django-cors-headers/)
> which can be easily added to our existing project.
Это мидлварь, которая добавляет в хедер токен.

    pip install django-cors-headers

settings.py
```
corsheaders
```
Добавляем мидлварь
```
'corsheaders.middleware.CorsMiddleware', # new
'django.middleware.common.CommonMiddleware', # new
```




**Вернуть весь список** 
Метод: `GET`

    api/

**Обратиться к объекту по id** 
Метод: `GET`

    api/id 

 например`api/1`

**Добавить объект** 
Метод: `POST`
url: `api/`
Пример curl:

    curl -X POST -d "title='NEWTITLE'&description='NEWDESCR'" http://localhost:8000/api/

**Удалить объект**
Метод: `DELETE`
url: `api/id`
Пример curl:

    curl -X DELETE http://127.0.0.1:8000/api/1/

**Обновить объект**
метод: `PUT`
url: `/api/id`
Пример curl:

    curl -X PUT -d "title='XXX'&description='adfasdf'" http://localhost:8000/api/2/

