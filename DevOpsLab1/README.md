# 1 Лабораторная
Настроить nginx по заданному тз:
- Должен работать по https c сертификатом
- Настроить принудительное перенаправление HTTP-запросов (порт 80) на HTTPS (порт 443) для обеспечения безопасного соединения.
- Использовать alias для создания псевдонимов путей к файлам или каталогам на сервере.
- Настроить виртуальные хосты для обслуживания нескольких доменных имен на одном сервере.
- Что угодно еще под требования проекта

## Начало

в качестве пет проектов будет:
- супер-простая html страничка (назовем домен `project-html.dev`)
- базовое веб api приложение (`project-api.dev`), развернутое в конейнере докера (образ взял с 3-й лабы)

работу буду делать на виртуалке (ubuntu 24.04 server)

подключаемся по ssh

<img width="1114" height="96" alt="image_2026-01-15_22-48-32" src="https://github.com/user-attachments/assets/73bc6019-3ce8-4eb6-bb51-e9fda40c01f1" />

для начала установим сам nginx `sudo apt install nginx -y`. проверим что все скачалось

<img width="1058" height="668" alt="image_2026-01-15_22-50-12" src="https://github.com/user-attachments/assets/1b04249c-b11b-4a4a-b98e-5a87031c0c95" />

и сразу запустим 

`sudo systemctl enable nginx`

`sudo systemctl start nginx`

потом добавим в `/etc/hosts` нужные домены:
- 127.0.0.1 project-html.dev
- 127.0.0.1 project-api.dev

<img width="631" height="319" alt="image_2026-01-16_00-42-23" src="https://github.com/user-attachments/assets/47f2b5d0-c621-4441-b6cf-8aacb4652a2c" />

## Сертификаты для https

для работы с https нам необходимы сертификаты. при помощи гугла выясняем что в рамках лабы можно использовать самоподписные сертификаты. не без помощи cladue.ai генерируем самоподписные сертификаты для двух проектов 

- project-html:
  
  ```
  sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/nginx/ssl/project-html.key \
  -out /etc/nginx/ssl/project-html.crt \
  -subj "/C=RU/ST=Moscow/L=Moscow/O=ProjectHTML/CN=project-html.dev"
  ```

  <img width="1391" height="350" alt="image_2026-01-16_01-48-26" src="https://github.com/user-attachments/assets/fc9cf2e6-93b1-4b88-a8fc-a38df8f0d132" />

- project-api:
  
  ```
  sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/nginx/ssl/project-api.key \
  -out /etc/nginx/ssl/project-api.crt \
  -subj "/C=RU/ST=Moscow/L=Moscow/O=ProjectAPI/CN=project-api.dev"
  ```
  
  <img width="1380" height="542" alt="image_2026-01-16_01-49-04" src="https://github.com/user-attachments/assets/7781b860-5db3-497e-8bd5-40f9a316bd54" />

смотрим всё ли получилось `ls -la /etc/nginx/ssl/`

<img width="533" height="250" alt="image" src="https://github.com/user-attachments/assets/84c522b0-3c51-4ce3-a223-7f4bbc50d181" />

вроде все есть. двигаемся дальше 

## Конфиг для project-html

тут сначала создадим html документ в папке /var/www/project-html/

<img width="629" height="410" alt="image_2026-01-15_23-42-17" src="https://github.com/user-attachments/assets/48a820b7-4e7a-46b6-9ee5-e2f936609d78" />

и папку /var/www/shared/images/. там создадим "изображение" котика

<img width="593" height="93" alt="image_2026-01-16_01-06-15" src="https://github.com/user-attachments/assets/467b5d8f-477e-4e34-85f8-8613d84ae805" />

### Редирект

```
server {
    listen 80;
    server_name project-html.dev;
    return 301 https://$server_name$request_uri;
}
```

слушаем на 80 порту (стандартный порт для http)

перенаправляем на https. имя сервера uri подставятся сами через переменные 

### Алиасы и прочее

пишем еще раз

```
server { }
```

далее ставим корень в /var/www/project-html и индекс страницу index.html (которую создали выше)

```
root /var/www/project-html;
index index.html;
```

ставим сертификаты которые сгенерили выше 
```
ssl_certificate /etc/nginx/ssl/project-html.crt;
ssl_certificate_key /etc/nginx/ssl/project-html.key;
```

проставляем что только /index.html можем с рута открыть
```
location = / {
  try_files /index.html =404;
}
```

