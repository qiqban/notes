# Kafka

kafka_2.11-0.10.1.0.tgz


前提: 启动zookeeper

启动kafka:
bin/kafka-server-start.sh config/server.properties

create topic:
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test

list topics:
bin/kafka-topics.sh --list --zookeeper localhost:2181

修改分区:
bin/kafka-topics.sh --alter --zookeeper localhost:2181 --partitions 2 --topic test

描述topic:
bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic test

product messages:
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test

consume messages:
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
