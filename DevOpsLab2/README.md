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

<img width="254" height="261" alt="image_2025-12-25_15-45-52" src="https://github.com/user-attachments/assets/b9b7c6f1-4268-4b27-8696-efe3c4107179" />

соблюдая плохие практики написания, создадим образ

<img width="548" height="226" alt="image_2025-12-25_17-57-08" src="https://github.com/user-attachments/assets/4c661cb4-cf31-445c-8d48-f87ca43ba791" />

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
  - выполним команду `docker run -d --name bad-app -p 5046:5046 app-with-bad-dockerfile`
    - `-d`: флаг для работы на заднем фоне  
    - `--name bad-app`: дадим имя контейнеру
    - `-p 5046:5046`: порты
  - <img width="1090" height="42" alt="image_2025-12-25_17-12-31" src="https://github.com/user-attachments/assets/a2724659-3a2a-463d-ab95-48e7db0e2845" />
- проверим, что все работает, отправив запрос через Postman
- <img width="619" height="985" alt="image_2025-12-25_17-12-43" src="https://github.com/user-attachments/assets/fabf21ab-446e-47e9-bc55-adadfe4572e4" />

готово. плохой контейнер работает. теперь разберем, что тут не так

### ошибки

еще раз посмотрим на докерфайл

<img width="548" height="226" alt="image_2025-12-25_17-57-08" src="https://github.com/user-attachments/assets/4c661cb4-cf31-445c-8d48-f87ca43ba791" />

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
  - `CMD ["dotnet", "run", "--project", "DevOpsLab2.csproj"]`
    - ввиду использования тяжелого образа проект приходится запускать так
    - плюсом мы запускаем от root, не установив другого пользователя 

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
- делаем контейнер `docker run -d --name good-app app-with-good-dockerfile`
- <img width="1027" height="46" alt="image_2025-12-25_17-35-06" src="https://github.com/user-attachments/assets/fa8445eb-8c9a-4fa5-87f7-7ec2cb0d0e0b" />
- проверяем через Postman
- <img width="455" height="965" alt="image_2025-12-25_17-35-31" src="https://github.com/user-attachments/assets/51c1cbbe-e67f-4efb-8616-19a537e99f4e" />

### что исправили

смотрим на докерфайл

<img width="538" height="399" alt="image" src="https://github.com/user-attachments/assets/ab07b6bc-6ba3-446f-858c-97d2247cb256" />

(см https://github.com/bropleaseletmein/clouds/blob/main/DevOpsLab2/Dockerfile.Good)

- `FROM mcr.microsoft.com/dotnet/sdk:8.0.416-alpine3.23 AS build`
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
- `FROM mcr.microsoft.com/dotnet/runtime:10.0-alpine3.23 AS run`
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





 









  

