tasks 31 

from kafka import KafkaConsumer

consumer = KafkaConsumer('group-topic', bootstrap_servers=['localhost:9092'],
                        group_id='my-group')

for message in consumer:
    print(f'Consumer: {consumer.group_id}, Message: {message.value.decode("utf-8")}')

tasks 32 


import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.streams.KafkaStreams;
import org.apache.kafka.streams.StreamsBuilder;
import org.apache.kafka.streams.kstream.KStream;

import java.util.Properties;

public class KafkaStreamsExample {
    public static void main(String[] args) {
        Properties props = new Properties();
        props.put(StreamsConfig.APPLICATION_ID_CONFIG, "my-stream-app");
        props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass().getName());
        props.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass().getName());

        StreamsBuilder builder = new StreamsBuilder();
        KStream<String, String> stream = builder.stream("stream-topic");
        stream.mapValues(value -> "Processed: " + value)
              .to("processed-topic");

        KafkaStreams streams = new KafkaStreams(builder.build(), props);
        streams.start();

        Runtime.getRuntime().addShutdownHook(new Thread(streams::close));
    }
}

tasks 33 


import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaProducer;
import org.apache.flink.util.Collector;

public class KafkaFlinkWordCount {
    public static void main(String[] args) throws Exception {
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        // Kafka Source
        Properties props = new Properties();
        props.setProperty("bootstrap.servers", "localhost:9092");
        props.setProperty("group.id", "flink-consumer");
        DataStream<String> stream = env.addSource(new FlinkKafkaConsumer<>("flink-topic", new SimpleStringSchema(), props));

        // Word count
        DataStream<Tuple2<String, Integer>> wordCounts = stream.flatMap(new FlatMapFunction<String, Tuple2<String, Integer>>() {
            @Override
            public void flatMap(String value, Collector<Tuple2<String, Integer>> out) throws Exception {
                String[] words = value.split("\s+");
                for (String word : words) {
                    out.collect(new Tuple2<>(word, 1));
                }
            }
        })
        .keyBy(0)
        .sum(1);

        // Kafka Sink
        props.setProperty("bootstrap.servers", "localhost:9092");
        props.setProperty("group.id", "flink-producer");
        wordCounts.addSink(new FlinkKafkaProducer<>("flink-wordcount-topic", new SimpleStringSchema(), props));

        env.execute("Kafka Flink WordCount");
    }
}

tasks 34 

{
  "name": "kafka-s3-sink",
  "connector.class": "io.confluent.connect.s3.S3SinkConnector",
  "tasks.max": "1",
  "topic": "my-topic",
  "s3.region": "us-east-1",
  "s3.bucket.name": "my-bucket",
  "s3.path.prefix": "data/",
  "s3.access.key": "your_access_key",
  "s3.secret.key": "your_secret_key",
  "key.converter": "org.apache.kafka.connect.json.JsonConverter",
  "value.converter": "org.apache.kafka.connect.json.JsonConverter"
}

tasks 35 

curl -X POST "${KAFKA_CONNECT_HOST}/connectors" -H "Content-Type: application/json" -d '{ \
    "name": "my-new-connector", \
    "config": { \
      "connector.class": "io.confluent.connect.jdbc.JdbcSinkConnector", \
      "tasks.max": 1,
      "topics": "mysql-table01,mysql-table02", \
      "connection.url": "jdbc:postgresql://postgres:5432/catalog", \
      "connection.user": "postgres", \
      "connection.password": "postgres", \
      "auto.create": "true" \
    } \
  }'

tasks 36


{
  "name": "my-debezium-mysql-connector",
  "config": {
    "tasks.msx": 1,
    "connector.class": "io.debezium.connector.mysql.MySqlConnector",
    "database.hostname": "${MYSQL_HOST}",
    "database.serverTimezone": "Europe/Moscow",
    "database.port": "${MYSQL_PORT}",
    "database.user": "${MYSQL_USER}",
    "database.password": "${MYSQL_PASS}",
    "database.server.id": "223355",
    "database.server.name": "monolyth_db",
    "table.whitelist": "${MYSQL_DB}.table_name1",
    "database.history.kafka.bootstrap.servers": "${KAFKA_BROKER}",
    "database.history.kafka.topic": "monolyth_db.debezium.history",
    "database.history.skip.unparseable.ddl": true,
    "snapshot.mode": "initial",
    "time.precision.mode": "connect"
  }
}

  "database.hostname": "${MYSQL_HOST}",
    "database.serverTimezone": "Europe/Moscow",
    "database.port": "${MYSQL_PORT}",
    "database.user": "${MYSQL_USER}",
    "database.password": "${MYSQL_PASS}",

    tasks 37 

    CREATE TABLE orders (
	order_id SERIAL PRIMARY KEY,
	status VARCHAR(10) NOT NULL,
	notes VARCHAR(255) NOT NULL,
	created_on TIMESTAMP NOT NULL,
	last_modified TIMESTAMP
);

INSERT INTO "orders"
	("order_id", "status", "notes", "created_on", "last_modified")
