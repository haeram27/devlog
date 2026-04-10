# DB 접속 command

## postgres

```bash
docker exec -it postgres-container psql -P pager=off  -p 5432 -d cppoltp
docker exec -it postgres-container psql -P pager=off  -p 5432 -d cppoltp -f any.sql
docker exec -it postgres-container psql -P pager=off  -p 5432 -d cppoltp -c '<sql stmt>'
docker exec -it postgres-container psql -P pager=off  -p 5432 -d cppoltp -c 'SELECT * FROM pg_catalog.pg_tables;'
```

## mongodb

```bash
mongosh mongodb://id:pass@1.1.1.1:27017/dbname
mongosh mongodb://id:pass@1.1.1.1:27017/?authMechanism=DEFAULT&authSource=dbname
mongosh mongodb+srv://id:pass@hostname/?authMechanism=DEFAULT&authSource=dbname
mongosh mongodb://1.1.1.1:27017 --authenticationDatabase=admin -u adminUser -p adminPass --eval 'sample_airbnb.listingsAndReviews.find().batchSize(-1).limit(10)'
```

## redis

```bash
redis-cli -h 127.0.0.1 -p 6379 -a password

## with command
redis-cli -h 127.0.0.1 -p 6379 -a password get mykey
echo -e "set key1 val1\nget key1\ninfo" | redis-cli
```

## kafka

|스크립트 이름|역할|
|---|---|
|kafka-configs.sh|브로커 상태 및 설정 관리|
|kafka-topics.sh|토픽 관리|
|kafka-console-producer.sh|메세지 생성|
|kafka-console-consumer.sh|메세지 소비|
|kafka-consumer-groups.sh|컨슈머 그룹 상태 확인|

### 1. 토픽 관리 (kafka-topics.sh)
토픽을 생성, 조회, 수정, 삭제할 때 사용합니다.

- 토픽 리스트 확인:

```bash
bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
```

- 토픽 생성:

```bash
bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --topic my-topic --partitions 3 --replication-factor 1
```

### 2. 메시지 생성 (kafka-console-producer.sh)
특정 토픽으로 메시지를 직접 전송(Produce)할 때 사용합니다.

```bash
# 실행 후 입력창이 뜨면 메시지를 입력하고 Enter를 누르세요.
bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic my-topic
```

### 3. 메시지 소비 (kafka-console-consumer.sh)

토픽에 저장된 메시지를 실시간으로 확인(Consume)할 때 사용합니다.

- 처음부터 모든 메시지를 다 읽어오고 싶을 때 (--from-beginning)

```bash
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic my-topic --from-beginning
```