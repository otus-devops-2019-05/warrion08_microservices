# warrion08_microservices
warrion08 microservices repository

## **Оглавление:**
- ### Технология контейнеризации. Ввдение в Docker(#ДЗ №12)


#### Технология контейнеризации. Ввдение в Docker(#ДЗ №12)

##### Установка Docker

Установка Docker проводим по инструкции [отсюда](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
Для настройки запуска из под существующего пользователя[инструкция](https://docs.docker.com/install/linux/linux-postinstall/)

Команды:
```
docker version - посмотреть версию Docker
docker run "имя контейнера" - создание и запуск контейнера из образа. Каждый раз запускает новый контейнер. Флаг --rm при запуске, то после остановки контейнер вместе с содержимым остается на диске.
docker ps - список запущенных контейнеров
docker ps -a - список всех контейнеров
docker images - список сохранненых образов
docker start - запускает остановленный контейнер
docker attach - подсоединяет терминал к созданному контейнеру
docker create - используется, когда не нужно стартовать контейнер сразу
docker exec - запускает новый процесс внутри контейнера
docker commit <container_id> <yourName>/<imageName> - создает image из контейнера
docker kill - сигнал SIGKILL контейнеру
docker stop - сигнал SIGTERM контейнеру, а через 10 секунд посылает SIGKILL
docker system df - сколько дискового пространства занято контейнерами, образами и вольюмами, а так же сколько из них не используется и возможно удалить.
docker rm <container_id> - удаление контейнера. При указании ключа -f можно удалять работающий контейнер.
docker rmi <image_id> - удаление образа

``` 
##### Docker-machine

Docker-machine - это встроенный в докер механизм для создания хостов и установки на них docker engine (server).

Команда создания - docker-machine create <имя>. Имен может быть много, переключение между ними через eval $(docker-machine env <имя>). Переключение на локальный докер - eval $(docker-machine env --unset). Удаление - docker-machine rm <имя>. docker-machine создает хост для докер демона со указываемым образом в --googlemachine-image. Образы которые используются для построения докер контейнеров к этому никак не относятся. Все докер команды, которые запускаются в той же консоли после eval $(docker-machine env <имя>) работают с удаленным докер демоном в GCP.

```
Dockerfile-файл содержащий описание нашего докер образа. Dockerfile всегда должен начинаться с инструкции FROM (единственная инструкция, которая может быть указана до FROM - это ARG)
```

##### Docker Hub
```
Docker Hub - это облачный registry сервис от компании Docker. В него можно выгружать и загружать из него докер образы. Docker по умолчанию скачивает образы из докер хаба.
```
Команды:
```
docker login-аутенфикация на локальной машине
docker tag reddit:latest warrion08/otus-reddit:1.0 - привязка тэга к образу
docker push warrion08/otus-reddit:1.0 - загрузка образа
docker run --name reddit -d -p 9292:9292 <your-login>/otus-reddit:1.0 - установка обрза и запуск контейнера из Docker Hub на локальном хосте
docker logs reddit -f -просмотр логов контейнера
docker exec -it reddit bash - войти в контейнер
docker start reddit - запуск контейнера
docker stop reddit && docker rm reddit - остановка и удаление контейнера
docker inspect <your-login>/otus-reddit:1.0 - просмотр инфу о образе
```
