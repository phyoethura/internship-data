# Strimzi Kafka Setup Instructions

## Step 1: Add Strimzi Helm Repository

1. Add the Strimzi Helm repository:

    ```bash
    helm repo add strimzi https://strimzi.io/charts/
    ```

    - **Explanation**: This command adds the Strimzi Helm chart repository to your Helm configuration, allowing you to install charts from this repository.

2. Update your Helm repositories to ensure you have the latest information:

    ```bash
    helm repo update
    ```

    - **Explanation**: This command updates the local Helm chart repository cache with the latest information from the added repositories.

## Step 2: Create Namespace for Strimzi

1. Create a namespace for Strimzi:

    ```bash
    kubectl create namespace kafka
    ```

    - **Explanation**: This command creates a new namespace named `kafka` in your Kubernetes cluster.

## Step 3: Install Strimzi Operator

1. Install the Strimzi Operator version 0.36.0:

    ```bash
    helm install strimzi-operator strimzi/strimzi-kafka-operator --namespace kafka --version 0.36.0
    ```

    - **Explanation**: This command installs the Strimzi Kafka Operator in the `kafka` namespace using the specified version.

## Step 4: Create Kafka Cluster

1. Create a file named `kafka-cluster-poc.yaml` with the following content:

    ```yaml name=kafka-cluster-poc.yaml
    apiVersion: kafka.strimzi.io/v1beta2
    kind: Kafka
    metadata:
      name: kafka-cluster-poc
    spec:
      kafka:
        version: 3.4.0
        logging:
          type: inline
          loggers:
            kafka.root.logger.level: "INFO, WARN, ERROR, DEBUG"
        replicas: 3
        listeners:
          - name: plain
            port: 9092
            type: internal
            tls: false
            authentication:
             type: scram-sha-512
          - name: external
            port: 9094
            type: loadbalancer
            tls: false
            authentication:
             type: scram-sha-512
            configuration:
              bootstrap:
                loadBalancerIP: 10.1.0.76
              brokers:
              - broker: 0
                loadBalancerIP: 10.1.0.77
              - broker: 1
                loadBalancerIP: 10.1.0.78
              - broker: 2
                loadBalancerIP: 10.1.0.79
        authorization:
          type: simple
        config:
          default.replication.factor: 3
          min.insync.replicas: 3
          offsets.topic.replication.factor: 3
          transaction.state.log.replication.factor: 3
          transaction.state.log.min.isr: 2
          log.message.format.version: "3.4"
          inter.broker.protocol.version: "3.4"
          message.max.bytes: 5242880
          replica.fetch.max.bytes: 5242880
          replica.fetch.response.max.bytes: 5242880
          log4j.rootLogger: "INFO, stdout, kafkaAppender, Error"
          log4j.logger.org.I0Itec.zkclient.ZkClient: "INFO"
          log4j.logger.org.apache.zookeeper: "INFO"
          log4j.logger.kafka: "INFO, WARN, ERROR"
          log4j.logger.org.apache.kafka: "INFO"
          log4j.logger.kafka.controller: "INFO, controllerAppender"
          log4j.logger.kafka.log.LogCleaner: "INFO, cleanerAppender"
          log4j.logger.state.change.logger: "INFO, stateChangeAppender"
          log4j.logger.kafka.authorizer.logger: "INFO, authorizerAppender"
          log4j.logger.kafka.network.Processor: "WARN, requestAppender"
        metricsConfig:
          type: jmxPrometheusExporter
          valueFrom:
            configMapKeyRef:
              name: kafka-metrics
              key: kafka-metrics-config.yaml
        storage:
          type: jbod
          volumes:
          - id: 0
            type: persistent-claim
            size: 10Gi
            class: openebs-zfs-tidb
            deleteClaim: false
        template:
          kafkaContainer:
            env:
            - name: TZ
              value: "Asia/Bangkok"
          pod:
            affinity:
              podAntiAffinity:
                preferredDuringSchedulingIgnoredDuringExecution:
                - podAffinityTerm:
                    labelSelector:
                      matchLabels:
                        strimzi.io/cluster: kafka-cluster-poc
                        strimzi.io/name: kafka-cluster-poc-kafka
                    topologyKey: kubernetes.io/hostname
                  weight: 100
      kafkaExporter:
        topicRegex: ".*"
        groupRegex: ".*"
      zookeeper:
        metricsConfig:
          type: jmxPrometheusExporter
          valueFrom:
            configMapKeyRef:
              name: zookeeper-metrics
              key: zookeeper-metrics.yaml
        replicas: 3
        config:
          zookeeper.root.logger: "INFO, CONSOLE"
          zookeeper.console.threshold: "INFO"
          zookeeper.log.threshold: "INFO"
          log4j.appender.CONSOLE.Threshold: "INFO"
          log4j.appender.ROLLINGFILE.Threshold: "INFO"
          log4j.appender.TRACEFILE.Threshold: "INFO"
          log4j.rootLogger: "INFO, CONSOLE, WARN, ERROR"
        storage:
          type: persistent-claim
          size: 10Gi
          class: openebs-zfs-tidb
          deleteClaim: false
        template:
          zookeeperContainer:
            env:
            - name: TZ
              value: "Asia/Bangkok"
          pod:
            affinity:
              podAntiAffinity:
                preferredDuringSchedulingIgnoredDuringExecution:
                - podAffinityTerm:
                    labelSelector:
                      matchLabels:
                        strimzi.io/cluster: kafka-cluster-poc
                        strimzi.io/name: kafka-cluster-poc-zookeeper
                    topologyKey: kubernetes.io/hostname
                  weight: 100
      entityOperator:
        topicOperator:
          logging:
            type: inline
            loggers:
              rootLogger.level: "INFO, WARN, ERROR"
        userOperator:
          logging:
            type: inline
            loggers:
              rootLogger.level: "INFO, WARN, ERROR"
      cruiseControl:
        logging:
          type: inline
          loggers:
            rootLogger.level: "INFO, DEBUG, ERROR"
    ```

