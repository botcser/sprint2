Проект уже настроен и запускается. Конфиг файл redis уже заложен. Кластер создается в compose сам.

docker compose up -d

#docker exec -it configSrv mongosh --port 27017

> rs.initiate(
  {
    _id : "config_server",
       configsvr: true,
    members: [
      { _id : 0, host : "configSrv:27017" }
    ]
  }
);
> exit();

redis-cli cluster nodes
