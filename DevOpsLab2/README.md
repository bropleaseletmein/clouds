# 2 Лабораторная
## Обычная:
- Написать “плохой” Dockerfile, в котором есть не менее трех “bad practices” по написанию докерфайлов
- Написать “хороший” Dockerfile, в котором эти плохие практики исправлены
- В Readme описать каждую из плохих практик в плохом докерфайле, почему она плохая и как в хорошем она была исправлена, как исправление повлияло на результат
- В Readme описать 2 плохих практики по работе с контейнерами. ! Не по написанию докерфайлов, а о том, как даже используя хороший докерфайл можно накосячить именно в работе с контейнерами.

# ход выполнения работы
- ## плохой Dockerfile:
для начала создадим дефолтный проект для dotnet:

вводим команду `dotnet new webapi`

получаем следующее

<img width="873" height="857" alt="image_2025-12-25_15-45-11" src="https://github.com/user-attachments/assets/21ee1112-92df-4b67-9624-abd44c04dd72" />

добавим докерфайл в наш проект

<img width="254" height="261" alt="image_2025-12-25_15-45-52" src="https://github.com/user-attachments/assets/bdfd0925-565c-4a31-8450-a840a40c7735" />

соблюдая плохие практики написания, создадим образ

<img width="517" height="257" alt="image" src="https://github.com/user-attachments/assets/515e989f-00fc-4662-a024-d1887ad9ec3b" />

