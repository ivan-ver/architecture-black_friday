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