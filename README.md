# DevOps Infrastructure & Automation Sandbox

Репозиторий представляет собой интерактивную практическую лабораторию по изучению и внедрению подходов **Infrastructure as Code (IaC)**, автоматизации конфигураций, оркестрации контейнеров и развертыванию инфраструктуры разработки.
## 🛠 Стек технологий
* **Конфигурация и автоматизация:** Ansible (Playbooks, Advanced Roles, Jinja2 Templates, Ansible Vault)
* **Виртуализация и окружение:** Vagrant, VirtualBox
* **Оркестрация и контейнеризация:** Docker, Docker Swarm
* **Веб-серверы и проксирование:** Nginx

<hr>

### Ansible Ad-Hoc

**Настройка внутри виртуалки**

Выполняем в WSL эту команду, чтобы один раз разрешить пользователю `ubuntu` на виртуалке работать без пароля (система попросит пароль в последний раз):
```sh
ssh -t server-local "echo 'ubuntu ALL=(ALL) NOPASSWD:ALL' | sudo tee /etc/sudoers.d/ubuntu && sudo chmod 440 /etc/sudoers.d/ubuntu"
```

**Добавление нового пользователя на сервер**
```sh
ansible -i hosts.ini -m user -a "name=ubuntu_2 state=present" -b demo
```

**Полное удаление пользователя вместе со всеми его личными файлами и домашней директорией**
```sh
ansible -i hosts.ini -m user -a "name=ubuntu_2 state=absent remove=yes" -b demo
```

<hr>

### Ansible Playbook

**Запуск сценария управления пользователями**
```sh
ansible-playbook -i hosts.ini user.yml
```
