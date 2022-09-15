## ДЗ 12: Технология контейнеризации. Введение в Docker

В процессе выполнения ДЗ было сделано

1. Установка Docker
2. Работа с Docker, создания образа, запуск контейнера
3. Использование базовых команд Docker
4. Сравнение Docker образа и Docker контейнера


## ДЗ 13: Docker-контейнеры

В процессе выполнения ДЗ было сделано

1. Установка Docker mashine
2. Поднятие инстанса в YC и инициализация на нем докер хост системы
3. Создание Dockerfile для приложения reddit, создание образа
4. Публикация образа на Docker Hub

Команды YC:

```
# создание сети
yc vpc network create app-network

# создание подсети
yc vpc subnet create --name app-subnet

# создание инстанса
yc compute instance create \
 --name docker-host \
 --cores=2 \
 --core-fraction=5 \
 --memory=4 \
 --create-boot-disk image-folder-id=standard-images,image-family=ubuntu-1804-lts,size=15GB \
 --network-interface subnet-name=app-subnet,nat-ip-version=ipv4 \
 --zone=ru-central1-a \
 --metadata serial-port-enable=1 \
 --ssh-key ~/.ssh/yc-user.pub
```

Команды Docker mashine:

```
# создание
docker-machine create \
  --driver generic \
  --generic-ip-address=62.84.114.108 \
  --generic-ssh-user yc-user \
  --generic-ssh-key ~/.ssh/id_rsa \
  docker-host

# просмотр
docker-machine ls

# удаление
docker-machine rm docker-host
```

Команды Docker:

```
# сборка образа из Dockerfile
docker build -t reddit:latest .

# отправка образа в Docker Hub
docker login
docker tag reddit:latest astrviktor/otus-reddit:1.0
docker push astrviktor/otus-reddit:1.0

# запуск контейнера
docker run --name reddit -d -p 9292:9292 astrviktor/otus-reddit:1.0
```

## ДЗ 14: Docker-образы. Микросервисы

В процессе выполнения ДЗ было сделано

1. Разбивка приложения reddit на отдельные образы
2. Оптимизация образов по размеру
3. Работа с Docker volume

Команды сборки образов:

```
docker pull mongo:latest
docker build -t astrviktor/post:1.0 ./post-py
docker build -t astrviktor/comment:1.0 ./comment
docker build -t astrviktor/ui:1.0 ./ui
```

Команды запуска контейнеров:

```
docker network create reddit
docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db mongo:latest
docker run -d --network=reddit --network-alias=post astrviktor/post:1.0
docker run -d --network=reddit --network-alias=comment astrviktor/comment:1.0
docker run -d --network=reddit -p 9292:9292 astrviktor/ui:1.0
```

Создание Docker volume:

```
docker volume create reddit_db
docker run -d --network=reddit --network-alias=post_db \
 --network-alias=comment_db -v reddit_db:/data/db mongo:latest
```


## ДЗ 15: Docker: сети, docker-compose

В процессе выполнения ДЗ было сделано

1. Изучение работы с сетями в docker
2. Запуск через docker-compose
3. Использование переменных для docker-compose

Команды для проверки и сравнения сетевых интерфейсов

```
docker run -ti --rm --network none joffotron/docker-net-tools -c ifconfig
docker run -ti --rm --network host joffotron/docker-net-tools -c ifconfig 
docker run --network host -d nginx

docker kill $(docker ps -q)
```

Создание bridge сети и запуск проекта в этой сети

```
docker network create reddit --driver bridge

docker run -d --network=reddit mongo:latest
docker run -d --network=reddit astrviktor/post:1.0
docker run -d --network=reddit astrviktor/comment:1.0
docker run -d --network=reddit -p 9292:9292 astrviktor/ui:1.0 

docker run -d --network=reddit --network-alias=post_db --networkalias=comment_db mongo:latest
docker run -d --network=reddit --network-alias=post astrviktor/post:1.0
docker run -d --network=reddit --network-alias=comment astrviktor/comment:1.0
docker run -d --network=reddit -p 9292:9292 astrviktor/ui:1.0

docker kill $(docker ps -q)
```

Создание 2-x bridge сетей и запуск проекта в 2-х сетях