2. Apply the Kafka cluster configuration:

    ```bash
    kubectl apply -f kafka-cluster-poc.yaml -n kafka
    ```

    - **Explanation**: This command creates a Kafka cluster named `kafka-cluster-poc` with the specified configuration in the `kafka` namespace.

## Step 5: Create ConfigMaps for Metrics

1. Create a file named `kafka-metrics-config.yaml` with the following content:

    ```yaml name=kafka-metrics-config.yaml
    kind: ConfigMap
    apiVersion: v1
    metadata:
      name: kafka-metrics
      labels:
        app: strimzi
    data:
      kafka-metrics-config.yaml: |
        lowercaseOutputName: true
        rules:
        - pattern: kafka.server<type=(.+), name=(.+), clientId=(.+), topic=(.+), partition=(.*)><>Value
          name: kafka_server_$1_$2
          type: GAUGE
          labels:
           clientId: "$3"
           topic: "$4"
           partition: "$5"
        - pattern: kafka.server<type=(.+), name=(.+), clientId=(.+), brokerHost=(.+), brokerPort=(.+)><>Value
          name: kafka_server_$1_$2
          type: GAUGE
          labels:
           clientId: "$3"
           broker: "$4:$5"
        - pattern: kafka.server<type=(.+), cipher=(.+), protocol=(.+), listener=(.+), networkProcessor=(.+)><>connections
          name: kafka_server_$1_connections_tls_info
          type: GAUGE
          labels:
            cipher: "$2"
            protocol: "$3"
            listener: "$4"
            networkProcessor: "$5"
        - pattern: kafka.server<type=(.+), clientSoftwareName=(.+), clientSoftwareVersion=(.+), listener=(.+), networkProcessor=(.+)><>connections
          name: kafka_server_$1_connections_software
          type: GAUGE
          labels:
            clientSoftwareName: "$2"
            clientSoftwareVersion: "$3"
            listener: "$4"
            networkProcessor: "$5"
        - pattern: "kafka.server<type=(.+), listener=(.+), networkProcessor=(.+)><>(.+):"
          name: kafka_server_$1_$4
          type: GAUGE
          labels:
           listener: "$2"
           networkProcessor: "$3"
        - pattern: kafka.server<type=(.+), listener=(.+), networkProcessor=(.+)><>(.+)
          name: kafka_server_$1_$4
          type: GAUGE
          labels:
           listener: "$2"
           networkProcessor: "$3"
        # Some percent metrics use MeanRate attribute
        # Ex) kafka.server<type=(KafkaRequestHandlerPool), name=(RequestHandlerAvgIdlePercent)><>MeanRate
        - pattern: kafka.(\w+)<type=(.+), name=(.+)Percent\w*><>MeanRate
          name: kafka_$1_$2_$3_percent
          type: GAUGE
        # Generic gauges for percents
        - pattern: kafka.(\w+)<type=(.+), name=(.+)Percent\w*><>Value
          name: kafka_$1_$2_$3_percent
          type: GAUGE
        - pattern: kafka.(\w+)<type=(.+), name=(.+)Percent\w*, (.+)=(.+)><>Value
          name: kafka_$1_$2_$3_percent
          type: GAUGE
          labels:
            "$4": "$5"
        # Generic per-second counters with 0-2 key/value pairs
        - pattern: kafka.(\w+)<type=(.+), name=(.+)PerSec\w*, (.+)=(.+), (.+)=(.+)><>Count
          name: kafka_$1_$2_$3_total
          type: COUNTER
          labels:
            "$4": "$5"
            "$6": "$7"
        - pattern: kafka.(\w+)<type=(.+), name=(.+)PerSec\w*, (.+)=(.+)><>Count
          name: kafka_$1_$2_$3_total
          type: COUNTER
          labels:
            "$4": "$5"
        - pattern: kafka.(\w+)<type=(.+), name=(.+)PerSec\w*><>Count
          name: kafka_$1_$2_$3_total
          type: COUNTER
        # Generic gauges with 0-2 key/value pairs
        - pattern: kafka.(\w+)<type=(.+), name=(.+), (.+)=(.+), (.+)=(.+)><>Value
          name: kafka_$1_$2_$3
          type: GAUGE
          labels:
            "$4": "$5"
            "$6": "$7"
        - pattern: kafka.(\w+)<type=(.+), name=(.+), (.+)=(.+)><>Value
          name: kafka_$1_$2_$3
          type: GAUGE
          labels:
            "$4": "$5"
        - pattern: kafka.(\w+)<type=(.+), name=(.+)><>Value
          name: kafka_$1_$2_$3
          type: GAUGE
        # Emulate Prometheus 'Summary' metrics for the exported 'Histogram's.
        # Note that these are missing the '_sum' metric!
        - pattern: kafka.(\w+)<type=(.+), name=(.+), (.+)=(.+), (.+)=(.+)><>Count
          name: kafka_$1_$2_$3_count
          type: COUNTER
          labels:
            "$4": "$5"
            "$6": "$7"
        - pattern: kafka.(\w+)<type=(.+), name=(.+), (.+)=(.*), (.+)=(.+)><>(\d+)thPercentile
          name: kafka_$1_$2_$3
          type: GAUGE
          labels:
            "$4": "$5"
            "$6": "$7"
            quantile: "0.$8"
        - pattern: kafka.(\w+)<type=(.+), name=(.+), (.+)=(.+)><>Count
          name: kafka_$1_$2_$3_count
          type: COUNTER
          labels:
            "$4": "$5"
        - pattern: kafka.(\w+)<type=(.+), name=(.+), (.+)=(.*)><>(\d+)thPercentile
          name: kafka_$1_$2_$3
          type: GAUGE
          labels:
            "$4": "$5"
            quantile: "0.$6"
        - pattern: kafka.(\w+)<type=(.+), name=(.+)><>Count
          name: kafka_$1_$2_$3_count
          type: COUNTER
        - pattern: kafka.(\w+)<type=(.+), name=(.+)><>(\d+)thPercentile
          name: kafka_$1_$2_$3
          type: GAUGE
          labels:
            quantile: "0.$4"
    ```

