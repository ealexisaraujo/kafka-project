# Kafka Cluster en Kubernetes

Este proyecto implementa un cluster de Apache Kafka en Kubernetes utilizando la imagen de Bitnami. El cluster incluye Zookeeper para la gestión de la coordinación y un cliente Kafka para pruebas.

## Arquitectura

El proyecto está compuesto por:
- Cluster de Kafka con 3 brokers
- Servidor Zookeeper
- Cliente Kafka para pruebas
- Configuración SASL para seguridad

## Prerrequisitos

- Kubernetes cluster (versión 1.19+)
- kubectl configurado y funcionando
- Acceso a un StorageClass que soporte ReadWriteOnce (por defecto usa 'standard')

## Estructura del Proyecto

```bash
kafka-project/
├── README.md
├── client
│   ├── client-configmap.yaml
│   ├── kafka-client-pod.yaml
│   └── kafka-client-secret.yaml
├── kafka
│   ├── kafka-broker-config.yaml
│   ├── kafka-service.yaml
│   └── kafka-statefulset.yaml
├── namespace.yaml
└── zookeeper
    ├── zookeeper-service.yaml
    └── zookeeper-statefulset.yaml
```

## Instalación

1. Crear el namespace:

```bash
kubectl apply -f namespace.yaml
```

2. Desplegar Zookeeper:

```bash
kubectl apply -f zookeeper/
```

3. Desplegar Kafka:

```bash
kubectl apply -f kafka/
```

4. Desplegar el cliente Kafka:

```bash
kubectl apply -f client/
```

## Verificación del Despliegue

Verificar que todos los pods están corriendo:

```bash
kubectl get pods -n kafka
```

## Uso del Cliente Kafka

### Crear un Topic

```bash
kubectl exec -it -n kafka kafka-client -- kafka-topics.sh \
--create --topic test-topic \
--partitions 1 --replication-factor 3 \
--bootstrap-server kafka:9092
```

### Producir Mensajes

```bash
kubectl exec -it -n kafka kafka-client -- kafka-console-producer.sh \
--topic test-topic \
--bootstrap-server kafka:9092
```

### Consumir Mensajes

```bash
kubectl exec -it -n kafka kafka-client -- kafka-console-consumer.sh \
--topic test-topic \
--from-beginning \
--bootstrap-server kafka:9092
```

## Configuración

### Recursos del Cluster
- Kafka: 3 replicas con 1Gi de almacenamiento cada una
- Zookeeper: 1 replica con 2Gi de almacenamiento
- Recursos de CPU y memoria configurados en los StatefulSets

### Seguridad
El cluster utiliza autenticación SASL_PLAINTEXT con mecanismo SCRAM-SHA-256.

## Mantenimiento

### Escalado
Para escalar el número de brokers de Kafka:

```bash
kubectl scale statefulset kafka -n kafka --replicas=<número_deseado>
```

### Limpieza
Para eliminar todo el despliegue:

```bash
kubectl delete namespace kafka
```

## Notas Importantes

- Los volúmenes persistentes no se eliminan automáticamente al eliminar el namespace
- Asegúrate de tener suficientes recursos en tu cluster antes de desplegar
- La configuración actual está pensada para un entorno de desarrollo/pruebas

## Versiones

- Kafka: 3.9.0
- Zookeeper: 3.9.3
- Imágenes base: Debian 12