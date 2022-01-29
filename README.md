# Домашнее задание к занятию "10.02. Системы мониторинга"

## Обязательные задания

1. Опишите основные плюсы и минусы pull и push систем мониторинга.

push
- Плюсы
  - Возможность настраивать параметры отправки пакетов на каждом агенте
  - Удобство настройки при динамическом создании хостов
  - Возможность настроить репликацию данных в разные системы мониторинга, повышение надежности в случае отказа одной системы сбора данных.
- Минуы
  - Нет возможности централизованно контролировать/настраивать агенты мониторинга

pull
- Плюсы
  - Единая точка конфигурирования
  - Возможность настройки прокси-сервера до агентов
  - Проще контролировать агенты мониторина, опрос ведется только тех, которые настроены в системе
- Минусы
  - В случае отказа системы сбора данных, возможна потеря метрик.
  - Сложнее конфигурировать новые агенты

2. Какие из ниже перечисленных систем относятся к push модели, а какие к pull? А может есть гибридные?

    - Prometheus - гибридная
    - TICK - гибридная
    - Zabbix - pull
    - VictoriaMetrics - push
    - Nagios - pull

3. Склонируйте себе [репозиторий](https://github.com/influxdata/sandbox/tree/master) и запустите TICK-стэк, 
используя технологии docker и docker-compose.

В виде решения на это упражнение приведите выводы команд с вашего компьютера (виртуальной машины):

    - curl http://localhost:8086/ping
    - curl http://localhost:8888
    - curl http://localhost:9092/kapacitor/v1/ping

А также скриншот веб-интерфейса ПО chronograf (`http://localhost:8888`). 

P.S.: если при запуске некоторые контейнеры будут падать с ошибкой - проставьте им режим `Z`, например
`./data:/var/lib:Z`  

### Решение  


- curl http://localhost:8086/ping
```
Пустой ответ
```

- curl http://localhost:8888
```
<!DOCTYPE html><html><head><meta http-equiv="Content-type" content="text/html; charset=utf-8"><title>Chronograf</title><link rel="icon shortcut" href="/favicon.fa749080.ico"><link rel="stylesheet" href="/src.3dbae016.css"></head><body> <div id="react-root" data-basepath=""></div> <script src="/src.fab22342.js"></script> </body></html>
```
curl http://localhost:9092/kapacitor/v1/ping
```
Пустой ответ
```
Скрин:  
![alt text](https://github.com/rdegtyarev/mnt-10-02/blob/main/images/3.png)

---

4. Перейдите в веб-интерфейс Chronograf (`http://localhost:8888`) и откройте вкладку `Data explorer`.

    - Нажмите на кнопку `Add a query`
    - Изучите вывод интерфейса и выберите БД `telegraf.autogen`
    - В `measurments` выберите mem->host->telegraf_container_id , а в `fields` выберите used_percent. 
    Внизу появится график утилизации оперативной памяти в контейнере telegraf.
    - Вверху вы можете увидеть запрос, аналогичный SQL-синтаксису. 
    Поэкспериментируйте с запросом, попробуйте изменить группировку и интервал наблюдений.
```diff
-  Чтобы это заработало нужно в telegraf.conf добавить [[inputs.mem]]
```

Для выполнения задания приведите скриншот с отображением метрик утилизации места на диске 
(disk->host->telegraf_container_id) из веб-интерфейса.
```diff
-  Чтобы это заработало нужно в telegraf.conf добавить [[inputs.disk]]
```
### Решение  
Скрин:  
![alt text](https://github.com/rdegtyarev/mnt-10-02/blob/main/images/4.png)

---

5. Изучите список [telegraf inputs](https://github.com/influxdata/telegraf/tree/master/plugins/inputs). 
Добавьте в конфигурацию telegraf следующий плагин - [docker](https://github.com/influxdata/telegraf/tree/master/plugins/inputs/docker):
```
[[inputs.docker]]
  endpoint = "unix:///var/run/docker.sock"
```

```diff
- inputs.docker уже был добавлен в telegraf.conf
```

Дополнительно вам может потребоваться донастройка контейнера telegraf в `docker-compose.yml` дополнительного volume и 
режима privileged:
```
  telegraf:
    image: telegraf:1.4.0
    privileged: true
    volumes:
      - ./etc/telegraf.conf:/etc/telegraf/telegraf.conf:Z
      - /var/run/docker.sock:/var/run/docker.sock:Z
    links:
      - influxdb
    ports:
      - "8092:8092/udp"
      - "8094:8094"
      - "8125:8125/udp"
```

После настройке перезапустите telegraf, обновите веб интерфейс и приведите скриншотом список `measurments` в 
веб-интерфейсе базы telegraf.autogen . Там должны появиться метрики, связанные с docker.

Факультативно можете изучить какие метрики собирает telegraf после выполнения данного задания.

### Решение  

Скрин:  
![alt text](https://github.com/rdegtyarev/mnt-10-02/blob/main/images/5.png)
