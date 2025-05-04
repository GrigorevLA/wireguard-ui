# wireguard-ui

проект Wireguard UI
https://github.com/ngoduykhanh/wireguard-ui/

Установка
Покупаем vps, vds сервер на Debian, Ubuntu

Заходим на сервер

ssh root@<ip адрес сервера>
Вводим пароль

Обновляем по  

apt update && apt upgrade -y
Устанавливаем wireguard

apt install wireguard -y
Включаем ip forwarding на сервере

echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
sysctl -p
устанавливаем iptables:

apt install iptables -y
Устанавливаем wireguard-ui

создаем каталог wireguard-ui и преходим в него

mkdir wireguard-ui
cd wireguard-ui
Качаем tar архив из релизов https://github.com/ngoduykhanh/wireguard-ui/releases

wget <ссылка на tar.gz файл>
Смотрим имя загруженного файла:

ls -la
Распаковываем скачанный файл:

tar -xvf <скачанный архив>
Запускаем wireguard-ui для создания конфигурации

./wireguard-ui
Останавливаем wireguard-ui Нажимаем в консоли Ctrl+C

Запускаем службу для wireguard: После запуска web интерфейса у нас создастся файл конфигурации для wireguard по пути /wtc/wireguard/wg0.conf. Теперь можно включить и запускать wireguard как сервис, чтобы при перезапуске компьютера работал wireguard.

systemctl enable wg-quick@wg0.service
Создаем службы для перезапуска wireguard в случае изменения конфигурации. Выполняем в терминале следующие команды:

cd /etc/systemd/system/
cat << EOF > wgui.service
[Unit]
Description=Restart WireGuard
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/bin/systemctl restart wg-quick@wg0.service

[Install]
RequiredBy=wgui.path
EOF
cd /etc/systemd/system/
cat << EOF > wgui.path
[Unit]
Description=Watch /etc/wireguard/wg0.conf for changes

[Path]
PathModified=/etc/wireguard/wg0.conf

[Install]
WantedBy=multi-user.target
EOF
systemctl enable wgui.{path,service}
systemctl start wgui.{path,service}
Создаём файл для автоматического запуска веб-интерфейса в случае перезапуска компьютера:

cd /etc/systemd/system/
cat << EOF > wireguard-ui.service
[Unit]
Description=WireGuard UI Service
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/root/wireguard-ui
ExecStart=/root/wireguard-ui/wireguard-ui
Restart=on-failure

[Install]
WantedBy=multi-user.target

EOF
systemctl enable wireguard-ui.service
systemctl start wireguard-ui.service
Заходим в веб интерфейс wireguard-ui

в браузере вводим http://<адрес сервера>:5000
вводим логин и пароль admin admin
в настройках Wiregard Server:
в поле Post Up Script вводим:
iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
в поле Post Down Script вводим:
iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
, где eth0 - имя сетевого интерфейса, через который будут подключаться клиенты. Скорее всего это eth0, но убедиться можно выполнив команду ip a на сервере и найдя имя интерфейса с внешним ip адресом по которому вы подключаетесь к серверу.

Сохраняем Save и применяем конфигурацию Apply Config.
Добавляем пользователей

Добавляете пользователей
После добавления пользователей надо перезагрузить wireguard применив конфигурацию - нажав кнопку Apply Config в веб интерфейсе.
делимся конфигурационным файлом или qr-кодом с пользователем.
