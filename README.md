# BINGO!

:: Вариант решения финального задания по тренировкам DevOps от Yandex ::

Инструкции по развертыванию инфраструктуры на Yandex Cloud :
1. Поместите `authorized_key.json` вашего сервисного аккаунта с достаточными привилегиями
2. Укажите Folder ID директории Yandex Cloud в файле `variables.tf`
3. Будучи в корневой директории с файлом `main.tf` введите `terraform apply` и `yes`
4. ???
5. Вы восхитительны!

Инфраструктура разворачивается в течение 5 минут, после чего вы получаете полностью работоспособное приложение по адресу `bingo.neverservers.ru`

Для получения SSH доступа до виртуальных машин - используйте `id_rsa` (закрытый SSH ключ)

:: Схема взаимодействия нод в инфрастуктуре ::

![285672564-d5d3fee6-f29f-46e6-b0f9-51ee1092a1de](https://github.com/detree05/my-little-project/assets/125824800/be4b8ec3-b0b3-442b-a7d8-b94b90cd8af1)

:: С помощью чего было реализовано решение ::

- Terraform (автоматизация развертки)
- Nginx (обратный прокси, балансировщик, кэш)
- PostgreSQL (база данных)
- Docker (контейнеризованные приложения (PostgreSQL, bingo, nginx))
- Docker Compose (автоматизация развертки контейнеров на виртуальных машинах и их контроль)
