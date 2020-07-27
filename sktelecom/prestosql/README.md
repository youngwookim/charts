# PrestoSQL cluster on Kubernetes with Helm

## Prerequisite

- kubectl (<https://kubernetes.io/docs/tasks/tools/install-kubectl>)
- Helm (<https://helm.sh/docs/using_helm/#installing-helm>)
- Configure kubectl to connect to the Kubernetes cluster.

## How to setup a Presto cluster for demo

### Start Presto with Helm

Below command will start one presto coordinator and one presto worker as an example.

```bash
helm install prestosql -n prestosql .
```

#### To check deployment status

```bash
kubectl get all -n prestosql
```

### Sample queries to execute

- List all catalogs

```
presto:default> show catalogs;
```

- List All tables

```
presto:default> show tables;
```

- Show schema

```
presto:default> DESCRIBE catalog1.dontcare.airlinestats;
```
```
        Column        |  Type   | Extra | Comment
----------------------+---------+-------+---------
 flightnum            | integer |       |
 origin               | varchar |       |
 quarter              | integer |       |
 lateaircraftdelay    | integer |       |
 divactualelapsedtime | integer |       |
......

```

Then you could verify the new worker nodes are added by:

```bash
presto:default> select * from system.runtime.nodes;
               node_id                |        http_uri        |      node_version      | coordinator | state
--------------------------------------+------------------------+------------------------+-------------+--------
 d7123f72-bdd4-417e-adc2-9cfed5bbb12c | http://10.1.0.165:8080 | 0.236-SNAPSHOT-dbf3a80 | true        | active
 fe9b8813-7bfc-4b4e-aa84-9d50c5e87c57 | http://10.1.0.166:8080 | 0.236-SNAPSHOT-dbf3a80 | false       | active
(2 rows)

```

## Configuring the Chart

The chart can be customized using the following configurable parameters:

| Parameter                                      | Description                                                                                                                                                                | Default                                                            |
|------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------|
| `image.repository`                             | Presto Container image repo                                                                                                                                                 | `gcr.io/sktaic-datahub/prestosql`                                                |
| `image.tag`                                    | Presto Container image tag                                                                                                                                                  | `339`                                                   |
| `image.pullPolicy`                             | Presto Container image pull policy                                                                                                                                          | `IfNotPresent`                                                     |
|------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------|
| `coordinator.name`                              | Name of Presto Coordinator                                                                                                                                                   | `coordinator`                                                       |
| `coordinator.port`                              | Presto Coordinator port                                                                                                                                                      | `8080`                                                             |
| `coordinator.replicaCount`                      | Presto Coordinator replicas                                                                                                                                                  | `1`                                                                |
| `coordinator.query.maxMemory`                          | Max Memory Usage per Query                                                             | `4GB`                                       |
| `coordinator.query.maxMemoryPerNode`                          | Max Memory Usage per Node per Query                                                            | `1GB`                                       |
| `coordinator.query.maxTotalMemoryPerNode`                          | Max Total Memory Usage per Node                                                             | `2GB`                                       |
| `coordinator.discovery.serverEnabled`                          | Enable Discovery Server                                                                                                                                                             | `true`                                                 |
| `coordinator.discovery.uri`                          | Discovery Server URI                                                                                                                                                            | `http://presto-coordinator:8080`                                                 |
| `coordinator.jvm`                          | Coordinator JVM Configs                                                                                                                                                             | `-server -Xmx16G -XX:+UseG1GC -XX:G1HeapRegionSize=32M -XX:+UseGCOverheadLimit -XX:+ExplicitGCInvokesConcurrent -XX:+HeapDumpOnOutOfMemoryError -XX:+ExitOnOutOfMemoryError`                                                             |
| `coordinator.log`                          | Log File Content                                                                                                                                                             | `com.facebook.presto=INFO`                                                             |
| `coordinator.node.environment`                          | Node Environment                                                                                                                                                             | `production`                                                             |
| `coordinator.node.id`                          | Node ID (Default is empty, Presto will assign automatically)                                                                                                                                                             | ``                                                             |
| `coordinator.node.dataDir`                          | Data Directory                                                                                                                                                             | `/home/presto/data`                                                             |
| `coordinator.node.schedulerIncludeCoordinator`                          |  If Schedule Query on Coordinator                                                                                                                                                            | `true`                                                             |
| `coordinator.persistence.enabled`               | Use a PVC to persist Presto Coordinator Data                                                                                                                                 | `true`                                                             |
| `coordinator.persistence.accessMode`            | Access mode of data volume                                                                                                                                                 | `ReadWriteOnce`                                                    |
| `coordinator.persistence.size`                  | Size of data volume                                                                                                                                                        | `4G`                                                               |
| `coordinator.persistence.storageClass`          | Storage class of backing PVC                                                                                                                                               | `""`                                                               |
| `coordinator.service.port`                      | Service Port                                                                                                                                                               | `8080`                                                             |
| `coordinator.external.enabled`                  | If True, exposes Presto Coordinator externally                                                                                                                               | `true`                                                            |
| `coordinator.external.type`                     | Service Type                                                                                                                                                               | `LoadBalancer`                                                     |
| `coordinator.external.port`                     | Service Port                                                                                                                                                               | `8080`                                                             |
| `coordinator.resources`                         | Presto Coordinator resource requests and limits                                                                                                                              | `{}`                                                               |
| `coordinator.nodeSelector`                      | Node labels for controller pod assignment                                                                                                                                  | `{}`                                                               |
| `coordinator.affinity`                          | Defines affinities and anti-affinities for pods as defined in: <https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity> preferences | `{}`                                                               |
| `coordinator.tolerations`                       | List of node tolerations for the pods. <https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/>                                                           | `[]`                                                               |
| `coordinator.podAnnotations`                    | Annotations to be added to controller pod                                                                                                                                  | `{}`                                                               |
| `coordinator.updateStrategy.type`               | StatefulSet update strategy to use.                                                                                                                                        | `RollingUpdate`                                                    |
|------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------|
| `worker.name`                              | Name of Presto Worker                                                                                                                                                   | `worker`                                                       |
| `worker.port`                              | Presto Worker port                                                                                                                                                      | `8080`                                                             |
| `worker.replicaCount`                      | Presto Worker replicas                                                                                                                                                  | `1`                                                                |
| `worker.query.maxMemory`                          | Max Memory Usage per Query                                                             | `8GB`                                       |
| `worker.query.maxMemoryPerNode`                          | Max Memory Usage per Node per Query                                                            | `4GB`                                       |
| `worker.query.maxTotalMemoryPerNode`                          | Max Total Memory Usage per Node                                                             | `8GB`                                       |
| `worker.discovery.uri`                          | Discovery Server URI                                                                                                                                                            | `http://presto-coordinator:8080`                                                 |
| `worker.jvm`                          | Worker JVM Configs                                                                                                                                                             | `-server -Xmx64G -XX:+UseG1GC -XX:G1HeapRegionSize=32M -XX:+UseGCOverheadLimit -XX:+ExplicitGCInvokesConcurrent -XX:+HeapDumpOnOutOfMemoryError -XX:+ExitOnOutOfMemoryError`                                                             |
| `worker.log`                          | Log File Content                                                                                                                                                             | `com.facebook.presto=INFO`                                                             |
| `worker.node.environment`                          | Node Environment                                                                                                                                                             | `production`                                                             |
| `worker.node.id`                          | Node ID (Default is empty, Presto will assign automatically)                                                                                                                                                             | ``                                                             |
| `worker.node.dataDir`                          | Data Directory                                                                                                                                                             | `/home/presto/data`                                                             |
| `worker.persistence.enabled`               | Use a PVC to persist Presto Worker Data                                                                                                                                 | `true`                                                             |
| `worker.persistence.accessMode`            | Access mode of data volume                                                                                                                                                 | `ReadWriteOnce`                                                    |
| `worker.persistence.size`                  | Size of data volume                                                                                                                                                        | `4G`                                                               |
| `worker.persistence.storageClass`          | Storage class of backing PVC                                                                                                                                               | `""`                                                               |
| `worker.service.port`                      | Service Port                                                                                                                                                               | `8080`                                                             |
| `worker.resources`                         | Presto Worker resource requests and limits                                                                                                                              | `{}`                                                               |
| `worker.nodeSelector`                      | Node labels for controller pod assignment                                                                                                                                  | `{}`                                                               |
| `worker.affinity`                          | Defines affinities and anti-affinities for pods as defined in: <https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity> preferences | `{}`                                                               |
| `worker.tolerations`                       | List of node tolerations for the pods. <https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/>                                                           | `[]`                                                               |
| `worker.podAnnotations`                    | Annotations to be added to controller pod                                                                                                                                  | `{}`                                                               |
| `worker.updateStrategy.type`               | StatefulSet update strategy to use.                                                                                                                                        | `RollingUpdate`                                                    |
|------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------|

Specify parameters using `--set key=value[,key=value]` argument to `helm install`

Alternatively a YAML file that specifies the values for the parameters can be provided like this:

```bash
helm install --name prestosql -f values.yaml .
```
