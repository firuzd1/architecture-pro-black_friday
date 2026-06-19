# mongo-sharding-repl

## Запуск

```shell
docker compose up -d
```

## Инициализация

### 1. Config server
```shell
docker compose exec configSrv mongosh --port 27017 --quiet --eval "rs.initiate({_id: 'config_server', configsvr: true, members: [{_id: 0, host: 'configSrv:27017'}]})"
```

### 2. Shard1 (3 реплики)
```shell
docker compose exec shard1-1 mongosh --port 27018 --quiet --eval "rs.initiate({_id: 'shard1', members: [{_id: 0, host: 'shard1-1:27018'}, {_id: 1, host: 'shard1-2:27018'}, {_id: 2, host: 'shard1-3:27018'}]})"
```

### 3. Shard2 (3 реплики)
```shell
docker compose exec shard2-1 mongosh --port 27019 --quiet --eval "rs.initiate({_id: 'shard2', members: [{_id: 0, host: 'shard2-1:27019'}, {_id: 1, host: 'shard2-2:27019'}, {_id: 2, host: 'shard2-3:27019'}]})"
```

### 4. Добавляем шарды в кластер
```shell
docker compose exec mongos_router mongosh --port 27020 --quiet --eval "sh.addShard('shard1/shard1-1:27018,shard1-2:27018,shard1-3:27018'); sh.addShard('shard2/shard2-1:27019,shard2-2:27019,shard2-3:27019')"
```

### 5. Включаем шардирование
```shell
docker compose exec mongos_router mongosh --port 27020 --quiet --eval "sh.enableSharding('somedb'); sh.shardCollection('somedb.helloDoc', {'name': 'hashed'})"
```

### 6. Заполняем базу
```shell
docker compose exec mongos_router mongosh --port 27020
```
В консоли mongosh:
```javascript
use somedb
for(let i=0; i<1000; i++){db.helloDoc.insertOne({name:'user_'+i, age:Math.floor(Math.random()*60)+18})}
db.helloDoc.countDocuments()
```