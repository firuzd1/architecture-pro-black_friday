# architecture-pro-black_friday

## Структура репозитория

- `mongo-sharding` — шардирование MongoDB (задание 2)
- `mongo-sharding-repl` — шардирование + репликация (задание 3)
- `sharding-repl-cache` — шардирование + репликация + кеширование Redis (задание 4)
- `schema.drawio` — итоговые схемы архитектуры (задания 1, 5, 6)

## Как запустить финальный стенд (sharding-repl-cache)

### 1. Запуск
```shell
cd sharding-repl-cache
docker compose up -d
```

### 2. Инициализация config server
```shell
docker compose exec configSrv mongosh --port 27017 --quiet --eval "rs.initiate({_id: 'config_server', configsvr: true, members: [{_id: 0, host: 'configSrv:27017'}]})"
```

### 3. Инициализация shard1
```shell
docker compose exec shard1-1 mongosh --port 27018 --quiet --eval "rs.initiate({_id: 'shard1', members: [{_id: 0, host: 'shard1-1:27018'}, {_id: 1, host: 'shard1-2:27018'}, {_id: 2, host: 'shard1-3:27018'}]})"
```

### 4. Инициализация shard2
```shell
docker compose exec shard2-1 mongosh --port 27019 --quiet --eval "rs.initiate({_id: 'shard2', members: [{_id: 0, host: 'shard2-1:27019'}, {_id: 1, host: 'shard2-2:27019'}, {_id: 2, host: 'shard2-3:27019'}]})"
```

### 5. Добавление шардов в кластер
```shell
docker compose exec mongos_router mongosh --port 27020 --quiet --eval "sh.addShard('shard1/shard1-1:27018,shard1-2:27018,shard1-3:27018'); sh.addShard('shard2/shard2-1:27019,shard2-2:27019,shard2-3:27019')"
```

### 6. Включение шардирования
```shell
docker compose exec mongos_router mongosh --port 27020 --quiet --eval "sh.enableSharding('somedb'); sh.shardCollection('somedb.helloDoc', {'name': 'hashed'})"
```

### 7. Заполнение базы данными
```shell
docker compose exec mongos_router mongosh --port 27020
```
В консоли mongosh:
```javascript
use somedb
for(let i=0; i<1000; i++){db.helloDoc.insertOne({name:'user_'+i, age:Math.floor(Math.random()*60)+18})}
db.helloDoc.countDocuments()
```

## Как проверить

Открыть в браузере http://localhost:8080

Проверка кеша: http://localhost:8080/helloDoc/users — второй запрос выполняется менее 100мс.