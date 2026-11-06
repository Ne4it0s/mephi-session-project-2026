# mephi-session-project-2026
Отчет по сессионному заданию Жидков Никита М265232

Примечание: для простоты администрирования терминал запущен от root (sudo -i)

##Раздел 1: Установка дистрибутива
  - Динамическая адресация и имя хоста заданы в терминале уже запущенной ВМ.
  - Выход в Интернет подтверждается связностью с 8.8.8.8 (файл ping.out)

)

##Раздел 2: Управление ПО
  - Обновление дистрибутива - 'dnf upgrade'
  - Установка необходимых пакетов - 'dnf install'
  - Скачивание tcpdump без установки - 'dnf download tcpdump --destdir=/tmp'
  - Установка через rpm - 'rpm -ivh /tmp/tcpdump-*.x86_64.rpm'

)

##Раздел 3: Управление ФС
  - Отключив ВМ, создается новый vdi
  - Полностью размечается в ext4 - 'parted -s /dev/sdb mklabel gpt mkpart primary ext4 1MiB 100%'
  - Добавляется метка - 'mkfs.ext4 -L MEPHI_WEB /dev/sdb1'
  - Добавление диска в автомонтирование - 'echo "LABEL=MEPHI_WEB /mephi-web ext4 defaults 0 2" >> /etc/fstab'
  - Ручное монтирование диска - 'mount /mephi-web'

)

##Раздел 4: Управление сервисами
  - Добавление nginx в автозагрузку и ручной запуск - 'systemctl enable --now nginx'
  - Поиск логов, связанных с nginx - 'journalctl -u nginx -b > journalctl.out'

)

##Раздел 6: Аутентификация
  - Раздел выполнен раньше, чтобы пользователи созданные в разделе 5 получали необходимые параметры автоматически.
  - Смена пароля каждые 90 дней - PASS_MAX_DAYS=90 в /etc/login.defs
  - Минимальная длина пароля 12 символов - minlen=12 в /etc/security/pwquality.conf
  - Запрет локального входа группе curators - 'echo "-:(curators):LOCAL" >> /etc/security/access.conf 
                                            sed -i '/^account.*include.*system-auth/i account required pam_access.so' /etc/pam.d/login'

)

##Раздел 5: Управление доступом
  - Создание группы с конкретным GID - 'groupadd curators -g 4444'
  - Создание пользователя с конкретным UID - 'useradd user1 -u 5501 -G developers' (группа developers включает в себя разработчиков для упрощения администрирования)
  - Директории проекта назначается группа-владелец developers (для SGID-бита), но владелец остается root - 'chown root:developers /data/mephi-2026'
  - Сам SGID-бит - 'chmod 2770 /data/mephi-2026'
  - Создается ACL доступа к директории - 'setfacl -m g:developers:rwx,g:curators:rx,o::--- /data/mephi-2026'
  - Создается ACL по умолчанию - 'setfacl -d -m u::rwx,g::rwx,g:developers:rwx,g:curators:rx,o::--- /data/mephi-2026 (маски считаются автоматически - rwx)'
  - Родительской папке выдаются права на поиск для всех - 'chmod 755 /data'
  - У tcpdump убирается SUID-бит - 'chmod 755 /usr/sbin/tcpdump'
  - Исполняемому файлу выдаются только необходимые привилегии для сбора трафика - 'setcap cap_net_raw,cap_net_admin=ep /usr/sbin/tcpdump'
  - Проверка статуса SELinux - 'getenforce'
  - Изменение корневой директории nginx - root=/mephi-web; в /etc/nginx/nginx.conf
