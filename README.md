# Высокодоступный кластер PostgreSQL на базе Patroni. С DNS точкой клиентского доступа, с поддержкой геораспределения.

[![GitHub license](https://img.shields.io/github/license/IlgizMamyshev/pgsql_cluster)](./LICENSE) 
![GitHub stars](https://img.shields.io/github/stars/IlgizMamyshev/pgsql_cluster)

---
![Banner](https://github.com/IlgizMamyshev/pgsql_cluster/blob/master/doc/PostreSQLBanner1600x400.png)

### Развертывание кластера высокой доступности PostgreSQL на основе Patroni. Автоматизация с помощью Ansible.

Этот Ansible playbook разработан для развёртывания высокодоступного кластера PostgreSQL на выделенных физических серверах для производственной среды.  
Развёртывание может быть выполнено в виртуальной среде для тестовой среды или небольших проектов.  

В дополнение к развертыванию новых кластеров этот playbook также поддерживает развертывание кластера поверх уже существующего и работающего PostgreSQL. Вы можете преобразовать выделенный экземпляр PostgreSQL в кластер высокой доступности (укажите переменную `postgresql_exists='true'` в файле инвентаризации).  
  
**Внимание!** Ваш экземпляр PostgreSQL будет остановлен перед запуском в составе кластера (запланируйте небольшой простой баз данных).

> :heavy_exclamation_mark: Пожалуйста, проведите тестирование, прежде чем использовать в производственной среде.
  
  
#### Основные возможности:
- развёртывание кластера [Patroni](https://patroni.readthedocs.io/en/latest/) с СУБД PostgreSQL или Postgres Pro;
- использование встроенного механизма распределённого консенсуса или использование внешней DCS (etcd);
- настройка watchdog для Patroni (защита от split-brain);
- настройка параметров ядра операционной системы Linux;
- настройка сетевого брандмауэра;
- DNS точка подключения клиентов ([DNS Connection Point](https://github.com/IlgizMamyshev/dnscp));
- поддержка геораспределенного кластера ([DNS Connection Point](https://github.com/IlgizMamyshev/dnscp));
- развёртывание HAProxy для балансировки доступа к репликам только для чтения;
- развёртывание кластера [etcd](https://etcd.io/docs/v3.5/op-guide/);

##### Высокодоступный кластер, на базе Patroni, etcd и DNSCP:  
![PGSQLCluster](https://github.com/IlgizMamyshev/pgsql_cluster/blob/master/doc/PGSQLClusterTypeA.png)

Другие варианты реализации архитектуры высокой доступности - смотреть [примеры](./doc/README.md).

#### Компоненты высокой доступности:
[**Patroni**](https://github.com/zalando/patroni) - это шаблон для создания решения высокой доступности с использованием Python и распределенного хранилища конфигурации, [*собственного*](https://github.com/zalando/patroni/pull/375) или такого как ZooKeeper, etcd, Consul или Kubernetes. Используется для автоматизации управления экземплярами PostgreSQL и автоматического аварийного переключения.

:white_check_mark: протестировано: `Patroni 2.01.07`

[**etcd**](https://github.com/etcd-io/etcd) - это распределенное надежное хранилище ключей и значений для наиболее важных данных распределенной системы. etcd написан на Go и использует алгоритм консенсуса [Raft](https://raft.github.io/) для управления высокодоступным реплицированным журналом. Он используется Patroni для хранения информации о состоянии кластера и параметрах конфигурации PostgreSQL.

:white_check_mark: протестировано: `etcd 3.5.6`

[Что такое Распределенный Консенсус (Distributed Consensus)?](http://thesecretlivesofdata.com/raft/)

[**DNS Connection Point for Patroni**](https://github.com/IlgizMamyshev/dnscp) используется для обеспечения единой точки входа. DNSCP обеспечивает регистрацию DNS-записи как единой точки входа для клиентов и позволяет использовать один или более виртуальных IP-адресов (VIP), принадлежащих одной или нескольким подсетям. DNSCP использует функцию обратных вызовов ([callback](https://patroni.readthedocs.io/en/latest/SETTINGS.html)) [Patroni](https://github.com/zalando/patroni). 

#### СУБД PostgreSQL:
[**PostgreSQL**](https://www.postgresql.org) - реляционная база данных с открытым исходным кодом. При использовании ОС Astra Linux возможно использование PostgreSQL в составе репозитория ОС.  
Поддерживаются все поддерживаемые версии PostgreSQL.

:white_check_mark: протестировано: `PostgreSQL 11, 14, 15, 16`

[**Postgres Pro**](https://www.postgrespro.ru) - Российская система управления базами данных на основе PostgreSQL. Коммерческий продукт.
Поддерживаются все версии Postgres Pro, редакции Standard и Enterprise.

:white_check_mark: протестировано: `Postgres Pro 14, 15`

#### Операционные Системы:
- **Debian**: 9, 10, 11
- **Astra Linux**: CE (основан на Debian 9), SE (основан на Debian 10)

:white_check_mark: протестировано: `Astra Linux CE 2.12, Astra Linux SE 1.7`

#### Ansible:
Для автоматизации развёртывания Решения используется [Ansible](https://www.ansible.com) - система управления конфигурациями. При использовании ОС Astra Linux возможно использование Ansible из состава репозитория операционной системы.
Минимальная поддерживаемая версия Ansible - 2.7.

## Требования
Этот playbook требует root привилегий или sudo.

Ansible ([Что такое Ansible](https://www.ansible.com/resources/videos/quick-start-video)?)

## Требования к портам
Список необходимых портов TCP, которые должны быть открыты на узлах кластера кластера СУБД:

- `5432` (PostgreSQL)
- `8008` (Patroni Rest API)
- `2379`, `2380` (etcd)
- `2379` (Patroni RAFT)

#### Связанные ссылки:
- [Планирование портов и протоколов](/doc/protocol_workloads.md)

## Рекомендации
- **Linux (Операционная Система)**: 

Обновите все операционные системы перед развёртыванием;

Присоедините серверы-узлы кластера СУБД к домену [Microsoft Active Directory](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview) или [Astra Linux Directory](https://wiki.astralinux.ru/display/doc/Astra+Linux+Directory). Присоединение к домену является требованием, если вы хотите использовать аутентифицированный доступ к DNS-серверу при регистрации DNS-имени точки клиентского доступа.

- **Patroni RAFT**: 

Patroni может не зависеть от сторонних систем DCS (Distributed Consensus Store, типа etcd, Consul, ZooKeeper) за счёт собственной реализации [RAFT](https://patroni.readthedocs.io/en/latest/SETTINGS.html#raft-settings).

- **DCS (Распределённое Хранилище Конфигурации (Distributed Configuration Store))**: 

Быстрые диски и надежная сеть являются наиболее важными факторами производительности и стабильности кластера etcd.

Избегайте хранения данных etcd на одном диске вместе с другими процессами (такими как база данных), интенсивно использующими ресурсы дисковой подсистемы!  
Храните данные etcd и PostgreSQL на **разных** дисках (см. переменную `etcd_data_dir`), по возможности используйте диски ssd.  
Рекомендуется изучить руководства [по выбору оборудования](https://etcd.io/docs/v3.3.12/op-guide/hardware/) и [настройке](https://etcd.io/docs/v3.3.12/tuning/) etcd.

Для высоконагруженных СУБД рассмотрите установку кластера etcd на выделенных серверах.  

Изучите вопросы планирования кластера etcd. [Примеры](https://www.cybertec-postgresql.com/en/introduction-and-how-to-etcd-clusters-for-patroni/).

- **Как предотвратить потерю данных в случае автоматической отработки отказа (synchronous_modes and pg_rewind)**:

По соображениям надёжности синхронная репликация в данном шаблоне включена. Автоматическая отработка отказа выполняется с переходом только на синхронную реплику.

Чтобы ещё более ужесточить требования к надёжности и свести к минимуму риск потери данных при автоматическом переходе на другой ресурс, вы можете настроить параметры следующим образом:
- [synchronous_mode](https://patroni.readthedocs.io/en/latest/replication_modes.html#synchronous-mode): 'true' (включен по умолчанию в данном playbook)
- [synchronous_mode_strict](https://patroni.readthedocs.io/en/latest/replication_modes.html#synchronous-mode): 'true' (выключен  по умолчанию)
- [synchronous_commit](https://postgrespro.ru/docs/postgrespro/14/runtime-config-wal#GUC-SYNCHRONOUS-COMMIT): 'on' (или 'remote_apply') ('on'  по умолчанию)
- [use_pg_rewind](https://postgrespro.ru/docs/postgrespro/14/app-pgrewind): '[false](https://patroni.readthedocs.io/en/latest/SETTINGS.html#dynamic-configuration-settings)' (включен по умолчанию)

## Развёртывание: быстрый старт
1. Подготовьте серверы
Установите операционную систему, настройте репозитории операционной системы и [PostgreSQL](https://www.postgresql.org/download/linux/), установите обновления операционной системы. \
Переименуйте серверы: `sudo hostnamectl set-hostname pgsql-n1.im.local` \
Присоедините серверы к домену Active Directory, если требуется.

2. [Установите Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) на сервер управления, свой компьютер или ноутбук
##### Пример 1 (установка, используя репозиторий [Astra Linux](https://wiki.astralinux.ru/pages/viewpage.action?pageId=27362819)):
`sudo apt update` \
`sudo apt install ansible` \

`--Установка пакета для работы с pyOpenSSL` \
`sudo apt-get install libssl-dev` \
`sudo pip3 install --upgrade pip` \
`sudo pip install pyOpenSSL`

##### Пример 2 (установка, используя [pip](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-with-pip) ):
`sudo apt update && sudo apt install python3-pip sshpass git -y` \
`sudo pip3 install ansible` \

`--Установка пакета для работы с pyOpenSSL` \
`sudo apt-get install libssl-dev` \
`sudo pip3 install --upgrade pip` \
`sudo pip install pyOpenSSL`

3. Скачайте или клонируйте этот репозиторий

`git clone https://github.com/IlgizMamyshev/pgsql_cluster.git`

4. Перейдите в каталог с файлами playbook

`cd pgsql_cluster/`

5. Отредактируйте файл инвентаризации

##### Задайте IP-адреса и параметры подключения (`ansible_user`, `ansible_ssh_pass` ...)

`vim inventory`

6. Отредактируйте значения переменных в файле vars/[main.yml](./vars/main.yml)

`vim vars/main.yml`

6.1 Запустите playbook для установки кластера etcd (опционально, если используете etcd DCS вместо Patroni RAFT):

`sudo ansible-playbook deploy_etcdcluster.yml -K` \
После успешного развёртывания etcd в /vars/[main.yml](./vars/main.yml) укажите `dcs_exists: true` и `dcs_type: "etcd"`

6.2 Запустите playbook для установки кластера PostgreSQL:

`sudo ansible-playbook deploy_pgcluster.yml -K`

## Переменные
Смотри файлы vars/[main.yml](./vars/main.yml), [system.yml](./vars/system.yml) и [Debian.yml](./vars/Debian.yml), чтобы узнать подробности.

## Проверка после развёртывания
### Patroni
##### Статус сервиса
```
sudo systemctl status patroni.service
```

##### Журнал событий
```
sudo grep -i patroni /var/log/syslog
```
```
sudo tail -n 15 /var/log/syslog
```

##### Здоровье кластера
```
sudo patronictl -c /etc/patroni/patroni.yml list
```
```
+ Cluster: PGSQL-CL (1234567890123456789) ----------------+----+-----------+
| Member          | Host         | Role         | State   | TL | Lag in MB |
+-----------------+--------------+--------------+---------+----+-----------+
| PGSQL-N2        | 172.16.33.22 | Sync Standby | running |  3 |         0 |
| PGSQL-N3        | 172.16.33.33 | Leader       | running |  3 |           |
+-----------------+--------------+--------------+---------+----+-----------+
```

### DNSCP for Patroni
##### Журнал событий
```
sudo grep -i callback /var/log/syslog
```
##### Задание Планировщика для обновления динамической DNS-записи
```
sudo cat /var/spool/cron/crontabs/postgres
```

##### RAFT
```
sudo syncobj_admin -conn pgsql-n2:2379 -pass Password -status
```

### HA Proxy
##### Статистика
`http://pgsql-n2.im.local:7000` \
`http://pgsql-n3.im.local:7000`

### PostgreSQL
##### Журнал событий
```
sudo tail -n 20 /var/log/postgresql/postgresql-15.log
```

##### Тестовое подключение
```
/opt/pgpro/std-14/bin/psql -p 5432 -U postgres -d postgres -c "SELECT version();"
```

### etcd
##### Статус сервиса
```
sudo systemctl status etcd.service
```

##### Здоровье кластера
```
sudo ETCDCTL_API=2 etcdctl --ca-file="/etc/etcd/ssl/ca.crt" --endpoints https://127.0.0.1:2379 --cert-file=/etc/etcd/ssl/server.crt --key-file=/etc/etcd/ssl/server.key cluster-health
```
```
sudo su
ENDPOINTS=$(ETCDCTL_API=3 etcdctl member list --cacert="/etc/etcd/ssl/ca.crt" --cert=/etc/etcd/ssl/server.crt --key=/etc/etcd/ssl/server.key | grep -o '[^ ]\+:2379' | paste -s -d,)
ETCDCTL_API=3 etcdctl --endpoints=$ENDPOINTS endpoint health --write-out=table --cacert="/etc/etcd/ssl/ca.crt" --cert=/etc/etcd/ssl/server.crt --key=/etc/etcd/ssl/server.key
```
```
+----------------------------+--------+-------------+-------+
|          ENDPOINT          | HEALTH |    TOOK     | ERROR |
+----------------------------+--------+-------------+-------+
| https://172.16.32.11:2379  |   true | 23.921885ms |       |
| https://172.16.64.22:2379  |   true | 35.531942ms |       |
| https://172.16.96.33:2379  |   true | 36.971386ms |       |
+----------------------------+--------+-------------+-------+
```

##### Список узлов кластера
```
ETCDCTL_API=3 sudo etcdctl member list -w table --cacert="/etc/etcd/ssl/ca.crt" --cert=/etc/etcd/ssl/server.crt --key=/etc/etcd/ssl/server.key
```
```
+------------------+---------+-----------------+----------------------------+----------------------------+------------+
|        ID        | STATUS  |      NAME       |         PEER ADDRS         |        CLIENT ADDRS        | IS LEARNER |
+------------------+---------+-----------------+----------------------------+----------------------------+------------+
| 1111111111111111 | started | ETCD-N1         | https://172.16.32.11:2380  | https://172.16.32.11:2379  |      false |
| 2222222222222222 | started | ETCD-N2         | https://172.16.64.22:2380  | https://172.16.64.22:2379  |      false |
| 3333333333333333 | started | ETCD-N3         | https://172.16.96.33:2380  | https://172.16.96.33:2379  |      false |
+------------------+---------+-----------------+----------------------------+----------------------------+------------+
```
  
  
```
sudo su
ENDPOINTS=$(ETCDCTL_API=3 etcdctl member list --cacert="/etc/etcd/ssl/ca.crt" --cert=/etc/etcd/ssl/server.crt --key=/etc/etcd/ssl/server.key | grep -o '[^ ]\+:2379' | paste -s -d,)
ETCDCTL_API=3 etcdctl --endpoints=$ENDPOINTS endpoint status --write-out=table --cacert="/etc/etcd/ssl/ca.crt" --cert=/etc/etcd/ssl/server.crt --key=/etc/etcd/ssl/server.key
```
```
+----------------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|          ENDPOINT          |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
+----------------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
| https://172.16.32.11:2379  | 1111111111111111 |   3.5.6 |   20 kB |     false |      false |         2 |         23 |                 23 |        |
| https://172.16.64.22:2379  | 2222222222222222 |   3.5.6 |   20 kB |      true |      false |         2 |         23 |                 23 |        |
| https://172.16.96.33:2379  | 3333333333333333 |   3.5.6 |   20 kB |     false |      false |         2 |         23 |                 23 |        |
+----------------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
```

##### Конфигурация PostgreSQL в etcd
```
sudo ETCDCTL_API=2 etcdctl --ca-file="/etc/etcd/ssl/ca.crt" --endpoints https://127.0.0.1:2379 --cert-file=/etc/etcd/ssl/server.crt --key-file=/etc/etcd/ssl/server.key get service/pgsql-cluster/config
```

## Обслуживание
Данный вопрос выходит за рамки данного описания.  

Предлагается изучить следующие дополнительные материалы:

- [Учебник: Управление высокодоступными кластерами PostgreSQL с Patroni](https://pgconf.ru/en/2018/108567)
- [Документация Patroni](https://patroni.readthedocs.io/en/latest/)
- [Руководство по эксплуатации etcd](https://etcd.io/docs/v3.3.12/op-guide/)

## Аварийное восстановление

Кластер высокой доступности обеспечивает механизм автоматического перехода на другой ресурс и не охватывает все сценарии аварийного восстановления.
Вы должны позаботиться о резервном копировании своих данных самостоятельно.
##### etcd
> Узлы Patroni сбрасывают состояние параметров DCS на диск при каждом изменении конфигурации в файл patroni.dynamic.json, расположенный в каталоге данных PostgreSQL. Мастеру (узел Patroni с ролью Лидера) разрешено восстанавливать эти параметры из дампа на диске, если они полностью отсутствуют в DCS или если они недействительны.

Тем не менее, рекомендуется ознакомиться с руководством по аварийному восстановлению кластера etcd:
- [Аварийное восстановление etcd](https://etcd.io/docs/v3.3.12/op-guide/recovery)

##### PostgreSQL (базы данных)
Рекомендуемые средства резервного копирования и восстановления:
* [pg_probackup](https://github.com/postgrespro/pg_probackup)

Не забывайте тестировать свои резервные копии.

## Как начать развёртывание с начала
Если вам нужно начать с самого начала, используйте для очистки следующие команды:
- на всех узлах СУБД остановить сервис Patroni и удалить кластер баз данных (каталог с базами данных, PGDATA), каталог с конфигурацией Patroni:
    ```shell
    sudo systemctl stop patroni
    sudo rm -rf /var/lib/postgresql/ # будьте осторожны, если есть другие экземпляры PostgreSQL
    sudo rm -rf /etc/patroni/
    ```
- затем, если используется etcd, удалите запись в etcd (можно запустить на любом узле etcd):
    ```shell 
    sudo ETCDCTL_API=2 etcdctl --ca-file="/etc/etcd/ssl/ca.crt" --endpoints https://127.0.0.1:2379 --cert-file=/etc/etcd/ssl/server.crt --key-file=/etc/etcd/ssl/server.key rm --dir --recursive /service/
    ```

## Лицензия
Под лицензией MIT License. Подробнее см. в файле [LICENSE](./LICENSE) .

## Автор
Илгиз Мамышев (Microsoft SQL Server, PostgreSQL DBA) \
[https://imamyshev.wordpress.com](https://imamyshev.wordpress.com/2022/05/29/dns-connection-point-for-patroni/) \
Виталий Кухарик (PostgreSQL DBA) - автор проекта [postgresql_cluster](https://github.com/vitabaks/postgresql_cluster) на кодовой базе которого построен [pgsql_cluster](https://github.com/IlgizMamyshev/pgsql_cluster/)

## Обратная связь, отчеты об ошибках, запросы и т.п.
[Добро пожаловать](https://github.com/IlgizMamyshev/pgsql_cluster/issues)!
