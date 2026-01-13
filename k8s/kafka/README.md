# Kafka Setup for Muzika Platform

This directory contains Kubernetes manifests for setting up Kafka using Strimzi operator.

## Prerequisites

1. Kubernetes cluster with enough resources (at least 3 nodes recommended)
2. kubectl configured to access your cluster
3. Storage class for persistent volumes (check with `kubectl get storageclass`)

## Installation Steps

### 1. Install Strimzi Kafka Operator

```bash
kubectl create namespace kafka
kubectl create -f 'https://strimzi.io/install/latest?namespace=kafka' -n kafka
```

Wait for the operator to be ready:
```bash
kubectl wait deployment strimzi-cluster-operator --for=condition=Available -n kafka --timeout=300s
```

### 2. Create Kafka Namespace

```bash
kubectl apply -f namespace.yaml
```

### 3. Deploy Kafka Cluster

```bash
kubectl apply -f kafka-cluster.yaml
```

This will create:
- A Kafka cluster with 3 brokers
- A ZooKeeper cluster with 3 nodes
- Entity Operator for managing topics and users

Wait for the cluster to be ready:
```bash
kubectl wait kafka/kafka-cluster --for=condition=Ready -n kafka --timeout=600s
```

### 4. Create Kafka Topics

```bash
kubectl apply -f kafka-topics.yaml
```

Verify topics are created:
```bash
kubectl get kafkatopics -n kafka
```

### Check Kafka Services

```bash
kubectl get svc -n kafka | grep kafka-cluster
```

You should see:
- `kafka-cluster-kafka-bootstrap` - Bootstrap service (used by applications)
- `kafka-cluster-kafka-brokers` - Broker services
- `kafka-cluster-zookeeper-client` - ZooKeeper client service

## Topics Created

The following topics are created for the Muzika platform:

1. **user-created** - Published by AuthorizationManager, consumed by PlaylistManager and QueueManager
2. **request-random-song** - Published by QueueManager, consumed by BandcampApi
3. **request-slskd-song** - Published by BandcampApi, consumed by SlskdDownload and QueueManager
4. **loaded-song** - Published by SlskdDownload, consumed by QueueManager
5. **liked** - Published by QueueManager, consumed by PlaylistManager
6. **unliked** - Published by QueueManager, consumed by PlaylistManager

All topics have:
- 3 partitions (for parallelism)
- 3 replicas (for high availability)
- 7 days retention
- Delete cleanup policy

## Bootstrap Server

The bootstrap server for applications is:
```
kafka-cluster-kafka-bootstrap.kafka.svc.cluster.local:9092
```

This is already configured in your application ConfigMaps.

## Troubleshooting

### Check Kafka Cluster Status

```bash
kubectl describe kafka kafka-cluster -n kafka
```

### Check Pod Logs

```bash
kubectl logs kafka-cluster-kafka-0 -n kafka
```

### Check Topic Status

```bash
kubectl describe kafkatopic <topic-name> -n kafka
```
