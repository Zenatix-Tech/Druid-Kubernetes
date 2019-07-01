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

**curl requests**
```bash

# markUnused using interval to coordinator
curl -X 'POST' -H 'Content-Type:application/json' -d '{ "interval" : "2015-09-11T00:00:00.000Z/2015-09-14T00:00:00.000Z" }' http://localhost:8081/druid/coordinator/v1/datasources/deletion-tutorial/markUnused


# markUnused using segment ID to coordinator
curl -X 'POST' -H 'Content-Type:application/json' -d '{ "segmentIds": ["wikipedia_2015-09-12T00:00:00.000Z_2015-09-13T00:00:00.000Z_2019-06-26T10:39:34.338Z" ]}' http://localhost:8081/druid/coordinator/v1/datasources/deletion-tutorial/markUnused

# delete request to overlord
curl -X 'POST' -H 'Content-Type:application/json' -d '{ "type": "kill", "dataSource": "wikipedia", "interval" : "2015-09-11/2015-09-14" }' http://localhost:8090/druid/indexer/v1/task

# delete datasource to coordinator
curl -X 'DELETE' -H 'Content-Type:application/json' http://localhost:8081/druid/coordinator/v1/datasources/wikipedia

curl -X 'DELETE' -H 'Content-Type:application/json' http://localhost:8081/druid/coordinator/v1/datasources/wikipedia/segments/wikipedia_2015-09-12T00:00:00.000Z_2015-09-13T00:00:00.000Z_2019-06-26T10:39:34.338Z

https://druid.apache.org/docs/latest/tutorials/tutorial-delete-data.html

```