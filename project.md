### «Реализация отказоустойчивого кластера Patroni+Consul+PgBouncer+HAProxy+Keepalived и мониторинг с использованием Zabbix и Grafana»

<br><br>

В рамках проектной работы на платформе Яндекс.Облако был развернут высокодоступный и катастрофоустойчивый кластер Patroni размещенный в трех зонах доступности.


**Состав инфрастуктурной схемы:**

1. Кластер PostgreSQL из 3-х нод, на каждой установлены:

- **Patroni** для автоматического управления, репликации и аварийного переключения  кластера PostgreSQL {порт REST API **8008**};
    
- **Consul** в качестве системы распределенного хранилища данных для автоматического обнаружения и управления состоянием нод кластера Patroni {порт API Consul **8500**};
    
- **PgBouncer** для эффективного распределения запросов, выступающий в роли пуллера соединений {принимает соединения на порту **6432** и адресует запросы Patroni на 5432};

2. Кластер HAProxy из 2-х нод, на каждой установлено:

- **Keepalived** для реализации виртуального IP-адреса между нодами и выступающего в качестве единой точки входа для клиентов;
    
- **HAProxy** непосредственно в роли балансировщика нагрузки для распределения запросов клиентов между нодами Patroni {принимает соединения на порту **5000** и адресует их в сторону PgBouncer на 6432}; uri статистики **:10001/haproxy_stats** с авторизацией;
    

3. Сервер мониторинга **Zabbix** с инструментом визуализации данных **Grafana** для отслеживания состояния и реагирования на проблемы.


Примечание:

В Яндекс.Облаке не удалось запустить виртуальный IP-адрес Keepalived, вместо него в качестве единой точки входа использовался Yandex Network Load Balancer.


| **Назначение**    | **Сетевое имя** | **IP**        | **Зона доступности** |
| ----------------- | --------------- | ------------- | -------------------- |
| Virtual IP (VRPP) | \-              | Public IP     | ABC                  |
| Кластер HAProxy   | haproxy-node-1  | 192.168.1.50  | A                    |
|                   | haproxy-node-2  | 192.168.2.50  | B                    |
| Кластер Patroni   | patroni-node-1  | 192.168.1.100 | A                    |
|                   | patroni-node-2  | 192.168.2.100 | B                    |
|                   | patroni-node-3  | 192.168.3.100 | C                    |
| Север Zabbix            | zabbix-node-1   | 192.168.1.20  | A                    |


Версии используемого ПО на нодах кластеров Patroni и HAProxy:

- Ubuntu 22.04 LTS
- Consul: 1.16.1
- Patroni: 3.1.0
- PostgreSQL: 15.4.1
- Keepalived v2.2.4
- HAProxy 2.8.2

Версии ПО на сервере мониторинга:

- Ubuntu 22.04 LTS
- Zabbix 6.4.6
- Grafana 10.0.3

<br><br>

**VM в консоли YC:**


