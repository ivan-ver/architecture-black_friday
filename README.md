# pymongo-api

## Как запустить

Запускаем mongodb и приложение

```shell
docker compose up -d
```

Заполняем mongodb данными

```shell
./scripts/mongo-init.sh
```

## Как проверить

### Если вы запускаете проект на локальной машине

Откройте в браузере http://localhost:8080

### Если вы запускаете проект на предоставленной виртуальной машине

Узнать белый ip виртуальной машины

```shell
curl --silent http://ifconfig.me
```

Откройте в браузере http://<ip виртуальной машины>:8080

## Доступные эндпоинты

Список доступных эндпоинтов, swagger http://<ip виртуальной машины>:8080/docs

# Решение:
## Задание 1

[схема вар-1](schemas%2Ftask-1%2Fschema_v1.drawio)

![schema_v1.png](schemas%2Ftask-1%2Fimg%2Fschema_v1.png)

[схема вар-2](schemas%2Ftask-1%2Fschema_v2.drawio)

![schema_v2.png](schemas%2Ftask-1%2Fimg%2Fschema_v2.png)

[схема вар-3](schemas%2Ftask-1%2Fschema_v3.drawio)

![schema_v3.png](schemas%2Ftask-1%2Fimg%2Fschema_v3.png)


## Задание 2
Для запуска проекта выполняем следующие команды:

1) запуск контейнеров:

```shell
docker-compose -f mongo-sharing.yaml up -d
```

2) инициализация сервера конфигурации:

```shell
docker exec -it config-srv mongosh --port 27017 
```

далее выполняется команда:

```shell
rs.initiate(
  {
    _id : "config-server",
       configsvr: true,
    members: [
      { _id : 0, host : "config-srv:27017" }
    ]
  }
);
```
```shell
exit();
```

3) инициализация шардов:

шард №1

```shell
docker exec -it shard-1 mongosh --port 27018
```
```shell
rs.initiate(
    {
      _id : "shard-1",
      members: [
        { _id : 0, host : "shard-1:27018" },
      ]
    }
);
```
```shell
exit()
```

шард №2

```shell
docker exec -it shard-2 mongosh --port 27019
```
```shell
rs.initiate(
    {
      _id : "shard-2",
      members: [
        { _id : 1, host : "shard-2:27019" }
      ]
    }
);
```
```shell
exit()
```

4) Инициализация роутера:

```shell
docker exec -it mongos-router mongosh --port 27020
```
```shell
sh.addShard( "shard-1/shard-1:27018");
sh.addShard( "shard-2/shard-2:27019");
```

5) создание БД и документа:

```shell
sh.enableSharding("somedb");
sh.shardCollection("somedb.helloDoc", { "name" : "hashed" } );
```

6) заполнение тестовыми данными:

```shell
use somedb;
for(var i = 0; i < 1000; i++) db.helloDoc.insert({age:i, name:"ly"+i});
```
```shell
exit()
```

ТЕСТИРОВАНИЕ РАБОТОСПОСОБНОСТИ

1) Проверка работы сервиса осуществляется через swagger:

![fig-1.png](img%2Ffig-1.png)

фиг. 1 - swagger-страница приложения api_app

При вызове:

```shell
curl -X 'GET' 'http://localhost:8086/' -H 'accept: application/json'
```

возвращается ответ

```json
{
  "mongo_topology_type": "Sharded",
  "mongo_replicaset_name": null,
  "mongo_db": "somedb",
  "read_preference": "Primary()",
  "mongo_nodes": [
    [
      "mongos-router",
      27020
    ]
  ],
  "mongo_primary_host": null,
  "mongo_secondary_hosts": [],
  "mongo_is_primary": true,
  "mongo_is_mongos": true,
  "collections": {
    "helloDoc": {
      "documents_count": 1002
    }
  },
  "shards": {
    "shard-1": "shard-1/shard-1:27018",
    "shard-2": "shard-2/shard-2:27019"
  },
  "cache_enabled": false,
  "status": "OK"
}
```

![fig-2.png](img%2Ffig-2.png)

фиг. 2 - результат выполнения запроса

Видно, что имеются два шарда (**shard-1**, **shard-2**), а общее количество записей **1002**.

2) Проверка успешности шардирования:

Проверка общего количества записей через команды:

```shell
docker exec -it mongos-router mongosh --port 27020
```
```shell
use somedb;
db.helloDoc.countDocuments();
```
Общее количество - **1002**

![fig-3.png](img%2Ffig-3.png)

фиг. 3 - общее количество записей в БД

Проверка количества записей в шарде **shard-1**

```shell
docker exec -it shard-1 mongosh --port 27018
```
```shell
use somedb;
db.helloDoc.countDocuments();
```

Количество **494**

![fig-4.png](img%2Ffig-4.png)

фиг. 4 - количество записей в shard-1


Проверка количества записей в шарде **shard-2**

```shell
docker exec -it shard-2 mongosh --port 27019
```
```shell
use somedb;
db.helloDoc.countDocuments();
```

Количество **508**