2. Apply the ConfigMap for Kafka metrics:

    ```bash
    kubectl apply -f kafka-metrics-config.yaml -n kafka
    ```

    - **Explanation**: This command creates a ConfigMap named `kafka-metrics` in the `kafka` namespace with the configuration for exporting Kafka metrics.

3. Create a file named `zookeeper-metrics-config.yaml` with the following content:

    ```yaml name=zookeeper-metrics-config.yaml
    kind: ConfigMap
    apiVersion: v1
    metadata:
      name: zookeeper-metrics
      labels:
        app: strimzi
    data:
      zookeeper-metrics.yaml: |
        lowercaseOutputName: true
        rules:
        - pattern: "org.apache.ZooKeeperService<name0=ReplicatedServer_id(\\d+)><>(\\w+)"
          name: "zookeeper_$2"
          type: GAUGE
        - pattern: "org.apache.ZooKeeperService<name0=ReplicatedServer_id(\\d+), name1=replica.(\\d+)><>(\\w+)"
          name: "zookeeper_$3"
          type: GAUGE
          labels:
            replicaId: "$2"
        - pattern: "org.apache.ZooKeeperService<name0=ReplicatedServer_id(\\d+), name1=replica.(\\d+), name2=(\\w+)><>(Packets\\w+)"
          name: "zookeeper_$4"
          type: COUNTER
          labels:
            replicaId: "$2"
            memberType: "$3"
        - pattern: "org.apache.ZooKeeperService<name0=ReplicatedServer_id(\\d+), name1=replica.(\\d+), name2=(\\w+)><>(\\w+)"
          name: "zookeeper_$4"
          type: GAUGE
          labels:
            replicaId: "$2"
            memberType: "$3"
        - pattern: "org.apache.ZooKeeperService<name0=ReplicatedServer_id(\\d+), name1=replica.(\\d+), name2=(\\w+), name3=(\\w+)><>(\\w+)"
          name: "zookeeper_$4_$5"
          type: GAUGE
          labels:
            replicaId: "$2"
            memberType: "$3"
    ```

