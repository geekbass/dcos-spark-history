# Spark History on DC/OS 
The [Apache Spark History Server](https://spark.apache.org/docs/latest/monitoring.html) allows you to view the state of running and completed Spark jobs. This project allows for users to run any form of distributed filesystem (EFS, NFS, NAS, etc...) as the underlying storage instead of the default HDFS. Prefeably the FS would be available across all Private Agents on the cluster that Spark runs its jobs. We are also using the default Spark container provided by Mesosphere so that it matches the version of Spark being run on the cluster. This will need to be periodically updated as needed.

You can access the UI at: https://DCOS-URL/service/spark-history

### Deploying the History Server on DC/OS
1) You must ensure that the FS that you are going to use is already created and given correct permissions to the SPARK_USER (root default). Modify the json to fit your need: id, SPARK_HISTORY_OPTS, volumes, etc...

- The volumes must be visible from the host and mounted to container RW

2) Deploy...
```sh
dcos marathon app add spark-history.json
```

### Logging to Spark History
Append the following in your Spark Submit to log to History Server

```sh
--conf spark.eventLog.enabled=true #Default is False
--conf spark.eventLog.dir=file:///NETWORK/FS/YOU/CREATED/ #Specifies location on the nodes where network FS exists
--conf spark.mesos.executor.docker.volumes=/HOST/FS/PATH/YOU/CREATED:/CONTAINER/PATH/SAME/AS/HOST/PATH:rw #Mount host volume under container to write the logs
```

NOTE: The 'spark.mesos.executor.docker.volumes' is important. If you dont mount the Host volume into the container launched it not see the volume. You will see an error about the filesystem not existing. 

Example using the DC/OS CLI:
```sh
dcos spark run --submit-args="\
--conf spark.eventLog.enabled=true  \
--conf spark.eventLog.dir=file:///mnt/network-fs/spark-history  \
--conf spark.mesos.executor.docker.volumes=/mnt/network-fs/spark-history:/mnt/network-fs/spark-history:rw \
--class org.apache.spark.examples.SparkPi https://downloads.mesosphere.com/spark/assets/spark-examples_2.11-2.0.1.jar 30"
```

### Some notes
- For custom Spark History settings, you must pass them as a value in the SPARK_HISTORY_OPTS key. You can find Spark History options [here](https://spark.apache.org/docs/latest/monitoring.html#spark-configuration-options). 
- Default deployment will clearing up logs that are older than 7 days
- 18080 as bridge to docker and host with a LB VIP. 
- Labels so we can access the service from the DCOS UI: DCOS_SERVICE_SCHEME, DCOS_SERVICE_NAME, DCOS_SERVICE_PORT_INDEX