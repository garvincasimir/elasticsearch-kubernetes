# Elasticsearch on kubernetes
Sample repo for running Elasticsearch on Kubernetes using StatefulSets

To deploy:
```
kubectl apply -f https://raw.githubusercontent.com/garvincasimir/elasticsearch-kubernetes/master/es-cluster.yaml
```
 ### Controllers
This setup features a pair of StatefulSets, one for the master nodes and one for the data nodes. 

### Volumes
The data mounts are backed by emptyDir volumes but can easily be swapped out with something more durable.

### Services
Both node types use the DNS name of the headless master service for discovery. The data service is used to interact with the cluster http endpoints.

### Configuring Elasticsearch
The Elasticsearch config values are managed in ConfigMaps. These map to the node types and are mounted into the elasticsearch config folder on a per-file basis. This is so config files that come with the image will not be unintentionally overwritten. Please view the [official Elastic.io docs](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html) for more details on configuring Elasticsearch.

### Pods
Each pod includes an init container and a container running the [official elasticsearch image](https://www.docker.elastic.co). The init container is used to set *vm.max_map_count*. Please see the[docs](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html#docker-cli-run-prod-mode) for further details on this and other production settings. I recommend pre-configuing your nodes and ommitting this init container. 

### Resource Limits
Use data.jvm.options to adjust the JVM heap size. This must be done serpareately for each node type.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: es-master-cfg
data:
  jvm.options: |
    -Xms256m
    -Xmx256m
```

### Scaling
When scaling the master nodes remember to change the [mininum master nodes](https://www.elastic.co/guide/en/elasticsearch/guide/1.x/_important_configuration_changes.html#_minimum_master_nodes) setting to a suitable value. New nodes will join the cluster automatically. 

If you need to scale compute and storage separately consider adding a Deployment controller for Elasticsearch [coordinating only nodes](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#coordinating-only-node).

### Other considerations
There are many other features this cluster can benefit from such as:
  * Automated Snapshots using timed tasks
  *  Pod disruption budgets
  *  Pod Affinity