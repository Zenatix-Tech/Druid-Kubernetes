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

#purging/deleting
kubectl delete -f k8s/druid
kubectl delete -f k8s/zookeeper
kubectl delete -f k8s/postgres

```

**Commands**
```bash
kubectl exec -it druid-middlemanager-0 bash -n druid

mkdir /var/druid/tmp

```

**Test Ingestion Spec**
```json
{
  "type" : "index",
  "spec" : {
    "dataSchema" : {
      "dataSource" : "wikipedia",
      "parser" : {
        "type" : "string",
        "parseSpec" : {
          "format" : "json",
          "dimensionsSpec" : {
            "dimensions" : [
              "channel",
              "cityName",
              "comment",
              "countryIsoCode",
              "countryName",
              "isAnonymous",
              "isMinor",
              "isNew",
              "isRobot",
              "isUnpatrolled",
              "metroCode",
              "namespace",
              "page",
              "regionIsoCode",
              "regionName",
              "user",
              { "name": "added", "type": "long" },
              { "name": "deleted", "type": "long" },
              { "name": "delta", "type": "long" }
            ]
          },
          "timestampSpec": {
            "column": "time",
            "format": "iso"
          }
        }
      },
      "metricsSpec" : [],
      "granularitySpec" : {
        "type" : "uniform",
        "segmentGranularity" : "day",
        "queryGranularity" : "none",
        "intervals" : ["2015-09-12/2015-09-13"],
        "rollup" : false
      }
    },
    "ioConfig" : {
      "type" : "index",
      "firehose" : {
        "type" : "local",
        "baseDir" : "quickstart/tutorial/",
        "filter" : "wikiticker-2015-09-12-sampled.json.gz"
      },
      "appendToExisting" : false
    },
    "tuningConfig" : {
      "type" : "index",
      "maxRowsPerSegment" : 5000000,
      "maxRowsInMemory" : 25000
    }
  }
}
```

```json
{"type": "kafka", "dataSchema": {"dataSource": "live-Samhi", "parser": {"type": "string", "parseSpec": {"format": "json", "timestampSpec": {"column": "time", "format": "auto"}, "dimensionsSpec": {"spatialDimensions": [{"dimName": "coordinates", "dims": ["site_latitude", "site_longitude"]}], "dimensions": ["uuid", "path", {"name": "reading", "type": "float"}, "stream_path", "stream_uuid", "device_tags", "device_is_virtual", "device_id", "device_display_name", "device_path", "device_type", "device_category", "physical_parameter_display", "physical_parameter_unit", "physical_parameter_type", "physical_parameter_name", "site_longitude", "site_latitude", "site_name", "customer_name", "site_Display_Name", "metric_tod_metadata", "metric_operational_metadata", "live_parent", "live_region", "live_purpose", "live_room_number", "live_pie_breakup", "live_room_direction"]}}}, "metricsSpec": [{"type": "count", "name": "count"}], "granularitySpec": {"type": "uniform", "segmentGranularity": "HOUR", "rollup": false}}, "tuningConfig": {"type": "kafka", "maxSavedParseExceptions": 1000, "forceExtendableShardSpecs": true}, "ioConfig": {"topic": "druid_telemetry_data_Samhi", "taskCount": 1, "replicas": 1, "taskDuration": "PT120S", "completionTimeout": "PT5M", "useEarliestOffset": true, "consumerProperties": {"bootstrap.servers": "ke-cp-kafka-headless.kafka:9092"}}}
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