# docker - Домашка
1. Создайте свой кастомный образ nginx на базе alpine. 
* После запуска nginx должен отдавать кастомную страницу (достаточно изменить дефолтную страницу nginx)
* Определите разницу между контейнером и образом
* Вывод опишите в домашнем задании.
* Ответьте на вопрос: Можно ли в контейнере собрать ядро?
* Собранный образ необходимо запушить в docker hub и дать ссылку на ваш
репозиторий.

## Выполнение ДЗ
1. Создание кастомного образа nginx на базе alpine  (ссылка на хаб https://hub.docker.com/repository/docker/pawkalida/nginx)
```
создаём Dockerfile

FROM alpine:latest
ENV NGINX_VERSION nginx-1.19.1
EXPOSE 80
EXPOSE 443
RUN apk --update --no-cache add build-base \
        openssl-dev \
        pcre-dev \
        zlib-dev \
        wget

RUN mkdir -p /tmp/src && \
    cd /tmp/src && \
    wget http://nginx.org/download/${NGINX_VERSION}.tar.gz && \
    tar zxf ${NGINX_VERSION}.tar.gz && \
    cd ${NGINX_VERSION} && \
    ./configure --sbin-path=/usr/bin/nginx \
        --conf-path=/etc/nginx/nginx.conf \
        --pid-path=/var/run/nginx.pid \
        --error-log-path=/var/log/nginx/error.log \
        --http-log-path=/var/log/nginx/access.log \
        --with-pcre \
        --with-http_ssl_module && \
    make && \
    make install && \
    rm -rf /tmp/src
COPY ./alpine/index.html /usr/local/nginx/html/index.html
CMD ["nginx", "-g", "daemon off;"]
```
* Запускаем сборку образа командой: docker build -t pawkalida/nginx:v1 .
```
Sending build context to Docker daemon 73.73 kB
   ...
Removing intermediate container 91993033d2db
Successfully built 5317a5d59a6c
```
Смотрим наличие собранного образа docker images
```
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
pawkalida/nginx     v1                  5317a5d59a6c        14 seconds ago       224 MB
```
Подключаемся к хабу, проходим авторизацию и пушим созданный образ docker push pawkalida/nginx:v1
```
The push refers to a repository [docker.io/pawkalida/nginx]
0d40a7736bf1: Pushed
e17ef3bd5583: Pushed
fad2067ebcbb: Pushed
50644c29ef5a: Mounted from library/alpine
v1: digest: sha256:c64abec9300dc65ca0132889a3e8aa7b8c0f8bb97a32aa4deade866ad6f29d65 size: 1158
```
Запускаем контейнер и проверяем, перейдя по адресу виртуалки и изменённому порту командой
docker run -d -p 8686:80 pawkalida/nginx:v1
```
[root@centos docker]# docker ps -a
CONTAINER ID        IMAGE                COMMAND                  CREATED              STATUS                     PORTS                           NAMES
06900ba3cbcd        pawkalida/nginx:v1   "nginx -g 'daemon ..."   About a minute ago   Up About a minute          443/tcp, 0.0.0.0:8686->80/tcp   compassionate_murdock
```
![Image alt](https://github.com/paulDashkevich/docker/blob/master/dummy.png)
Разница между контейнером и образом
```
Контейнеры создаются из образов с помощью команды docker run
Чтобы создать контейнер, движок Docker берет образ, добавляет доступный для записи верхний слой и инициализирует различные параметры (сетевые порты, имя контейнера, идентификатор и лимиты ресурсов).
Все операции на запись внутри контейнера сохраняются в этом верхнем слое и когда контейнер удаляется, верхний слой, который был доступен для записи, также удаляется, в то время как нижние слоя остаются неизменными.
Таким образом: контейнер добавляет к образу (read-only) верхний слой, с которым можно выполнять нужные манипуляции.
+ c одного образа можно запустить одновременно много контейнеров
***  В контейнере можно собрать ядро, но не загружаться с него  ***
```
