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
sh.shardCollection("somedb.helloDoc", { "name" : "hashed" } );

use somedb;
for(var i = 0; i < 1000; i++) db.helloDoc.insert({age:i, name:"my"+i});
exit();

docker exec -it redis_1 /bin/sh
redis-cli cluster nodes

Будет:
188a3b4dbad37664c6cc83eee372e7cbd36d41a4 173.17.0.6:6379@16379 slave 8fd7af63fe7a6bbac0fa9cf8c069f3a9a7b8b7e0 0 1726778883455 1 connected
42a2f21b0f2b2897efff1e46f3c7b86c1a73f437 173.17.0.7:6379@16379 slave 45fa22bd16e30f6ba7e91c0cf983fd16fc129a3a 0 1726778884962 2 connected
f2d72c24758eccca758885168bc708d07b737e2b 173.17.0.5:6379@16379 slave 91f3daa5add1997a47be15099dbfaefe9789da2e 0 1726778883000 3 connected
8fd7af63fe7a6bbac0fa9cf8c069f3a9a7b8b7e0 173.17.0.2:6379@16379 myself,master - 0 0 1 connected 0-5460
91f3daa5add1997a47be15099dbfaefe9789da2e 173.17.0.4:6379@16379 master - 0 1726778884000 3 connected 10923-16383
45fa22bd16e30f6ba7e91c0cf983fd16fc129a3a 173.17.0.3:6379@16379 master - 0 1726778884460 2 connected 5461-10922
