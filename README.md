# DevOps Infrastructure & Automation Sandbox

Репозиторий представляет собой интерактивную практическую лабораторию по изучению и внедрению подходов **Infrastructure as Code (IaC)**, автоматизации конфигураций, оркестрации контейнеров и развертыванию инфраструктуры разработки.

## 🛠 Стек технологий

* **Конфигурация и автоматизация:** Ansible (Playbooks, Advanced Roles, Jinja2 Templates, Ansible Vault)
* **Виртуализация и окружение:** Vagrant, VirtualBox
* **Оркестрация и контейнеризация:** Docker, Docker Swarm
* **Веб-серверы и проксирование:** Nginx

<hr>

## 📋 Требования

* **Vagrant** — управление виртуальными окружениями
* **VirtualBox** — гипервизор для виртуальных машин
* **Ansible** — автоматизация конфигураций
* **SSH ключ** (`id_ed25519_virtualbox`) — аутентификация на нодах кластера

 <hr>

## 🔑 SSH ключ

Треббуется для аутентификации на виртуальных машинах кластера.

### Генерация ключа

```sh
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_virtualbox -C "wsl-to-virtualbox"
```

Используется алгоритм **Ed25519** — современный, быстрый и более безопасный по сравнению с RSA. Ключи Ed25519 короче, генерируются мгновенно и устойчивы к атакам по сторонним каналам.

### Копирование ключа на виртуальную машину

После первого запуска `vagrant up` скопируйте публичный ключ на хост-машину:

```sh
ssh-copy-id -p 2222 -i ~/.ssh/id_ed25519_virtualbox.pub ubuntu@localhost
```

 <hr>

## Настройка

### Демо-сервер

Инвентарь `demo-server/demo` содержит одну удалённую ноду `server-local` для отработки базовых команд Ansible.

### Разрешение sudo без пароля (одноразово)

```sh
ssh -t server-local "echo 'ubuntu ALL=(ALL) NOPASSWD:ALL' | sudo tee /etc/sudoers.d/ubuntu && sudo chmod 440 /etc/sudoers.d/ubuntu"
```

## Ansible Ad-Hoc

**Создание пользователя**

```sh
ansible -i demo-server -m user -a "name=ubuntu_2 state=present" -b demo
```

**Удаление пользователя с домашней директорией**

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

**Запуск playbook для создания пользователя**

```sh
ansible-playbook -i demo-server examples/user_base.yml
```

**Запуск playbook для конфигурации сервера**

```sh
ansible-playbook -i demo-server examples/server_config.yml
```

<hr>

## Архитектура кластера

Кластер состоит из 5 виртуальных машин на базе **Ubuntu 22.04** (bento/ubuntu-22.04).

| Параметр | Значение |
|----------|----------|
| ОЗУ | 3072 MB |
| CPU | 1 ядро |
| Сеть (private) | `10.11.10.11` — `10.11.10.15` |
| Сетевой интерфейс | VirtualBox internal network |

### Forwarded ports

| Нода | Host Port | Guest Port |
|------|-----------|------------|
| server-1 | 2201 | 22 |
| server-2 | 2202 | 22 |
| server-3 | 2203 | 22 |
| server-4 | 2204 | 22 |
| server-5 | 2205 | 22 |

<hr>

## Управление кластером

### Создание кластера

```sh
vagrant up
```

### SSH-доступ к ноде

```sh
ssh -i ~/.ssh/id_ed25519_virtualbox -p 2201 vagrant@127.0.0.1
```

### Конфигурация серверов

Установка Docker на все ноды кластера через Ansible:

```sh
ansible-playbook -i cluster/inventory cluster/config.yml
```

### Удаление кластера

```sh
vagrant destroy -f
rm -rf .vagrant
```

<hr>

## Ansible Configuration

Файл `ansible.cfg` содержит глобальные настройки:

* `interpreter_python = auto_silent` — автоматическое определение Python без вывода предупреждений
* `pipelining = True` — ускорение выполнения задач за счёт снижения количества SSH-сессий
