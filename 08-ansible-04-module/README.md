# Ответы на домашнее задание к занятию "08.04 Создание собственных modules"

## Подготовка к выполнению
1. Создайте пустой публичных репозиторий в любом своём проекте: `my_own_collection`
2. Скачайте репозиторий ansible: `git clone https://github.com/ansible/ansible.git` по любому удобному вам пути
3. Зайдите в директорию ansible: `cd ansible`
4. Создайте виртуальное окружение: `python3 -m venv venv`
5. Активируйте виртуальное окружение: `. venv/bin/activate`. Дальнейшие действия производятся только в виртуальном окружении
6. Установите зависимости `pip install -r requirements.txt`
7. Запустить настройку окружения `. hacking/env-setup`
8. Если все шаги прошли успешно - выйти из виртуального окружения `deactivate`
9. Ваше окружение настроено, для того чтобы запустить его, нужно находиться в директории `ansible` и выполнить конструкцию `. venv/bin/activate && . hacking/env-setup`

## Основная часть

Наша цель - написать собственный module, который мы можем использовать в своей role, через playbook. Всё это должно быть собрано в виде collection и отправлено в наш репозиторий.

1. В виртуальном окружении создать новый `my_own_module.py` файл

```bash
cd lib/ansible/modules
touch my_own_module.py
```

