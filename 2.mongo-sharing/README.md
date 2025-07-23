# Задание 2
Для запуска проекта выполняем следующие команды:

1) запуск контейнеров:

```shell
docker-compose -f compose.yaml up -d
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

фиг. 5 - количество записей в shard-2

Шардирование работает штатно