```
docker network create back_net --subnet=10.0.2.0/24
docker network create front_net --subnet=10.0.1.0/24

docker run -d --network=front_net -p 9292:9292 --name ui astrviktor/ui:1.0
docker run -d --network=back_net --name comment astrviktor/comment:1.0
docker run -d --network=back_net --name post astrviktor/post:1.0
docker run -d --network=back_net --name mongo_db \
 --network-alias=post_db --network-alias=comment_db mongo:latest 

docker network connect front_net post
docker network connect front_net comment
 
docker kill $(docker ps -q)
```

Запуск через docker-compose

```
export USERNAME=astrviktor
docker-compose up -d
docker-compose ps
docker-compose down
```

Поменять базовое имя проекта в docker-compose

```
docker-compose --project-name <name> up -d
```


## ДЗ 16: Устройство Gitlab CI. Построение процесса непрерывной поставки

В процессе выполнения ДЗ было сделано

1. Подготовка инсталляцию Gitlab CI в Docker
2. Подготовка репозиторий с кодом приложения
3. Подключение Gitlab Runner
4. Описание для приложения этапов пайплайна
5. Работа с окружениями

Команда для создания VM для gitlab

```
# создание инстанса для gitlab

yc compute instance create \
 --name gitlab-host \
 --cores=2 \
 --core-fraction=5 \
 --memory=4 \
 --create-boot-disk image-folder-id=standard-images,image-family=ubuntu-1804-lts,size=50GB \
 --network-interface subnet-name=app-subnet,nat-ip-version=ipv4 \
 --zone=ru-central1-a \
 --metadata serial-port-enable=1 \
 --ssh-key ~/.ssh/yc-user.pub
 
# подключение
ssh yc-user@51.250.74.121

# установка docker
https://docs.docker.com/engine/install/ubuntu/
```

Запуск Gitlab
```
docker-compose up -d
```

## ДЗ 17: Введение в мониторинг. Системы мониторинга.

В процессе выполнения ДЗ было сделано

1. Подготовка окружения и запуск Prometheus
2. Сборка своего образа Prometheus с prometheus.yml
3. Добавление в docker-compose сервиса node-exporter
4. Отправка образов в Docker Hub


Команда для создания VM

```
yc compute instance create \
 --name docker-host \
 --cores=2 \
 --core-fraction=5 \
 --memory=2 \
 --create-boot-disk image-folder-id=standard-images,image-family=ubuntu-1804-lts,size=15GB \
 --network-interface subnet-name=app-subnet,nat-ip-version=ipv4 \
 --zone=ru-central1-a \
 --metadata serial-port-enable=1 \
 --ssh-key ~/.ssh/yc-user.pub
```

Команда для запуска Prometheus в Docker

```
docker run --rm -p 9090:9090 -d --name prometheus prom/prometheus
```

Сборка своего образа Prometheus

```
export USER_NAME=astrviktor
docker build -t $USER_NAME/prometheus .
```

Отправка образов в Docker Hub

```
docker login

docker push $USER_NAME/ui
docker push $USER_NAME/comment
docker push $USER_NAME/post
docker push $USER_NAME/prometheus
```

## ДЗ 18: Логирование и распределенная трассировка.

В процессе выполнения ДЗ было сделано

1. Пересборка микросервисов с добавлением функционала логирования
2. Подготовка docker-compose-logging.yml и образа fluentd
3. Просмотр структурированных логов через Kibana
4. Подключение Zipkin и просмотр трейсов

Пересборка микросервисов

```
export USER_NAME=astrviktor
cd ./src/ui && bash docker_build.sh && docker push $USER_NAME/ui
cd ../post-py && bash docker_build.sh && docker push $USER_NAME/post
cd ../comment && bash docker_build.sh && docker push $USER_NAME/comment
```

Команда для создания VM

```
yc compute instance create \
 --name logging \
 --cores=2 \
 --core-fraction=5 \
 --memory=4 \
 --create-boot-disk image-folder-id=standard-images,image-family=ubuntu-1804-lts,size=15GB \
 --network-interface subnet-name=app-subnet,nat-ip-version=ipv4 \
 --zone=ru-central1-a \
 --metadata serial-port-enable=1 \
 --ssh-key ~/.ssh/yc-user.pub
```

Сборка fluentd

```
cd logging/fluentd
docker build -t $USER_NAME/fluentd .
```

Запуск сервисов и логирования

```
cd docker
docker-compose -f docker-compose-logging.yml up -d
docker-compose up -d
```



