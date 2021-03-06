# Bxdocker
Bxdocker позволяет легко и просто запускать **Bitrix CMS** на **Docker**.

## Введение
Bxdocker облегчает разработку на Битрикс предоставляя готовые сервисы PHP, NGINX, MySQL, phpMyAdmin и многие другие.

### Преимущества данной сборки
- Сервис PHP запакован в отдельный образ, чтобы избавить разработчиков от долгого компилирования.
- Остальные сервисы так же "причёсаны" и разворачиваются моментально.
- Ничего лишнего.

## Порядок разработки в Windows
Если вы работаете в Windows, то требуется установить виртуальную машину.
Желательно использовать Virtualbox, сделать сеть "Сетевой мост", поставить Ubuntu Server.
Сетевой мост даст возможность обращаться к машине по IP и не делать лишних пробросов портов.
Ваш рабочий проект должен хранится в двух местах, первое - локальная папка с проектами на хосте (открывается в IDE), второе - виртуальная машина
(например `/var/www/bitrix`). Проект на хосте мапится в IDE к гостевой OC.

## Зависимости
- Git
```bash
apt-get install -y git
```
- Docker & Docker-Compose
```bash
cd /usr/local/src && wget -qO- https://get.docker.com/ | sh && \
curl -L "https://github.com/docker/compose/releases/download/1.18.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose && \
chmod +x /usr/local/bin/docker-compose && \
echo "alias dc='docker-compose'" >> ~/.bash_aliases && \
source ~/.bashrc
```

### Начало работы
- Склонируйте репозиторий bxdocker
```bash
git clone https://github.com/creio/bxdocker.git
```

- Выполните настройку окружения

Скопируйте файл `.env_template` в `.env`

```bash
cp -f .env_template .env
```

Если у вас мак или windows, то удалите строчку /etc/localtime:/etc/localtime/:ro из docker-compose

- По умолчнию используется nginx php7, эти настройки можно изменить в файле `.env`. Также можно задать путь к каталогу с сайтом и параметры базы данных MySQL.
- В настройках подключения требуется указывать имя сервиса, например для подключения к mysql нужно указывать "mysql", а не "localhost". Пример [конфига](configs/.settings.php)  с подклчюением к mysql и memcached.
- Если восстанавливаем сайт в старой кодировке, исправить в php конфигах. `mbstring.internal_encoding=ISO-8859-1` или `mbstring.func_overload=0`.

```bash
PHP_VERSION=php7           # Версия php 
WEB_SERVER_TYPE=nginx      # Веб-сервер nginx/apache
MYSQL_DATABASE=bitrix      # Имя базы данных
MYSQL_USER=bitrix          # Пользователь базы данных
MYSQL_PASSWORD=123         # Пароль для доступа к базе данных
MYSQL_ROOT_PASSWORD=123    # Пароль для пользователя root от базы данных
INTERFACE=0.0.0.0          # На данный интерфейс будут проксироваться порты `ip a`
SITE_PATH=/var/www/bitrix  # !!!Путь к директории Вашего сайта
```

- Запустите docker.service
```bash
sudo systemctl start docker
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl stop docker
sudo systemctl status docker
```

- Команды для работы.
```bash
docker-compose -h
docker-compose up -d                    # -d В режиме демона
docker-compose up -d --build
docker-compose up -d --force-recreate
docker image prune -f
docker-compose ps                       # Процессы
docker-compose stop                     # Остановка
docker-compose down                     # Остановка контейнеров
docker-compose down --volumes           # Остановка и удаление контейнеров и сети
docker-compose exec web_server bash     # Подключение к контейнеру web_server(консоль)
```
Чтобы проверить, что все сервисы запустились посмотрите список процессов ```docker ps```.  
Посмотрите все прослушиваемые порты, должны быть 80, 11211, 9000 ```netstat -plnt```.  
Откройте IP машины в браузере.

- Сайт будет доступен по адресу localhost `ip a`, если не указали INTERFACE. Чтобы сайт открывался по доменному имени, например, loc.dev, добавляем запись в файл /etc/hosts. https://wiki.archlinux.org/index.php/Network_configuration_(%D0%A0%D1%83%D1%81%D1%81%D0%BA%D0%B8%D0%B9)

```
127.0.0.1       loc.dev
```

- Команды docker.
```bash
docker ps -a                   # Lists containers (and tells you which images they are spun from)

docker rm <container_id>       # Removes a container
docker rm $(docker ps -a -q)   # delete all stopped containers with
docker kill $(docker ps -q)    # kill all running containers with

docker images                  # Lists images

docker rmi <image_id>          # Removes an image
docker rmi -f <image_id>       # Forces removal
docker rmi $(docker images -q) # delete all images with

docker network ls
docker network prune
```

## Примечание
- Если вы хотите начать с чистой установки Битрикса, скачайте файл [bitrixsetup.php](http://www.1c-bitrix.ru/download/scripts/bitrixsetup.php) в папку с сайтом. По умолчанию стоит папка ```/var/www/bitrix/```
- Для загрузки резервной копии в контейнер используйте команду: ```cat /var/www/bitrix/backup.sql | docker exec -i mysql /usr/bin/mysql -u root -p123 bitrix```

- phpMyAdmin на порту `:8080`

## Полезные скрипты

[bitrixsetup.php](http://www.1c-bitrix.ru/download/scripts/bitrixsetup.php)  
[restore.php](http://www.1c-bitrix.ru/download/scripts/restore.php)  
[bitrix_server_test.php](http://dev.1c-bitrix.ru/download/scripts/bitrix_server_test.php)

## Отличие от виртуальной машины Битрикс
Виртуальная машина от разработчиков битрикс решает ту же задачу, что и Bxdocker - предоставляет готовое окружение. Разница лишь в том, что Docker намного удобнее, проще и легче в поддержке.

Как только вы запускаете виртуалку, Docker сервисы автоматически стартуют, т.е. вы запускаете свой минихостинг для проекта и он сразу доступен.

Если у вас появится новый проект и поменяется окружение, достаточно скопировать чистую виртуалку (если вы на винде), скопировать папку Bxdocker, добавить или заменить сервисы и запустить.

P.S.
Виртуальная машина от разработчиков битрикс на Apache, а у нас на Nginx, а он работает намного быстрее и кушает меньше памяти.
