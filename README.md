# DevOps Infrastructure & Automation Sandbox

Репозиторий представляет собой интерактивную практическую лабораторию по изучению и внедрению подходов **Infrastructure as Code (IaC)**, автоматизации конфигураций, оркестрации контейнеров и развертыванию инфраструктуры разработки.
## 🛠 Стек технологий
* **Конфигурация и автоматизация:** Ansible (Playbooks, Advanced Roles, Jinja2 Templates, Ansible Vault)
* **Виртуализация и окружение:** Vagrant, VirtualBox
* **Оркестрация и контейнеризация:** Docker, Docker Swarm
* **Веб-серверы и проксирование:** Nginx

<hr>

## Настройка

### Разрешение sudo без пароля (одноразово)

```sh
ssh -t server-local "echo 'ubuntu ALL=(ALL) NOPASSWD:ALL' | sudo tee /etc/sudoers.d/ubuntu && sudo chmod 440 /etc/sudoers.d/ubuntu"
```

## Ansible Ad-Hoc

**Создание пользователя**
```sh
ansible -i hosts.ini -m user -a "name=ubuntu_2 state=present" -b demo
```

**Удаление пользователя с домашней директорией**
```sh
ansible -i hosts.ini -m user -a "name=ubuntu_2 state=absent remove=yes" -b demo
```

## Playbooks

| Файл | Описание |
|------|----------|
| `examples/user_base.yml` | Минимальный пример: создание пользователя |
| `examples/user_blocks.yml` | Block/rescue, условное выполнение (только Ubuntu), apt |
| `examples/tasks_async.yml` | Асинхронные задачи с async_status и polling |
| `examples/server_config.yml` | Установка Docker CE, Docker Compose, добавление пользователя в группу docker, ребут |

**Запуск playbook**
```sh
ansible-playbook -i hosts.ini examples/user_base.yml
```

**Запуск с альтернативным инвентарём**
```sh
ansible-playbook -i demo-server/demo examples/server_config.yml
