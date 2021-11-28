# Ответы на домашнее задание к занятию "09.06 Gitlab"

## Подготовка к выполнению

1. Необходимо [зарегистрироваться](https://about.gitlab.com/free-trial/)
2. Создайте свой новый проект
3. Создайте новый репозиторий в gitlab, наполните его [файлами](./repository)
4. Проект должен быть публичным, остальные настройки по желанию

## Основная часть

### DevOps

В репозитории содержится код проекта на python. Проект - RESTful API сервис. Ваша задача автоматизировать сборку образа с выполнением python-скрипта:
1. Образ собирается на основе [centos:7](https://hub.docker.com/_/centos?tab=tags&page=1&ordering=last_updated)
2. Python версии не ниже 3.7
3. Установлены зависимости: `flask` `flask-jsonpify` `flask-restful`
4. Создана директория `/python_api`
5. Скрипт из репозитория размещён в /python_api
6. Точка вызова: запуск скрипта
7. Если сборка происходит на ветке `master`: Образ должен пушится в docker registry вашего gitlab `python-api:latest`, иначе этот шаг нужно пропустить

### Ответ:

Ссылка на репозиторий: https://gitlab.com/malik-vn/mnt-7/-/tree/main

### Product Owner

Вашему проекту нужна бизнесовая доработка: необходимо поменять JSON ответа на вызов метода GET `/rest/api/get_info`, необходимо создать Issue в котором указать:
1. Какой метод необходимо исправить
2. Текст с `{ "message": "Already started" }` на `{ "message": "Running"}`
3. Issue поставить label: feature

### Ответ:

![](https://github.com/VitalyMozhaev/mnt-homeworks/blob/main/09-ci-06-gitlab/issue.png)

### Developer

Вам пришел новый Issue на доработку, вам необходимо:
1. Создать отдельную ветку, связанную с этим issue
2. Внести изменения по тексту из задания
3. Подготовить Merge Requst, влить необходимые изменения в `master`, проверить, что сборка прошла успешно

### Ответ:

![](https://github.com/VitalyMozhaev/mnt-homeworks/blob/main/09-ci-06-gitlab/merge_request.png)

![](https://github.com/VitalyMozhaev/mnt-homeworks/blob/main/09-ci-06-gitlab/merge_request_build.png)

### Tester

Разработчики выполнили новый Issue, необходимо проверить валидность изменений:
1. Поднять докер-контейнер с образом `python-api:latest` и проверить возврат метода на корректность
2. Закрыть Issue с комментарием об успешности прохождения, указав желаемый результат и фактически достигнутый

### Ответ:

Тест на отдельной виртуалке:

```bash
docker pull registry.gitlab.com/malik-vn/mnt-7/python-api
docker run -d -p 5290:5290 --name python-api registry.gitlab.com/malik-vn/mnt-7/python-api:latest

curl localhost:5290/rest/api/get_info
{"version": 3, "method": "GET", "message": "Running"}
```

Автоматизировал тест после сборки через stage test (необязательная часть)

```bash
docker pull registry.gitlab.com/malik-vn/mnt-7/python-api
docker run -d -p 5290:5290 --name python-api registry.gitlab.com/malik-vn/mnt-7/python-api:latest
docker exec python-api curl -s http://0.0.0.0:5290/rest/api/get_info | grep Running && echo "Test succeeded"
{"version": 3, "method": "GET", "message": "Running"}
Test succeeded
```

## Итог

После успешного прохождения всех ролей - отправьте ссылку на ваш проект в гитлаб, как решение домашнего задания

### Ответ:

Ссылка на репозиторий: https://gitlab.com/malik-vn/mnt-7

## Необязательная часть

Автомазируйте работу тестировщика, пусть у вас будет отдельный конвейер, который автоматически поднимает контейнер и выполняет проверку, например, при помощи curl.
На основе вывода - будет приниматься решение об успешности прохождения тестирования

### Ответ:

Дописал stage test в pipeline.

Ссылка на .gitlab-ci.yml: https://gitlab.com/malik-vn/mnt-7/-/blob/main/.gitlab-ci.yml

Сборка с тестом:

![](https://github.com/VitalyMozhaev/mnt-homeworks/blob/main/09-ci-06-gitlab/Build_and_Test.png)