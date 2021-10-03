# Домашнее задание к занятию "08.01 Введение в Ansible"

## Подготовка к выполнению
1. Установите ansible версии 2.10 или выше.

```text
pip3 install ansible --user
...
Successfully installed MarkupSafe-2.0.1 ansible-4.6.0 ansible-core-2.11.5 jinja2-3.0.1 packaging-21.0 pyparsing-2.4.7 resolvelib-0.5.4

# Для запуска также необходимо прописать в переменные окружения
export PATH=$PATH:~/.local/bin
```

2. Создайте свой собственный публичный репозиторий на github с произвольным именем.
3. Скачайте [playbook](./playbook/) из репозитория с домашним заданием и перенесите его в свой репозиторий.

## Основная часть
1. Попробуйте запустить playbook на окружении из `test.yml`, зафиксируйте какое значение имеет факт `some_fact` для указанного хоста при выполнении playbook'a.

```text
ansible-playbook site.yml -i inventory/test.yml

PLAY [Print os facts] **********************************************************

TASK [Gathering Facts] *********************************************************
ok: [localhost]

TASK [Print OS] ****************************************************************
ok: [localhost] => {
    "msg": "Ubuntu"
}

TASK [Print fact] **************************************************************
ok: [localhost] => {
    "msg": 12
}

PLAY RECAP *********************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

# Факт some_fact = 12, это значение берётся из group_vars/all/examp.yml
```

2. Найдите файл с переменными (group_vars) в котором задаётся найденное в первом пункте значение и поменяйте его на 'all default fact'.

```text
# После внесения изменений

ansible-playbook site.yml -i inventory/test.yml

...

TASK [Print fact] **************************************************************
ok: [localhost] => {
    "msg": "all default fact"
}

PLAY RECAP *********************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

3. Воспользуйтесь подготовленным (используется `docker`) или создайте собственное окружение для проведения дальнейших испытаний.

```text
docker run -d --name centos7 pycontribs/centos:7 sleep 10000000
docker run -d --name ubuntu pycontribs/ubuntu sleep 10000000
```

4. Проведите запуск playbook на окружении из `prod.yml`. Зафиксируйте полученные значения `some_fact` для каждого из `managed host`.

```text
ansible-playbook site.yml -i inventory/prod.yml

PLAY [Print os facts] **********************************************************

TASK [Gathering Facts] *********************************************************
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] ****************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] **************************************************************
ok: [centos7] => {
    "msg": "el"
}
ok: [ubuntu] => {
    "msg": "deb"
}

PLAY RECAP *********************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

5. Добавьте факты в `group_vars` каждой из групп хостов так, чтобы для `some_fact` получились следующие значения: для `deb` - 'deb default fact', для `el` - 'el default fact'.

```text
Устанваливаем значения:
/group_vars/deb/examp.yml - some_fact: "deb default fact"
/group_vars/el/examp.yml - some_fact: "el default fact"
```

6.  Повторите запуск playbook на окружении `prod.yml`. Убедитесь, что выдаются корректные значения для всех хостов.

```text
ansible-playbook site.yml -i inventory/prod.yml

PLAY [Print os facts] **********************************************************

TASK [Gathering Facts] *********************************************************
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] ****************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] **************************************************************
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}

PLAY RECAP *********************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

7. При помощи `ansible-vault` зашифруйте факты в `group_vars/deb` и `group_vars/el` с паролем `netology`.

```text
ansible-vault encrypt group_vars/deb/examp.yml
New Vault password:
Confirm New Vault password:
Encryption successful

cat group_vars/deb/examp.yml
$ANSIBLE_VAULT;1.1;AES256
32363731623039363536396537383936383239633233353035663966623466636538663931613434
3639653065653333366130623766306638336635393964380a326239343637313537306135626161
34393365633534366134333130663932303165376332363133663863633336326435356464626633
3032313663323562610a343162333839373331353434346635323032646263643038313739313561
61333532663531323661333664313238363535313337323939643634393736376234626532346638
3634646636326434376565336536376462616137326334633330