![fig-5.png](img%2Ffig-5.png)

фиг. 4 - количество записей в shard-2

Шардирование работает штатно

## Задание 3

Измененное описание контейнеров: [mongo-sharing-repl.yaml](mongo-sharing-repl.yaml)

Для запуска проекта выполняем следующие команды:

1) запуск контейнеров:

```shell
docker-compose -f mongo-sharing-repl.yaml up -d
```

2) инициализация сервера конфигурации:

```shell
docker exec -it config-srv mongosh --port 27017 
```

далее выполняется команда:

```shell
rs.initiate(
  {
    _id : "config-server",
       configsvr: true,
    members: [
      { _id : 0, host : "config-srv:27017" }
    ]
  }
);
```
```shell
exit();
```

3) инициализация шардов:

шард №1

```shell
docker exec -it shard-1-primary mongosh --port 27018
```
```shell
rs.initiate(
    {
      _id : "shard-1",
      members: [
        { _id: 0, host: "shard-1-primary:27018" },
        { _id: 1, host: "shard-1-replica-1:27018" },
        { _id: 2, host: "shard-1-replica-2:27018" }
      ]
    }
);
```
```shell
exit()
```

шард №2

```shell
docker exec -it shard-2-primary mongosh --port 27019
```
```shell
rs.initiate(
    {
      _id : "shard-2",
      members: [
        { _id: 0, host: "shard-2-primary:27019" },
        { _id: 1, host: "shard-2-replica-1:27019" },
        { _id: 2, host: "shard-2-replica-2:27019" }
      ]
    }
);
```
```shell
exit()
```

4) Инициализация роутера:

```shell
docker exec -it mongos-router mongosh --port 27020
```
```shell
sh.addShard("shard-1/shard-1-primary:27018,shard-1-replica-1:27018,shard-1-replica-2:27018");
sh.addShard("shard-2/shard-2-primary:27019,shard-2-replica-1:27019,shard-2-replica-2:27019");
```

6) создание БД и документа:

```shell
docker exec -it mongos-router mongosh --port 27020
```
```shell
sh.enableSharding("somedb");
sh.shardCollection("somedb.helloDoc", { "name" : "hashed" } );
```

7) заполнение тестовыми данными:

```shell
use somedb;
for(var i = 0; i < 1000; i++) db.helloDoc.insert({age:i, name:"ly"+i});
```
```shell
exit()
```

ТЕСТИРОВАНИЕ РАБОТОСПОСОБНОСТИ

1) Проверка работы сервиса осуществляется через swagger:
   
При вызове:

```shell
curl -X 'GET' 'http://localhost:8086/' -H 'accept: application/json'
```

возвращается ответ

```json
{
  "mongo_topology_type": "Sharded",
  "mongo_replicaset_name": null,
  "mongo_db": "somedb",
  "read_preference": "Primary()",
  "mongo_nodes": [
    [
      "mongos-router",
      27020
    ]
  ],
  "mongo_primary_host": null,
  "mongo_secondary_hosts": [],
  "mongo_is_primary": true,
  "mongo_is_mongos": true,
  "collections": {
    "helloDoc": {
      "documents_count": 1000
    }
  },
  "shards": {
    "shard-1": "shard-1/shard-1-primary:27018,shard-1-replica-1:27018,shard-1-replica-2:27018",
    "shard-2": "shard-2/shard-2-primary:27019,shard-2-replica-1:27019,shard-2-replica-2:27019"
  },
  "cache_enabled": false,
  "status": "OK"
}
```

![fig-6.png](img%2Ffig-6.png)

фиг. 6 - результат выполнения запроса

1) Проверка работоспособности реплицирования:

последовательно для всех трех реплик первого шарда выполнить команды:

```shell
docker exec -it <имя контейнера> mongosh --port <порт контейнера>
```
```shell
use somedb
db.helloDoc.countDocuments()
```

количество записей составляет **492**

![fig-7.png](img%2Ffig-7.png)

фиг. 7 - количество записей в репликах первого шарда


Аналогично для реплик ворого шарда. Количество составило **508**.

![fig-8.png](img%2Ffig-8.png)

фиг. 8 - количество записей в репликах второго шарда

## Задание 1

Обновленный файл контейнеров: [sharding-repl-cache.yaml](sharding-repl-cache.yaml)

Для проверки (если не останавливались контейнеры из предыдущего задания) необходимо остановить контейнер **pymongo_api**.

далее выполнить команды:

```shell
docker-compose -f sharding-repl-cache.yaml up redis -d 
```
```shell
docker-compose -f sharding-repl-cache.yaml up pymongo_api -d
```

Для проверки скорости запроса и ответа использовал программу **bruno**. Выполнил запрос *GET http://localhost:8086/helloDoc/users*
Результат представлен на фиг. 9

![fig-9.png](img%2Ffig-9.png)

фиг. 9 - результат первого запроса

Время запроса/ответа составило **1,2 с**

Повторный запрос:

![fig-10.png](img%2Ffig-10.png)

фиг. 9 - результат второго запроса

Время запроса/ответа составило **8 мс**