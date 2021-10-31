# Ответы на домашнее задание к занятию "09.03 Jenkins"

## Подготовка к выполнению

1. Установить jenkins по любой из [инструкций](https://www.jenkins.io/download/)

```bash
# Для установки jenkins на localhost нужна java
sudo apt-get install default-jdk
java -version

# Установка Jenkins
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt-get update
sudo apt-get install jenkins
sudo systemctl enable jenkins
sudo systemctl status jenkins

# Пароль для первого входа
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

2. Запустить и проверить работоспособность
3. Сделать первоначальную настройку
4. Настроить под свои нужды
5. Поднять отдельный cloud

```bash
# Для правильного подключения к docker из jenkins, нужно предоставить доступ к docker.sock
sudo chmod 777 /var/run/docker.sock
```

6. Для динамических агентов можно использовать [образ](https://hub.docker.com/repository/docker/aragast/agent)
7. Обязательный параметр: поставить label для динамических агентов: `ansible_docker`
8.  Сделать форк репозитория с [playbook](https://github.com/aragastmatb/example-playbook)

## Основная часть

1. Сделать Freestyle Job, который будет запускать `ansible-playbook` из форка репозитория

```bash
# Сначала пробовал запускать на динамическом слейве в таком варианте:
ansible-vault decrypt secret --vault-password-file vault_pass
cp ./secret ~/.ssh/id_rsa
chmod 400 ~/.ssh/id_rsa
ansible-galaxy install -r requirements.yml -p roles
ansible-playbook site.yml -i inventory/prod.yml
ls -lah roles

# Но не получается - падает на этапе requirements.yml с ошибкой "Host key verification failed":
...
+ ansible-galaxy install -r requirements.yml -p roles
Starting galaxy role install process
[WARNING]: - java was NOT installed successfully: - command /usr/bin/git clone
git@github.com:netology-code/mnt-homeworks-ansible.git java failed in directory
/root/.ansible/tmp/ansible-local-135dlbvxhak/tmptxx7lpqq (rc=128) - Host key
verification failed.  fatal: Could not read from remote repository.  Please
make sure you have the correct access rights and the repository exists.
ERROR! - you can use --ignore-errors to skip failed roles and finish processing the list.
Build step 'Execute shell' marked build as failure
Finished: FAILURE

# Заменил на другой вариант и заработало:
git clone https://github.com/netology-code/mnt-homeworks-ansible.git
mkdir -p ./roles/java
mv mnt-homeworks-ansible ./roles/java
ansible-playbook site.yml -i inventory/prod.yml
ls -lah roles
```

2. Сделать Declarative Pipeline, который будет выкачивать репозиторий с плейбукой и запускать её

```text
# Не удалось быстро настроить работу pipeline для репозитория https://github.com/netology-code/mnt-homeworks-ansible.git,
# поэтому переделал на простенький playbook из предыдущей домашки https://github.com/VitalyMozhaev/mnt-homeworks-ansible-1
pipeline {
    agent {
        node {
            label 'ansible_docker'
        }
    }

    stages {
        stage('Prepare') {
            steps {
                git 'https://github.com/VitalyMozhaev/mnt-homeworks-ansible-1'
            }
        }
        stage('Check') {
            steps {
                sh 'ansible --version'
            }
        }
        stage('Playbook') {
            steps {
                sh 'ansible-playbook site.yml -i inventory/test.yml'
            }
        }
    }
}
```

3. Перенести Declarative Pipeline в репозиторий в файл `Jenkinsfile`

screen + screen

4. Перенастроить Job на использование `Jenkinsfile` из репозитория

https://github.com/VitalyMozhaev/pipeline_declarative

5. Создать Scripted Pipeline, наполнить его скриптом из [pipeline](./pipeline)
6. Заменить credentialsId на свой собственный

```
# В скрипте не хватает загрузки зависимостей и предварительной расшифровки ключа
node("ansible_docker"){
    stage("Git checkout"){
        git branch: 'main', credentialsId: 'cdb63529-4db3-439a-85d0-8ceca32c3057', url: 'https://github.com/VitalyMozhaev/example-playbook'
    }
    stage("Check ssh key"){
        secret_check=true
    }
    stage("Run playbook"){
        if (secret_check){
            sh 'ansible-vault decrypt secret --vault-password-file vault_pass'
            sh 'cp ./secret ~/.ssh/id_rsa && chmod 400 ~/.ssh/id_rsa'
            sh 'ansible-galaxy install -r requirements.yml -p roles'
            sh 'ansible-playbook site.yml -i inventory/test.yml'
        }
        else{
            echo 'no more keys'
        }
    }
}

# Но в таком варианте у меня опять же не заработал, надо поковыряться.
# Скорее всего где-то неправильно прописаны ключи
```

```text
# Для теста сделал простой playbook
node("ansible_docker"){
    stage("Git checkout"){
        git 'https://github.com/VitalyMozhaev/mnt-homeworks-ansible-1'
    }
    stage("Check ssh key"){
        secret_check=true
    }
    stage("Run playbook"){
        if (secret_check){
            sh 'ansible-playbook site.yml -i inventory/test.yml'
        }
        else{
            echo 'no more keys'
        }
    }
}
```

7. Проверить работоспособность, исправить ошибки, исправленный Pipeline вложить в репозитрий в файл `ScriptedJenkinsfile`

screen

8. Отправить ссылку на репозиторий в ответе


## Необязательная часть

1. Создать скрипт на groovy, который будет собирать все Job, которые завершились хотя бы раз неуспешно. Добавить скрипт в репозиторий с решеним с названием `AllJobFailure.groovy`
2. Установить customtools plugin
3. Поднять инстанс с локальным nexus, выложить туда в анонимный доступ  .tar.gz с `ansible`  версии 2.9.x
4. Создать джобу, которая будет использовать `ansible` из `customtool`
5. Джоба должна просто исполнять команду `ansible --version`, в ответ прислать лог исполнения джобы 
