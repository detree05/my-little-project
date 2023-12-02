
# BINGO!
Вариант решения финального задания по  DevOps тренировкам.
# Схема инфраструктуры
![BINGO Cluster](https://github.com/detree05/my-little-project/assets/125824800/6b9150d5-3e3d-4640-bae5-dac1c943a3f0)
## Развертывание через Terraform

Клонируйте репозиторий, поместите в директорию с ним `authorized_key.json` сервисного аккаунта YC и укажите `Folder ID` в `variables.tf`, после чего выполните следующие команды
```
terraform init
terraform apply
```
После подтверждения изменений Terraform, сервис будет развернут в течение 5-8 минут в директории Вашего облака и будет доступен по адресу `bingo.neverservers.ru`

## Коды

| Код | Как нашел    |
| :-------- | :------- |
| `yoohoo_server_launched` | `Запустил сервер в первый раз` |
| `index_page_is_awesome` | `Обратился к корневой директории сервиса` |
| `google_dns_is_not_http` | `Решение проблемы с DNS, найденной через tcpdump` |

  
## Проблемы

| Проблема | Поиск    | Решение | Причина решения |
| :--------- | :---------| :--------- |  :--------- |
| `Первый старт bingo` | `man? --help?` | `Прописал bingo --help` | `-` |
| `Ошибка bingo run_server` | `strace bingo` | `Создал конфиг в /opt/bingo` | `Увидел в strace, что bingo пытается обратиться к несуществующему файлу` |
| `Ошибка bingo run_server` | `strace bingo` | `Создал базу данных PostgreSQL` | `Увидел в strace, что bingo пытается подключиться к БД, указанной в конфиге` |
| `Долгий для автоматизации bingo prepare_db` | `-` | `Сделал dump и автоматическое заполнение из него при развертке` | `Была дилемма между заполнением БД прямо в облаке после развертки, либо просто подгружать уже готовую БД. Выбрал второе решение из-за скорости и удобства` |
| `Две ноды?` | `-` | `Принял контейнер за ноду` | `Так как решение всей задачи я начал с автоматизации, я решил что для этого отлично подойдет Docker и Docker Compose` |
| `Инфраструктура` | `-` | `-` | `Думал над тем, как реализовать инфраструктуру и балансировщик. Отталкивался от второго домашнего задания, но так и не пришел ни к чему, решил сделать набросок плана инфраструктуры и выбрал решение с двумя виртуальными машинами: Одна под БД - Одна под сервис с балансировщиком. Так как балансировщик должен был быть прямо на хост машине, решил использовать контейнеризованный NGINX` |
| `Образы Docker и автоматизация развертывания на виртуальных машинах` | `Интернет. Много.` | `Docker Compose, cloud-init` | `Нужно было как-то автоматизировать развертку всего и везде, начиная от готовых сервисов в контейнерах, заканчивая тем, чтобы эти контейнеры хотя бы стартовали без внешнего вмешательства. Готовые сервисы реализовал через собственные Docker образы, куда поместил все необходимое для работы и задал конфиг файлы. Автоматизацию развертки на виртуальных машинах реализовал через cloud-init и compose.yaml` |
| `Долгий старт bingo` | `strace, tcpdump` | `iptables` | `Увидел, что бинарник очень долго долбится к DNS по 80 порту, хотя это немного некорректно. Решил исправить через iptables -t nat -A PREROUTING -p tcp -d 8.8.8.8 --dport 80 -j DNAT --to-destination 8.8.8.8:53, перенаправляя все пакеты, идущие на 8.8.8.8:80 - на 8.8.8.8:53` |
| `Кэширование` | `Документация NGINX` | `Реализовал кэширование location /long_dummy` | `Просто взял решение из коробки, которое было в документации и сделал кэширование сугубо для этого запроса` |
| `HTTP + HTTPS + HTTP3` | `Документация NGINX` | `listen 80; listen 443; Образ с http3 модулем` | `HTTP и HTTPS без переадресаций были реализованы через два listen, HTTP3 реализован через готовый Docker образ с модулем, необходимо было лишь подкрутить пару вещей в nginx.conf` |
| `I feel bad` | `-` | `Корректный healthcheck` | `Проверяя сервис под нагрузкой, заметил, что помимо моментов, когда сервис просто самоуничтожается, он может отдавать I feel bad в ответ на все запросы в течение нескольких минут, что не очень хорошо. Решил проблему через Docker Compose healthcheck - "curl ip-address/ping \| grep 'pong' \|\| kill 1"` |
| `Жирные запросы к БД` | `Лекции, интернет. Много.` | `Индексация и закручивание гаек в конфигурации PostgreSQL` | `Решение из коробки проходило все тесты, кроме трех - GET'ы. Отваливались из-за timeout. Сначала закрутил гайки в конфигурации и это решило два из трех тестов. Затем, прочитав тонну информации, проиндексировал все таблицы в БД, что решило оставшийся тест` |
| `Статические IP адреса` | `Документация Terraform Yandex Cloud` | `Изменение main.tf и использование DNS серверов в Облаке` | `Изначально вся инфраструктура строилась на статических адресах, что на самом деле я считал костылем. Прочитав документацию, обнаружил, что в принципе можно реализовать динамические адреса через DNS записи, т.е. независимо от того, какой адрес будет у машины - обращаться они будут именно по доменному имени, которое в свою очередь будет изменяться при развертывании (IP машины можно передавать в main.tf)` |
| `Просто, чтобы красиво` | `-` | `-` | `Чтобы каждый раз не глазеть, какой же там IP у машины с сервисом, оформил внешний DNS в main.tf с моим поддоменом, в домене же сделал NS запись, которая переадресовывает запросы на DNS сервер Yandex. Поэтому сервис в любом случае будет доступен по одному единственному адресу - bingo.neverservers.ru (сертификат соответственно прикрутил тоже настоящий)` |
