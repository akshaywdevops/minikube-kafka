# Apache Kafka on Minikube

### Stack used 
- Minikube
- Docker
- Apache Kafka - 2.7.0
- Zookeeper - 3.4.10
- CMAK (Cluster Manager Apache Kafka) - 3.0.0.5

### Image Creation
Build Docker image first for Kafka, Zookeeper and CMAK

- Kafka
  ```bash
     $ cd kafka/docker
     $ docker build -t <repoid>/kafka:2.7 .
     $ cd -
  ```
- Zookeeper
  ```bash
     $ cd zookeeper/docker
     $ docker build -t <repoid>/zookeeper:3.4.10 .
     $ cd -
  ```
- CMAK
  ```bash
     $ cd kafka
     $ docker build -t <repoid>/cmak:3.0.0.5 -f Dockerfile-cmak .
     $ cd -
  ```

### Cluster creation
Create Apache Kafka cluster 

- Zookeeper
  ```bash
     $ kubectl create ns custom-kafka
     $ kubectl apply -f zookeeper/statefulset.yaml -n custom-kafka
     $ kubectl apply -f zookeeper/service.yaml -n custom-kafka
  ```

- Kafka
  ```bash
     $ kubectl apply -f kafka/statefulset.yaml -n custom-kafka
     $ kubectl apply -f kafka/service.yaml -n custom-kafka
  ```

- CMAK
  ```bash
     $ kubectl apply -f kafka/kafka-manager.yaml -n custom-kafka
  ```

Once Cluster is deployed add the cluster in CMAK
- Access CMAK dashboard on `$(minikube ip):nodeport-of-kafka-manager-svc`
- Add cluster - Use `zk-cs.custom-kafka.svc.cluster.local:2181` for zookeeper address and add the cluster
- If you faced error `Yikes! KeeperErrorCode = Unimplemented for /kafka-manager/mutex` then access zookeeper pod (zk-0) and perform following actions to add cluster in CMAK
  ```bash
     $ kubectl -n custom-kafka exec -it zk-0 -- bash
     $ cd /opt/zookeeper/bin/
     $ ./zkCli.sh
     [zk: localhost:2181(CONNECTED) 1] ls /kafka-manager
     [zk: localhost:2181(CONNECTED) 2] create /kafka-manager/mutex ""
     [zk: localhost:2181(CONNECTED) 3] create /kafka-manager/mutex/locks ""
     [zk: localhost:2181(CONNECTED) 4] create /kafka-manager/mutex/leases ""
     [zk: localhost:2181(CONNECTED) 5] exit
  ``` 


### References
- https://github.com/mmohamed/kafka-kubernetes
