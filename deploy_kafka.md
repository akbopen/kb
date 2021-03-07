# Create Zookeeper
1. Deploy Zookeeper cluster `zookeeper_deploy.yaml`

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: zookeeper1
spec:
  replicas: 1
  selector:
    app: zookeeper1
  template:
    metadata:
      labels:
        app: zookeeper1
    spec:
      containers:
      - name: zookeeper1
        image: digitalwonderland/zookeeper
        ports:
        - containerPort: 2181
        - containerPort: 2888
        - containerPort: 3888
        env:
        - name: ZOOKEEPER_ID
          value: "1"
        - name: ZOOKEEPER_SERVER_1
          value: zoo1
        - name: ZOOKEEPER_SERVER_2
          value: zoo2
---
apiVersion: v1
kind: ReplicationController
metadata:
  name: zookeeper2
spec:
  replicas: 1
  selector:
    app: zookeeper2
  template:
    metadata:
      labels:
        app: zookeeper2
    spec:
      containers:
      - name: zookeeper2
        image: digitalwonderland/zookeeper
        ports:
        - containerPort: 2181
        - containerPort: 2888
        - containerPort: 3888
        env:
        - name: ZOOKEEPER_ID
          value: "2"
        - name: ZOOKEEPER_SERVER_1
          value: zoo1
        - name: ZOOKEEPER_SERVER_2
          value: zoo2
```

2. create deployment
```
kubectl create -f zookeeper_deploy.yaml
# view pods
kubectl get pods
# to check with the logs
kubectl logs [POD_NAME]
```

3. Start Zookeeper service `zookeeper-service.yaml`
```
kind: Service
apiVersion: v1
metadata:
  name: zoo1
spec:
  type: LoadBalancer
  ports:
  - name: port-2181
    port: 2181
    protocol: TCP 
  - name: port-2888
    port: 2888
    protocol: TCP 
  - name: port-3888
    port: 3888
    protocol: TCP 
  selector:
    app: zookeeper1

---
kind: Service
apiVersion: v1
metadata:
  name: zoo2
spec:
  type: LoadBalancer
  ports:
  - name: port-2181
    port: 2181
    protocol: TCP 
  - name: port-2888
    port: 2888
    protocol: TCP 
  - name: port-3888
    port: 3888
    protocol: TCP 
  selector:
    app: zookeeper2
```

4. Create service
```
kubectl create -f zookeeper-service.yaml
# view services
kubectl get services
```


# Deploy Kafka
1. Create kafka service template: `kafka-service.yaml`

```
apiVersion: v1
kind: Service
metadata:
  name: kaf1
spec:
  type: LoadBalancer
  ports:
    - name: port-9092
      port: 9092
      protocol: TCP
  selector:
    app: kafka1
---
apiVersion: v1
kind: Service
metadata:
  name: kaf2
spec:
  type: LoadBalancer
  ports:
    - name: port-9092
      port: 9092
      protocol: TCP
  selector:
    app: kafka2
  ```
2. Create kafka service
```
kubectl create -f kafka-service.yaml
# view services
kubectl get services
```
3. Collect loadbalance dns name
```
kubectl get services
```
4. Create kafaka deployment

`kafka-cluster.yaml`
```
apiVersion: v1
kind: ReplicationController
metadata:
  name: kafka1
spec:
  replicas: 1
  selector:
    app: kafka1
  template:
    metadata:
      labels:
        app: kafka1
    spec:
      containers:
      - name: kafka1
        image: wurstmeister/kafka
        ports:
        - containerPort: 9092
        env:
        - name: KAFKA_BROKER_ID
          value: "1"
        - name: KAFKA_ADVERTISED_PORT
          value: "9092"
        - name: KAFKA_ADVERTISED_HOST_NAME
          value: <dns_name_kafka_service_1>
        - name: KAFKA_ZOOKEEPER_CONNECT
          value: zoo1:2181,zoo2:2181
---
apiVersion: v1
kind: ReplicationController
metadata:
  name: kafka2
spec:
  replicas: 1
  selector:
    app: kafka2
  template:
    metadata:
      labels:
        app: kafka2
    spec:
      containers:
      - name: kafka2
        image: wurstmeister/kafka
        ports:
        - containerPort: 9092
        env:
        - name: KAFKA_BROKER_ID
          value: "2"
        - name: KAFKA_ADVERTISED_PORT
          value: "9092"
        - name: KAFKA_ADVERTISED_HOST_NAME
          value: <dns_name_kafka_service_2>
        - name: KAFKA_ZOOKEEPER_CONNECT
          value: zoo1:2181,zoo2:2181
```
5. Create kafka cluster
```
kubectl create -f kafka-cluster.yaml
# view services
kubectl get pod
```
6. Test with connecting to Kafka broker

```
kafkacat -L -b <kafka broker host>:<kafka broker port>
```
7. Create Publisher

```
kafkacat -P -b <kafka broker host>:<kafka broker port> <publisher-name>
```

8. Create Consumer

```
kafkacat -C -b <kafka broker host>:<kafka broker port> <publisher-name>
```