VALUES
	(1, 'PENDING', '', now(), NULL),
	(2, 'ACCEPTED', 'Some notes', now(), now());

 tasks 38 


   zookeeper:
    image: confluentinc/cp-zookeeper:6.2.1
    volumes:
      - zookeeper-data:/var/lib/zookeeper/data:Z
      - zookeeper-log:/var/lib/zookeeper/log:Z
    environment:
      ZOOKEEPER_CLIENT_PORT: '2181'
      ZOOKEEPER_ADMIN_ENABLE_SERVER: 'false'

  kafka:
    image: confluentinc/cp-kafka:6.2.1
    volumes:
      - kafka-data:/var/lib/kafka/data:Z
    ports:
      - 9091:9091
    environment:
      KAFKA_BROKER_ID: '0'
      KAFKA_ZOOKEEPER_CONNECT: 'zookeeper:2181'
      KAFKA_NUM_PARTITIONS: '12'
      KAFKA_COMPRESSION_TYPE: 'gzip'
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: '1'
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: '1'
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: '1'
      KAFKA_ADVERTISED_LISTENERS: 'PLAINTEXT://kafka:9092'
      KAFKA_CONFLUENT_SUPPORT_METRICS_ENABLE: 'false'
      KAFKA_JMX_PORT: '9091'
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: 'true'
      KAFKA_AUTHORIZER_CLASS_NAME: 'kafka.security.auth.SimpleAclAuthorizer'
      KAFKA_ALLOW_EVERYONE_IF_NO_ACL_FOUND: 'true'
    links:
      - zookeeper


      tasks 39 



      name=jdbc-source-orders
connector.class=io.confluent.connect.jdbc.JdbcSourceConnector
connection.url=jdbc:postgresql://db:5432/orders
connection.user=postgres
connection.password=${file:/etc/kafka-connect/kafka-connect.properties:jdbc.source.orders.password}
incrementing.column.name=order_id
mode=incrementing
table.whitelist=orders
topic.prefix=connect.
transforms=createKeyStruct,extractStructValue,addNamespace


tasks 40 

{
    "schema": { // 1 Схема значения
        "type": "struct",
        "fields": [
            {
                "type": "struct",
                "fields": [
{
"type": "int32",
"optional": false,
"field": "id"
},
{
"type": "string",
"optional": false,
 "field": "first_name"
},
{
"type": "string",
"optional": false,
"field": "last_name"
},
{
"type": "string",
"optional": false,
field": "email"
}
],
                "optional": true,
                "name": "PostgreSQL_server.inventory.customers.Value", // 2 Название схемы для полей before и after
                "field": "before"
            },
            {
                "type": "struct",
                "fields": [
{
"type": "int32",
"optional": false,
"field": "id"
},
{
"type": "string",
"optional": false,
"field": "first_name"
},
{
"type": "string",
"optional": false,
"field": "last_name"
},
{
"type": "string",
"optional": false,
"field": "email"
}
],
                "optional": true,
                "name": "PostgreSQL_server.inventory.customers.Value",
                "field": "after"
            },
            {
                "type": "struct",
                "fields": [
{
type": "string",
"optional": false,
"field": "version"
},
{"type": "string",
optional": false,
"field": "connector"
},
{
                        "type": "string",
                        "optional": false,
                        "field": "name"
                    },
                    {
                        "type": "int64",
                        "optional": false,
                        "field": "ts_ms"
                    },
                    {
                        "type": "boolean",
                        "optional": true,
                        "default": false,
                        "field": "snapshot"

{
}
                        "type": "string",
                        "optional": false,
                        "field": "db"
                    },
                    {
                        "type": "string",
                        "optional": false,
                        "field": "schema"
                    },
                    {
                        "type": "string",
                        "optional": false,
                        "field": "table"
                    },
                    {
                        "type": "int64",
                        "optional": true,
"field": "txId"
},
{
type": "int64",
"ooptional": true,
field": "lsn"
},
{
"type": "int64",
"optional": true,
field": "xmin"
}
],
                "optional": false,
                "name": "io.debezium.connector.postgresql.Source", // 3 Схема поля source . Специфична для PostgreSQL коннектора.
                "field": "source"
            },
            {
                "type": "string",
                "optional": false,
                "field": "op"
            },
            {
                "type": "int64",
                "optional": true,
                "field": "ts_ms"
            }
        ],
        "optional": false,
        "name": "PostgreSQL_server.inventory.customers.Envelope" // 4 Схема всей структуры payload-a. 
    },
    "payload": { // 5 Сами данные value - информация предоставляемая change event.
        "before": null, // 6 Опциональное поле, характеризующее состояние данных в БД до того, как событие произошло.
        "after": { // 7 Опциональное поле, характеризующее состояние данных в БД после того, как событие произошло
            "id": 1,
            "first_name": "Anne",
            "last_name": "Kretchmar",
            "email": "annek@noanswer.org"
        },
        "source": { // 8 Обязательное поле, которое характеризует источник метаданных о событии.
            "version": "2.4.1.Final", // версия Debezium
            "connector": "postgresql", // название коннектора 
            "name": "PostgreSQL_server", // имя сервера Postgres
            "ts_ms": 1559033904863, // Время события в БД 
            "snapshot": true, // Является ли событие частью снепшота 
            "db": "postgres", // название базы
            "sequence": "[\"24023119\",\"24023128\"]", //  приведенный к строке массив JSON с доп информацией об оффсетах. Первое значение - последний закомиченный оффсет LSN. Второе - текущий оффсет LSN. 
            "schema": "public", // название схемы
            "table": "customers", // таблица, содержащая новую запись 
            "txId": 555, // ID транзакции, в которой произошла операция 
            "lsn": 24023128, // оффсет операции в базе данных (LSN) 
            "xmin": null
        },
        "op": "c", // 9 Обязательное поле, характеризующее тип операции: с = create, u = update, d = delete, r = read (applies only to snapshots), t = truncate, m = message
        "ts_ms": 1559033904863 // 10 Опциональное поле, характеризующее время обработки события коннектором Debezium
    }
















  
