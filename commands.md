# 1. Перевірка Docker та Compose

docker version
docker compose version

# 2. Перевірка портів

Get-NetTCPConnection -State Listen -LocalPort 80,443 -ErrorAction SilentlyContinue

Порти 80 та 443 вільні.

# 3. Налаштування локального домену

notepad C:\Windows\System32\drivers\etc\hosts

Додано запис:

127.0.0.1 docker-compose.loc

ipconfig /flushdns
ping docker-compose.loc

# 4. Завантаження стартового проєкту

git clone --depth 1 https://github.com/lisnayk/znu-lab-starters.git "$env:TEMP\znu-lab-starters"

Copy-Item -Path "$env:TEMP\znu-lab-starters\conteinerization\labs\06-compose\starter\*" -Destination . -Recurse -Force

# 5. Перевірка Python API

docker build -t lab6-python-check ./python_app

docker run -d --rm --name lab6-python-check -p 127.0.0.1:5050:5000 lab6-python-check

curl.exe http://127.0.0.1:5050/articles

Результат: JSON із трьома статтями.

docker stop lab6-python-check
docker image rm lab6-python-check

# 6. Підготовка змінних середовища

Copy-Item .env.example .env

Паролі задано у файлі .env.
Файл виключено з Git.

# 7. Генерація сертифіката та збірка frontend

docker compose config --quiet

docker compose up certboot frontend-build

docker compose ps --all

Результат:

certboot: Exited (0)
frontend-build: Exited (0)

# 8. Перевірка вмісту томів

docker run --rm -v lab6_certs:/data alpine:3.22 ls -l /data

Результат:

server.crt
server.key

docker run --rm -v lab6_frontend-dist:/data alpine:3.22 ls -l /data

Результат:

index.html
favicon.ico
assets

# 9. Запуск MySQL, PHP та Python

docker compose config --quiet

docker compose up -d --build python-app php-app mysql

docker compose ps --all

docker compose logs mysql

MySQL отримав статус healthy.
Виконано скрипт db.sql.
Створено базу mydb та користувача app.

# 10. Запуск усіх сервісів

docker compose config --quiet

docker compose down

docker compose up -d --build

docker compose ps --all

Результат:

certboot: Exited (0)
frontend-build: Exited (0)
mysql: Up (healthy)
php-app: Up
python-app: Up
web: Up

Сайт доступний за адресою:

https://docker-compose.loc

# 11. Перевірка API

curl.exe -k https://docker-compose.loc/api/articles

Отримано три статті з Python API.

curl.exe -k https://docker-compose.loc/api/users

Отримано п'ять користувачів, включаючи timkov.

# 12. Перевірка перенаправлення HTTP на HTTPS

curl.exe -I http://docker-compose.loc

Результат:

HTTP/1.1 301 Moved Permanently
Location: https://docker-compose.loc/

# 13. Перевірка розділення мереж

docker compose exec web getent hosts mysql

$LASTEXITCODE

Результат: 2.
Nginx не знаходить MySQL за іменем сервісу.

docker compose exec php-app getent hosts mysql

Результат:

172.18.0.2 mysql

PHP має доступ до мережі бази даних.

# 14. Перевірка складу мереж

docker network inspect lab6_front --format '{{range .Containers}}{{println .Name}}{{end}}'

Результат:

lab6-web-1
lab6-python-app-1
lab6-php-app-1

docker network inspect lab6_back --format '{{range .Containers}}{{println .Name}}{{end}}'

Результат:

lab6-mysql-1
lab6-php-app-1

docker compose ps

Опубліковано лише порти 80 та 443 сервісу web.

# 15. Додавання тестового запису

docker compose exec mysql mysql -u app -p mydb

SQL:

INSERT INTO users (username, email)
VALUES ('lab6-test', 'test@example.com');

SELECT * FROM users;

exit

# 16. Перевірка збереження даних

docker compose down

docker compose up -d --wait

curl.exe -k https://docker-compose.loc/api/users

Тестовий запис lab6-test зберігається,
оскільки том db-data не видалено.

# 17. Перевірка повторної ініціалізації бази

docker compose down --volumes

docker compose up -d --wait

curl.exe -k https://docker-compose.loc/api/users

Запис lab6-test відсутній.
Початкові п'ять користувачів відновлено зі скрипту db.sql.
Запис timkov присутній.

# 18. Завантаження проєкту на GitHub

git init
git add .
git status
git commit -m "Add laboratory work 6"
git branch -M main
git push -u origin main

Репозиторій:

https://github.com/TailRed1/Docker-Lab6