проставляем алиас на /images/ -> перенаправляем поиск в `/var/www/shared/images/` (там у нас лежит "изображение" котика. т.е. при запаросе на /images/cat.png мы будем искать не с рута (`/var/www/project-html`), а в `/var/www/shared/images/`
```
location /images/ {
  alias /var/www/shared/images/;
}
```

и во всех остальнных случаях нельзя ничего делать - кидаем 404 - не найдено
```
location / {
  return 404;
}
```

плюсом добавил логи 
```
  access_log /var/log/nginx/project-html_access.log;
  error_log /var/log/nginx/project-html_error.log;
```

итоговый файл такой 
```
server {
    listen 80;
    server_name project-html.dev;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name project-html.dev;

    root /var/www/project-html;
    index index.html;

    ssl_certificate /etc/nginx/ssl/project-html.crt;
    ssl_certificate_key /etc/nginx/ssl/project-html.key;

    location = / {
        try_files /index.html =404;
    }

    location /images/ {
        alias /var/www/shared/images/;
    }

    location / {
        return 404;
    }

    access_log /var/log/nginx/project-html_access.log;
    error_log /var/log/nginx/project-html_error.log;
}
```

через `nano` запишем всё это в файл

<img width="546" height="737" alt="image_2026-01-16_00-35-32" src="https://github.com/user-attachments/assets/4c633c27-69be-4df5-a5c0-581b87842aa7" />

## Конфиг для project-api

### Редирект

тут так же как и с project-html

```
server {
    listen 80;
    server_name project-api.dev;
    return 301 https://$server_name$request_uri;
}
```

### Прочее

и итоговый конфиг выглядит так
```
server {
    listen 80;
    server_name project-api.dev;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name project-api.dev;

    ssl_certificate /etc/nginx/ssl/project-api.crt;
    ssl_certificate_key /etc/nginx/ssl/project-api.key;

    location / {
        proxy_pass http://localhost:8080;
    }

    access_log /var/log/nginx/project-api_access.log;
    error_log /var/log/nginx/project-api_error.log;
}
```

единственное что добавили - это проксирование на порт 8080 локалсхоста - этот порт будет запущенный контейнер слушать
```
location / {
  proxy_pass http://localhost:8080;
}
```

ну и убрали лишнее тут это не нужно тк просто проект с апи:
- `root /var/www/project-html;`
- `index index.html;`
- алиасы

пишем конфиг в нужный файл

<img width="568" height="441" alt="image_2026-01-16_01-20-41" src="https://github.com/user-attachments/assets/b5f17b24-7981-44d0-8f0f-fd5149012f4b" />

## Запускаем 

создадим сначала линки 
- `sudo ln -s /etc/nginx/sites-available/project-html /etc/nginx/sites-enabled/project-html`
- `sudo ln -s /etc/nginx/sites-available/project-api /etc/nginx/sites-enabled/project-api`

перезапускаем nginx `sudo systemctl restart nginx`

поднимем контейнер `docker run -d -p 8080:8080 lab3:latest test-api-for-nginx`

и начинаем проверять 

### Редирект

выполняем `curl -k --head http://project-html.dev`

<img width="613" height="203" alt="image" src="https://github.com/user-attachments/assets/f223ecbe-d3a5-4ecd-b38d-04e3eb817d1e" />

как видим тут идет редирект: `Location: https://project-html.dev/`

то же самое для второго проекта `curl -k --head http://project-api.dev`

<img width="570" height="175" alt="image" src="https://github.com/user-attachments/assets/67304fc4-1151-466f-8ec9-a3cf5462e269" />

### Алиасы

`curl -k https://project-html.dev/images/cat.png`

<img width="642" height="61" alt="image" src="https://github.com/user-attachments/assets/07c79ded-38bd-4371-9403-6d5784e6eabc" />

получили картинку не с рута, а по другому пути

### Просто запросы

`curl -k https://project-html.dev`

<img width="554" height="299" alt="image" src="https://github.com/user-attachments/assets/fdd3d5d2-79fb-4efe-988c-532f3eba00a1" />

`curl -k https://project-api.dev/weatherforecast | jq` (пайпим с jq - чтобы отформатировать)

<img width="771" height="754" alt="image" src="https://github.com/user-attachments/assets/22d4ff98-0cd6-4bf2-9f38-6700cf86dc48" />

как видим у нас два приложения с разными хостами на одном сервере - все работает

# Итог
- сгенерили самоподписные сертификаты и используем их
- настроили перенаправление HTTP-запросов (порт 80) на HTTPS (порт 443) 
- создали алиас /images/ -> /var/shared/images/ (а не с рута смотрим)
- настроили виртуальные хосты (обслуживаем несколько доменных имен на одном сервере)
- и немного защитились от path traversal - можно делать запросы только на `/images/` и `/` (сам рут - или index.html c него. остально нельзя)






















