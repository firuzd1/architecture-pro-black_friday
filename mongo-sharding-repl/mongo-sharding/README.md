\# mongo-sharding



\## Запуск



```shell

docker compose up -d

```



\## Инициализация шардирования



\### 1. Инициализируем config server



```shell

docker compose exec -T configSrv mongosh --port 27017 --quiet <<EOF

rs.initiate({

&#x20; \_id: "config\_server",

&#x20; configsvr: true,

&#x20; members: \[{ \_id: 0, host: "configSrv:27017" }]

})

EOF

```



\### 2. Инициализируем shard1



```shell

docker compose exec -T shard1 mongosh --port 27018 --quiet <<EOF

rs.initiate({

&#x20; \_id: "shard1",

&#x20; members: \[{ \_id: 0, host: "shard1:27018" }]

})

EOF

```



\### 3. Инициализируем shard2



```shell

docker compose exec -T shard2 mongosh --port 27019 --quiet <<EOF

rs.initiate({

&#x20; \_id: "shard2",

&#x20; members: \[{ \_id: 0, host: "shard2:27019" }]

})

EOF

```



\### 4. Добавляем шарды в кластер через router



```shell

docker compose exec -T mongos\_router mongosh --port 27020 --quiet <<EOF

sh.addShard("shard1/shard1:27018")

sh.addShard("shard2/shard2:27019")

sh.enableSharding("somedb")

sh.shardCollection("somedb.helloDoc", { "name": "hashed" })

EOF

```



\### 5. Заполняем базу данными



```shell

docker compose exec -T mongos\_router mongosh --port 27020 --quiet <<EOF

use somedb

for (let i = 0; i < 1000; i++) {

&#x20; db.helloDoc.insertOne({ name: "user\_" + i, age: Math.floor(Math.random() \* 60) + 18 })

}

EOF

```



\### 6. Проверяем количество документов



```shell

docker compose exec -T mongos\_router mongosh --port 27020 --quiet <<EOF

use somedb

db.helloDoc.countDocuments()

EOF

```



```shell

docker compose exec -T shard1 mongosh --port 27018 --quiet <<EOF

use somedb

db.helloDoc.countDocuments()

EOF

```



```shell

docker compose exec -T shard2 mongosh --port 27019 --quiet <<EOF

use somedb

db.helloDoc.countDocuments()

EOF

```

