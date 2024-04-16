# Install and config IPsec-VPN-SERVER on some VPS

Данный playbook предназначен для автоматизированной установки и конфигурирования VPN сервера, так-же предусмотренна базовая настройка безопасности для конфигурируемого VPS.

Playbook состоит из последовательно выполняемых ролей, некоторые роли не обязательны к исполнению.

Описание необходимых действий для успешной настройки VPN-сервера

### 00. Задание переменных
#### Все ключевые переменные можно определить в файле [group_vars/all.yml](./group_vars/all.yml)
- При необходимости, переменные можно переопределить в playbook [main.yml](./main.yml)
```yml
  vars:
    vpn_user: some_user # Пользователь с правами root для создаваемого VPN-сервера
    users_users:
      - user: '{{ vpn_user }}'
        name: "VPN admin"
        key: id_rsa.pub # Публичный ssh-ключ, который будет передан на сервер для подключения к нему, файл должен лежать в ./roles/add-users/files
    users_global_admin_users:
      - '{{ vpn_user }}'
    security_ssh_port: 222 # Номер измененного порта для подключения по ssh (меняем для безопасности)
    security_ssh_allowed_users: ['{{ vpn_user }}']
    security_sudoers_passwordless: ['{{ vpn_user }}']
```

- При необходимости, параметры для подключения к вашему VPS-серверу можно задать в файле [inventory/hosts.yml](./inventory/hosts.yml)
```yml
      ansible_host: 0.0.0.0 # ip-адрес вашего сервера
      ansible_port: 22 # порт для подключения по ssh
      ansible_user: user_name # имя пользователя с root правами на вашем VPS-сервере
      ansible_ssh_private_key_file: "~/.ssh/id_rsa" # путь к вашему ssh-ключу
```

- Запуск плейбука на новом сервере в котором еще нет ни пользователей, ни ключей
```bash
ansible-playbook -k -u root main.yml -i inventory/hosts.yml
# Вводим пароль от root, ваданный VPS-провайдером
```


### 01. Добавление пользователей для работы на VPS-сервере
Функцию добавления пользователей выполняет роль [add-users](./roles/add-users/)

Была использована данная роль: [Ansible role to create user accounts](https://github.com/cogini/ansible-role-users)


### 02. Установка docker и docker-compose
Функцию установки docker выполняет роль [docker-install](./roles/docker-install/). 
Так-же данная роль задает пользователя от которого будет запускаться docker, 
пользователь берется из переменной **vpn_user**

Была использована данная роль: [Ansible Role: Docker](https://github.com/geerlingguy/ansible-role-docker)


### 03. Установка IPsec-VPN сервера
Функцию установки VPN-сервера выполняет роль [ipsec-vpn-server-install](./roles/ipsec-vpn-server-install/).

Данная роль копирует на VPS-сервер [docker-compose файл](./roles/ipsec-vpn-server-install/files/docker-compose.yml),
и с его помощью запускает docker-контейнер с работающим ipsec-vpn-сервером.

Запуск VPN сервера производится с помощью контейнера [Docker image to run an IPsec VPN server, with IPsec/L2TP, Cisco IPsec and IKEv2](https://github.com/hwdsl2/docker-ipsec-vpn-server/tree/master)

Учетные данные для подключения к vpn-серверу генерируются автоматически и выводятся в консоль.

Так-же данные для аунтификацию могут быть введены вручную, для этого необходимо раскоментировать соответствующие строки 
в файле [vpn.env](./roles/ipsec-vpn-server-install/files/vpn.env)
```ini
VPN_IPSEC_PSK=your_ipsec_pre_shared_key # ключ должен быть определенной длины (не менее 20 символов)
VPN_USER=your_vpn_username
VPN_PASSWORD=your_vpn_password
```
Можно добавить дополнительных пользователей:
```ini
VPN_ADDL_USERS=additional_username_1 additional_username_2
VPN_ADDL_PASSWORDS=additional_password_1 additional_password_2
```

[Инструкции по подключению клиентов](https://github.com/hwdsl2/setup-ipsec-vpn/blob/master/docs/clients.md#configure-ipsecl2tp-vpn-clients)


### 04. Настройка базовой безопасности linux
Функцию настройки базовой безопасности выполняет роль [basics-security](./roles/basics-security/)

По умолчанию роль закомичена и выполняться не будет.
Данная роль настраивает базовые опции безопасности linux-сервера в интернете:
- Переопределяет ssh-порт для подключения к серверу
- Запрещяет root-доступ к серверу. Внимание!!! После применения данной роли к серверу нельзя будет подключиться использую стадартный root-логин, выданный VPS-провайдером
- Отключает логин/пароль аунтификацию. Можно подключиться только использую ssh-ключ

Была использована данная роль: [Ansible Role: Security (Basics)](https://github.com/geerlingguy/ansible-role-security)
