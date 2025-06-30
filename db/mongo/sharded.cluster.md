# Create Shareded Cluster

mongodb sharded cluster 구성 방법 정리

구성 내용:

- mongos
- config server PSA replicat set
- shard1 server primary only replicat set
- shard2 server primary only replicat set

## 구성 중 주의 사항

config/shard server는 각각 replica set을 구성해야 한다.  
replica set 구성은 각 replica set의 primary server에서 설정 명령을 실행해야 한다.

mongos에 각 shard replica를 추가한다.  
shard 추가 명령은 mongos에서 실행한다.  
mongos에서 shard 구성시 메타 데이터는 config server admin db에 저장된다.

sharded cluster 구성 중 임의로 각 서버 process가 종료되면 안된다.
구성 중 정상 설정이 완료 되기 전에 process가 종료되면 데이터가 정상 설정 되지 못하게 되며, 이후 복구 방법은 없다.

sharded cluster는 각 서버 process의 시작 및 종료시 순서가 매우 중요하다.
작업 중 임의로 config server나 shard 서버가 종료되는 경우 복구 할 수 없는 오류가 발생할 가능성이 매우 높다.

## sharded cluster 설정 순서

- cluster 구성을 위한 모든 mongod/mongos 서버 실행
- config server replica set 구성
- shard server replica set 구성
- mongos에 shard 서버 추가
- 데이터 저장용 db 생성
- database에 shard enabling

## docker-compose.yml

```yaml
services:
  # Config Servers (replica set: csrs)
  mongo-cluster-config1:
    image: mongo:7
    command: mongod --configsvr --replSet csrs --port 27019
    container_name: mongo-cluster-config1
    networks:
      - mongo-cluster

  mongo-cluster-config2:
    image: mongo:7
    command: mongod --configsvr --replSet csrs --port 27019
    container_name: mongo-cluster-config2
    networks:
      - mongo-cluster

  mongo-cluster-config3:
    image: mongo:7
    command: mongod --configsvr --replSet csrs --port 27019
    container_name: mongo-cluster-config3
    networks:
      - mongo-cluster

  # Shard 1
  mongo-cluster-shard1:
    image: mongo:7
    command: mongod --shardsvr --replSet shard1 --port 27018
    container_name: mongo-cluster-shard1
    networks:
      - mongo-cluster

  # Shard 2
  mongo-cluster-shard2:
    image: mongo:7
    command: mongod --shardsvr --replSet shard2 --port 27018
    container_name: mongo-cluster-shard2
    networks:
      - mongo-cluster

  # mongos
  mongo-cluster-mongos:
    image: mongo:7
    depends_on:
      - mongo-cluster-config1
      - mongo-cluster-config2
      - mongo-cluster-config3
      - mongo-cluster-shard1
      - mongo-cluster-shard2
    command: >
      mongos --configdb csrs/mongo-cluster-config1:27019,mongo-cluster-config2:27019,mongo-cluster-config3:27019 --port 27017
    container_name: mongo-cluster-mongos
    ports:
      - "27017:27017"
    networks:
      - mongo-cluster

networks:
  mongo-cluster:
    driver: bridge
```

## config server replica set 구성

config server replica set(csrs) 생성 설정

```bash
docker exec -it mongo-cluster-config1 mongosh --port 27019
```

```js
rs.initiate({
  _id: "csrs",
  configsvr: true,
  members: [
    { _id: 0, host: "mongo-cluster-config1:27019" },
    { _id: 1, host: "mongo-cluster-config2:27019" },
    { _id: 2, host: "mongo-cluster-config3:27019" }
  ]
})
```

## shard1 server replica set 구성

shard1 server replica set(shard1) 생성 설정

```bash
docker exec -it mongo-cluster-shard1 mongosh --port 27018
```

```js
rs.initiate({
  _id: "shard1",
  members: [
    { _id: 0, host: "mongo-cluster-shard1:27018" }
  ]
})
```

## shard2 server replica set 구성

shard2 server replica set(shard2) 생성 설정

```bash
docker exec -it mongo-cluster-shard2 mongosh --port 27018
```

```js
rs.initiate({
  _id: "shard2",
  members: [
    { _id: 0, host: "mongo-cluster-shard2:27018" }
  ]
})
```

## mongos 에 shard server 추가

```bash
docker exec -it mongo-cluster-mongos mongosh --port 27017
```

```js
sh.addShard("shard1/mongo-cluster-shard1:27018")
sh.addShard("shard2/mongo-cluster-shard2:27018")
```
