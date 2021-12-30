# Django site

Докеризированный сайт на Django для экспериментов с Kubernetes.

Внутри конейнера Django запускается с помощью Nginx Unit, не путать с Nginx. Сервер Nginx Unit выполняет сразу две функции: как веб-сервер он раздаёт файлы статики и медиа, а в роли сервера-приложений он запускает Python и Django. Таким образом Nginx Unit заменяет собой связку из двух сервисов Nginx и Gunicorn/uWSGI. [Подробнее про Nginx Unit](https://unit.nginx.org/).

## Как запустить dev-версию

Запустите базу данных и сайт:

```shell-session
$ docker-compose up
```

В новом терминале не выключая сайт запустите команды для настройки базы данных:

```shell-session
$ docker-compose run web ./manage.py migrate  # создаём/обновляем таблицы в БД
$ docker-compose run web ./manage.py createsuperuser
```

Для тонкой настройки используйте переменные окружения. Список доступных переменных можно найти внутри файла `docker-compose.yml`.

## Переменные окружения

Образ с Django считывает настройки из переменных окружения:

`SECRET_KEY` -- обязательная секретная настройка Django. Это соль для генерации хэшей. Значение может быть любым, важно лишь, чтобы оно никому не было известно. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#secret-key).

`DEBUG` -- настройка Django для включения отладочного режима. Принимает значения `TRUE` или `FALSE`. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#std:setting-DEBUG).

`ALLOWED_HOSTS` -- настройка Django со списком разрешённых адресов. Если запрос прилетит на другой адрес, то сайт ответит ошибкой 400. Можно перечислить несколько адресов через запятую, например `127.0.0.1,192.168.0.1,site.test`. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#allowed-hosts).

`DATABASE_URL` -- адрес для подключения к базе данных PostgreSQL. Другие СУБД сайт не поддерживает. [Формат записи](https://github.com/jacobian/dj-database-url#url-schema).

## Как запустить вебсайт в Minikube
1. Установить `minikube` и `kubectl`, а также Docker и Virtualbox
2. Сделать `git clone`
3. Установить [Helm](https://helm.sh/) и [Helm Chart for Postgres](https://artifacthub.io/packages/helm/bitnami/postgresql) для локальной Postgres или же поднять БД другим способом
4. Сформировать url БД формата `postgres://USER:PASSWORD@HOST:PORT/NAME`
5. Внести в django-config переменные среды - `DEBUG`, `DATABASE_URL`, `ALLOWED_HOSTS`, `SECRET_KEY`
6. Добавить Docker-образ в minikube командой `minikube image build -t django_app backend_main_django`
7. Создать ConfigMap командой `kubectl apply -f django-config.yaml`
8. Создать ClusterIP и Deployment командой `kubectl apply -f django-service.yaml`
9. Создать Ingress для сервиса командой `kubectl apply -f django-ingress.yaml`
10. Выполнить миграции через `kubectl apply -f migrations-job.yaml`, дождаться выполнения команды и удалить Job командой `kubectl delete job django-migrate`
11. Запустить CrontabJob для очистки сессий командой `kubectl apply -f django-clearsessions.yaml`
12. Получить IP Minikube командой `minikube ip` и добавить строку формата `YOUR_IP YOUR_DOMAIN` в файл `hosts`
13. Создать суперпользователя:
 * Командой `kubectl exec <django_pod_name> -it -- bs` подключиться к поду.
 * Командой `python manage.py createsuperuser` создать пользователя

