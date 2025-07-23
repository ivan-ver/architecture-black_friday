# Задание 3

Измененное описание контейнеров: [compose.yaml](compose.yaml)

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