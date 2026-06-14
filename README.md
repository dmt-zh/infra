## Ansible Ad-Hoc

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


## Ansible Playbook

**Запуск сценария управления пользователями**
```sh
ansible-playbook -i hosts.ini user.yml
```
