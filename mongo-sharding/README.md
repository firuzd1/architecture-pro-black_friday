# mongo-sharding

## Запуск

```shell
docker compose up -d
```

## Инициализация шардирования

### 1. Config server
```shell
docker compose exec configSrv mongosh --port 27017 --quiet --eval "rs.initiate({_id: 'config_server', configsvr: true, members: [{_id: 0, host: 'configSrv:27017'}]})"
```

### 2. Shard1
```shell
docker compose exec shard1 mongosh --port 27018 --quiet --eval "rs.initiate({_id: 'shard1', members: [{_id: 0, host: 'shard1:27018'}]})"
```

### 3. Shard2
```shell
docker compose exec shard2 mongosh --port 27019 --quiet --eval "rs.initiate({_id: 'shard2', members: [{_id: 0, host: 'shard2:27019'}]})"
```

### 4. Добавляем шарды и включаем шардирование
```shell
docker compose exec mongos_router mongosh --port 27020 --quiet --eval "sh.addShard('shard1/shard1:27018'); sh.addShard('shard2/shard2:27019'); sh.enableSharding('somedb'); sh.shardCollection('somedb.helloDoc', {'name': 'hashed'})"
```

### 5. Заполняем базу данными
```shell
docker compose exec mongos_router mongosh --port 27020
```
В консоли mongosh:
```javascript
use somedb
for(let i=0; i<1000; i++){db.helloDoc.insertOne({name:'user_'+i, age:Math.floor(Math.random()*60)+18})}
db.helloDoc.countDocuments()
```

### 6. Проверяем количество на каждом шарде
```shell
docker compose exec mongos_router mongosh --port 27020
```
В консоли mongosh:
```javascript
use somedb
db.helloDoc.getShardDistribution()
```