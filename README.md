# warrion08_microservices
warrion08 microservices repository

## **Оглавление:**
- ### Технология контейнеризации. Ввдение в Docker(#ДЗ №12)
- ### Docker-образы. Микросервисы (№ДЗ №13)
- ### Docker.Сети. Docker-Comprose (#ДЗ №14)
- ### Устройство Gitlab CI. Построение процесса непрерывной поставки (#ДЗ №15)
- ### Введение в мониторинг. Системы мониторинга (#ДЗ №16)
- ### Мониторинг приложения и инфраструктуры (#ДЗ №17)


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

Команда создания - docker-machine create <имя>.
Имен может быть много, переключение между ними через eval $(docker-machine env <имя>).
Переключение на локальный докер - eval $(docker-machine env --unset).
Удаление - docker-machine rm <имя>.
docker-machine создает хост для докер демона с указываемым образом в --googlemachine-image.
Образы которые используются для построения докер контейнеров к этому никак не относятся.
Все докер команды, которые запускаются в той же консоли после eval $(docker-machine env <имя>) работают с удаленным докер демоном в GCP.

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
### Docker-образы. Микросервисы (№ДЗ №13)

#### Сборка и запуск приложений в контейнерах
Скачаем исходные коды приложения и положим их в корень нашего репозитория в папку src. Таким образом у нас получится структура:

- src/post-py
- src/comment
- src/ui

Каждая из этих директорий является сервисом и будет превращена в контейнер, поэтому напишем докерфайлы к каждому сервису.

Подключимся к хосту, скачаем последний образ монги и выполним команды сборки образов:
```
eval $(docker-machine env docker-host)
docker pull mongo:latest
docker build -t <your-dockerhub-login>/post:1.0 ./post-py
docker build -t <your-dockerhub-login>/comment:1.0 ./comment
docker build -t <your-dockerhub-login>/ui:1.0 ./ui
```
Создадим сеть и запустим созданные контейнеры:
```
docker network create reddit
docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db mongo:latest
docker run -d --network=reddit --network-alias=post warrion08/post:1.0
docker run -d --network=reddit --network-alias=comment warrion08/comment:1.0
docker run -d --network=reddit -p 9292:9292 warrion08/ui:1.0
```
Проверка: http://104.155.3.117:9292/

#### Подключение volume к контейнеру

Создаем Volume:
`docker volume create reddit_db`

Убьем старые контейнеры и запустим новые, при этом подключив к контейнеру с базой созданный volume. Таким образом, база будет сохраняться на volume и данные не пропадут со смертью контейнера
```
docker kill $(docker ps -q)
docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db -v reddit_db:/data/db mongo:latest
docker run -d --network=reddit --network-alias=post ssjotus/post:1.0
docker run -d --network=reddit --network-alias=comment sjotus/comment:1.0
docker run -d --network=reddit -p 9292:9292 sjotus/ui:2.0
```
#### Docker.Сети. Docker-Comprose (#ДЗ №14)

#### Работа с сетью Docker

Основные Driver`s для работы с сетью в докере:
```
none
host
macvlan
bridge
overlay
```

##### none
Этот драйвер означает, что у контейнера не будет никаких внешних сетевых интерфейсов. Только loopback.

Выполним команду, что бы проверить это:

`docker run -it --rm --network none joffotron/docker-net-tools -c ifconfig`

##### host
При использовании этого драйвера, контейнер подключается напрямую к хосту. Т.е. контейнер использует network namespace хоста. В данном режиме сеть не управляется докером, контейнер будет иметь тот же адрес, что и докер-хост, и 2 разных сервиса в разных контейнерах не смогут слушать один и тот же порт.

Это самый производительный режим сетевого драйвера (в части сетевых задержек).

Выполним команду для проверки сетевых интерфейсов внури контейнера, и команду для проверки сетевых интерфейсов на докер-хосте:
```
Проверим сетевые интерфейсы внутри контейнера
`docker run -ti --rm --network host joffotron/docker-net-tools -c ifconfig`

