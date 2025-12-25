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





  

