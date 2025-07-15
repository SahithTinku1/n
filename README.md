

# **Kafka on EC2 – Secure Public Setup Guide**

This guide walks you through the steps to install and configure Apache Kafka on an AWS EC2 instance, allowing external access using a public IP. All personal identifiers have been removed—safe for sharing or presenting.

---

## **Step 1: Download and Extract Kafka**

First, we’ll grab the Kafka archive and extract it.

```bash
wget https://archive.apache.org/dist/kafka/3.3.1/kafka_2.12-3.3.1.tgz
tar -xvf kafka_2.12-3.3.1.tgz
cd kafka_2.12-3.3.1
```

---

## **Step 2: Connect to Your EC2 Instance**

SSH into your EC2 instance. Replace the placeholders below with your actual key file and EC2 DNS.

```bash
ssh -i "<your-key.pem>" ec2-user@<your-ec2-public-dns>
```

Example format:
`ec2-user@ec2-xx-xxx-xxx-xx.ap-south-1.compute.amazonaws.com`

---

## **Step 3: Install Java (OpenJDK 8)**

Kafka needs Java to run. If it’s not already installed, do this:

```bash
sudo yum install java-1.8.0-openjdk -y
java -version
```

Make sure the version check returns a Java 1.8 build.

---

## **Step 4: Configure Kafka for Public Access**

Let’s now make Kafka accessible externally.

Navigate to Kafka's configuration file:

```bash
cd kafka_2.12-3.3.1
nano config/server.properties
```

Inside the file, **do the following**:

1. Comment out the default advertised listeners:

   ```properties
   # advertised.listeners=PLAINTEXT://:9092
   ```

2. Add your public IP (replace this with yours):

   ```properties
   advertised.listeners=PLAINTEXT://<your-public-ip>:9092
   ```

3. Make sure this line is also there:

   ```properties
   listeners=PLAINTEXT://0.0.0.0:9092
   ```

Save and exit the file (`Ctrl + X`, then `Y`, then `Enter`).

---

## **Step 5: Start ZooKeeper**

Kafka depends on ZooKeeper. Let’s start it:

```bash
cd kafka_2.12-3.3.1
bin/zookeeper-server-start.sh config/zookeeper.properties
```

Leave this terminal open and running.

---

## **Step 6: Start Kafka Broker (In a New Terminal)**

Open a new SSH terminal and reconnect to EC2:

```bash
ssh -i "<your-key.pem>" ec2-user@<your-ec2-public-dns>
```

Then, start the Kafka server with reduced memory usage:

```bash
export KAFKA_HEAP_OPTS="-Xmx256M -Xms128M"
cd kafka_2.12-3.3.1
bin/kafka-server-start.sh config/server.properties
```

---

## **Step 7: Create a Kafka Topic**

Once Kafka is running, create a topic named `demo_testing`:

```bash
cd kafka_2.12-3.3.1
bin/kafka-topics.sh --create \
  --topic demo_testing \
  --bootstrap-server <your-public-ip>:9092 \
  --replication-factor 1 \
  --partitions 1
```

---

## **Step 8: Launch a Kafka Producer**

Let’s now send some messages into that topic:

```bash
bin/kafka-console-producer.sh \
  --topic demo_testing \
  --bootstrap-server <your-public-ip>:9092
```

Type a few lines and press `Enter` after each to produce messages.

---

## **Step 9: Launch a Kafka Consumer**

In another terminal or SSH session, consume messages from the topic:

```bash
bin/kafka-console-consumer.sh \
  --topic demo_testing \
  --bootstrap-server <your-public-ip>:9092 \
  --from-beginning
```

You’ll see the messages you typed as a producer, now consumed.

---

## ✅ **Security & Access Checklist**

* 🔓 **Ensure Port 9092 is open** in your EC2 security group to allow external access.
* 🚫 **Never upload your `.pem` key** or real IP address to public platforms.
* 🔁 **Rotate SSH keys** periodically and track usage.
* 🔐 For production use, **always secure Kafka using SSL/TLS and authentication mechanisms**.


