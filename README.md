# Kubernetes Event-Driven Autoscaling (KEDA)

KEDA is an open-source Kubernetes-based event-driven autoscaler that extends Kubernetes' scaling capabilities beyond what the native Horizontal Pod Autoscaler (HPA) provides. KEDA allows you to scale applications based on event sources and metrics outside of Kubernetes itself.

## How KEDA Works

1. KEDA monitors external event sources or metrics (message queues, databases, monitoring systems)
2. When events or metrics exceed defined thresholds, KEDA activates and scales your deployments
3. When event processing needs decrease, KEDA scales down deployments, potentially to zero
4. KEDA works alongside the native Kubernetes HPA to provide more versatile scaling options

## Key Components

- **ScaledObject**: Custom resource that defines scaling rules for a deployment
- **ScaledJob**: Custom resource for event-driven jobs (based on Kubernetes Jobs)
- **Scalers**: Connectors to various event sources (70+ available)
- **Metrics Adapter**: Exposes metrics to the Kubernetes metrics API

## Basic Example

Here's a KEDA ScaledObject that scales a deployment based on RabbitMQ queue length:

```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: rabbitmq-scaler
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: consumer-deployment
  minReplicaCount: 0
  maxReplicaCount: 30
  pollingInterval: 15
  cooldownPeriod: 30
  triggers:
  - type: rabbitmq
    metadata:
      protocol: amqp
      queueName: orders
      host: rabbitmq.default.svc:5672
      queueLength: "50"
      username: user
      passwordFromEnv: RABBITMQ_PASSWORD
```

In this example:
- KEDA targets a deployment named `consumer-deployment`
- It scales between 0 and 30 replicas
- It checks the queue every 15 seconds
- When the queue length exceeds 50 messages, scaling begins
- After scaling activity, it waits 30 seconds before scaling again

## Installing KEDA

```bash
# Using Helm
helm repo add kedacore https://kedacore.github.io/charts
helm repo update
helm install keda kedacore/keda --namespace keda --create-namespace

# Using kubectl
kubectl apply -f https://github.com/kedacore/keda/releases/download/v2.10.1/keda-2.10.1.yaml
```

## Popular Event Sources (Scalers)

KEDA supports 70+ scalers, including:

- Message queues: RabbitMQ, Kafka, Azure Service Bus, AWS SQS
- Databases: PostgreSQL, MySQL, MongoDB, Redis
- Cloud services: AWS, Azure, GCP metrics
- Prometheus metrics
- Cron schedules
- HTTP endpoints

## Advanced Features

### Scaling to Zero

Unlike HPA, KEDA can scale deployments to zero pods when there's no work to be done:

```yaml
spec:
  minReplicaCount: 0
  # other settings...
```

### Authentication

KEDA supports various authentication methods for event sources:

```yaml
triggers:
- type: azure-servicebus
  metadata:
    connectionFromEnv: SERVICE_BUS_CONNECTION_STRING
    queueName: orders
    queueLength: "5"
```

### Scaling Jobs

For batch processing, use ScaledJob:

```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledJob
metadata:
  name: batch-processor
spec:
  jobTargetRef:
    template:
      spec:
        containers:
        - name: processor
          image: processor:latest
  pollingInterval: 30
  maxReplicaCount: 100
  triggers:
  - type: kafka
    metadata:
      bootstrapServers: kafka.svc:9092
      consumerGroup: batch-group
      topic: batch-jobs
      lagThreshold: "50"
```

## Best Practices

1. Start with conservative scaling parameters and adjust as needed
2. Set appropriate scaling thresholds based on your application's capacity
3. Consider the cold start time of your applications when scaling from zero
4. Use authentication secrets securely (reference as environment variables)
5. Monitor KEDA's behavior during initial deployment
6. Set reasonable cooldown periods to prevent thrashing
7. Use appropriate scaling metrics that correlate with actual load

## Advantages of KEDA

1. Scales based on event sources beyond CPU/memory
2. Can scale to zero to save resources when idle
3. Integrates with numerous external systems
4. Works alongside existing Kubernetes autoscaling
5. Reduces operational costs by matching resources to actual demand
6. Ideal for event-driven and serverless architectures on Kubernetes

KEDA is particularly valuable for event-driven applications, message queue processors, batch jobs, and applications with variable workloads that might benefit from scaling to zero during idle periods.