![console_yc](https://github.com/korobkin-s/otus-project/blob/main/media/console.png)

<br><br>

**Визуализация инфраструктурной схемы в виде диаграммы:**

![diagram](https://github.com/korobkin-s/otus-project/blob/main/media/infrastructure_diagram.png)

## Основные этапы проектной работы


### Развёртывание сервисов



#### Инструкции по кластеру Patroni:  


- **Нода №1** - установка и настройка [Consul](https://github.com/korobkin-s/otus-project/blob/main/patroni-node-1/Consul.md), [Patroni](https://github.com/korobkin-s/otus-project/blob/main/patroni-node-1/Patroni.md), [PgBouncer](https://github.com/korobkin-s/otus-project/blob/main/patroni-node-1/PgBouncer.md), [Zabbix-агент](https://github.com/korobkin-s/otus-project/blob/main/zabbix-node-1/%D0%A3%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0%20Zabbix-%D0%B0%D0%B3%D0%B5%D0%BD%D1%82%D0%B0.md)
  
- **Нода №2** - установка и настройка [Consul](https://github.com/korobkin-s/otus-project/blob/main/patroni-node-2/Consul.md), [Patroni](https://github.com/korobkin-s/otus-project/blob/main/patroni-node-2/Patroni.md), [PgBouncer](https://github.com/korobkin-s/otus-project/blob/main/patroni-node-2/PgBouncer.md), [Zabbix-агент](https://github.com/korobkin-s/otus-project/blob/main/zabbix-node-1/%D0%A3%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0%20Zabbix-%D0%B0%D0%B3%D0%B5%D0%BD%D1%82%D0%B0.md)
  
- **Нода №3** - установка и настройка [Consul](https://github.com/korobkin-s/otus-project/blob/main/patroni-node-3/Consul.md), [Patroni](https://github.com/korobkin-s/otus-project/blob/main/patroni-node-3/Patroni.md), [PgBouncer](https://github.com/korobkin-s/otus-project/blob/main/patroni-node-3/PgBouncer.md), [Zabbix-агент](https://github.com/korobkin-s/otus-project/blob/main/zabbix-node-1/%D0%A3%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0%20Zabbix-%D0%B0%D0%B3%D0%B5%D0%BD%D1%82%D0%B0.md)


#### Инструкции по кластеру HAProxy:

- **Нода №1** - установка и настройка [HAProxy](https://github.com/korobkin-s/otus-project/blob/main/haproxy-node-1/HAProxy.md), [Keepalived](https://github.com/korobkin-s/otus-project/blob/main/haproxy-node-1/Keepalived.md), [Zabbix-агент](https://github.com/korobkin-s/otus-project/blob/main/zabbix-node-1/%D0%A3%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0%20Zabbix-%D0%B0%D0%B3%D0%B5%D0%BD%D1%82%D0%B0.md)

- **Нода №2** - установка и настройка [HAProxy](https://github.com/korobkin-s/otus-project/blob/main/haproxy-node-2/HAProxy.md), [Keepalived](https://github.com/korobkin-s/otus-project/blob/main/haproxy-node-2/Keepalived.md), [Zabbix-агент](https://github.com/korobkin-s/otus-project/blob/main/zabbix-node-1/%D0%A3%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0%20Zabbix-%D0%B0%D0%B3%D0%B5%D0%BD%D1%82%D0%B0.md)



**Мониторинг:**

- **Zabbix** - [установка](https://github.com/korobkin-s/otus-project/blob/main/zabbix-node-1/%D0%A3%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0%20Zabbix-%D1%81%D0%B5%D1%80%D0%B2%D0%B5%D1%80%D0%B0.md)

- **Grafana** - [установка](https://github.com/korobkin-s/otus-project/blob/main/zabbix-node-1/%D0%A3%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0%20Grafana.md)


<br><br>


### Тестирование отказа

В кластере предварительно создана тестовая БД `otus` с простейшей таблицей `loading`:

```
CREATE TABLE loading (
    id SERIAL PRIMARY KEY,
    status TEXT,
    record_timestamp TIMESTAMP
);
```


Для имитации взаимодействия с базой был написан [скрипт](https://github.com/korobkin-s/otus-project/blob/main/load/%D0%A1%D0%BA%D1%80%D0%B8%D0%BF%D1%82%20%D0%B8%D0%BC%D0%B8%D1%82%D0%B0%D1%86%D0%B8%D0%B8%20%D0%BD%D0%B0%D0%B3%D1%80%D1%83%D0%B7%D0%BA%D0%B8.md) на Python, который запускается на отдельной VM с помощью [cron](https://github.com/korobkin-s/otus-project/blob/main/load/%D0%97%D0%B0%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5%20%D0%B2%20Cron.md) каждые 5 секунд. Скрипт выполняет подключения к базе данных и вставляет в таблицу 50 записей.

Также ведется логирование (`/mnt/load.log`):

- по факту успешной записи в базу информация об этом вносится в файл лога;
- в случае недоступности сервера в течении 3 секунд информация об этом также фиксируется в логе;


**Тестирование отказа одной из нод HAProxy**

В процессе работы скрипта останавливаем демон HAProxy на мастер-ноде и наблюдаем за событиями в логе - при очередной попытке записи в базу 2 раза подряд возникает ошибка подключения к серверу, на третий раз соединения с базой принимает реплик-нода HAProxy и процесс записи в БД продолжается:


```
2023-08-23 16:52:01,083 - INFO - Запросы успешно выполнены
2023-08-23 16:52:01,084 - INFO - Соединение с базой данных закрыто
2023-08-23 16:52:06,175 - INFO - Запросы успешно выполнены
2023-08-23 16:52:06,175 - INFO - Соединение с базой данных закрыто
2023-08-23 16:52:11,246 - INFO - Запросы успешно выполнены
2023-08-23 16:52:11,246 - INFO - Соединение с базой данных закрыто
2023-08-23 16:52:16,302 - ERROR - Ошибка при выполнении запросов: connection to server at "84.252.128.85", port 5000 failed: Connection refused

2023-08-23 16:52:21,354 - ERROR - Ошибка при выполнении запросов: connection to server at "84.252.128.85", port 5000 failed: Connection refused

2023-08-23 16:52:27,016 - INFO - Запросы успешно выполнены
2023-08-23 16:52:27,016 - INFO - Соединение с базой данных закрыто
2023-08-23 16:52:32,666 - INFO - Запросы успешно выполнены
2023-08-23 16:52:32,666 - INFO - Соединение с базой данных закрыто
```



**Тестирование отказа ноды Patroni**

Определяем лидер-ноду в кластере Patroni и останавливаем на ней демон Patroni:

```
root@patroni-node-1:~# patronictl -c /etc/patroni/patroni.yml list
+ Cluster: postgres --------------+---------+-----------+----+-----------+
| Member         | Host           | Role    | State     | TL | Lag in MB |
+----------------+----------------+---------+-----------+----+-----------+
| patroni-node-1 | patroni-node-1 | Leader  | running   |  2 |           |
| patroni-node-2 | patroni-node-2 | Replica | streaming |  2 |         0 |
| patroni-node-3 | patroni-node-3 | Replica | streaming |  2 |         0 |
+----------------+----------------+---------+-----------+----+-----------+
root@patroni-node-1:~#
```


Состояние кластера после этого:

```
root@patroni-node-2:~# patronictl -c /etc/patroni/patroni.yml list
+ Cluster: postgres --------------+---------+---------+----+-----------+
| Member         | Host           | Role    | State   | TL | Lag in MB |
+----------------+----------------+---------+---------+----+-----------+
| patroni-node-1 | patroni-node-1 | Replica | stopped |    |   unknown |
| patroni-node-2 | patroni-node-2 | Leader  | running |  3 |           |
| patroni-node-3 | patroni-node-3 | Replica | running |  2 |         0 |
+----------------+----------------+---------+---------+----+-----------+
root@patroni-node-2:~#
```


В процессе очередной записи в БД рвется соединение с сервером, но т.к. происходит назначение другого лидера, следующее соединение и запись в БД успешно завершаются:

```
2023-08-23 17:06:42,557 - INFO - Запросы успешно выполнены
2023-08-23 17:06:42,558 - INFO - Соединение с базой данных закрыто
2023-08-23 17:06:47,637 - INFO - Запросы успешно выполнены
2023-08-23 17:06:47,637 - INFO - Соединение с базой данных закрыто
2023-08-23 17:07:01,731 - INFO - Запросы успешно выполнены
2023-08-23 17:07:01,732 - INFO - Соединение с базой данных закрыто
2023-08-23 17:07:12,458 - ERROR - Ошибка при выполнении запросов: server closed the connection unexpectedly
	This probably means the server terminated abnormally
	before or while processing the request.

2023-08-23 17:07:12,458 - INFO - Соединение с базой данных закрыто
2023-08-23 17:07:17,741 - INFO - Запросы успешно выполнены
2023-08-23 17:07:17,741 - INFO - Соединение с базой данных закрыто
2023-08-23 17:07:23,011 - INFO - Запросы успешно выполнены
2023-08-23 17:07:23,011 - INFO - Соединение с базой данных закрыто
2023-08-23 17:07:28,282 - INFO - Запросы успешно выполнены
```

<br><br>


### Мониторинг

Мониторинг реализован с помощью системы мониторинга Zabbix в связке с инструментом визуализации Grafana.

К нодам HAProxy присоединены стандартные шаблоны **HAProxy by HTTP**, **ICMP Ping**, **Linux by Zabbix agent**

К нодам Patroni присоединены стандартные шаблоны **ICMP Ping**, **Linux by Zabbix agent** и дополнительно [кастомные](https://github.com/korobkin-s/otus-project/blob/main/zabbix-node-1/patroni_templates.yaml) - настроил парсинг ролей нод и состояния репликации с API **8008/patroni**


![zabbix_metric](https://github.com/korobkin-s/otus-project/blob/main/media/zabbix_metric.png)


![zabbix_host](https://github.com/korobkin-s/otus-project/blob/main/media/zabbix_host.png)

<br><br>

В Grafana вывел на дашборд состояние ролей и репликации нод Patroni, утилизацию на них CPU и пинг. И по HAProxy статистика трафика, информация по сеансам и утилизация CPU:

![grafana](https://github.com/korobkin-s/otus-project/blob/main/media/grafana.png)
