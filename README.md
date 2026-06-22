# DevOps Infrastructure & Automation Sandbox

Репозиторий представляет собой интерактивную практическую лабораторию по изучению и внедрению подходов **Infrastructure as Code (IaC)**, автоматизации конфигураций, оркестрации контейнеров и развертыванию инфраструктуры разработки.

## 🛠 Стек технологий

* **Конфигурация и автоматизация:** Ansible (Playbooks, Advanced Roles, Jinja2 Templates, Ansible Vault)
* **Виртуализация и окружение:** Vagrant, VirtualBox
* **Оркестрация и контейнеризация:** Docker, Docker Swarm
* **Веб-серверы и проксирование:** Nginx

<hr>

## 📋 Требования

* **Vagrant** (установлен в Windows, запускается через bash-обертку в WSL2)
* **VirtualBox** (установлен на хост-системе Windows)
* **Ansible** (установлен внутри дистрибутива WSL2)
* **SSH ключ** (`id_ed25519_virtualbox`) — сгенерированный внутри WSL2

<hr>

## 🔧 Настройка окружения WSL2 & Vagrant Bridge

### 1. Глобальная конфигурация сети (`.wslconfig`)

Создайте или отредактируйте файл `.wslconfig` в корне домашнего каталога вашего пользователя **в Windows** (Путь: `C:\Users\Пользователь\.wslconfig`): 
```ini [wsl2]
networkingMode=mirrored
localhostForwarding=true
```
**Важно:** После сохранения файла обязательно перезапустите подсистему WSL. Для этого откройте PowerShell в Windows и выполните: `wsl --shutdown`.

### 2. Интеграция CLI Vagrant (`~/.bashrc`)

Чтобы вызывать Windows-версию Vagrant напрямую из терминала Linux без прописывания `.exe`,  добавьте в самый конец файла `~/.bashrc` внутри WSL2 следующую функцию-обертку:
```sh
vagrant() {
    export VAGRANT_WSL_ENABLE_WINDOWS_ACCESS="1"
    "/mnt/c/Program Files/Vagrant/bin/vagrant.exe" "$@"
}
```
После сохранения файла примените настройки в терминале: `source ~/.bashrc`.

 <hr>

## 🔑 SSH ключ

Для беспарольной автоматизации и доступа Ansible к нодам используется современный алгоритм **Ed25519**. Он быстрее, безопаснее старого RSA, а ключи устойчивы к атакам по сторонним каналам.

**Генерация ключа в WSL2**
```sh
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_virtualbox -C "wsl-to-virtualbox"
```

**Автоматическая инжекция ключа**
Скрипт в `Vagrantfile` при сборке машин (`vagrant up`) самостоятельно заберет публичный ключ из WSL по сетевому UNC-пути и пропишет его в `authorized_keys` пользователя `vagrant` на каждом сервере.

 <hr>

## 🏗 Архитектура кластера Docker Swarm

Кластер состоит из 5 виртуальных машин на базе **Ubuntu 22.04** (bento/ubuntu-22.04). Кластер разворачивается в изолированной внутренней сети VirtualBox, что изолирует трафик от хост-машины и предотвращает конфликты сетевых режимов WSL2.
  
### Спецификация нод (на одну ВМ)

- **ОС:** Ubuntu 22.04 LTS (Bento оптимизированный)
- **ОЗУ:** 3072 MB
- **CPU:** 1 vCPU (оптимально для стабильности хоста при оверкоммите ресурсов)

### Карта сетевой маршрутизации кластера

| **Имя ноды** | **Роль в Swarm**    | **Внутренний IP (Cluster)** | **Внешний Порт (Host Forward)** |
| ------------ | ------------------- | --------------------------- | ------------------------------- |
| **server-1** | Manager (Leader)    | `10.11.10.11`               | `2201`                          |
| **server-2** | Manager (Reachable) | `10.11.10.12`               | `2202`                          |
| **server-3** | Manager (Reachable) | `10.11.10.13`               | `2203`                          |
| **server-4** | Worker              | `10.11.10.14`               | `2204`                          |
| **server-5** | Worker              | `10.11.10.15`               | `2205`                          |
<hr>

## 🚀 Управление и развертывание

**Поднятие «голой» инфраструктуры**

Развертывание пяти чистых операционных систем в VirtualBox:
```sh
vagrant up
```

**Оркестрация и деплой кластера через Ansible**

Запуск комплексного плейбука. `Ansible` автоматически установит` Docker CE` на все ноды, добавит пользователя в группы, перезагрузит серверы, инициализирует Swarm-лидера, безопасно извлечет токены через контекст `hostvars` и подключит оставшиеся менеджеры и воркеры в единую сеть:
```sh
ansible-playbook -i cluster/inventory cluster/config.yml
```

**Верификация кластера**

Чтобы убедиться, что кластер собран правильно, подключитесь к первому менеджеру по SSH:
```sh
ssh -i ~/.ssh/id_ed25519_virtualbox -p 2201 vagrant@127.0.0.1
```

И выполните команду проверки нод:
```sh
docker node ls
```

<hr>

## 🧹 Утилизация ресурсов (Гигиена стенда)

Когда работа над плейбуками завершена, не оставляйте машины запущенными или замороженными, чтобы не забивать RAM и дисковое пространство хоста.
- Временная остановка (выключение): `vagrant halt`
- Полное уничтожение стенда и очистка диска SSD:
```sh
vagrant destroy -f
rm -rf .vagrant
```

<hr>

# Демо-сервер

Инвентарь `demo-server/demo` содержит одну удалённую ноду `server-local` для отработки базовых команд Ansible.

Разрешение sudo без пароля (одноразово):
```sh
ssh -t server-local "echo 'ubuntu ALL=(ALL) NOPASSWD:ALL' | sudo tee /etc/sudoers.d/ubuntu && sudo chmod 440 /etc/sudoers.d/ubuntu"
```

## Ansible Ad-Hoc команды

Создание пользователя:
```sh
ansible -i demo-server -m user -a "name=ubuntu_2 state=present" -b demo
```

Удаление пользователя с домашней директорией:
```sh
ansible -i demo-server -m user -a "name=ubuntu_2 state=absent remove=yes" -b demo
```

## Playbooks

| Файл | Описание |
|------|----------|
| `examples/user_base.yml` | Минимальный пример: создание пользователя |
| `examples/user_blocks.yml` | Block/rescue, условное выполнение (только Ubuntu), apt |
| `examples/tasks_async.yml` | Асинхронные задачи с async_status и polling |
| `examples/server_config.yml` | Установка Docker CE, Docker Compose, добавление пользователя в группу docker, ребут |

Запуск playbook для создания пользователя:
```sh
ansible-playbook -i demo-server examples/user_base.yml
```

Запуск playbook для конфигурации сервера:
```sh
ansible-playbook -i demo-server examples/server_config.yml
```

<hr>


## ⚙️ Системная конфигурация Ansible

Файл `ansible.cfg` оптимизирован под высокую скорость работы с большим количеством нод:
* `interpreter_python = auto_silent` — автоматическое определение Python без вывода предупреждений;
* `pipelining = True` — выполняет цепочки задач в рамках одной SSH-сессии без повторных подключений, ускоряя деплой Swarm в 2–3 раза.