Проверим сетевые интерфейсы на докер-хосте
`docker-machine ssh docker-host ifconfig`
Данные от этих 2-х команд в данном случае будут одинаковыми, т.к., как уже было сказано, контейнер подключается напрямую к к сетевому неймспейсу хоста.
```

##### bridge
Этот драйвер используется по умолчанию при запуске контейнеров. При использовании этого драйвера в хост-системе создается мост между интерфейсов хоста и виртуальным интерфесом. Виртуальный интерфейс контейнера подключается к мосту и весь трафик будет ходить через него. Так же, докер управляет правилами iptables, в которых помимо фильтрации трафика выполняет NAT при обращении к контейнеру или из контейнера наружу.
```
!! Важно. Докер-сеть с драйвером bridge (docker0) создается по умолчанию при установке докера, но данная сеть не поддерживает service discovery, а значит через встроенные средства докер у контейнеров не получится общаться друг с другом через имена (только через ip адреса).
```

###### Команды Docker для создания подсетей(пример)
```
docker network create back_net --subnet=10.0.2.0/24
docker network create front_net --subnet=10.0.1.0/24
```

При запуске контейнера докер может покдючить к нему только 1 сеть, поэтому дополнительно подключим вторую сеть к контейнерам post и comment
```
docker network connect front_net post
docker network connect front_net comment
```
###### Docker-comprose

Установка 
`pip install docker-compose или https://docs.docker.com/compose/install/#install-compose`

Команды:
```
# поднятие контейнеров
docker-compose up
# поднятие контейнеров в режиме detach
docker-compose up -d
# команда up скачивает недостающие образы или собирает образ из докер файла, после чего запускает его

# Остановка и удаление контейнера
docker-compose down

# Запуск и остановка контейнеров
docker-compose start
docker-compose stop
# start запускает ранее остановленные контейнеры. Stop просто останавливает контейнеры без их удаления

# Просмотр информации о работе compose
docker-compose ps
```
Docker-compose поддерживает интерполяцию переменных окружения. Так же, поддерживает автоматическую загрузку переменных из файла с расширением .env

###### Именование проекта в docker-compose

По-умолчанию докер композ составляет имена запущеных контейнеров по следующей схеме:
```
БазовоеИмяПроекта_ИмяСервиса_НомерИнстанса
БазовоеИмяПроекта по-умолчанию определяется как имя каталога, в котором находится docker-compose.yml. Это имя можно изменить, при запуске композа:
```
Изменить наименование можно с помощью параметра -p:
```
start compose
docker-compose -p <БазовоеИмяПроекта> up
Stop compose
docker-compose -p <БазовоеИмяПроекта> down
Либо задав переменную окружения COMPOSE_PROJECT_NAME
```
#### Устройство Gitlab CI. Построение процесса непрерывной поставки (#ДЗ №15)

##### Инсталляция Gitlab CI

Инсталляцию виртуальной машины осуществим через Docker-machine
```
docker-machine create --driver google \
 --google-machine-image https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1604-lts \
 --google-machine-type n1-standard-1 \
 --google-zone europe-west1-b \
 --google-disk-size 60 \
 --gitlab-ci
```
С помощью команды `docker-machine ssh gitlab-ci` подключимся по ssh в созданную VM
Доставим в VM docker-compose `sudo apt-get install docker-compose`
Создадим папки и файл docker-compose.yml `sudo mkdir -p /srv/gitlab/config /srv/gitlab/data /srv/gitlab/logs && cd /srv/gitlab/ && sudo touch docker-compose.yml`
Правим файл docker-compose.yml и вносим в него конфу с изменением на наш внешний IP
Собираем контейнер `docker-compose up -d`
Проверяем: http://104.155.3.117

##### Настройка Gitlab

