# Pawster 🐾  
Pawster — это социальная сеть для обмена фотографиями ваших любимых питомцев. Проект использует контейнеризацию через Docker и автоматический процесс CI/CD, настроенный с помощью GitHub Actions.

## Установка и запуск

1. Клонируйте репозиторий на свой локальный компьютер:
    ```bash
    git clone git@github.com:Alexgnchrv/Pawster.git
    cd Pawster
    ```

2. Создайте файл `.env` и заполните его своими данными, используя образец `.env.example`, который находится в корневой директории проекта.

## Создание и загрузка Docker-образов

1. Построение Docker-образов:
    ```bash
    cd frontend
    docker build -t username/pawster_frontend .

    cd ../backend
    docker build -t username/pawster_backend .

    cd ../nginx
    docker build -t username/pawster_gateway .
    ```

2. Загрузка образов в DockerHub:
    ```bash
    docker push username/pawster_frontend
    docker push username/pawster_backend
    docker push username/pawster_gateway
    ```

## Деплой на сервер

1. Подключитесь к удаленному серверу:
    ```bash
    ssh -i путь_до_файла_с_SSH_ключом/название_файла_с_SSH_ключом имя_пользователя@ip_адрес_сервера
    ```

2. Создайте директорию проекта на сервере:
    ```bash
    mkdir pawster
    ```

3. Установите Docker и Docker Compose на сервер:
    ```bash
    sudo apt update
    sudo apt install curl
    curl -fSL https://get.docker.com -o get-docker.sh
    sudo sh ./get-docker.sh
    sudo apt install docker-compose-plugin 
    ```

4. Копируйте файлы `docker-compose.production.yml` и `.env` на сервер:
    ```bash
    scp -i path_to_SSH/SSH_name docker-compose.production.yml username@server_ip:/home/username/kittygram/docker-compose.production.yml
    ```

5. Запустите Docker Compose в режиме демона:
    ```bash
    sudo docker compose -f docker-compose.production.yml up -d
    ```

6. Выполните миграции и соберите статические файлы:
    ```bash
    sudo docker compose -f docker-compose.production.yml exec backend python manage.py migrate
    sudo docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic
    sudo docker compose -f docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/
    ```

## Настройка Nginx

1. Редактируйте конфигурацию Nginx:
    ```bash
    sudo nano /etc/nginx/sites-enabled/default
    ```

2. Обновите настройки `location`:
    ```nginx
    location / {
        proxy_set_header Host $http_host;
        proxy_pass http://127.0.0.1:9000;
    }
    ```

3. Проверьте конфигурацию Nginx:
    ```bash
    sudo nginx -t
    ```

4. Если ошибок нет, перезапустите Nginx:
    ```bash
    sudo service nginx reload
    ```

## Настройка CI/CD

1. Файл workflow находится в директории `.github/workflows/main.yml`. Он автоматизирует процесс тестирования и деплоя на сервер.

2. Для настройки GitHub Actions добавьте следующие секреты в настройки репозитория на GitHub:
    - **DOCKER_USERNAME** — логин DockerHub
    - **DOCKER_PASSWORD** — пароль DockerHub
    - **HOST** — IP-адрес сервера
    - **USER** — имя пользователя на сервере
    - **SSH_KEY** — приватный SSH-ключ (получите с помощью `cat ~/.ssh/id_rsa`)
    - **SSH_PASSPHRASE** — пароль для SSH-ключа
    - **TELEGRAM_TO** — ID вашего Telegram-аккаунта (узнайте у `@userinfobot`)
    - **TELEGRAM_TOKEN** — токен вашего Telegram-бота (создайте через `@BotFather`)

## Проверка перед отправкой

1. Убедитесь, что Kittygram доступен по указанному доменному имени.
2. Пуш в ветку `main` запускает автоматическое тестирование и деплой.
3. После успешного деплоя приходит уведомление в Telegram.
4. Все контейнеры успешно работают на сервере.
