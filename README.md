# Run Kafka Cluster

`docker compose up --build`

## Healthcheck

Each Kafka container binds `PLAINTEXT://0.0.0.0:9092` internally.  
`nc -z localhost 9092` confirms the broker socket is listening,  
which happens ~30-60s after container start (KRAFT cluster formation time).

# Kafka CLI

You can use any broker as bootstrap broker to access the cluster:

```bash
docker compose exec kafka1 bash
```

```bash
/opt/kafka/bin/kafka-topics.sh --list --bootstrap-server localhost:9092
/opt/kafka/bin/kafka-topics.sh --create --topic cluster-test1 --bootstrap-server localhost:9092
/opt/kafka/bin/kafka-topics.sh --describe --topic cluster-test1 --bootstrap-server localhost:9092
```

Default partitions and replicas for topics defined in `KAFKA_NUM_PARTITIONS` and `KAFKA_DEFAULT_REPLICATION_FACTOR`.

**Create another topic with 6 partitions and 2 replicas:**
```bash
/opt/kafka/bin/kafka-topics.sh --create --topic cluster-test2 --partitions 6 --replication-factor 2 --bootstrap-server localhost:9092
```

| Topic        | Partition | Leader | Replicas | ISR  |
|--------------|-----------|--------|----------|------|
| cluster-test2| 0         | 2      | 2,3      | 2,3  |
| cluster-test2| 1         | 3      | 3,1      | 3,1  |
| cluster-test2| 2         | 1      | 1,2      | 1,2  |
| cluster-test2| 3         | 3      | 3,2      | 3,2  |
| cluster-test2| 4         | 2      | 2,1      | 2,1  |
| cluster-test2| 5         | 1      | 1,3      | 1,3  |

1 topic with 6 partitions and a different leader for each partition plus indication of where is the second replica apart the one on the broker

**Get offsets in each partition**
```bash
/opt/kafka/bin/kafka-get-offsets.sh --topic cluster-test2 --bootstrap-server localhost:9092
```
| Topic-Partition-Offset |
|------------------------|
| cluster-test2:0:0      |
| cluster-test2:1:0      |
| cluster-test2:2:0      |
| cluster-test2:3:0      |
| cluster-test2:4:0      |
| cluster-test2:5:0      |

offset always 0 as there are no msg

**Create messages**
```bash
/opt/kafka/bin/kafka-console-producer.sh --topic cluster-test2 --bootstrap-server localhost:9092

hello world
kafka testing
another message
```

| Topic-Partition-Offset |
|------------------------|
| cluster-test2:0:2      |
| cluster-test2:1:1      |
| cluster-test2:2:0      |
| cluster-test2:3:0      |
| cluster-test2:4:0      |
| cluster-test2:5:0      |

Without key, Kafka assigns the message to partitions in a round-robin
this distributes messages evenly across partitions for load balancing but does not guarantee message order across partitions.

**Read from a specific partition (in another broker)**
`docker compose exec kafka2 bash`

```bash
/opt/kafka/bin/kafka-console-consumer.sh --topic cluster-test2 --partition 1 --offset earliest --bootstrap-server localhost:9092
/opt/kafka/bin/kafka-console-consumer.sh --topic cluster-test2 --partition 0 --offset 1 --bootstrap-server localhost:9092 -- kafka testing
```
