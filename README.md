Installation

Pull the images
```
docker pull debezium/zookeeper:1.2
```

Watch some logs in terminals
```
watch docker stack ps --no-trunc debezium
docker service logs -f debezium_zookeeper
docker service logs -f debezium_kafka
docker service logs -f debezium_connect
```

Create the project directory
```
mkdir -p ~/dev/docker/projects/debezium;
```

Create the volume directories
```
mkdir -p ~/dev/docker/projects/debezium/volumes/zookeeper/data;
mkdir -p ~/dev/docker/projects/debezium/volumes/zookeeper/txns;
mkdir -p ~/dev/docker/projects/debezium/volumes/zookeeper/conf;
mkdir -p ~/dev/docker/projects/debezium/volumes/zookeeper/logs;
mkdir -p ~/dev/docker/projects/debezium/volumes/kafka/data;
mkdir -p ~/dev/docker/projects/debezium/volumes/kafka/logs;
mkdir -p ~/dev/docker/projects/debezium/volumes/connect/logs;
mkdir -p ~/dev/docker/projects/debezium/volumes/connect/config;
```

Viewing the Kafka server log
```
tail -400f volumes/kafka/logs/server.log
```

Cheking the status of Kafka Connect
```
curl -H "Accept:application/json" localhost:8083
```

Deploying the MySQL connector
```
curl -i -X POST -H "Accept:application/json" \
  -H "Content-Type:application/json" localhost:8083/connectors/ \
  -d '{
  "name": "useraccount-connector",
  "config": {
    "connector.class": "io.debezium.connector.mysql.MySqlConnector",
    "tasks.max": "1",
    "database.hostname": "mysql",
    "database.port": "3306",
    "database.user": "debezium",
    "database.password": "dbz",
    "database.server.id": "1234",
    "database.server.name": "mylocaldb",
    "database.whitelist": "useraccount",
    "table.whitelist": "useraccount.user_account,useraccount.user_role",
    "database.history.kafka.bootstrap.servers": "kafka:9092",
    "database.history.kafka.topic": "schema-changes.useraccount",
    "database.history.skip.unparseable.ddl": "true"
  }
}'
```

Getting the list of connectors of Kafka Connect
```
curl -H "Accept:application/json" localhost:8083/connectors/
```

Reviewing the connector tasks
```
curl -i -X GET -H "Accept:application/json" localhost:8083/connectors/useraccount-connector
```

Watching a topic
```  
docker run -it --rm --name watcher -e ZOOKEEPER_CONNECT=zookeeper:2181 -e KAFKA_BROKER=kafka:9092 --network common debezium/kafka:1.2 watch-topic -a -k mylocaldb.useraccount.user_role
```  

See
https://debezium.io/documentation/reference/1.3/tutorial.html
https://rmoff.net/2018/08/02/kafka-listeners-explained/
https://github.com/devshawn/kafka-connect-healthcheck

Start the services
```  
cd ~/dev/docker/projects/debezium
docker stack deploy --compose-file docker-compose-dev.yml debezium
```

Stopping the common services
```  
docker stack rm debezium
```
