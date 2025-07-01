# ElasticSearch: installation with Docker Compose

- [ElasticSearch: installation with Docker Compose](#elasticsearch-installation-with-docker-compose)
  - [refs](#refs)
  - [Troubleshooting Virtual Memory misconfiguration](#troubleshooting-virtual-memory-misconfiguration)
    - [how to fix](#how-to-fix)
  - [Check alive elasticsearch using certs](#check-alive-elasticsearch-using-certs)

---

## refs

- <https://www.elastic.co/blog/getting-started-with-the-elastic-stack-and-docker-compose>
- <https://github.com/elkninja/elastic-stack-docker-part-one>

## Troubleshooting Virtual Memory misconfiguration

```bash
$ sysctl vm.max_map_count
vm.max_map_count = 65530
```

`65530` is default and too low.

ElasticSearch node may can NOT be launched properly with below error.

```json
{"@timestamp":"2023-04-14T13:16:22.148Z", "log.level":"ERROR", "message":"node validation exception\n[1] bootstrap checks failed. You must address the points described in the following [1] lines before starting Elasticsearch.\nbootstrap check failure [1] of [1]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]", "ecs.version": "1.2.0","service.name":"ES_ECS","event.dataset":"elasticsearch.server","process.thread.name":"main","log.logger":"org.elasticsearch.bootstrap.Elasticsearch","elasticsearch.node.name":"es01","elasticsearch.cluster.name":"docker-cluster"}
```

### how to fix

```bash
$ sudo sysctl -w vm.max_map_count=262144
vm.max_map_count = 26214
```

## Check alive elasticsearch using certs

```bash
docker cp elastic-stack-docker-part-one-es01-1:/usr/share/elasticsearch/config/certs/ca/ca.crt /tmp/.

curl --cacert /tmp/ca.crt -u elastic:changeme https://localhost:9200
```