4. Apply the ConfigMap for Zookeeper metrics:

    ```bash
    kubectl apply -f zookeeper-metrics-config.yaml -n kafka
    ```

    - **Explanation**: This command creates a ConfigMap named `zookeeper-metrics` in the `kafka` namespace with the configuration for exporting Zookeeper metrics.


## Step 6: Deploy Kafdrop

1. Create a file named `kafdrop-dc-deployment.yaml` with the following content:

    ```yaml name=kafdrop-dc-deployment.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    labels:
        app.kubernetes.io/instance: kafdrop-dc
        app.kubernetes.io/name: kafdrop-dc
    name: kafdrop-dc
    namespace: kafka
    spec:
    replicas: 1
    selector:
        matchLabels:
        app.kubernetes.io/instance: kafdrop-dc
        app.kubernetes.io/name: kafdrop-dc
    template:
        metadata:
        creationTimestamp: null
        labels:
            app.kubernetes.io/instance: kafdrop-dc
            app.kubernetes.io/name: kafdrop-dc
        spec:
        containers:
        - env:
            - name: KAFKA_BROKERCONNECT
            value: 10.111.0.40:9094
            - name: KAFKA_PROPERTIES
            value: c2VjdXJpdHkucHJvdG9jb2w9U0FTTF9QTEFJTlRFWFQKc2FzbC5tZWNoYW5pc209U0NSQU0tU0hBLTUxMgpzYXNsLmphYXMuY29uZmlnPW9yZy5hcGFjaGUua2Fma2EuY29tbW9uLnNlY3VyaXR5LnNjcmFtLlNjcmFtTG9naW5Nb2R1bGUgcmVxdWlyZWQgdXNlcm5hbWU9Imx1Y2t5IiBwYXNzd29yZD0iYVc5U01NZ3UyNlZCaFNsQWdPRUJYZks3eUJ6RmFMdksiOwo=
            - name: KAFKA_TRUSTSTORE
            - name: KAFKA_KEYSTORE
            - name: JVM_OPTS
            value: -Xms512M -Xmx512M
            - name: JMX_PORT
            value: "8686"
            - name: HOST
            - name: SERVER_SERVLET_CONTEXTPATH
            - name: KAFKA_PROPERTIES_FILE
            value: kafka.properties
            - name: KAFKA_TRUSTSTORE_FILE
            value: kafka.truststore.jks
            - name: KAFKA_KEYSTORE_FILE
            value: kafka.keystore.jks
            - name: SERVER_PORT
            value: "9000"
            - name: CMD_ARGS
            value: --topic.deleteEnabled=false --topic.createEnabled=false
            image: obsidiandynamics/kafdrop:4.0.1
            imagePullPolicy: Always
            name: kafdrop-dc
            ports:
            - containerPort: 9000
            name: http
            protocol: TCP
            resources:
            requests:
                cpu: 500m
                memory: 1Gi
    ---
    apiVersion: v1
    kind: Service
    metadata:
    name: kafdrop-dc-svc
    namespace: kafka
    annotations:
        metallb.universe.tf/allow-shared-ip: "key-to-share-10.111.0.49"
    spec:
    type: LoadBalancer
    loadBalancerIP: 10.111.0.49
    ports:
        - name: kafdrop-dc
        protocol: TCP
        port: 9037
        targetPort: 9000
    selector:
        app.kubernetes.io/name: kafdrop-dc  
    ```
