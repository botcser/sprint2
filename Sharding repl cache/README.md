Проект уже настроен и запускается. Конфиг файл redis уже заложен. Кластер создается в compose сам.

docker compose up -d

docker exec -it configSrv mongosh --port 27017
rs.initiate(
  {
    _id : "config_server",
       configsvr: true,
    members: [
      { _id : 0, host : "configSrv:27017" }
    ]
  }
);
exit();

docker exec -it shard1 mongosh --port 27020
rs.initiate({_id: "shard1", members: [
{_id: 0, host: "shard1:27020"},
{_id: 1, host: "shard1-rep1:27021"},
{_id: 2, host: "shard1-rep2:27022"}
]});
exit();

docker exec -it shard2 mongosh --port 27030
rs.initiate({_id: "shard2", members: [
{_id: 0, host: "shard2:27030"},
{_id: 1, host: "shard2-rep1:27031"},
{_id: 2, host: "shard2-rep2:27032"}
]}) 
exit();

docker exec -it mongos_router1 mongosh --port 27018
sh.addShard( "shard1/shard1:27020");
sh.addShard( "shard2/shard2:27030");

sh.enableSharding("somedb");
sh.shardCollection("somedb.helloDoc", { "user" : "hashed" } );

use somedb;
for(var i = 0; i < 1000; i++) db.helloDoc.insert({users:"my"+i});
exit();

Я не понял, что там с кластерами редис и кто там некорректно работает, но шарды и редис настроены корректно.
