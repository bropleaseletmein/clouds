# 3 Лабораторная
- Написать “плохой” CI/CD файл, который работает, но в нем есть не менее трех “bad practices” по написанию CI/CD
- Написать “хороший” CI/CD, в котором эти плохие практики исправлены
- В Readme описать каждую из плохих практик в плохом файле, почему она плохая и как в хорошем она была исправлена, как исправление повлияло на результат

## Начнем с небольшой настройки
папйлайны будут:
- собирать докер-образ, в докерфайле будут прописаны все шаги (по невероятным стечениям обсоятельств взял докер-файл с лабы 2) - восстановление зависимостей, сама сборка
- прогонять юнит-тесты
- деплоить на сервер

я захотел немнного приблизить процесс к реальности, но посколько у меня нет сервера и прочих вещей куда можно задеплоить своё приложение, я буду использовать виртуальную машину как сервер. буду использовать self-hosted раннер (просто захотелось посмотреть как это будет выглядеть + чтобы не париться над тем как соединить раннеры гитхаба с моей виртуальной машиной). 

## Начнем с настроки ВМ и self-hosted раннера
- скачаем образ ОС (я выбрал такой - ubuntu-24.04.3-live-server-amd64.iso)
- зайдем в виртуал бокс и начнем настроку
- <img width="708" height="597" alt="image_2026-01-13_16-58-54" src="https://github.com/user-attachments/assets/513b1e9f-812e-44bb-af8e-30e413056449" />
- выделим память
- <img width="707" height="594" alt="image_2026-01-13_17-00-27" src="https://github.com/user-attachments/assets/92daf768-f883-4e74-af29-9fb4c4176ab4" />
- пробросим порты - пусть будет
  - для HTTP хост:5000 -> вм:80
  - для SSH  хост:2222 -> вм:22
- <img width="843" height="493" alt="image_2026-01-13_17-08-54" src="https://github.com/user-attachments/assets/d20452bc-b1d4-4e0a-be75-4a996fa9f481" />
- запускаем, настраиваем (почти везде жмем "Done" :) придумываем логин / пароль, ждем
- <img width="1280" height="800" alt="VirtualBox_server-for-lab3_13_01_2026_17_21_22" src="https://github.com/user-attachments/assets/d233be99-a2e5-45a3-a5ce-1b25258ea6b5" />
- входим в систему
- <img width="1280" height="800" alt="VirtualBox_server-for-lab3_13_01_2026_17_42_44" src="https://github.com/user-attachments/assets/7b6f036c-4cdc-4a20-aee1-a5ef3ebcc08f" />
- скачиваем docker: `supo apt install docker.io -y` (всему верим, на все отвечаем y), проверяем установку
- <img width="1280" height="800" alt="VirtualBox_server-for-lab3_13_01_2026_17_56_13" src="https://github.com/user-attachments/assets/90d51a74-4a90-4529-9880-333b6b8da4d0" />
- создем репозиторий (если нужен будет доступ - открою)
- <img width="1329" height="1007" alt="image_2026-01-13_18-06-06" src="https://github.com/user-attachments/assets/d2a34c6a-aca4-4be9-a66a-09ca1507d2d5" />
- идем на страничку с self-hosted раннером и следуем инструкции (токен на всякий замазал)
- <img width="812" height="1059" alt="image_2026-01-13_18-09-51" src="https://github.com/user-attachments/assets/d2952e50-cc56-43b3-a085-59bfdc280e9c" />
- создаем директорию
- <img width="1280" height="800" alt="VirtualBox_server-for-lab3_13_01_2026_18_16_23" src="https://github.com/user-attachments/assets/3f92b578-cfc4-4401-bf5a-e3653ba23c44" />
- (тут я устал работать через интерфейс вм поэтому подключился по SSH)
- <img width="1110" height="612" alt="image_2026-01-13_18-23-56" src="https://github.com/user-attachments/assets/e0847d69-ba83-4089-9c33-380f083a558f" />
- скачиваем все, проверяем установку, распаковываем
- <img width="1097" height="623" alt="image_2026-01-13_18-25-41" src="https://github.com/user-attachments/assets/93cadf46-e51d-43c1-9c41-d6e3f10e9965" />
- настраиваем (токен тоже на всякий замазал)
- <img width="1350" height="696" alt="image_2026-01-13_18-29-50" src="https://github.com/user-attachments/assets/278568e7-8ab6-4b0e-89f5-34754cd2f01a" />
- раннер появился
- <img width="795" height="228" alt="image_2026-01-13_18-34-04" src="https://github.com/user-attachments/assets/f4cf8c01-5243-4d6f-94ae-2c7a52660a64" />
- запускаем процесс (сам раннер)
- <img width="1904" height="457" alt="image_2026-01-13_18-33-30" src="https://github.com/user-attachments/assets/4498f7aa-2640-410a-9c13-33b801a1bf51" />
- раннер готов слушать наш репозиторий
- <img width="797" height="292" alt="image_2026-01-13_18-34-12" src="https://github.com/user-attachments/assets/ae1a086d-6d34-4f15-b988-5313e53a311f" />