## Step 7: Create Kafka Topic

1. Create a file named `kafka-topic.yaml` with the following content:

    ```yaml name=kafka-topic.yaml
    apiVersion: kafka.strimzi.io/v1beta2
    kind: KafkaTopic
    metadata:
      name: my-topic
      labels:
        strimzi.io/cluster: my-cluster
    spec:
      partitions: 3
      replicas: 3
      config:
        retention.ms: 7200000
        segment.bytes: 1073741824
    ```

2. Apply the Kafka topic configuration:

    ```bash
    kubectl apply -f kafka-topic.yaml -n kafka
    ```

    - **Explanation**: This command creates a Kafka topic named `my-topic` with 3 partitions and 3 replicas. The topic has a retention period of 7200000 milliseconds and segment size of 1 GiB.

## Step 8: Create Kafka User

1. Create a file named `kafka-user.yaml` with the following content:

    ```yaml name=kafka-user.yaml
    apiVersion: kafka.strimzi.io/v1beta1
    kind: KafkaUser
    metadata:
      name: kafdrop-user
      labels:
        strimzi.io/cluster: kafka-cluster-poc
    spec:
      authentication:
        type: scram-sha-512
      authorization:
        type: simple
        acls:
          - resource:
              type: topic
              name: '*'
              patternType: literal
            operation: All
          - resource:
              type: cluster
              name: '*'
              patternType: literal
            operation: All
          - resource:
              type: group
              name: '*'
              patternType: literal
            operation: All
          - resource:
              type: transactionalId
              name: '*'
              patternType: literal
            operation: All
    ```

2. Apply the Kafka user configuration:

    ```bash
    kubectl apply -f kafka-user.yaml -n kafka
    ```

    - **Explanation**: This command creates a Kafka user named `kafdrop-user` with SCRAM-SHA-512 authentication and full access to all Kafka resources.

## Step 9: Configure Security Properties for Kafka