Теперь откроем в браузере http://104.155.3.117, гитлаб предложит изменить нам пароль от встроенного пользователя root. Далее, залогинимся и перейдем в глобальные настройки. Там выбираем settings -> Sign-up restrictions и снимаем галочку с sign-up enabled.

Cоздадим группу homework, а внутри неё создадим репозиторий example.

Добавим наш созданный репозиторий в remotes нашего репозитория с микросервисами и сделаем пуш:
```
git checkout -b gitlab-ci-1
git remote add gitlab http://104.155.3.117/homework/example.git
git push gitlab gitlab-ci-1
```
###### Регистрируем и запускаем Runner

На GCE хосте выполняем
```
$ docker run -d --name gitlab-runner --restart always \
-v /srv/gitlab-runner/config:/etc/gitlab-runner \
-v /var/run/docker.sock:/var/run/docker.sock \
gitlab/gitlab-runner:latest
```

###### Регистрируем Runner

`$ docker exec -it gitlab-runner gitlab-runner register --run-untagged --locked=false`

Добавим исходный код reddit в репозиторий
```
$ git clone https://github.com/express42/reddit.git && rm -rf ./reddit/.git
$ git add reddit/
$ git commit -m “Add reddit app”
$ git push gitlab gitlab-ci-1
```

Меняем описание пайплайна в .gitlab-ci.yml для тестирования приложения

Добавляем файл теста simpletest.rb

Добавляем библиотеку rack-test в reddit/Gemfile

###### Разделение на окружения
Изменяем .gitlab-ci.yml для dev окружения
```
deploy_dev_job:
  stage: review
  script:
    - echo 'Deploy'
  environment:
    name: dev
    url: http://dev.example.com
```
Изменяем .gitlab-ci.yml для staging и production окружений
```
staging:
  stage: stage
  when: manual
  only:
    - /^\d+\.\d+\.\d+/ 
  script:
    - echo 'Deploy'
  environment:
    name: stage
    url: https://beta.example.com

production:
  stage: production
  when: manual
  only:
    - /^\d+\.\d+\.\d+/ 
  script:
    - echo 'Deploy'
  environment:
    name: production
    url: https://example.com
Изменение, помеченное тэгом в git запустит полный пайплайн
```
```
$ git commit -a -m ‘#4 add logout button to profile page’
$ git tag 2.4.10
$ git push gitlab gitlab-ci-1 --tags
```
Динамическое окружение задано при помощи переменных в .gitlab-ci.yml. На каждую ветку в git отличную от master Gitlab CI будет определять новое окружение.
```
branch review:
 stage: review
 script: echo "Deploy to $CI_ENVIRONMENT_SLUG"
 environment:
 name: branch/$CI_COMMIT_REF_NAME
 url: http://$CI_ENVIRONMENT_SLUG.example.com
 only:
 - branches
 except:
 - master
```
#### Введение в мониторинг. Системы мониторинга (#ДЗ №16)

##### Запуск prometheus в контейнере
Перед запуском prometheus нам нужно подготовить окружение. 
Необходимо добавить правила фаервола в GCP:
```
gcloud compute firewall-rules create prometheus-default --allow tcp:9090
gcloud compute firewall-rules create puma-default --allow tcp:9292
```
Cоздаем ВМ с докером через docker-machine:

`export GOOGLE_PROJECT=docker-251308`
```
#create docker host
docker-machine create --driver google \
    --google-machine-image https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1604-lts \
    --google-machine-type n1-standard-1 \
    --google-zone europe-west1-b \
    docker-host
```
Подключаемся к удаленному хосту через докер-машин:
`eval $(docker-machine env docker-host)`

Запустим прометеус в контейнере. Будем использовать уже готовый образ с докер-хаба

`docker run --rm -p 9090:9090 -d --name prometheus  prom/prometheus`

Посмотреть web морду прометеуса можно по адресу: `http://104.199.66.231:9090`

Тормозим контейнер:
`docker stop prometheus`