## Теперь немного попишем код
- создадим дефолтное веб апи для дотнета `dotnet new webapi`
- <img width="1198" height="861" alt="image_2026-01-13_19-10-27" src="https://github.com/user-attachments/assets/39a03da2-e332-46f7-a27a-35f86dcf98da" />
- добавим докер файл с прошлой лабы (поменям имя проекта и dll)
- <img width="522" height="436" alt="image" src="https://github.com/user-attachments/assets/a3139556-d13b-4ee6-90b1-6440bc111fe7" />
- добавим `.dockeriignore` (брал отсюда - https://github.com/dotnet/runtime/blob/main/.dockerignore)

## К самим пайплайнам 
сначала напишем плохой пайплайн, убедимся что все работает, потом разберем что не так. поехали

## Плохой пайплайн
- я сделал чет в таком духе
- <img width="529" height="415" alt="image" src="https://github.com/user-attachments/assets/efa61877-3dd0-4148-956f-aab9bc4128d3" />
- попробовал запустить (залил код), получил вот такое
- <img width="1850" height="212" alt="image_2026-01-13_19-29-06" src="https://github.com/user-attachments/assets/b934a051-1d02-43e2-8abd-3844b6e7766f" />
- похоже была проблема с правами, команда `sudo usermod -aG docker andrew` помогла. пробуем запустить еще раз. вроде что-то есть:
- <img width="714" height="1125" alt="image_2026-01-13_19-42-14" src="https://github.com/user-attachments/assets/64574b84-2d23-45f2-9aac-0b38fb270c63" />
- пайплайн прошел
- теперь добавим новый функционал. к примеру новую ручку `/bad-workflow` которой сейчас нет. пробуем обратиться - 404. понятное дело
- <img width="2123" height="406" alt="image_2026-01-13_20-32-38" src="https://github.com/user-attachments/assets/d856ad49-5faa-46ad-b2fd-6895cf496ec6" />
- создадим новую ветку `git checkout -b feature/deploy-with-bad-workflow-test`. добавим в ней новую ручку. пушим. как крутые разработчики делаем мр
- <img width="1200" height="1187" alt="image_2026-01-13_20-32-17" src="https://github.com/user-attachments/assets/cb65d9d9-26a2-4b94-9e7e-5b8e17d39ee4" />
- и сразу же сливаем :)
- бежим смотреть на джобы - при слиянии в дев новый коммит, изменения должны были выложится на сервер. джоба успешная
- <img width="587" height="539" alt="image_2026-01-13_20-34-50" src="https://github.com/user-attachments/assets/234cb50d-3717-4bc8-aeb4-c2d802489740" />
- проверим новую ручку `/bad-workflow`
- <img width="2111" height="443" alt="image_2026-01-13_20-35-05" src="https://github.com/user-attachments/assets/3c21d0d0-8599-42a3-8ed0-bec24c77fc76" />
- все работает. по итогу при пуше в дев у нас автоматически
  - собрался образ
  - все выложилось на сервер: прошлый контейнер грохнули, создали новый, запустили в фоновом режиме
  - на севрере новый код

## Хороший пайлайн
разберем все ошибки прошлого пайлайно тут. а потом напишем нормальный и проверим


















