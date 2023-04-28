# Start Kafka
```bash
bin/zookeeper-server-start.sh config/zookeeper.propertie

bin/kafka-server-start.sh config/server.properties
```

# Topic Management
```bash
bin/kafka-topics.sh --list --bootstrap-server localhost:9092

bin/kafka-topics.sh --create --topic quickstart-events --bootstrap-server localhost:9092

bin/kafka-topics.sh --describe quickstart-events --bootstrap-server localhost:9092
```

# Write Events
```bash
bin/kafka-console-producer.sh --topic quickstart-events --bootstrap-server localhost:9092
```

# Read Events
```bash
bin/kafka-console-consumer.sh --topic quickstart-events --from-beginning --bootstrap-server localhost:9092
```