2. Наполнить его содержимым:
```python
#!/usr/bin/python

# Copyright: (c) 2018, Terry Jones <terry.jones@example.org>
# GNU General Public License v3.0+ (see COPYING or https://www.gnu.org/licenses/gpl-3.0.txt)
from __future__ import (absolute_import, division, print_function)
__metaclass__ = type

DOCUMENTATION = r'''
---
module: my_test

short_description: This is my test module

# If this is part of a collection, you need to use semantic versioning,
# i.e. the version is of the form "2.5.0" and not "2.4".
version_added: "1.0.0"

description: This is my longer description explaining my test module.

options:
    name:
        description: This is the message to send to the test module.
        required: true
        type: str
    new:
        description:
            - Control to demo if the result of this module is changed or not.
            - Parameter description can be a list as well.
        required: false
        type: bool
# Specify this value according to your collection
# in format of namespace.collection.doc_fragment_name
extends_documentation_fragment:
    - my_namespace.my_collection.my_doc_fragment_name

author:
    - Your Name (@yourGitHubHandle)
'''

EXAMPLES = r'''
# Pass in a message
- name: Test with a message
  my_namespace.my_collection.my_test:
    name: hello world

# pass in a message and have changed true
- name: Test with a message and changed output
  my_namespace.my_collection.my_test:
    name: hello world
    new: true

# fail the module
- name: Test failure of the module
  my_namespace.my_collection.my_test:
    name: fail me
'''

RETURN = r'''
# These are examples of possible return values, and in general should use other names for return values.
original_message:
    description: The original name param that was passed in.
    type: str
    returned: always
    sample: 'hello world'
message:
    description: The output message that the test module generates.
    type: str
    returned: always
    sample: 'goodbye'
'''

from ansible.module_utils.basic import AnsibleModule


def run_module():
    # define available arguments/parameters a user can pass to the module
    module_args = dict(
        name=dict(type='str', required=True),
        new=dict(type='bool', required=False, default=False)
    )

    # seed the result dict in the object
    # we primarily care about changed and state
    # changed is if this module effectively modified the target
    # state will include any data that you want your module to pass back
    # for consumption, for example, in a subsequent task
    result = dict(
        changed=False,
        original_message='',
        message=''
    )

    # the AnsibleModule object will be our abstraction working with Ansible
    # this includes instantiation, a couple of common attr would be the
    # args/params passed to the execution, as well as if the module
    # supports check mode
    module = AnsibleModule(
        argument_spec=module_args,
        supports_check_mode=True
    )

    # if the user is working with this module in only check mode we do not
    # want to make any changes to the environment, just return the current
    # state with no modifications
    if module.check_mode:
        module.exit_json(**result)

    # manipulate or modify the state as needed (this is going to be the
    # part where your module will do what it needs to do)
    result['original_message'] = module.params['name']
    result['message'] = 'goodbye'

    # use whatever logic you need to determine whether or not this module
    # made any modifications to your target
    if module.params['new']:
        result['changed'] = True

    # during the execution of the module, if there is an exception or a
    # conditional state that effectively causes a failure, run
    # AnsibleModule.fail_json() to pass in the message and the result
    if module.params['name'] == 'fail me':
        module.fail_json(msg='You requested this to fail', **result)

    # in the event of a successful module execution, you will want to
    # simple AnsibleModule.exit_json(), passing the key/value results
    module.exit_json(**result)


def main():
    run_module()


if __name__ == '__main__':
    main()
```
Или возьмите данное наполнение из [статьи](https://docs.ansible.com/ansible/latest/dev_guide/developing_modules_general.html#creating-a-module).

3. Заполните файл в соответствии с требованиями ansible так, чтобы он выполнял основную задачу: module должен создавать текстовый файл на удалённом хосте по пути, определённом в параметре `path`, с содержимым, определённым в параметре `content`.

```python
# Проверяем последний символ в стоке path, если нет /, добавляем его
if module.params['path'][:-1] == '/':
  gen_path = module.params['path'] + module.params['name']
else:
  gen_path = module.params['path'] + '/' + module.params['name']

# Проверяем, надо ли создавать или пересоздавать нужный файл
need_create = os.access(gen_path, os.F_OK) and not module.params['force']
if module.check_mode or need_create:
    module.exit_json(**result)

# Создаём путь до файла и если он уже есть, продолжаем без ошибки
os.makedirs(module.params['path'], exist_ok=True)
# Записываем побитово содержимое переменной content в нужный файл
with open(gen_path, 'wb') as newone:
  newone.write(to_bytes(module.params['content']))

# Выводим сообщение и параметры
result['original_message'] = 'Successful created'
result['message'] = 'End'
result['changed'] = True
result['path'] = module.params['path']
result['content'] = module.params['content']
```

4. Проверьте module на исполняемость локально.

Для локального теста создадим файл `ansible/payload.json` с аргументами:

```json
{
  "ANSIBLE_MODULE_ARGS": {
    "name": "Hello.txt",
    "path": "/tmp/my_own_files",
    "content": "Test content",
    "force": false
  }
}
```

```bash
# Запускаем локальный тест
python -m ansible.modules.my_own_module payload.json

{"changed": false, "original_message": "", "message": "", "invocation": 
{"module_args": {"name": "Hello", "path": "/tmp/test_create_file", "content": "content text", "force":false}}}
```

5. Напишите single task playbook и используйте module в нём.

```yml
---
  - name: Run the module my_own_module
    my_own_module:
      name: "{{ p_name }}"
      path: "{{ p_path }}"
      content: "{{ p_content }}"
      force: false
    register: testout
  - name: dump test output
    debug:
      msg: '{{ testout }}'
```

6. Проверьте через playbook на идемпотентность.

```bash
ansible-playbook playbook.yml

PLAY [Test for module my_own_module] *******************************************

TASK [Run the testing module] **************************************************
changed: [localhost]

TASK [dump test output] ********************************************************
ok: [localhost] => {
    "msg": {
        "changed": true,
        "content": "Test content",
        "failed": false,
        "gid": 1000,
        "group": "vagrant",
        "message": "End",
        "mode": "0775",
        "original_message": "Successful created",
        "owner": "vagrant",
        "path": "/tmp/my_own_files/",
        "size": 4096,
        "state": "directory",
        "uid": 1000
    }
}

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0


# Содержимое созданного файла
cat /tmp/my_own_files/Hello.txt
Test content


# Тест на идемпотентность
ansible-playbook playbook.yml

PLAY [Test for module my_own_module] *******************************************

TASK [Run the testing module] **************************************************
ok: [localhost]

TASK [dump test output] ********************************************************
ok: [localhost] => {
    "msg": {
        "changed": false,
        "failed": false,
        "message": "",
        "original_message": ""
    }
}

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

7. Выйдите из виртуального окружения.

```
deactivate
```

8. Инициализируйте новую collection: `ansible-galaxy collection init my_own_namespace.my_own_collection`

```bash
ansible-galaxy collection init my_own_namespace.my_own_collection
- Collection my_own_namespace.my_own_collection was created successfully

tree
.
└── my_own_collection
    ├── docs
    ├── galaxy.yml
    ├── plugins
    │   └── README.md
    ├── README.md
    └── roles

4 directories, 3 files
```

9. В данную collection перенесите свой module в соответствующую директорию.

```bash
 tree
.
└── my_own_collection
    ├── docs
    ├── galaxy.yml
    ├── plugins
    │   ├── modules
    │   │   └── my_own_module.py
    │   └── README.md
    ├── README.md
    └── roles

5 directories, 4 files
```

10. Single task playbook преобразуйте в single task role и перенесите в collection. У role должны быть default всех параметров module

```bash
tree
.
├── docs
├── galaxy.yml
├── plugins
│   ├── modules
│   │   └── my_own_module.py
│   └── README.md
├── README.md
└── roles
    └── my_own_role
        ├── defaults
        │   └── main.yml
        ├── handlers
        │   └── main.yml
        ├── meta
        │   └── main.yml
        ├── tasks
        │   └── main.yml
        └── vars
            └── main.yml

10 directories, 9 files
```

11. Создайте playbook для использования этой role.

```bash
tree
.
├── inventories
│   └── prod
│       └── hosts.yml
├── requirements.yml
└── site.yml
```

12. Заполните всю документацию по collection, выложите в свой репозиторий, поставьте тег `1.0.0` на этот коммит.
13. Создайте .tar.gz этой collection: `ansible-galaxy collection build` в корневой директории collection.

```
ansible-galaxy collection build
Created collection for my_own_namespace.my_own_collection at /home/vagrant/ansible/8.4/my_own_namespace/my_own_collection/my_own_namespace-my_own_collection-1.0.0.tar.gz
```

14. Создайте ещё одну директорию любого наименования, перенесите туда single task playbook и архив c collection.
15. Установите collection из локального архива: `ansible-galaxy collection install <archivename>.tar.gz`

```bash
ansible-galaxy collection install -p collections my_own_namespace-my_own_collection-1.0.0.tar.gz
Starting galaxy collection install process
Process install dependency map
Starting collection install process
Installing 'my_own_namespace.my_own_collection:1.0.0' to '/home/vagrant/ansible/8.4/my_own_namespace/playbook/collections/ansible_collections/my_own_namespace/my_own_collection'
my_own_namespace.my_own_collection:1.0.0 was installed successfully
```

16. Запустите playbook, убедитесь, что он работает.

```bash
sudo ansible-playbook site.yml -i inventories/prod/hosts.yml

PLAY [Playbook for my own collection] ******************************************

TASK [Gathering Facts] *********************************************************
ok: [localhost]

TASK [my_own_namespace.my_own_collection.my_own_role : Run the module my_own_module] ***
changed: [localhost]

TASK [my_own_namespace.my_own_collection.my_own_role : dump test output] *******
ok: [localhost] => {
    "msg": {
        "changed": true,
        "content": "Content for my module",
        "failed": false,
        "gid": 0,
        "group": "root",
        "message": "End",
        "mode": "0755",
        "original_message": "Successful created",
        "owner": "root",
        "path": "/tmp/my_own_files/",
        "size": 4096,
        "state": "directory",
        "uid": 0
    }
}

PLAY RECAP *********************************************************************
localhost                  : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0


# Проверяем идемпотентность:
sudo ansible-playbook site.yml -i inventories/prod/hosts.yml

PLAY [Playbook for my own collection] ******************************************

TASK [Gathering Facts] *********************************************************
ok: [localhost]

TASK [my_own_namespace.my_own_collection.my_own_role : Run the module my_own_module] ***
ok: [localhost]

TASK [my_own_namespace.my_own_collection.my_own_role : dump test output] *******
ok: [localhost] => {
    "msg": {
        "changed": false,
        "failed": false,
        "message": "",
        "original_message": ""
    }
}

PLAY RECAP *********************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

17. В ответ необходимо прислать ссылку на репозиторий с collection

### Репозиторий с коллекцией my_own_collection:

https://github.com/VitalyMozhaev/my_own_collection

### Тест коллекции my_own_collection с модулем my_own_module.py:

https://github.com/VitalyMozhaev/mnt-homeworks/tree/main/08-ansible-04-module/test

## Необязательная часть

1. Используйте свой полёт фантазии: Создайте свой собственный module для тех roles, что мы делали в рамках предыдущих лекций.
2. Соберите из roles и module отдельную collection.
3. Создайте новый репозиторий и выложите новую collection туда.

Если идей нет, но очень хочется попробовать что-то реализовать: реализовать module восстановления из backup elasticsearch.
