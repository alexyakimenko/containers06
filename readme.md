# IWNO6: Взаимодействие контейнеров

## Цель работы

Выполнив данную работу студент сможет управлять взаимодействием нескольких контейнеров.

## Задание

Создать php приложение на базе двух контейнеров: nginx, php-fpm.

## Подготовка

Для выполнения данной работы необходимо иметь установленный на компьютере Docker.

Для выполнения работы необходимо иметь опыт выполнения лабораторной работы №3

## Выполнение

Создаю репозиторий `containers06` и копирую его себе на компьютер

В директорию `mounts/site` клонирую сайт на `php`, созданный в рамках предмета по `php`.

Выполняю команду

```sh
git submodule add https://github.com/alexyakimenko/php_assessment_1.git mounts/site/
```

Создаю файл `.gitignore` в корне проекта и добавляю в него строки:

```sh
# Ignore files and directories
mounts/site/*
```

Создаю в директории `containers05` `файл nginx/default.conf` со следующим содержимым

```nginx
server {
    listen 80;
    server_name _;

    root /var/www/html/public;
    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        fastcgi_pass backend:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

## Запуск и тестирование

Создаю сеть `internal` для контейнеров

```sh
docker network create internal
```

Создаю контейнер `backend` со следующими свойствами:

- на базе образа `php:8.3-fpm`
- к контейнеру примонтирована директория `mounts/site` в `/var/www/html`
- работает в сети `internal`

В `cmd` запускаю следующую команду

```cmd
docker container run -d --name backend --network internal -v %cd%\mounts\site:/var/www/html php:8.3-fpm
```

Создаю контейнер `frontend` со следующими свойствами:

- на базе образа `nginx:1.23-alpine`
- с примонтированной директорией `mounts/site` в `/var/www/html`
- с примонтированным файлом `nginx/default.conf` в `/etc/nginx/conf.d/default.conf`
- порт `80` контейнера проброшен на порт `80` хоста
- работает в сети `internal`

В `cmd` запускаю следующую команду

```cmd
docker container run -d --name frontend --network internal -p 80:80 -v %cd%/mounts/site:/var/www/html -v %cd%/nginx/default.conf:/etc/nginx/conf.d/default.conf nginx:1.23-alpine
```

Проверяю работу сайта в браузере, перейдя по адресу `http://localhost`

## Ответы на вопросы

> **Q:** Каким образом в данном примере контейнеры могут взаимодействовать друг с другом?  
> **A:** Контейнеры взаимодействуют через общую сеть `internal`

---

> **Q:** Как видят контейнеры друг друга в рамках сети `internal`?  
> **A:** Контейнеры видят друг друга по их именам

---

> **Q:** Почему необходимо было переопределять конфигурацию `nginx`?  
> **A:** Стандартная конфигурация `nginx` не настроена для работы с `php`, поэтому мы указали как обрабатывать запросы и куда их перебрасывать

## Вывод

В ходе лабораторной работы была реализована работа `php`-приложения на основе двух `docker`-контейнеров: один с `nginx`, второй с `php-fpm`. Контейнеры были объединены в пользовательскую сеть `internal`, что позволило им взаимодействовать между собой по именам.