(см. https://github.com/bropleaseletmein/clouds/blob/main/DevOpsLab2/Dockerfile.Bad)

### сначала убедимся, что образ рабочий, а затем разберем ошибки и создадим исправленный вариант.

начнем с работоспособности

- вводим команду `docker build -f Dockerfile.Bad -t app-with-bad-dockerfile .` в нашей рабочей дериктории
  - `-f Dockerfile.Bad`: указываем путь до докерфайла
  - `-t app-with-bad-dockerfile`: тэг (имя) образа
  - `.`: конекст, будем копировать все с текущей директории 
- получаем 
- <img width="2526" height="465" alt="image_2025-12-25_17-02-57" src="https://github.com/user-attachments/assets/5a743e26-f59a-48dc-a244-04c647ba03b9" />
- выполним проверку, что образ есть, а так же посмотрим на его размер
- <img width="731" height="67" alt="image_2025-12-25_17-03-36" src="https://github.com/user-attachments/assets/7faeb81f-67b9-4afc-979a-b19320106515" />
- создадим контейнер по нашему образу. работать он будет заднем фоне
  - выполним команду `docker run -d --name bad-app -p 5046:8080 app-with-bad-dockerfile`
    - `-d`: флаг для работы на заднем фоне  
    - `--name bad-app`: дадим имя контейнеру
    - `-p 5046:8080`: порты (8080 - стандартный для контейнера)
  - <img width="1101" height="41" alt="image" src="https://github.com/user-attachments/assets/5b95c522-aa9b-4798-a99a-03a1edf3a221" />
- проверим, что все работает, отправив запрос через Postman
- <img width="619" height="985" alt="image_2025-12-25_17-12-43" src="https://github.com/user-attachments/assets/fabf21ab-446e-47e9-bc55-adadfe4572e4" />

готово. плохой контейнер работает. теперь разберем, что тут не так

### ошибки

еще раз посмотрим на докерфайл

<img width="517" height="257" alt="image" src="https://github.com/user-attachments/assets/88f018cd-3b5b-48be-ada1-7074f7a3ac58" />

(см. https://github.com/bropleaseletmein/clouds/blob/main/DevOpsLab2/Dockerfile.Bad)

разберем ошибки:
- `FROM mcr.microsoft.com/dotnet/sdk:latest`
  - во-первых мы тянем очень тяжелый базовый образ. в нем есть
    - Компилятор C# (Roslyn)
    - MSBuild (система сборки)
    - NuGet (пакетный менеджер)
    - dotnet CLI (командная строка)
    все это нужно только для сборки приложения. как только мы соберем его - оно попросту будет ненужно
  - плюсом мы тянем `:latest`
    - если к примеру у нас проект на 9 dotnet, а `:latest` вытянет 10, то ничего не соберется
  - `COPY . .`
    - здесь мы копируем все, а докер-игнор нет. поэтому в выходную директорию попадет всякий мусор по типу `/bin` или `/obj`. или туда могут уйти переменные окружения или любые другие нежелательные файлы
    - посмортим, что попало в выходную директорию при помощи команды `docker exec bad-app ls /app`
    - <img width="801" height="211" alt="image" src="https://github.com/user-attachments/assets/c7692e38-efc8-45f6-9f25-0e477b5e5e79" />
    - как видим здесь есть мусор, который для работы приложения не нужен
  - `CMD ["dotnet", "bin/Debug/net9.0/DevOpsLab2.dll"]`
    - так запускать плохо (что если 9 dotnet, что если другая конфигурая - release)
    - + мы запускаем от root, не установив другого пользователя 

- ## нормальный Dockerfile:

попытаемся создать нормальный докер файл

добавим новый файл

<img width="538" height="399" alt="image" src="https://github.com/user-attachments/assets/e08cc9e5-b047-4d21-8b2d-a2d48a3e29ab" />

(см https://github.com/bropleaseletmein/clouds/blob/main/DevOpsLab2/Dockerfile.Good)

добавим докер игнор 

<img width="485" height="227" alt="image_2025-12-25_17-16-04" src="https://github.com/user-attachments/assets/e75f6f7b-24fb-4020-853c-c55099cf9be9" />

## и так же как и в прошлый раз: сначала проверим, что все работает, затем рассмотрим, что исправили 

начнем с работоспособности
- docker build -f Dockerfile.Good -t app-with-good-dockerfile .
  - флаги уже были пояснены выше
- <img width="1184" height="599" alt="image" src="https://github.com/user-attachments/assets/b5e0a88f-94be-42cf-ae28-fa33e53e8b0f" />
- проверим что образ есть + посмотрим, сколько весит
- <img width="705" height="83" alt="image_2025-12-25_17-34-26" src="https://github.com/user-attachments/assets/e8a4764b-a57e-4d97-9ef5-2a96347b1dc7" />
- делаем контейнер `docker run -d -p 5045:8080 --name good-app app-with-good-dockerfile`
- <img width="1114" height="40" alt="image" src="https://github.com/user-attachments/assets/3a4586d2-dfaf-42bf-b4c2-af3425148953" />
- проверяем через Postman
- <img width="596" height="1000" alt="image" src="https://github.com/user-attachments/assets/c9159b27-f861-484e-9f5d-167b43deeef8" />

### что исправили

смотрим на докерфайл

<img width="538" height="399" alt="image" src="https://github.com/user-attachments/assets/ab07b6bc-6ba3-446f-858c-97d2247cb256" />

(см https://github.com/bropleaseletmein/clouds/blob/main/DevOpsLab2/Dockerfile.Good)

- `FROM mcr.microsoft.com/dotnet/sdk:9.0-alpine3.23 AS build`
  - использеум тяжелый базовый образ только для сборки
  - плюсом тянем нужную версию
  - ну и alpine используем - он легкий
- `WORKDIR /src`
  - тут установили другую директорию, в которой будет билд
- `COPY DevOpsLab2.csproj .` и `RUN dotnet restore`
  - тут вынес копирование и RUN в отдельные слои для кеширования
- `COPY . .`
  - тут копируем все, но от нежелательных вещей нас спасает докер-игнор (см выше)
- `RUN dotnet publish -c Release -o /app/publish`
  - нормально билдим приложение с нужной конфигурацией, выходная директория будет `/app/publish`. с нее потом будет брать полученный dll
- `FROM mcr.microsoft.com/dotnet/aspnet:9.0-alpine3.23 AS run`
  - стадия `run`: тянем базовый образ, в котором только все необходимое для рантайнма (JIT компилятор и тд)
  - указываем версию, плюсом используем alpine - она легкая
- `WORKDIR /app`
  - переместились в `/app` - наша финальная директория. `/src` нам уже не нужен
- `COPY --from=build /app/publish .`
  - копируем все с папки publish со стадии билда (тут будет нужный dll)
- `USER app`
  - поставим юзера app - он уже есть в базовом образе от майрософт
- `ENTRYPOINT [ "dotnet", "DevOpsLab2.dll" ]`
  - запускаемся - берем полученный DevOpsLab2.dll
  - сделал через `ENTRYPOINT`, а не `CMD`: `ENTRYPOINT` вызывается на старте коннтейнера, и больше аргументов нам не надо

в итоге получили +- нормальный докерфайл, исправили ошибки, значительно сократили размер

<img width="705" height="83" alt="image_2025-12-25_17-34-26" src="https://github.com/user-attachments/assets/61a858b9-7f6a-4bf5-8500-d5e511aad2d3" />

1.36гб у плохого vs 131мб у нормального 

## плохие практики работы с контейнерами 
- хранение данных в самом контейнере без использования томов. если мы перезапустим контейнер. то все данные будут удалены. как говорилось ранее, если имеется неоюлходимость сохраннять данные между перезапусками контейнеров, необходимо использовать тома
- запуск нескольких процессов (сервисов) в одном контейнере. например не рекомендуется сервис с апи и бд запускать на одном контейнере. они предназначены для одного приложения. при необходимости использовать несколько сервисов можно воспользоваться отдельным контейнером на каждый. управлять ими можно при помощи docker compose 



 









  