1. Create a `kafkapoc.properties` file with the following content:

    ```properties name=kafkapoc.properties
    security.protocol=SASL_PLAINTEXT
    sasl.mechanism=SCRAM-SHA-512
    sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username="bassall" password="aXeJK4g3f4CV8STzBGAzXGZhnxWfNdXU";
    ```

    - **Explanation**:
        - **security.protocol=SASL_PLAINTEXT**: This specifies that the communication protocol will use SASL for authentication, but the data will be transmitted in plaintext.
        - **sasl.mechanism=SCRAM-SHA-512**: This specifies that the SASL mechanism used for authentication is SCRAM (Salted Challenge Response Authentication Mechanism) with the SHA-512 hashing algorithm.
        - **sasl.jaas.config=...**: This provides the JAAS (Java Authentication and Authorization Service) configuration for SASL authentication, including the username and password.

2. Base64 encode the contents of the `kafkapoc.properties` file. The encoded content is:

    ```plaintext
    c2VjdXJpdHkucHJvdG9jb2w9U0FTTF9QTEFJTlRFWFQKc2FzbC5tZWNoYW5pc209U0NSQU0tU0hBLTUxMgpzYXNsLmphYXMuY29uZmlnPW9yZy5hcGFjaGUua2Fma2EuY29tbW9uLnNlY3VyaXR5LnNjcmFtLlNjcmFtTG9naW5Nb2R1bGUgcmVxdWlyZWQgdXNlcm5hbWU9ImJhc3NhbGwiIHBhc3N3b3JkPSJhWGVKSzRnM2Y0Q1Y4U1R6QkdBekhHWmhueFdGTmRYVSI7Cg==
    ```

3. Use the encoded content in the `kafdrop-dc-deployment.yaml` file to set the `KAFKA_PROPERTIES` environment variable.

## Step 10: Retrieve Kafka User Password

1. Create a script named `get_user_password.sh` with the following content:

    ```bash name=get_user_password.sh
    #!/bin/bash

    echo "Please input Kafka User"
    read kafka_user

    kubectl get secret $kafka_user -n kafka --template={{.data}} | cut -d ":" -f 3 | base64 -d
    ```

2. Run the script to retrieve the Kafka user password:

    ```bash
    ./get_user_password.sh
    ```

    - **Explanation**: This script prompts the user to input the Kafka username, retrieves the corresponding secret from Kubernetes, and decodes the password from base64.

## Step 11: Go Program to Produce Messages

1. Create a Go program named `producer.go` with the following content:

    ```go name=producer.go
    package main

    import (
        "context"
        "fmt"
        "log"
        "time"

        "github.com/segmentio/kafka-go"
        "github.com/segmentio/kafka-go/sasl/scram"
        "github.com/klauspost/compress/zstd"
    )

    type KafkaConfig struct {
        Username string
        Password string
    }

    func main() {
        config := KafkaConfig{
            Username: "bassread",
            Password: "zbznLgHkxMemKgsoXYiv4TOHheZltPCp",
        }

        kafkaAddress := "10.111.0.40:9094"
        topic := "my-topic-01"

        // Create SASL mechanism
        mechanism, err := scram.Mechanism(scram.SHA512, config.Username, config.Password)
        if err != nil {
            log.Fatalf("Failed to create SASL mechanism: %v", err)
        }

        // Create Zstandard compressor
        zstdCompressor, err := zstd.NewWriter(nil, zstd.WithEncoderLevel(zstd.SpeedBetterCompression))
        if err != nil {
            log.Fatalf("Failed to create Zstandard compressor: %v", err)
        }

        // Create Kafka writer
        writer := kafka.NewWriter(kafka.WriterConfig{
            Brokers: []string{kafkaAddress},
            Topic:   topic,
            Dialer: &kafka.Dialer{
                Timeout:       10 * time.Second,
                DualStack:     true,
                SASLMechanism: mechanism,
            },
            Async: false, // Set to true for asynchronous writes
            CompressionCodec: func() kafka.CompressionCodec {
                return kafka.Codec{
                    Name: "zstd",
                    Encode: func(msg kafka.Message) ([]byte, error) {
                        return zstdCompressor.EncodeAll(msg.Value, nil), nil
                    },
                }
            }(),
        })
        defer writer.Close()

        // Produce messages
        fmt.Print("Enter the number of times to loop: ")
        var loopCount int
        _, err = fmt.Scan(&loopCount)
        if err != nil {
            log.Fatalf("Failed to read input: %v", err)
        }

        for i := 0; i < loopCount; i++ {
            key := fmt.Sprintf("%d", i)
            value := fmt.Sprintf("Hello Test Production Number %d, Kafka! Produce messages from Go.", i)

            err := writer.WriteMessages(context.Background(), kafka.Message{
                Key:   []byte(key),
                Value: []byte(value),
            })
            if err != nil {
                log.Fatalf("Failed to write message: %v", err)
            }

            fmt.Printf("Message sent to Kafka: key=%s, value=%s\n", key, value)
            time.Sleep(1 * time.Second) // Add delay between messages
        }

        fmt.Println("Producer finished sending messages.")
    }
    ```

    - **Explanation**: This Go program produces messages to a Kafka topic using the specified SASL mechanism for authentication and Zstandard compression for message encoding. The user is prompted to enter the number of messages to produce, and the program sends each message to Kafka with a 1-second delay between messages.

