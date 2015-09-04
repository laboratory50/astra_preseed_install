# Автоматическая установка ОС «Astra Linux Special Edition»

Этот репозиторий содержит сценарии Ansible для создания сервера
автоматической установки по сети ОС Astra Linux Special Edition
на целевые машины.

Авторы: [Лаборатория 50](http://лаборатория50.рф) team@lab50.net.

## Лицензия

Все материалы распространяются на условиях
стандартной общественной лицензии [GNU (GPL)](http://www.gnu.org/copyleft/gpl.html) версии 3.

Полный текст лицензии находится в файле LICENSE.

Используются следующие пакеты:

 - Ansible версии 1.7+;
 - Dnsmasq — DHCP и TFTP сервер;
 - LigHTTPd/Apache 2/vsftpd — сервер раздачи репозитория/preseed файлов.

## Возможности

Проект в первую очередь предназначен для использования сисадминами и внедренцами,
работающими с Astra Linux Special Edition. Сценарии позволяют создать полностью
автономный сервер для сетевой установки ОС. Например, на ноутбуке.

Важные функции:

 - Раздача репозитория операционной системы.
 - Упрощенное создание сценариев установки (pressed-файлов) оптимизированных
   для Астры.
 - Поддержка программного RAID и LVM.
 - Привязка IP-адресов к MAC-адресам.
 - Использование разных сценариев установки для разных узлов (привязка к MAC-адресам).

## Подготовка

 1. Установите пакет ansible: `apt-get -y install ansible`.
 2. Создайте файл stage (на базе примера stage.sample). Пример рассчитан на установку
    на локальный узел.
 3. Создайте сценарий установки на базе примера site.yml: `cp site.yml my.yml`.
    Поставляемый файл рассчитан на установку на локальном узле пользователем с
    возможностью использования sudo. Если вы работаете под root-ом, значение
    `sudo` установите в `false`.
 4. Создайте свои сценарии установки ОС (preseed) в отдельном файле, например
    `preseeds.yml`.
 5. Сконфигурируйте параметры инсталляции в файле `group_vars/all`.

## Установка

Установка производится с помощью Ansible:

    ansible-playbook -i stage my.yml

## Сценарий установки ОС (preseed-файл)

Preseed-файл задает параметры автоматической установки Debian-подобных систем.
В проекте есть роль `preseed` которая облегчает создание этого файла путем
автоматической генерации на основании шаблона. Вы можете создавать любое
количество preseed-файлов.

Пример использования нескольких сценариев (в `my.yml`):

    …
      vars_files:
        - my.yml

    …
      roles:
        - { role: preseed, preseed: "{{ server }}" }
        - { role: preseed, preseed: "{{ client }}" }
    …

В примере будет создаваться два preseed-файла, определяемых переменными
`server` и `client` в файле `preseeds.yml`:

    server:
        name: server
        …

    client:
        name: client
        …

В роли preseed уже есть две стандартные роли: `standard` и `dmraid`.
Некоторые ограничения на натсройку ролей:

 1. Два варианта разбиения диска: LVM и программный RAID (RAID + LVM).

## Замечания для версии Астры 1.3

Данная конфигурация настроена на установку по сети ОС Astra Linux Special Edition.
Для этого доработан образ initrd.gz инсталлятора (по отношению к netinst
c оригинального диска с ОС Astra Linux 1.3):

 - изменен usr/share/localechooser/languagelist поддержка русскому языку изменена с 2 на 1.
   было `ru;2;RU;ru_RU.UTF-8;;console-setup` стало `ru;1;RU;ru_RU.UTF-8;;console-setup`
 - добавлен модуль dca.ko в lib/modules/3.2.0-27-generic/kernel/drivers/dca/
 - добавлены модули `dm-log.ko` `dm-mirror.ko` `dm-mod.ko` `dm-region-hash.ko`
   в `lib/modules/3.2.0-27-generic/kernel/drivers/md/`
 - добавлены ключи `lab50-archive-keyring.gpg` от собственного репа
   (он служит зеркалом при установке) в `usr/share/keyrings`

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
`grub-installer_1.78ubuntu8_amd64.udeb`, поскольку `grub-installer_1.70astra.se4_amd64.udeb` содержит
ошибки при установке grub на soft&fake raid.

