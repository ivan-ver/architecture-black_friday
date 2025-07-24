# Задание 4

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
```shell
exit()
```
5) создание БД и документа:

```shell
docker exec -it mongos-router mongosh --port 27020
```
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
Для проверки скорости запроса и ответа использовал программу **bruno**. Выполнил запрос *GET http://localhost:8086/helloDoc/users*
Результат представлен на фиг. 9

![fig-9.png](img%2Ffig-9.png)

фиг. 9 - результат первого запроса

Время запроса/ответа составило **1,2 с**

Повторный запрос:

![fig-10.png](img%2Ffig-10.png)

фиг. 10 - результат второго запроса

Время запроса/ответа составило **8 мс**