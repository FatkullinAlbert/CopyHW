# CopyHW
Домашнее задание на тему "Резервное копирование"
Задание:
Настроить стенд Vagrant с двумя виртуальными машинами: backup_server и client.
Настроить удаленный бекап каталога /etc c сервера client при помощи borgbackup. Резервные копии должны соответствовать следующим критериям:
директория для резервных копий /var/backup. Это должна быть отдельная точка монтирования. В данном случае для демонстрации размер не принципиален, достаточно будет и 2GB;
репозиторий дле резервных копий должен быть зашифрован ключом или паролем - на ваше усмотрение;
имя бекапа должно содержать информацию о времени снятия бекапа;
глубина бекапа должна быть год, хранить можно по последней копии на конец месяца, кроме последних трех.
Последние три месяца должны содержать копии на каждый день. Т.е. должна быть правильно настроена политика удаления старых бэкапов;
резервная копия снимается каждые 5 минут. Такой частый запуск в целях демонстрации;
написан скрипт для снятия резервных копий. Скрипт запускается из соответствующей Cron джобы, либо systemd timer-а - на ваше усмотрение;
настроено логирование процесса бекапа. Для упрощения можно весь вывод перенаправлять в logger с соответствующим тегом. Если настроите не в syslog, то обязательна ротация логов.
Для начала устанавливаем Vagrant, Ansible и VirtualBox
Установка Vagrant:
wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
apt update && sudo apt install vagrant
Установка Ansible:
apt install software-properties-common
add-apt-repository --yes --update ppa:ansible/ansible
apt install ansible
Установка VirtualBox:
apt install virtualbox
После устанвоки создаём директорию для наших ВМ и файлы для Ansible:
После создания запускаем наши ОС командой:
vagrant up
После поднятия ОС и исполнения playbook мы можем проверить работу.
Проверка сетевой связности:
ping 192.168.56.150
ping 192.168.56.160
Проверка таймера на клиенте:
vagrant ssh client
systemctl status borg-backup.timer
systemctl list-timers --all | grep borg
Проверка создания бэкапов:
sudo -i
borg list borg@192.168.56.160:/var/backup
# ввести парольную фразу
Проверка логов:
journalctl -u borg-backup.service -f
# или
tail -f /var/log/syslog | grep borg-backup
Проверка политики удаления:
borg prune --list --dry-run --keep-within 3m --keep-monthly 12 --keep-yearly 1 borg@192.168.56.160:/var/backup
Если все команды заработали, то домашнее задание выполнено!