ansible-vault encrypt group_vars/el/examp.yml
New Vault password:
Confirm New Vault password:
Encryption successful

cat group_vars/el/examp.yml
$ANSIBLE_VAULT;1.1;AES256
32316661313361663935643962336263663833653231316436613761333936366665353966633266
6230633662363936356332626439396564633332633731370a383935663930356664353833303166
66386136613563623133303039613835636237396562363633376232613761363035653063646164
6162646663646330300a626465623765643965393537646436666138386662353161353631316564
34373936666263623931303863643466346665636138326266346436373536343066323938666165
6162383362363233303262633062366436623730366365343466
```

8. Запустите playbook на окружении `prod.yml`. При запуске `ansible` должен запросить у вас пароль. Убедитесь в работоспособности.

```text
ansible-playbook site.yml -i inventory/prod.yml --ask-vault-pass
Vault password:

PLAY [Print os facts] **********************************************************

TASK [Gathering Facts] *********************************************************
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] ****************************************************************
ok: [centos7] => {
    "msg": "CentOS"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}

TASK [Print fact] **************************************************************
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}

PLAY RECAP *********************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

9. Посмотрите при помощи `ansible-doc` список плагинов для подключения. Выберите подходящий для работы на `control node`.

```text
control node у нас local
```

10. В `prod.yml` добавьте новую группу хостов с именем  `local`, в ней разместите localhost с необходимым типом подключения.

```text
---
  el:
    hosts:
      centos7:
        ansible_connection: docker
  deb:
    hosts:
      ubuntu:
        ansible_connection: docker
  local:
    hosts:
      localhost:
        ansible_connection: local
```

11. Запустите playbook на окружении `prod.yml`. При запуске `ansible` должен запросить у вас пароль. Убедитесь что факты `some_fact` для каждого из хостов определены из верных `group_vars`.

```text
ansible-playbook site.yml -i inventory/prod.yml --ask-vault-pass
Vault password:

PLAY [Print os facts] **********************************************************

TASK [Gathering Facts] *********************************************************
ok: [localhost]
ok: [ubuntu]
ok: [centos7]

TASK [Print OS] ****************************************************************
ok: [localhost] => {
    "msg": "Ubuntu"
}
ok: [ubuntu] => {
    "msg": "Ubuntu"
}
ok: [centos7] => {
    "msg": "CentOS"
}

TASK [Print fact] **************************************************************
ok: [localhost] => {
    "msg": "all default fact"
}
ok: [centos7] => {
    "msg": "el default fact"
}
ok: [ubuntu] => {
    "msg": "deb default fact"
}

PLAY RECAP *********************************************************************
centos7                    : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ubuntu                     : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

12. Заполните `README.md` ответами на вопросы. Сделайте `git push` в ветку `master`. В ответе отправьте ссылку на ваш открытый репозиторий с изменённым `playbook` и заполненным `README.md`.

Ссылка на репозиторий: https://github.com/VitalyMozhaev/mnt-homeworks-ansible

## Необязательная часть

```text
К сожалению, не успеваю сдавать ДЗ вовремя, поэтому приходится пропускать дополнительные задания, чтобы догнать поток.
```

1. При помощи `ansible-vault` расшифруйте все зашифрованные файлы с переменными.
2. Зашифруйте отдельное значение `PaSSw0rd` для переменной `some_fact` паролем `netology`. Добавьте полученное значение в `group_vars/all/exmp.yml`.
3. Запустите `playbook`, убедитесь, что для нужных хостов применился новый `fact`.
4. Добавьте новую группу хостов `fedora`, самостоятельно придумайте для неё переменную. В качестве образа можно использовать [этот](https://hub.docker.com/r/pycontribs/fedora).
5. Напишите скрипт на bash: автоматизируйте поднятие необходимых контейнеров, запуск ansible-playbook и остановку контейнеров.
6. Все изменения должны быть зафиксированы и отправлены в вашей личный репозиторий.
