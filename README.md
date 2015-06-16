# Автоматическая установка ОС ОС «Astra Linux Special Edition»

Этот репозиторий содержит Ansible-раскатку для сервера автоматической установки
по сети ОС Astra Linux Special Edition на целевые машины.

Авторы: [Лаборатория 50](http://лаборатория50.рф) team@lab50.net.

## Лицензия

Все материалы распространяются на условиях
стандартной общественной лицензии [GNU (GPL)](http://www.gnu.org/copyleft/gpl.html) версии 3.

Полный текст лицензии находится в файле LICENSE.

Используются следующие пакеты:

 - Dnsmasq
 - Apache

## Подготовка

Ставим пакет ansible -> `aptitude install ansible`

## Установка

Затем:

 0. Запуск `ansible-playbook -i <файл с описание хостов> <yml-файл c playbook>`
 1. Для запуска playbook на локальном хосте добавляем строчку `connection: local` после:

        - hosts: компьютер1
          user: root
          <cюда>

 2. Для использования sudo добавляем по строку `sudo: True`
    по аналогии с п.1, не забываем при запуске (см. п.0) добавлять --ask-sudo-pass,
    при этом строчку user:... комментируем.

### Про интрефейс для служб (dnsmasq и.т.д.)

Сетевые настройки для сервера установки задаются в секции network файла `groups_vars/all`
Стандартные настройки в шаблонах где нужен ip-адрес, маска, имя-адаптера (т.е. ethN)
подставляется `hostvars[inventory_hostname].ansible_eth0.ipv4.<address/netmask/device>`,
если вам нужен другой адаптер меняйте строку в секции network на
`hostvars[inventory_hostname].ansible_<ИМЯ АДАПТЕРА>.ipv4.<address/netmask/device>`
или пропишите явно (адрес, маску и имя).

## Замечания для версии Астры 1.3

Данная конфигурация настроена на установку по сети ОС Astra Linux Special Edition.
Для этого доработан образ initrd.gz инсталлятора (по отношению к netinst
c оригинального диска с ОС Astra Linux 1.3):

 - изменен usr/share/localechooser/languagelist поддержка русскому языку изменена с 2 на 1.
   было ru;2;RU;ru_RU.UTF-8;;console-setup стало ru;1;RU;ru_RU.UTF-8;;console-setup
 - добавлен модуль dca.ko в lib/modules/3.2.0-27-generic/kernel/drivers/dca/
 - добавлены модули dm-log.ko dm-mirror.ko dm-mod.ko dm-region-hash.ko
   в lib/modules/3.2.0-27-generic/kernel/drivers/md/
 - добавлены ключи lab50-archive-keyring.gpg от собственного репа
   (он служит зеркалом при установке) в usr/share/keyrings

Как это все сделать самостоятельно.

 1. распаковка и упаковка initrd.gz:

        #!/bin/sh
        if [ -d $1 ]; then
            rm -rf $1
        fi
        if [ -f initrd ]; then
            rm initrd
        fi
        if ! [ -f initrd.gz ]; then
            echo "initrd.gz not found"
            exit 1
        fi
        gunzip initrd.gz
        mkdir $1
        cd $1
        cpio -id < ../initrd

 2. сборка initrd.gz:

        #!/bin/sh
        if ! [ -d $1 ]; then
            echo "folder not found"
            exit 1
        fi
        if [ -f initrd ]; then
            rm initrd
        fi
        if [ -f initrd.gz ]; then
            rm initrd.gz
        fi
        cd $1
        find . | cpio --create --format='newc' > ../initrd
        cd ..
        gzip initrd

В качестве зеркала используется собственный репозиторий пакетов, в котором добавлен пакет
grub-installer_1.78ubuntu8_amd64.udeb, поскольку grub-installer_1.70astra.se4_amd64.udeb содержит
ошибки при установке grub на soft&fake raid.

