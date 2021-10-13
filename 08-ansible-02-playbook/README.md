# Ответы на домашнее задание к занятию "08.02 Работа с Playbook"

## Подготовка к выполнению
1. Создайте свой собственный (или используйте старый) публичный репозиторий на github с произвольным именем.

`Репозиторий:` https://github.com/VitalyMozhaev/mnt-homeworks-ansible-2

2. Скачайте [playbook](./playbook/) из репозитория с домашним заданием и перенесите его в свой репозиторий.
3. Подготовьте хосты в соотвтествии с группами из предподготовленного playbook. 
4. Скачайте дистрибутив [java](https://www.oracle.com/java/technologies/javase-jdk11-downloads.html) и положите его в директорию `playbook/files/`. 

## Основная часть
1. Приготовьте свой собственный inventory файл `prod.yml`.

```text
---
elasticsearch:
  hosts:
    localhost:
      ansible_connection: local
      ansible_user: root
kibana:
  hosts:
    localhost:
      ansible_connection: local
      ansible_user: root
```

2. Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает kibana.
3. При создании tasks рекомендую использовать модули: `get_url`, `template`, `unarchive`, `file`.
4. Tasks должны: скачать нужной версии дистрибутив, выполнить распаковку в выбранную директорию, сгенерировать конфигурацию с параметрами.

```text
- name: Install Kibana
  hosts: kibana
  tasks:
    - name: Upload tar.gz Kibana from remote URL
      get_url:
        url: "https://artifacts.elastic.co/downloads/kibana/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"
        dest: "/tmp/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"
        mode: 0755
        timeout: 60
        force: true
        validate_certs: false
      register: get_kibana
      until: get_kibana is succeeded
      tags: kibana
    - name: Create directrory for Kibana
      file:
        state: directory
        path: "{{ kibana_home }}"
      tags: kibana
    - name: Extract Kibana in the installation directory
      become: true
      unarchive:
        copy: false
        src: "/tmp/kibana-{{ kibana_version }}-linux-x86_64.tar.gz"
        dest: "{{ kibana_home }}"
        extra_opts: [--strip-components=1]
        creates: "{{ kibana_home }}/bin/kibana"
      tags:
        - kibana
    - name: Set environment Kibana
      become: true
      template:
        src: templates/kib.sh.j2
        dest: /etc/profile.d/kib.sh
      tags: kibana
```

5. Запустите `ansible-lint site.yml` и исправьте ошибки, если они есть.

```text
ansible-lint
WARNING  Listing 1 violation(s) that are fatal
yaml: too many blank lines (1 > 0) (empty-lines)
inventory/prod.yml:14

You can skip specific rules or tags by adding them to your configuration file:
# .ansible-lint
warn_list:  # or 'skip_list' to silence them completely
  - yaml  # Violations reported by yamllint

Finished with 1 failure(s), 0 warning(s) on 8 files.


# Исправил: убрал лишний пробел в конце файла inventory/prod.yml
```

6. Попробуйте запустить playbook на этом окружении с флагом `--check`.

```text
ansible-playbook site.yml -i inventory/prod.yml --check

PLAY [Install Java] ************************************************************

TASK [Gathering Facts] *********************************************************
ok: [localhost]

TASK [Set facts for Java 11 vars] **********************************************
ok: [localhost]

TASK [Upload .tar.gz file containing binaries from local storage] **************
changed: [localhost]

TASK [Ensure installation dir exists] ******************************************
ok: [localhost]

TASK [Extract java in the installation directory] ******************************
skipping: [localhost]

TASK [Export environment variables] ********************************************
ok: [localhost]

PLAY [Install Elasticsearch] ***************************************************

TASK [Gathering Facts] *********************************************************
ok: [localhost]

TASK [Upload tar.gz Elasticsearch from remote URL] *****************************
changed: [localhost]

TASK [Create directrory for Elasticsearch] *************************************
ok: [localhost]

TASK [Extract Elasticsearch in the installation directory] *********************
skipping: [localhost]

TASK [Set environment Elastic] *************************************************
changed: [localhost]

PLAY [Install Kibana] **********************************************************

TASK [Gathering Facts] *********************************************************
ok: [localhost]

TASK [Upload tar.gz Kibana from remote URL] ************************************
changed: [localhost]

TASK [Create directrory for Kibana] ********************************************
changed: [localhost]

TASK [Extract Kibana in the installation directory] ****************************
An exception occurred during task execution. To see the full traceback, use -vvv. The error was: NoneType: None
fatal: [localhost]: FAILED! => {"changed": false, "msg": "dest '/opt/kibana/7.10.1' must be an existing dir"}

PLAY RECAP *********************************************************************
localhost                  : ok=12   changed=5    unreachable=0    failed=1    skipped=2    rescued=0    ignored=0
```

7. Запустите playbook на `prod.yml` окружении с флагом `--diff`. Убедитесь, что изменения на системе произведены.

```text
sudo ansible-playbook site.yml -i inventory/prod.yml --diff

PLAY [Install Java] ************************************************************

TASK [Gathering Facts] *********************************************************
ok: [localhost]

TASK [Set facts for Java 11 vars] **********************************************
ok: [localhost]

TASK [Upload .tar.gz file containing binaries from local storage] **************
ok: [localhost]

TASK [Ensure installation dir exists] ******************************************
ok: [localhost]

TASK [Extract java in the installation directory] ******************************
skipping: [localhost]

TASK [Export environment variables] ********************************************
ok: [localhost]

PLAY [Install Elasticsearch] ***************************************************

TASK [Gathering Facts] *********************************************************
ok: [localhost]

TASK [Upload tar.gz Elasticsearch from remote URL] *****************************
ok: [localhost]

TASK [Create directrory for Elasticsearch] *************************************
ok: [localhost]

TASK [Extract Elasticsearch in the installation directory] *********************
skipping: [localhost]

TASK [Set environment Elastic] *************************************************
ok: [localhost]

PLAY [Install Kibana] **********************************************************

TASK [Gathering Facts] *********************************************************
ok: [localhost]

TASK [Upload tar.gz Kibana from remote URL] ************************************
ok: [localhost]

TASK [Create directrory for Kibana] ********************************************
--- before
+++ after
@@ -1,4 +1,4 @@
 {
     "path": "/opt/kibana/7.10.1",
-    "state": "absent"
+    "state": "directory"
 }

changed: [localhost]

TASK [Extract Kibana in the installation directory] ****************************
changed: [localhost]

TASK [Set environment Kibana] **************************************************
--- before
+++ after: /root/.ansible/tmp/ansible-local-4082iop8zwbr/tmpsh3wut23/kib.sh.j2
@@ -0,0 +1,6 @@
+# Warning: This file is Ansible Managed, manual changes will be overwritten onnext playbook run.
+#!/usr/bin/env bash
+
+# Kibana
+export KB_HOME=/opt/kibana/7.10.1
+export PATH=$PATH:$KB_HOME/bin

changed: [localhost]

PLAY RECAP *********************************************************************
localhost                  : ok=14   changed=3    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
```

8. Повторно запустите playbook с флагом `--diff` и убедитесь, что playbook идемпотентен.

```text
sudo ansible-playbook site.yml -i inventory/prod.yml --diff

PLAY [Install Java] ************************************************************

TASK [Gathering Facts] *********************************************************
ok: [localhost]

TASK [Set facts for Java 11 vars] **********************************************
ok: [localhost]

TASK [Upload .tar.gz file containing binaries from local storage] **************
ok: [localhost]

TASK [Ensure installation dir exists] ******************************************
ok: [localhost]

TASK [Extract java in the installation directory] ******************************
skipping: [localhost]

TASK [Export environment variables] ********************************************
ok: [localhost]

PLAY [Install Elasticsearch] ***************************************************

TASK [Gathering Facts] *********************************************************
ok: [localhost]

TASK [Upload tar.gz Elasticsearch from remote URL] *****************************
ok: [localhost]

TASK [Create directrory for Elasticsearch] *************************************
ok: [localhost]

TASK [Extract Elasticsearch in the installation directory] *********************
skipping: [localhost]

TASK [Set environment Elastic] *************************************************
ok: [localhost]

PLAY [Install Kibana] **********************************************************

TASK [Gathering Facts] *********************************************************
ok: [localhost]

TASK [Upload tar.gz Kibana from remote URL] ************************************
ok: [localhost]

TASK [Create directrory for Kibana] ********************************************
ok: [localhost]

TASK [Extract Kibana in the installation directory] ****************************
skipping: [localhost]

TASK [Set environment Kibana] **************************************************
ok: [localhost]

PLAY RECAP *********************************************************************
localhost                  : ok=13   changed=0    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0
```

9. Подготовьте README.md файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги.
10. Готовый playbook выложите в свой репозиторий, в ответ предоставьте ссылку на него.

`Репозиторий:` https://github.com/VitalyMozhaev/mnt-homeworks-ansible-2

## Необязательная часть

1. Приготовьте дополнительный хост для установки logstash.
2. Пропишите данный хост в `prod.yml` в новую группу `logstash`.
3. Дополните playbook ещё одним play, который будет исполнять установку logstash только на выделенный для него хост.
4. Все переменные для нового play определите в отдельный файл `group_vars/logstash/vars.yml`.
5. Logstash конфиг должен конфигурироваться в части ссылки на elasticsearch (можно взять, например его IP из facts или определить через vars).
6. Дополните README.md, протестируйте playbook, выложите новую версию в github. В ответ предоставьте ссылку на репозиторий.
