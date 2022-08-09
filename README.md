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


