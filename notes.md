# CD with debezium & Kafk, Prostgres

## PostgreSQL
```bash
docker exec -ti cdc_postgres /bin/bash

cat /var/lib/postgresql/data/postgresql.conf

pgcli -h localhost -p 5432 -U start_data_engineer

insert into bank.holding values (1000, 1, 'VFIAX', 10, now(), now());
```


## Debezium
```bash
curl -H "Accept:application/json" localhost:8083/connectors/

curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" \
localhost:8083/connectors/ -d '
{
  "name": "cdc-connector",
  "config": {
    "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
    "database.hostname": "postgres",
    "database.dbname": "postgres",
    "database.port": "5432",
    "database.user": "cdc_user",
    "database.password": "cdc_password",
    "table.whitelist": "bank.holding",
    "topic.prefix": "bankholding"
  }
}'
```

## Check Kafka Consumer
```bash
# List topics
docker run -it --rm --name cdc_consumer --link zookeeper:zookeeper --link kafka:kafka debezium/kafka:2.1 list-topics

# Read 1 message from topic
docker run -it --rm --name cdc_consumer --link zookeeper:zookeeper --link kafka:kafka debezium/kafka:2.1 watch-topic -a bankholding.bank.holding --max-messages 1
```