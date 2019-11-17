# kafka cleanup
```bash
docker exec -it kafkabroker /bin/bash
cd /opt/bitnami/kafka/bin/

./kafka-topics.sh --list --zookeeper zookeeper:2181

./kafka-topics.sh --delete --zookeeper zookeeper:2181 --topic orders
./kafka-topics.sh --delete --zookeeper zookeeper:2181 --topic customers

# Delete the lake from S3
```