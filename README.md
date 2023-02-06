Выполнил работу
Аит Мансур Фарид Группа 035
 
Проделанное в работе
Изменил файлы пайчарма: Удалил api.py. Перенес часть кода в main.py

Перед работой с Nginx, сначала создал файл requirements.txt, который содержит список предустановок и файл Dockerfile. С помощью Dockerfile автоматизировал создание окружения, установку зависимостей, запуск проекта.

С помощью команды docker build -t main_app:0.0.1 . создал образ. С помощью команды docker run -p 8080:8080 main_app:0.0.1 запустил контейнер с пробросом портов. Удостоверился в создании и работе образа с помощью программы Docker Desktop.

В корне директории проекта создал файл docker-compose.yaml. Вставил следующий код:

version: '3.4'

services:
    web:
        build:
            context: .
        ports:
            - "8080-8081:8080"
Данная инструкция запустит в docker-compose приложение web. В build указывается место расположения образа (в нашем случае Dockerfile лежит в корне директории) на основе которого собирается контейнер. В ports открываем порты 8080 и 8081 которые будут связаны с портом 8080 внутри контейнера. Запустил контейнеры с помощью команды docker-compose up --build --scale web=2 и проверил, что отвечают оба.

Создал файл nginx.conf с:

worker_processes auto;

events {

}

http {
    upstream main-app {
        hash $cookie_key;
        server web:8080;
        server web:8081;
    }

    server {
        listen 8080;

        location / {
            proxy_pass http://main-app;
        }
    }
}
Данный конфиг указывает nginx слушать порт 80 и весь трафик проксировать на сервера перечисленные в upstream. В строке hash $cookie_key; Nginx будет брать хэш от cookie с именем key и если такой хэш уже был - направит трафик на тот же сервер. После поднимаю nginx вместе с приложением web, добавляя в файл docker-compose часть кода

   nginx:
          image: nginx:latest
          container_name: my-nginx
          depends_on:
            - web
          volumes:
              - ./nginx/nginx.conf:/etc/nginx/nginx.conf
          ports:
              - 80:80
              - 443:443
В image указывается какой официальный образ забрать при сборке. В Volumes монтируется наш конфиг из рабочей директории прямиком в контейнер. Это позволит нам изменять конфиг nginx и не пересобирать заново docker-compose, достаточно будет только перезапустить nginx контейнер. Подключаю приложение web и nginx к новой сети app-net добавив

   networks:
    app-net:
        driver: bridge
Благодаря этому в конфиге nginx можно указывать DNS сервиса (web), который определен в docker-compose. Удаляю запущенные контейнеры с помошью команды

docker-compose down 
и запускаю сервис вместе с nginx командой

docker-compose up --build --scale web=2
Запуск приложение и остановка
Для запуска приложения
  docker-compose up --build --scale web=2
Для остановки приложения
  docker-compose down
Выявленные проблемы
Из-за того, что проживаю в общежитеии провайдер блокирует 80 и 8080 порт, из-за чего работа встала и было тяжело её проверить на работоспособность. Вместо изменения порта с 80 на использование по дефолту 81 просто запустил 2 дублирующих сервера на 8080 и 8081 порты связанные с 8080 внутри контейнера.

Используемые интернет ресурсы:
https://fastapi.tiangolo.com/ru/
https://unit.nginx.org/howto/fastapi/
https://docs.docker.com/get-started/02_our_app/
https://habr.com/ru/company/domclick/blog/548610/
