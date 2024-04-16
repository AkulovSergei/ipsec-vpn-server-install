# Установка IPSEC-VPN сервера


Очень примитивная роль, которая копирует на VPS-сервер [docker-compose файл](./files/docker-compose.yml),
и с его помощью запускает docker-контейнер с работающим ipsec-vpn-сервером.

Запуск VPN сервера производится с помощью контейнера [Docker image to run an IPsec VPN server, with IPsec/L2TP, Cisco IPsec and IKEv2](https://github.com/hwdsl2/docker-ipsec-vpn-server/tree/master)

Учетные данные для подключения к vpn-серверу генерируются автоматически и выводятся в консоль.

Так-же данные для аунтификацию могут быть введены вручную, для этого необходимо раскоментировать соответствующие строки 
в файле [vpn.env](./files/vpn.env)
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
