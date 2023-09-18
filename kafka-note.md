
docker pull wurstmeister/zookeeper
docker run -d --name zookeeper -p 2181:2181 -t wurstmeister/zookeeper
docker pull wurstmeister/kafka

docker run -d --name kafka0 -p 9092:9092 -e KAFKA_BROKER_ID=0 -e KAFKA_ZOOKEEPER_CONNECT=1.94.20.103:2181 -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://1.94.20.103:9092 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 -t wurstmeister/kafka
docker run -d --name kafka1 -p 9093:9093 -e KAFKA_BROKER_ID=1 -e KAFKA_ZOOKEEPER_CONNECT=1.94.20.103:2181 -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://1.94.20.103:9093 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9093 -t wurstmeister/kafka
docker run -d --name kafka2 -p 9094:9094 -e KAFKA_BROKER_ID=2 -e KAFKA_ZOOKEEPER_CONNECT=1.94.20.103:2181 -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://1.94.20.103:9094 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9094 -t wurstmeister/kafka




bin/kafka-topics.sh --zookeeper 1.94.20.103:2181 --create --topic topic-test --partitions 3 --replication-factor 3

bin/kafka-topics.sh --zookeeper 1.94.20.103:2181 --list

bin/kafka-topics.sh --zookeeper 1.94.20.103:2181 --describe --topic topic-test

bin/kafka-console-consume.sh --bootstrap-server 1.94.20.103:9094 --topic topic-test





docker stop zookeeper
docker rm zookeeper
docker run -d --name zookeeper -p 2181:2181 -t wurstmeister/zookeeper
docker stop kafka0 kafka1 kafka2
docker rm kafka0 kafka1 kafka2
docker run -d --name kafka0 -p 9092:9092 -e KAFKA_BROKER_ID=0 -e KAFKA_ZOOKEEPER_CONNECT=1.94.20.103:2181 -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://1.94.20.103:9092 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 -t wurstmeister/kafka
docker run -d --name kafka1 -p 9093:9093 -e KAFKA_BROKER_ID=1 -e KAFKA_ZOOKEEPER_CONNECT=1.94.20.103:2181 -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://1.94.20.103:9093 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9093 -t wurstmeister/kafka
docker run -d --name kafka2 -p 9094:9094 -e KAFKA_BROKER_ID=2 -e KAFKA_ZOOKEEPER_CONNECT=1.94.20.103:2181 -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://1.94.20.103:9094 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9094 -t wurstmeister/kafka