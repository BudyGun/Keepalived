# Домашнее задание к занятию 1 «Disaster recovery и Keepalived»

### Цель задания
В результате выполнения этого задания вы научитесь:
1. Настраивать отслеживание интерфейса для протокола HSRP;
2. Настраивать сервис Keepalived для использования плавающего IP

------

### Чеклист готовности к домашнему заданию

1. Установлена программа Cisco Packet Tracer
2. Установлена операционная система Ubuntu на виртуальную машину и имеется доступ к терминалу
3. Сделан клон этой виртуальной машины, они находятся в одной подсети и имеют разные IP адреса
4. Просмотрены конфигурационные файлы, рассматриваемые на лекции, которые находятся по [ссылке](1/)


------

### Инструкция по выполнению домашнего задания

1. Сделайте fork [репозитория c шаблоном решения](https://github.com/netology-code/sys-pattern-homework) к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/gitlab-hw или https://github.com/имя-вашего-репозитория/8-03-hw).
2. Выполните клонирование этого репозитория к себе на ПК с помощью команды git clone.
3. Выполните домашнее задание и заполните у себя локально этот файл README.md:
   - впишите вверху название занятия и ваши фамилию и имя;
   - в каждом задании добавьте решение в требуемом виде: текст/код/скриншоты/ссылка;
   - для корректного добавления скриншотов воспользуйтесь инструкцией [«Как вставить скриншот в шаблон с решением»](https://github.com/netology-code/sys-pattern-homework/blob/main/screen-instruction.md);
   - при оформлении используйте возможности языка разметки md. Коротко об этом можно посмотреть в [инструкции по MarkDown](https://github.com/netology-code/sys-pattern-homework/blob/main/md-instruction.md).
4. После завершения работы над домашним заданием сделайте коммит (git commit -m "comment") и отправьте его на Github (git push origin).
5. Для проверки домашнего задания преподавателем в личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
6. Любые вопросы задавайте в чате учебной группы и/или в разделе «Вопросы по заданию» в личном кабинете.



------


### Задание 1
- Дана [схема](1/hsrp_advanced.pkt) для Cisco Packet Tracer, рассматриваемая в лекции.
- На данной схеме уже настроено отслеживание интерфейсов маршрутизаторов Gi0/1 (для нулевой группы)
- Необходимо аналогично настроить отслеживание состояния интерфейсов Gi0/0 (для первой группы).
- Для проверки корректности настройки, разорвите один из кабелей между одним из маршрутизаторов и Switch0 и запустите ping между PC0 и Server0.
- На проверку отправьте получившуюся схему в формате pkt и скриншот, где виден процесс настройки маршрутизатора.

### Решение 1

![1](https://github.com/BudyGun/Keepalived/blob/main/img/pkt1.png)
![1](https://github.com/BudyGun/Keepalived/blob/main/img/pkt2.png)
![1](https://github.com/BudyGun/Keepalived/blob/main/img/pkt3.png)




### Задание 2
- Запустите две виртуальные машины Linux, установите и настройте сервис Keepalived как в лекции, используя пример конфигурационного [файла](1/keepalived-simple.conf).
- Настройте любой веб-сервер (например, nginx или simple python server) на двух виртуальных машинах
- Напишите Bash-скрипт, который будет проверять доступность порта данного веб-сервера и существование файла index.html в root-директории данного веб-сервера.
- Настройте Keepalived так, чтобы он запускал данный скрипт каждые 3 секунды и переносил виртуальный IP на другой сервер, если bash-скрипт завершался с кодом, отличным от нуля (то есть порт веб-сервера был недоступен или отсутствовал index.html). Используйте для этого секцию vrrp_script
- На проверку отправьте получившейся bash-скрипт и конфигурационный файл keepalived, а также скриншот с демонстрацией переезда плавающего ip на другой сервер в случае недоступности порта или файла index.html




### Решение 2

Создал две виртуальные машины с запущенным web-серверами: 192.168.1.4 - Server 1  и 192.168.1.11 - Server 2

![1](https://github.com/BudyGun/Keepalived/blob/main/img/k1.png)
![1](https://github.com/BudyGun/Keepalived/blob/main/img/k2.png)

Выбрал виртуальный ip-адрес - 192.168.1.250

Конфигурационный файл /etc/keepalived/keepalived.conf первого сервера со статусом MASTER и приоритетом 255:

```
global_defs {
    enable_script_security
}

vrrp_script check_script {
      script "/home/vboxuser/keepalived/script.sh"
      interval 3
}

vrrp_instance www {
        state MASTER
        interface enp0s3
        virtual_router_id 4
        priority 255
        advert_int 1

        virtual_ipaddress {
             192.168.1.250/24
        }
}
```

Конфигурационный файл /etc/keepalived/keepalived.conf второго сервера со статусом BACKUP и приоритетом 200:
```
global_defs {
    enable_script_security
}

rrp_script check_script {
      script "/home/vboxuser2/keepalived/script.sh"
      interval 3
}

vrrp_instance www {
        state BACKUP
        interface enp0s3
        virtual_router_id 4
        priority 200
        advert_int 1

        virtual_ipaddress {
              192.168.1.250/24
        }
}
```

Скрипт файла script.sh
```
#!/bin/bash
if [[ $(netstat -tuln | grep LISTEN | grep :80) ]] && [[ -f /var/www/html/index.html ]]; then
        exit 0
else
        sudo systemctl stop keepalived
fi
```

Запускаю keepalived сервис на обоих серверах и захожу на виртуальный адрес 192.168.1.250. Вижу страничку с мастер сервера 1. При этом статус второго сервера - Backup.

![1](https://github.com/BudyGun/Keepalived/blob/main/img/k10.png)
![1](https://github.com/BudyGun/Keepalived/blob/main/img/k11.png)

Отключаю keepalived сервис на первом сервере мастер, вижу что страничка грузится со второго сервера:
![1](https://github.com/BudyGun/Keepalived/blob/main/img/k12.png)

при этом второй сервер становится мастер:
![1](https://github.com/BudyGun/Keepalived/blob/main/img/k13.png)






