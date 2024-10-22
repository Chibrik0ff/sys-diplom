#  Дипломная работа по профессии «`Системный администратор`» - `Чибриков Станислав`

## Инфраструктура

Устанавливаем Terraform и ansible на основную машину 

Всю инфраструктуру описываем в [main.tf](https://github.com/Chibrik0ff/sys-diplom/blob/main/diplom/main.tf) 


Вся необходимая информация для авторизации (token_id,cloud_id,folder_id,zone_a,zone_b) вынесены в переменные  terraform.tfvars 

Отдельно создан фаил с переменными [variables.tf](https://github.com/Chibrik0ff/sys-diplom/blob/main/diplom/variables.tf)

Создаем:
1) Server 1 
2) Server 2 
3) Bastion
4) Zabbix-server
5) Kibana-server
6) Elasticsearch-server

![screen](https://github.com/Chibrik0ff/sys-diplom/blob/main/img/1.png)

![screen](https://github.com/Chibrik0ff/sys-diplom/blob/main/img/2.png)

![screen](https://github.com/Chibrik0ff/sys-diplom/blob/main/img/3.png)

Главный плейбук называется [playbook.yml](https://github.com/Chibrik0ff/sys-diplom/blob/main/diplom/ansible/playbook.yml) 

При его запуске происходит установка и настройка всех необходимых утилит, программ, конфигураций и т.д 

Но далее каждый плейбук я буду описывать отдельно. 

## Сеть

Настройки сети вынесены в [network.tf](https://github.com/Chibrik0ff/sys-diplom/blob/main/diplom/network.tf)

Созданы 4 подсети:

subnet-private1 для web-server-1 \ зона a

subnet-private2 для web-server-2 \ зона b

subnet-private2 для Elasticsearch \ зона d

subnet-public1 для Zabbix, Kibana, ALB, Bastion

+ Настройки маршрутизации для приватных сетей

![screen](https://github.com/Chibrik0ff/sys-diplom/blob/main/img/4.png)


Добавляем настройки [security_group.tf](https://github.com/Chibrik0ff/sys-diplom/blob/main/diplom/security_group.tf) - для взаимодействия vm между собой, ключевым образом через bastion host 

Для обеспечения концепции bastion host, используем фаил 
[local_files.tf](https://github.com/Chibrik0ff/sys-diplom/blob/main/diplom/local_files.tf) который предоставляет ip адреса для ansible.

ansible считывает всю необходимую информацию из [hosts.cfg](https://github.com/Chibrik0ff/sys-diplom/blob/main/diplom/ansible/hosts.cfg)

Для которой используется шаблон: [hosts.tpl](https://github.com/Chibrik0ff/sys-diplom/blob/main/diplom/hosts.tpl) 

![screen](https://github.com/Chibrik0ff/sys-diplom/blob/main/img/5.png)

Прописано правило , которое запускает SSH соединение через Bastion
 
```
[all:vars]
ansible_ssh_common_args="-o ProxyCommand=\"ssh -q ubuntu@158.160.172.204 -o IdentityFile=~/.ssh/bastion -o Port=22 -W %h:%p\""
```

Проверка пинга всех хостов 

![screen](https://github.com/Chibrik0ff/sys-diplom/blob/main/img/6.png)

Пример подключения по ssh через bastion 

![screen](https://github.com/Chibrik0ff/sys-diplom/blob/main/img/7.png)

## Сайт 

Ранее мы создали Server 1 и Server 2 для нашего сайта, а так же создали Target Group,Backend Group,HTTP router,Application load balancer

![screen](https://github.com/Chibrik0ff/sys-diplom/blob/main/img/8.png)

![screen](https://github.com/Chibrik0ff/sys-diplom/blob/main/img/9.png)

![screen](https://github.com/Chibrik0ff/sys-diplom/blob/main/img/10.png)

![screen](https://github.com/Chibrik0ff/sys-diplom/blob/main/img/11.png)


Теперь установим на него Nginx, с помощью роли geerlingguy.nginx и перенесем свою страницу сайта [nginx.yml](https://github.com/Chibrik0ff/sys-diplom/blob/main/diplom/ansible/nginx.yml)

## Перейдем по ip адресу балансировщика и увидим одностраничный сайт : [158.160.160.53:80](http://158.160.160.53:80)

![screen](https://github.com/Chibrik0ff/sys-diplom/blob/main/img/12.png)

Протестируем сайт: curl -v 

```
*   Trying 158.160.160.53:80...
* Connected to 158.160.160.53 (158.160.160.53) port 80 (#0)
> GET / HTTP/1.1
> Host: 158.160.160.53
> User-Agent: curl/7.81.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< server: ycalb
< date: Tue, 22 Oct 2024 10:55:29 GMT
< content-type: text/html
< content-length: 1186
< last-modified: Tue, 22 Oct 2024 07:23:10 GMT
< etag: "666406de-4a2"
< accept-ranges: bytes
< 
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Визитка Системного администратора</title>
<style>
    body {
        font-family: Arial, sans-serif;
        margin: 0;
        padding: 0;
    }
    .header {
        background-color: #333;
        color: white;
        text-align: center;
        padding: 10px 0;
    }
    .content {
        text-align: center;
        margin-top: 20px;
    }
    img {
        max-width: 100%;
        height: auto;
        margin-top: 20px;
    }
    .footer {
        background-color: #333;
        color: white;
        text-align: center;
        padding: 10px 0;
    }
</style>
</head>
<body>
    <div class="header">
        <h1>Дипломная работа по профессии системный администратор</h1>
    </div>
    <div class="content">
        <p>Участник группы SYS-32 Netology</p>
        <p>Спасибо netology.ru за обучение!</p>
    </div>
    <div class="footer">
        <p>&copy; Чибриков Станислав, 2024</p>
    </div>
</body>
</html>
* Connection #0 to host 158.160.160.53 left intact
```
## Мониторинг 

### Zabbix-server

Для Мониторинга наших серверов установим Zabbix сервер на VM 

Для начала установим и создадим базу данных PostgreSQL с помощью плейбука [psql.yml](https://github.com/Chibrik0ff/sys-diplom/blob/main/diplom/ansible/psql.yml)

Установим сам Zabbix на сервер и заменим файл конфигурации на свой [zabbix_server.yml](https://github.com/Chibrik0ff/sys-diplom/blob/main/diplom/ansible/zabbix_server.yml) 

Файл конфигурации:
[zabbix_server.conf](https://github.com/Chibrik0ff/sys-diplom/blob/main/diplom/zabbix_server.conf)

Осталось зайти на сервер и выполнить 2 команды:
```
zcat /usr/share/zabbix-sql-scripts/postgresql/server.sql.gz | psql zabbix_db
systemctl restart zabbix-server zabbix-agent apache2
```
После заходим на админку, прописываем хостов, создаем удобные графики и списки. 

### Zabbix server доступен по адресу: <http://158.160.169.173/zabbix/>
### Логин: Admin
### Пароль: zabbix

![screen](https://github.com/Chibrik0ff/sys-diplom/blob/main/img/13.png)

![screen](https://github.com/Chibrik0ff/sys-diplom/blob/main/img/14.png)

### Zabbix-agentd

Далее установим Zabbix agent и заменим файлы конфигурации на все хосты [zabbix_agent.yml](https://github.com/Chibrik0ff/sys-diplom/blob/main/diplom/ansible/zabbix_agent.yml)

Файл конфигурации [zabbix_agentd.conf](https://github.com/Chibrik0ff/sys-diplom/blob/main/diplom/zabbix_agentd.conf), установит соединение с нашим Zabbix-server

## Логи

### Elasticsearch 
Устанавливаем на хост elas , elasticsearch и копируем конфигурацию плейбук:[elasticsearch.yml](https://github.com/Chibrik0ff/sys-diplom/blob/main/diplom/ansible/elasticsearch.yml)

Elastic установился и работает , статус yellow из-за кол-ва реплик, не настраивал более 1.
```
ubuntu@fhm6rgj8j62jmbjbaa4f:~$ curl -XGET 'localhost:9200/_cluster/health?pretty'
{
  "cluster_name" : "chibrik0ff-elk",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 10,
  "active_shards" : 10,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 1,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 90.9090909090909
}
ubuntu@fhm6rgj8j62jmbjbaa4f:~$ 
```

Фаил конфигурации: [elasticsearch.yml](https://github.com/Chibrik0ff/sys-diplom/blob/main/diplom/elasticsearch.yml)

### Filebeat
Устанавливаем filebeat на Server 1 и Server 2, для сбора логов и отправки их в elasticsearch, копируем фаил конфигурации. плейбук: [filebeat.yml](https://github.com/Chibrik0ff/sys-diplom/blob/main/diplom/ansible/filebeat.yml)

Фаил конфигурации: [filebeat.yml](https://github.com/Chibrik0ff/sys-diplom/blob/main/diplom/filebeat.yml)

### Kibana 
Устанавливаем Kibana на kib, копируем конфигурацию и ждем запуска:

Плейбук: [kibana.yml](https://github.com/Chibrik0ff/sys-diplom/blob/main/diplom/ansible/kibana.yml)

Фаил конфигурации: [kibana.yml](https://github.com/Chibrik0ff/sys-diplom/blob/main/diplom/kibana.yml)

```
~/diplom/ansible » ansible -i hosts.cfg -m shell -a 'cat /etc/kibana/kibana.yml' kib -b 
kib | CHANGED | rc=0 >>
server.host: 0.0.0.0
elasticsearch.hosts: ["http://192.168.30.10:9200"]
```

### Зайти на web Elasticsearch <http://158.160.168.121:5601/login/>

![screen](https://github.com/Chibrik0ff/sys-diplom/blob/main/img/15.png)


Логи подтянулись автоматически, можем смотреть поток filebeat 


![screen](https://github.com/Chibrik0ff/sys-diplom/blob/main/img/16.png)

![screen](https://github.com/Chibrik0ff/sys-diplom/blob/main/img/17.png)

## Резервное копирование 

Резервное копирование настроенно через [snapshots.tf](https://github.com/Chibrik0ff/sys-diplom/blob/main/diplom/snapshots.tf) , на ежедневные снимки, с хранением в 7 дней. 

![screen](https://github.com/Chibrik0ff/sys-diplom/blob/main/img/18.png)








