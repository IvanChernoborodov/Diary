


## **Путь к пониманию Celery.**

Паттерн: *producer/consumer - производитель/потребитель(воркер)*


У веб-приложений существует **2** типа операций:

 - request-time -выполняется в цикле запрос ответ (является главным показателем скорости работы)
 - фоновые задачи - то, что происходит после отправки ответа с сервера (вне цикла запрос-ответ).

Итак, предположим есть сервис со следующим функционалом:
 -   Отправка уведомлений о подтверждении или рассылка сообщений
 -   Ежедневное сканирование и скрапинг некоторой информации из разных источников и сохранение их
 -   Анализ данных
 -   Удаление ненужных ресурсов
 -   экспорт документов/фотографии в различных форматах


Если все эти операции выполняются в цикле запрос-ответ, то это печаль. Даже с ростом трафика отправка уведомлений о успешной регистрации становится критичным. Так что нужно выносить эти задачи в фоновую обработку. 	

> Наиболее распространенным шаблоном программирования, используемым для  этого сценария, является  **Producer-Consume**r паттерн(архитектура). Его смысл состоит в том, что один или несколько потоков _производят_ данные, и параллельно этому один или несколько потоков _потребляют_ их.

Основные понятия:

 - Задача(task)
 - Производитель/клиент(producers)
 - Потребитель(worker/consumer)
 - Посредник(brocker)
 - Очередь(queue)

**Как работает эта архитектура:**
-   Producers(производители, в данном случае клиенты) создают данные или задачи(имеется в виду, что в ходе выполнения request-time фукций,  производится инициализация тех задач, которые мы хотим отправить в фоновую обработку). В Django например это может быть любая функция, которая обрабатывает HttpRequest, внутри этой функции и будет вызов нашей задачи(task).
-   Задачи(tasks помещаются в очередь(queue), которая называется очередью задач(task queues).
-   Consumers(воркеры) несут ответственность за потребление данных или выполнение задач.

Разберем более подробно.
 Celery работает через сообщения, обычно используя посредника(**broker**) для взаимодействия между клиентами(**producers**) и работниками(**workers**). Чтобы инициировать задачу, клиент(**producer**) добавляет сообщение в очередь(**queue**), затем посредник(**broker**) передает это сообщение работнику(**worker**).
 Чтобы было проще это понять - вот схема:
 ![](https://2.bp.blogspot.com/-wKQ7PqXnzCo/Whw-722kzCI/AAAAAAAAAA4/9vysZxmKHVA_ex7V4gPVeJraqkcBroWRgCK4BGAYYCw/s640/celery_architecture.jpg)

Итак, чтобы начать работу, нам нужно выбрать инструмент, который будет обрабатывать передачу сообщений.
В официальной документации мы можем увидеть -

> The RabbitMQ and Redis broker transports are feature complete, but
> there’s also support for a myriad of other experimental solutions,
> including using SQLite for local development.

Итого: два самых популярных решения
 **1. RabbitMQ
 2. Redis**
 
 
 
## Пример (Redis)

Вынесем функцию отправки email в фоновую обработку.

 1. Устанавливаем `celery`
 2. Устанавливаем `redis`
 3.  Добавляем файл `celery.py`
 4. Определяем что будем выносить в фоновую обработку
 5.  Добавляем файл `tasks.py`
 6. Тестируем
 7. Фоновое выполнение
 
 
 
 
    1.  `pip install celery`
2. ``pip install redis`` 
  Добавляем `django-redis` в `INSTALLED_APPS`
  Так же добавим конфигурации для redis сервера.

		REDIS_HOST = 'localhost'
		REDIS_PORT = '6379'
		BROKER_URL = 'redis://' + REDIS_HOST + ':' + REDIS_PORT + '/0'
		BROKER_TRANSPORT_OPTIONS = {'visibility_timeout': 3600} 
		CELERY_RESULT_BACKEND = 'redis://' + REDIS_HOST + ':' + REDIS_PORT + '/0'


3. Добавляем файл celery.py в директорию приложения

 

        import os
        from celery import Celery
        os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'app_name.settings')
        app = Celery('app_name')
        app.config_from_object('django.conf:settings')
        #Load task modules from all registered Django app configs.
        app.autodiscover_tasks()


4. Предположим у нас во вьюхе есть код, который будет отправлять сообщение на почту пользователю. 

	    def signup(request):
	    	html = 'registration/signup.html'
	    	if not request.user.is_authenticated:
	    		if request.method == 'POST':
	    			form = SignUpForm(request.POST)
	    			if form.is_valid():
	    				user = form.save(commit=False)
	    				user.is_active = False
	    				user.save()
	    				current_site = get_current_site(request)
	    				mail_subject = 'Активация вашего аккаунта'
	    				uid = urlsafe_base64_encode(force_bytes(user.pk)).decode()
	    				token = account_activation_token.make_token(user)
	    				message = render_to_string('acc_active_email.html', {
	    					'user': user,
	    					'domain': current_site.domain,
	    					'uid': uid,
	    					'token': token,
	    				})
	    				to_email = form.cleaned_data.get('email')
	    				email = EmailMessage(
	    					mail_subject, message, to=[to_email]
	    				)
	    				#Эту функцию нужно вынести в фоновую обработку
	    				email.send()
	 	    			return HttpResponse('Пожалуйста, подтвердите ваш email адрес для завершения регистрации, ссылка с активацией отправлена на ваш почтовый ящик')
	    		else:
	    			form = SignUpForm()
	    	else:
	    		return redirect('/account')
	    	return render(request, html, {'form': form})

5. Создаем файл `tasks.py` в директории приложения и выносим туда функцию отправки письма.

	     -*- coding: utf-8 -*-
	    from __future__ import unicode_literals
	    from users.celery import app
	    from celery import shared_task
	    
	    #@app.task
	    '''
	    The `@shared_task` decorator lets you create tasks without having any concrete app instance
	    '''
	    @shared_task
	    def sending_mail(mail_subject, message, to_email):
	    	email = EmailMessage(
	    				mail_subject,
	    				message,
	    				to=[to_email]
	    				)
	    	email.send()

Возвращаемся в `views.py`и импортируем нашу фоновую функцию.

    from users.tasks import sending_mail

Добавляем функцию в нашу вьюху, в то место, где была вызвана отправка письма

    #Эту функцию нужно вынести в фоновую обработку
    #email.send()
    #Обратите внимание, как мы вызываем метод `.delay` объекта задачи. Это означает, что мы отправляем задание на Celery, и мы не ожидаем результата.
    sending_mail.delay(mail_subject, message, to_email)
6. Тестируем.
Celery - это сервис, и нам нужно его запустить. Откройте новую консоль, убедитесь, что вы активируете соответствующий virtualenv, и перейдите в каталог проекта
`celery worker -A app_name --loglevel=debug --concurrency=4`
Эта команда запустит четыре процесса воркеров Celery.  Обратите внимание на то, что нет задержки, следите за логами в консоли Celery и посмотрите, правильно ли выполняются задачи. 
7. Фоновое выполнение 
https://realpython.com/asynchronous-tasks-with-django-and-celery/#running-remotely

