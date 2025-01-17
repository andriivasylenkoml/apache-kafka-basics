# apache-kafka-basics
A tutorial on Kafka to quickly learn how to work with it, gain fundamental skills and painlessly proceed to higher-level functionality in the future.

---

### **Launching Kafka**

Download Kafka archive:
```bash
wget https://archive.apache.org/dist/kafka/2.7.0/kafka_2.13-2.7.0.tgz
```

Unpack the archive:
```bash
tar -xzf kafka_2.13-2.7.0.tgz
```

Navigate to the Kafka directory:
```bash
cd kafka_2.13-2.7.0
```

#### **Starting ZooKeeper**
First, launch ZooKeeper. You will need the `./bin` folder, which contains scripts for starting brokers, configuring topics and partitions, and cluster reconfiguration. Run the script `zookeeper-server-start.sh`, passing it the configuration file `zookeeper.properties` from the `../config` folder.

Command:
```bash
./bin/zookeeper-server-start.sh config/zookeeper.properties
```

#### **Starting Kafka Broker**
Now start the Kafka broker. Similarly, in the `./bin` folder, run the `kafka-server-start.sh` script, passing it the configuration file `server.properties` from the `../config` folder.

Command:
```bash
./bin/kafka-server-start.sh config/server.properties
```

---

### **Writing and Reading Data**

#### **Creating a Topic**
To write data, you need to create a topic. Use the script `kafka-topics.sh` from the `./bin` folder, passing the `--create` option and the topic name, such as `--topic registrations`. Additionally, you must specify `--bootstrap-server`, which connects the script to the Kafka broker. Since we have only one broker, we use `localhost` and Kafka's default port, `9092`.

Command:
```bash
./bin/kafka-topics.sh --create --topic registrations --bootstrap-server localhost:9092
```

#### **Describing the Topic**
To check the topic's details:
```bash
./bin/kafka-topics.sh --describe --topic registrations --bootstrap-server localhost:9092
```

#### **Producing Messages**
Kafka includes a console utility for writing messages called `kafka-console-producer.sh`. Pass the topic name and the `--bootstrap-server` option. Once ready, you can start writing messages like `Hello World!`.

Command:
```bash
./bin/kafka-console-producer.sh --topic registrations --bootstrap-server localhost:9092
```

Example input after running the command:
```plaintext
>Hello World!
>Hello Slurm!
```

#### **Consuming Messages**
To read messages, use the console consumer `kafka-console-consumer.sh`, specifying the topic and `--bootstrap-server`:

Command:
```bash
./bin/kafka-console-consumer.sh --topic registrations --bootstrap-server localhost:9092
```

If you want to read messages written **before** starting the consumer, you need to override the default configuration by adding `--consumer-property auto.offset.reset=earliest` or use the shortcut `--from-beginning`.

Command:
```bash
./bin/kafka-console-consumer.sh --topic registrations --bootstrap-server localhost:9092 --consumer-property auto.offset.reset=earliest
```

Or:
```bash
./bin/kafka-console-consumer.sh --topic registrations --bootstrap-server localhost:9092 --from-beginning
```

#### **Using Consumer Groups**
You can assign a specific consumer group using the `--group` option. For example:
```bash
./bin/kafka-console-consumer.sh --topic registrations --bootstrap-server localhost:9092 --consumer-property auto.offset.reset=earliest --group slurm
```

#### **Viewing Consumer Group State**
To check the state of the consumer group:
```bash
./bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group slurm --describe
```

#### **Resetting Consumer Group Offset**
To reset the consumer group’s offset to the earliest available message:
```bash
./bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group slurm --reset-offsets --to-earliest --topic registrations --execute
```

---

### **Configuring Topic Retention**

#### **Changing Retention Settings**
To configure message retention for a topic, such as setting a retention period of 60 seconds:
```bash
./bin/kafka-configs.sh --bootstrap-server localhost:9092 --entity-type topics --entity-name registrations --alter --add-config retention.ms=60000
```

#### **Producing Continuous Messages**
Use a producer that continuously reads from a file and sends data to Kafka:
```bash
touch /tmp/data && tail -f -n0 /tmp/data | ./bin/kafka-console-producer.sh --topic registrations --bootstrap-server=localhost:9092 --sync
```

In another terminal, append data to the file continuously:
```bash
for i in $(seq 1 3600); do echo $"test${i}" >> /tmp/data; sleep 1; done
```

#### **Checking Segment Rollovers**
To force segment rollovers for a topic every 10 seconds:
```bash
./bin/kafka-configs.sh --bootstrap-server localhost:9092 --entity-type topics --entity-name registrations --alter --add-config segment.ms=10000
```

---

### **ZooKeeper Operations**

To interact with ZooKeeper, use the `zookeeper-shell.sh` script and connect to `localhost:2181`.

Command:
```bash
./bin/zookeeper-shell.sh localhost:2181
```

Some example ZooKeeper commands:
- List root directories:
  ```bash
  ls /
  ```
- Get controller information:
  ```bash
  get /controller
  ```
- Get partition state:
  ```bash
  get /brokers/topics/registrations/partitions/0/state
  ```
- View broker metadata:
  ```bash
  stat /brokers/ids/0
  ```

---

### **Log Compaction**

To enable **log compaction** for a topic, update the `cleanup.policy` configuration to include both `delete` (default) and `compact`:
```bash
./bin/kafka-configs.sh --bootstrap-server localhost:9092 --entity-type topics --entity-name registrations --alter --add-config cleanup.policy=delete,compact
```

---

### **Complete List of Commands**

1. **Download Kafka**:
   ```bash
   wget https://archive.apache.org/dist/kafka/2.7.0/kafka_2.13-2.7.0.tgz
   ```
2. **Extract Kafka**:
   ```bash
   tar -xzf kafka_2.13-2.7.0.tgz
   cd kafka_2.13-2.7.0
   ```
3. **Start ZooKeeper**:
   ```bash
   ./bin/zookeeper-server-start.sh config/zookeeper.properties
   ```
4. **Start Kafka Broker**:
   ```bash
   ./bin/kafka-server-start.sh config/server.properties
   ```
5. **Create Topic**:
   ```bash
   ./bin/kafka-topics.sh --create --topic registrations --bootstrap-server localhost:9092
   ```
6. **Describe Topic**:
   ```bash
   ./bin/kafka-topics.sh --describe --topic registrations --bootstrap-server localhost:9092
   ```
7. **Write Messages (Producer)**:
   ```bash
   ./bin/kafka-console-producer.sh --topic registrations --bootstrap-server localhost:9092
   ```
8. **Read Messages (Consumer)**:
   ```bash
   ./bin/kafka-console-consumer.sh --topic registrations --bootstrap-server localhost:9092 --from-beginning
   ```
9. **Set Retention Policy**:
   ```bash
   ./bin/kafka-configs.sh --bootstrap-server localhost:9092 --entity-type topics --entity-name registrations --alter --add-config retention.ms=60000
   ```
10. **Check Consumer Group State**:
    ```bash
    ./bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group slurm --describe
    ```
11. **Reset Consumer Offset**:
    ```bash
    ./bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group slurm --reset-offsets --to-earliest --topic registrations --execute
    ```
12. **Enable Log Compaction**:
    ```bash
    ./bin/kafka-configs.sh --bootstrap-server localhost:9092 --entity-type topics --entity-name registrations --alter --add-config cleanup.policy=delete,compact
    ```
---
### Source:
1. [Habr](https://habr.com/ru/companies/slurm/articles/719540/)
2. [CPT-4o](https://chatgpt.com/share/678a5b9a-cdd0-8007-8b3b-dee92f0d63ac)