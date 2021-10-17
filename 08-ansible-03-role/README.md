# Ответы на домашнее задание к занятию "08.03 Работа с Roles"

## Подготовка к выполнению
1. Создайте два пустых публичных репозитория в любом своём проекте: elastic-role и kibana-role.
2. Скачайте [role](./roles/) из репозитория с домашним заданием и перенесите его в свой репозиторий elastic-role.
3. Скачайте дистрибутив [java](https://www.oracle.com/java/technologies/javase-jdk11-downloads.html) и положите его в директорию `playbook/files/`. 
4. Установите molecule: `pip3 install molecule`
5. Добавьте публичную часть своего ключа к своему профилю в github.

```text
ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/vagrant/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/vagrant/.ssh/id_rsa
Your public key has been saved in /home/vagrant/.ssh/id_rsa.pub
...

cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3............Cc4c= vagrant@vagrant

Заходим на github.com -> Settings -> SSH and GPG keys -> New SSH key
Копируем id_rsa.pub в поле Key
```

## Основная часть

Наша основная цель - разбить наш playbook на отдельные roles. Задача: сделать roles для elastic, kibana и написать playbook для использования этих ролей. Ожидаемый результат: существуют два ваших репозитория с roles и один репозиторий с playbook.

1. Создать в старой версии playbook файл `requirements.yml` и заполнить его следующим содержимым:
   ```yaml
   ---
     - src: git@github.com:netology-code/mnt-homeworks-ansible.git
       scm: git
       version: "1.0.1"
       name: java 
   ```
2. При помощи `ansible-galaxy` скачать себе эту роль. Запустите  `molecule test`, посмотрите на вывод команды.

```text
ansible-galaxy install -r requirements.yml -p roles
Starting galaxy role install process
Enter passphrase for key '/home/vagrant/.ssh/id_rsa':
- extracting java to /home/vagrant/ansible/8.3/playbook/roles/java
- java (1.0.1) was installed successfully
```

```text
С molecule test для java "из коробки" (git@github.com:netology-code/mnt-homeworks-ansible.git) возникла проблема:
molecule test
INFO     default scenario test matrix: dependency, lint, cleanup, destroy, syntax, create, prepare, converge, idempotence, side_effect, verify, cleanup, destroy
INFO     Performing prerun...
INFO     Added ANSIBLE_LIBRARY=/home/vagrant/.cache/ansible-compat/df9dad/modules:/home/vagrant/.ansible/plugins/modules:/usr/share/ansible/plugins/modules
INFO     Added ANSIBLE_COLLECTIONS_PATH=/home/vagrant/.cache/ansible-compat/df9dad/collections:/home/vagrant/.ansible/collections:/usr/share/ansible/collections
INFO     Added ANSIBLE_ROLES_PATH=/home/vagrant/.cache/ansible-compat/df9dad/roles:/home/vagrant/.ansible/roles:/usr/share/ansible/roles:/etc/ansible/roles
ERROR    Computed fully qualified role name of java does not follow current galaxy requirements.
Please edit meta/main.yml and assure we can correctly determine full role name:

galaxy_info:
role_name: my_name  # if absent directory name hosting role is used instead
namespace: my_galaxy_namespace  # if absent, author is used instead
...

Поиск в интернете показал:
2 solutions:
- either change the value of "author" and change it to lowercase
- or inject "namespace: openstack"

Я залез в файл meta/main.yml, дописал "role_name" и исправил "author". Только после этого заработало.
Как по мне, то обучение по примерам, которые работают только после того, как их отдебажишь и допилишь, это плохой способ для платного обучения!
Пример для обучения должен работать "из коробки", чтобы понимать как правильно настраивать и запускать инструмент.
Хоть это и познавательно, но уж слишком много времени уходит на доработки примеров и реализацию домашних заданий.

-------------------------
После всех манипуляций molecule test заработал:
molecule test
INFO     default scenario test matrix: dependency, lint, cleanup, destroy, syntax, create, prepare, converge, idempotence, side_effect, verify, cleanup, destroy
INFO     Performing prerun...
INFO     Added ANSIBLE_LIBRARY=/home/vagrant/.cache/ansible-compat/1d6dd7/modules:/home/vagrant/.ansible/plugins/modules:/usr/share/ansible/plugins/modules
INFO     Added ANSIBLE_COLLECTIONS_PATH=/home/vagrant/.cache/ansible-compat/1d6dd7/collections:/home/vagrant/.ansible/collections:/usr/share/ansible/collections
INFO     Added ANSIBLE_ROLES_PATH=/home/vagrant/.cache/ansible-compat/1d6dd7/roles:/home/vagrant/.ansible/roles:/usr/share/ansible/roles:/etc/ansible/roles
INFO     Using /home/vagrant/.ansible/roles/alexey_metlyakov.java symlink to current repository in order to enable Ansible to find the role using its expected full name.
INFO     Running default > dependency
INFO     Running ansible-galaxy collection install -v community.docker:>=1.8.0
WARNING  Skipping, missing the requirements file.
WARNING  Skipping, missing the requirements file.
INFO     Running default > lint
INFO     Lint is disabled.
INFO     Running default > cleanup
WARNING  Skipping, cleanup playbook not configured.
INFO     Running default > destroy
INFO     Sanity checks: 'docker'

PLAY [Destroy] *****************************************************************

TASK [Destroy molecule instance(s)] ********************************************
changed: [localhost] => (item=centos8)
changed: [localhost] => (item=centos7)
changed: [localhost] => (item=ubuntu)

TASK [Wait for instance(s) deletion to complete] *******************************
ok: [localhost] => (item=centos8)
ok: [localhost] => (item=centos7)
ok: [localhost] => (item=ubuntu)

TASK [Delete docker networks(s)] ***********************************************

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0

INFO     Running default > syntax

...............
...............
...............

PLAY [Destroy] *****************************************************************

TASK [Destroy molecule instance(s)] ********************************************
changed: [localhost] => (item=centos8)
changed: [localhost] => (item=centos7)
changed: [localhost] => (item=ubuntu)

TASK [Wait for instance(s) deletion to complete] *******************************
changed: [localhost] => (item=centos8)
changed: [localhost] => (item=centos7)
changed: [localhost] => (item=ubuntu)

TASK [Delete docker networks(s)] ***********************************************

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0

INFO     Pruning extra files from scenario ephemeral directory
```

3. Перейдите в каталог с ролью elastic-role и создайте сценарий тестирования по умолчаню при помощи `molecule init scenario --driver-name docker`.

```text
molecule init scenario --driver-name docker
INFO     Initializing new scenario default...
INFO     Initialized scenario in /home/vagrant/ansible/8.3/playbook/roles/elastic-role/molecule/default successfully.
```

4. Добавьте несколько разных дистрибутивов (centos:8, ubuntu:latest) для инстансов и протестируйте роль, исправьте найденные ошибки, если они есть.

```text
molecule test
INFO     default scenario test matrix: dependency, lint, cleanup, destroy, syntax, create, prepare, converge, idempotence, side_effect, verify, cleanup, destroy
INFO     Performing prerun...
INFO     Added ANSIBLE_LIBRARY=/home/vagrant/.cache/ansible-compat/57ac47/modules:/home/vagrant/.ansible/plugins/modules:/usr/share/ansible/plugins/modules
INFO     Added ANSIBLE_COLLECTIONS_PATH=/home/vagrant/.cache/ansible-compat/57ac47/collections:/home/vagrant/.ansible/collections:/usr/share/ansible/collections
INFO     Added ANSIBLE_ROLES_PATH=/home/vagrant/.cache/ansible-compat/57ac47/roles:/home/vagrant/.ansible/roles:/usr/share/ansible/roles:/etc/ansible/roles
INFO     Using /home/vagrant/.ansible/roles/alexey_metlyakov.elastic symlink to current repository in order to enable Ansible to find the role using its expected full name.
INFO     Running default > dependency
WARNING  Skipping, missing the requirements file.
WARNING  Skipping, missing the requirements file.
INFO     Running default > lint
INFO     Lint is disabled.
INFO     Running default > cleanup
WARNING  Skipping, cleanup playbook not configured.
INFO     Running default > destroy
INFO     Sanity checks: 'docker'

PLAY [Destroy] *****************************************************************

TASK [Destroy molecule instance(s)] ********************************************
changed: [localhost] => (item=centos8)
changed: [localhost] => (item=centos7)
changed: [localhost] => (item=ubuntu)

TASK [Wait for instance(s) deletion to complete] *******************************
ok: [localhost] => (item=centos8)
ok: [localhost] => (item=centos7)
ok: [localhost] => (item=ubuntu)

TASK [Delete docker networks(s)] ***********************************************

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0

INFO     Running default > syntax

...............
...............
...............

PLAY [Destroy] *****************************************************************

TASK [Destroy molecule instance(s)] ********************************************
changed: [localhost] => (item=centos8)
changed: [localhost] => (item=centos7)
changed: [localhost] => (item=ubuntu)

TASK [Wait for instance(s) deletion to complete] *******************************
changed: [localhost] => (item=centos8)
changed: [localhost] => (item=centos7)
changed: [localhost] => (item=ubuntu)

TASK [Delete docker networks(s)] ***********************************************

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0

INFO     Pruning extra files from scenario ephemeral directory
```

5. Создайте новый каталог с ролью при помощи `molecule init role --driver-name docker kibana-role`. Можете использовать другой драйвер, который более удобен вам.

```text
cd roles/
molecule init role --driver-name docker kibana-role
INFO     Initializing new role kibana-role...
No config file found; using defaults
- Role kibana-role was created successfully
INFO     Initialized role in /home/vagrant/ansible/8.3/playbook/roles/kibana-role successfully.
```

6. На основе tasks из старого playbook заполните новую role. Разнесите переменные между `vars` и `default`. Проведите тестирование на разных дистрибитивах (centos:7, centos:8, ubuntu).

```text
Перенёс tasks для Kibana в kibana-role/tasks/main.yml, переменные в default и запустил тест:

molecule test
INFO     default scenario test matrix: dependency, lint, cleanup, destroy, syntax, create, prepare, converge, idempotence, side_effect, verify, cleanup, destroy
INFO     Performing prerun...
INFO     Added ANSIBLE_LIBRARY=/home/vagrant/.cache/ansible-compat/951136/modules:/home/vagrant/.ansible/plugins/modules:/usr/share/ansible/plugins/modules
INFO     Added ANSIBLE_COLLECTIONS_PATH=/home/vagrant/.cache/ansible-compat/951136/collections:/home/vagrant/.ansible/collections:/usr/share/ansible/collections
INFO     Added ANSIBLE_ROLES_PATH=/home/vagrant/.cache/ansible-compat/951136/roles:/home/vagrant/.ansible/roles:/usr/share/ansible/roles:/etc/ansible/roles
INFO     Using /home/vagrant/.ansible/roles/vitaly_mozhaev.kibana symlink to current repository in order to enable Ansible to find the role using its expected full name.
INFO     Running default > dependency
WARNING  Skipping, missing the requirements file.
WARNING  Skipping, missing the requirements file.
INFO     Running default > lint
INFO     Lint is disabled.
INFO     Running default > cleanup
WARNING  Skipping, cleanup playbook not configured.
INFO     Running default > destroy
INFO     Sanity checks: 'docker'

PLAY [Destroy] *****************************************************************

TASK [Destroy molecule instance(s)] ********************************************
changed: [localhost] => (item=centos8)
changed: [localhost] => (item=centos7)
changed: [localhost] => (item=ubuntu)

TASK [Wait for instance(s) deletion to complete] *******************************
ok: [localhost] => (item=centos8)
ok: [localhost] => (item=centos7)
ok: [localhost] => (item=ubuntu)

TASK [Delete docker networks(s)] ***********************************************

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0

INFO     Running default > syntax

...............
...............
...............

PLAY [Destroy] *****************************************************************

TASK [Destroy molecule instance(s)] ********************************************
changed: [localhost] => (item=centos8)
changed: [localhost] => (item=centos7)
changed: [localhost] => (item=ubuntu)

TASK [Wait for instance(s) deletion to complete] *******************************
FAILED - RETRYING: Wait for instance(s) deletion to complete (300 retries left).
changed: [localhost] => (item=centos8)
changed: [localhost] => (item=centos7)
changed: [localhost] => (item=ubuntu)

TASK [Delete docker networks(s)] ***********************************************

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0

INFO     Pruning extra files from scenario ephemeral directory
```

7. Выложите все roles в репозитории. Проставьте тэги, используя семантическую нумерацию.
8. Добавьте roles в `requirements.yml` в playbook.

```text
---
  - src: git@github.com:netology-code/mnt-homeworks-ansible.git
    scm: git
    version: "1.0.1"
    name: java
  - src: git@github.com:VitalyMozhaev/elastic-role.git
    scm: git
    version: "1.0.1"
    name: elastic
  - src: git@github.com:VitalyMozhaev/kibana-role.git
    scm: git
    version: "1.0.1"
    name: kibana
```

9. Переработайте playbook на использование roles.

```text
---
- name: Playbook roles Java and Elastic and Kibana
  hosts: all
  roles:
    - java
    - elastic
    - kibana
```

10. Выложите playbook в репозиторий.

```text
Проверяем сам playbook.
Если запускать без sudo, то почему-то не проходит шаг TASK [elastic : Upload tar.gz Elasticsearch from remote URL].
Запускаем с sudo:

sudo ansible-playbook site.yml -i inventories/tests/hosts.yml

PLAY [Playbook roles Java and Elastic and Kibana] ******************************

TASK [Gathering Facts] *********************************************************
ok: [localhost]

TASK [java : Upload .tar.gz file containing binaries from local storage] *******
skipping: [localhost]

TASK [java : Upload .tar.gz file conaining binaries from remote storage] *******
changed: [localhost]

TASK [java : Ensure installation dir exists] ***********************************
changed: [localhost]

TASK [java : Extract java in the installation directory] ***********************
changed: [localhost]

TASK [java : Export environment variables] *************************************
changed: [localhost]

TASK [elastic : Upload tar.gz Elasticsearch from remote URL] *******************
changed: [localhost]

TASK [elastic : Create directrory for Elasticsearch] ***************************
ok: [localhost]

TASK [elastic : Extract Elasticsearch in the installation directory] ***********
skipping: [localhost]

TASK [elastic : Set environment Elastic] ***************************************
changed: [localhost]

TASK [kibana : Upload tar.gz Kibana from remote URL] ***************************
changed: [localhost]

TASK [kibana : Create directrory for Kibana] ***********************************
ok: [localhost]

TASK [kibana : Extract Kibana in the installation directory] *******************
skipping: [localhost]

TASK [kibana : Set environment Kibana] *****************************************
ok: [localhost]

PLAY RECAP *********************************************************************
localhost                  : ok=11   changed=7    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0
```

11. В ответ приведите ссылки на оба репозитория с roles и одну ссылку на репозиторий с playbook.

`Ссылки на репозитории:`

https://github.com/VitalyMozhaev/elastic-role

https://github.com/VitalyMozhaev/kibana-role

https://github.com/VitalyMozhaev/mnt-homeworks-ansible-3

## Необязательная часть

1. Проделайте схожие манипуляции для создания роли logstash.
2. Создайте дополнительный набор tasks, который позволяет обновлять стек ELK.
3. В ролях добавьте тестирование в раздел `verify.yml`. Данный раздел должен проверять, что elastic запущен и возвращает успешный статус по API, web-интерфейс kibana отвечает без кодов ошибки, logstash через команду `logstash -e 'input { stdin { } } output { stdout {} }'`.
4. Убедитесь в работоспособности своего стека. Возможно, потребуется тестировать все роли одновременно.
5. Выложите свои roles в репозитории. В ответ приведите ссылки.
