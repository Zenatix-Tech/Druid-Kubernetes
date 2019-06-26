# Druid-Kubernetes
Deploy latest druid on kubernetes on AKS

**Components**

|       Service       | Port |
|:-------------------:|:----:|
| druid-zookeeper     | 2181 |
| druid-coordinator   | 8081 |
| druid-overlord      | 8090 |
| druid-middlemanager | 8091 |
| druid-historical    | 8083 |
| druid-broker        | 8082 |
| druid-router        | 8888 |
| druid-postgresql    | 5432 |

**Commands**
```bash
# build docker image
docker-compose -f docker/docker-compose.yaml build
docker-compose -f docker/docker-compose.yaml push

# deploy druid on kubernetes
kubectl apply -f k8s/azure
kubectl apply -f k8s/postgres
kubectl apply -f k8s/zookeeper
kubectl apply -f k8s/druid

```