## Step 12: Go Program to Consume Messages

1. Create a Go program named `consumer.go` with the following content:

    ```go name=consumer.go
    package main

    import (
        "context"
        "fmt"
        "log"
        "os"
        "time"

        "github.com/segmentio/kafka-go"
        "github.com/segmentio/kafka-go/sasl/scram"
    )

    type KafkaConfig struct {
        Username     string
        Password     string
        ConsumeGroup string
    }

    func main() {
        config := KafkaConfig{
            Username:     "lucky",
            Password:     "aW9SMMgu26VBhSlAgOEBXfK7yBzFaLvK",
            ConsumeGroup: "luxxy",
        }
        log.Println("Consume group", config.ConsumeGroup, "service is ready...")

        go func() {

            kafkaAddress := "10.111.0.40:9094"
            focusTopic := "a-topic"

            mechanism, err := scram.Mechanism(scram.SHA512, config.Username, config.Password)
            if err != nil {
                panic(err)
            }

            r := kafka.NewReader(kafka.ReaderConfig{
                Brokers: []string{kafkaAddress},
                GroupID: config.ConsumeGroup,
                Topic:   focusTopic,
                Dialer: &kafka.Dialer{
                    Timeout:       10 * time.Second,
                    DualStack:     true,
                    SASLMechanism: mechanism,
                },
                MinBytes: 10e3,
                MaxBytes: 10e6,
                MaxWait:  1 * time.Second,
            })
            defer r.Close()

            ctx := context.Background()
            for {
                m, err := r.FetchMessage(ctx)
                if err != nil {
                    fmt.Printf("%v - %v - Kafka: %v\n\n", focusTopic, config.ConsumeGroup, err)
                    continue
                }
                fmt.Printf("message at topic/partition/offset %v/%v/%v: %s = %s\n", m.Topic, m.Partition, m.Offset, string(m.Key), string(m.Value))

                if err := r.CommitMessages(ctx, m); err != nil {
                    fmt.Println("Kafka failed to commit messages:", err)
                }
            }

        }()

        c := make(chan os.Signal, 1)
        <-c
    }
    ```

    - **Explanation**: This Go program consumes messages from a Kafka topic using the specified SASL mechanism for authentication. The program continuously fetches messages from the topic and prints the message details (topic, partition, offset, key, and value) to the console. It also commits the messages to Kafka to acknowledge successful processing.

---

By following these steps, you will set up a Kafka cluster, create a topic and user, retrieve the user's password, and run Go programs to produce and consume messages from the Kafka topic.

[Back to Main README](../README.md)