##### Сборка собственного образа prometheus
Вся конфигурация Prometheus происходит через файлы конфигурации и опции командной строки.
В директории monitoring/prometheus создаем файл prometheus.yml
Создадим внутри директории monitoring диреткорию prometheus, внутри которой создадим Dockerfile
```
FROM prom/prometheus:v2.1.0
ADD prometheus.yml /etc/prometheus/
```
##### Оркестрация через docker-compose и сбор метрик
У нас уже есть докер-композ файл для поднятия наших сервисов, поэтому нам необходимо подключить туда поднятие прометеуса.
Но для начала пересоберем все образы наших сервисов через скрипт docker_build.sh, который находится в директории каждого сервиса в каталоге src.
Скрипт для сборки всего из корня репозтория:
```
for i in ui post-py comment; do cd src/$i; bash
docker_build.sh; cd -; done
```
После `$ docker-compose up -d`
Должны подняться контейнеры с сервисами и прометеус в котором можно посмотреть различные метрики по нашим сервисам. 
Проверим, что для сервиса базы данных установленны все алиасы (необходимо, что бы другие сервисы могли обращаться к сервису базы данных)
Теперь мы можем подключиться к прометеусу и посмотреть метрики. Зайдем по адресу http://<docker-host-ip>:9090 посмотрим на метрики `ui_healht, ui_health_comment_availability и ui_health_post_availability` убедимся что прометеус собирает метрики с наших сервисов.

##### Использование exporters
Экспортер-вспомогательный агент для сбора метрик. В ситуациях, когда мы не можем реализовать отдачу метрик Prometheus в коде приложения, мы можем использовать экспортер, который будет транслировать метрики приложения или системы в формате доступном для чтения Prometheus.

Настроим сбор метрик с докер-хоста. Для этого воспользуемся экспортером Node exporter. Экспортер будем так же запускать в контейнере, поэтому добавим его как сервис в docker-compose файл.

В конфиг прометеуса (prometheus.yml) добавим еще одну джобу, что бы прометеус следил за экспортером
```
scrape_configs:
...
  - job_name: 'node'
    static_configs:
      - targets:
        - 'node-exporter:9100'
```
И пересоберем контейнер с прометеусом:
```
export USER_NAME=warrion08
cd monitoring/prometheus && docker build -t $USER_NAME/prometheus .
```

https://cloud.docker.com/u/warrion08/repository/docker/warrion08/

#### Мониторинг приложения и инфраструктуры (#ДЗ №17)

#### Подготовка окружения
```
$ export GOOGLE_PROJECT=_ваш-проект_
$ docker-machine create --driver google \
--google-machine-image
https://www.googleapis.com/compute/v1/projects/ubuntu-oscloud/global/images/family/ubuntu-1604-lts \
--google-machine-type n1-standard-1 \
--google-zone europe-west1-b \
docker-host
...
Docker is up and running!
To see how to connect your Docker Client to the Docker Engine running on this
virtual machine, run: docker-machine env docker-host
$ eval $(docker-machine env docker-host)
# узнаем IP адрес
$ docker-machine ip docker-host
```
Разделил файл docker-compose.yml на два файла:
docker-compose.yml - описание приложения
docker-composemonitoring.yml - описание мониторинга приложения
Для запуска приложений используем `docker-compose up -d`, а для мониторинга - `docker-compose -f docker-compose-monitoring.yml up -d`

#### cAdvisor
Используется для мониторинга состояния докер контейнеров
На Web UI cAdvisor можно попасть по адресу `http://<dockermachine-host-ip>:8080`
Метрики для Prometheus `http://<dockermachine-host-ip>:8080/metrics`

#### Grafana
Используется для визуализации данных из Prometheus
На Web UI можно попасть по адресу `http://<dockermachine-host-ip>:3000`
Дашборды можно сделать самому или скачать `https://grafana.com/grafana/dashboards`

#### Alertmanager
Дополнительный компонент для системы мониторинга Prometheus, который отвечает за первичную обработку алертов и дальнейшую отправку оповещений по заданному назначению.
