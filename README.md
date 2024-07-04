```yaml
# This is only infra specific no need to add any micro-service here
version: "3.8"

services:
  zookeeper:
    image: zookeeper:3.5.6
    container_name: zookeeper
    ports:
      - "2181:2181"
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000 # 2sec used for heatbeat and timeouts
    networks:
      - bridge
    volumes:
      - zookeeper_data:/data
      - zookeeper_datalog:/datalog
  kafka:
    container_name: kafka
    image: docker.io/bitnami/kafka:latest
    ports:
      - "9092:9092"
      - "9093:9093"
    volumes:
      - kafka_data:/bitnami
    ulimits:
      nofile:
        soft: 256000
        hard: 256000
    networks:
      - bridge
    environment:
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181
      - ALLOW_PLAINTEXT_LISTENER=yes
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=INTERNAL:PLAINTEXT,EXTERNAL:PLAINTEXT # for internal and external client https://hub.docker.com/r/bitnami/kafka
      - KAFKA_CFG_LISTENERS=INTERNAL://:9092,EXTERNAL://:9093 # for internal and external client https://hub.docker.com/r/bitnami/kafka
      - KAFKA_CFG_ADVERTISED_LISTENERS=INTERNAL://kafka:9092,EXTERNAL://localhost:9093 # for internal and external client https://hub.docker.com/r/bitnami/kafka
      - KAFKA_INTER_BROKER_LISTENER_NAME=INTERNAL # for internal and external client https://hub.docker.com/r/bitnami/kafka
      - KAFKA_DELETE_TOPIC_ENABLE=true
      - "KAFKA_HEAP_OPTS=-Xmx1g -Xms1g"
    depends_on:
      - zookeeper

  kafka-ui:
    container_name: kafka-ui
    image: provectuslabs/kafka-ui:latest
    ports:
      - "28180:8080"
    networks:
      - bridge
    environment:
      - KAFKA_CLUSTERS_0_NAME=local
      - KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS=kafka:9092

volumes:
  kafka_data:
    driver: local
  zookeeper_data:
    driver: local
  zookeeper_datalog:
    driver: local
networks:
  bridge:

```
