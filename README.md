#Apache Hadoop 2.7.1 Docker image

Hadoop in Docker with Networking and Persistent Storage

The Portworx docker-hadoop project is a collection of Docker Images for running a multi-node Hadoop cluster. Whereas other Hadoop Docker Images are meant to be run in a single image, or one host, these images can be used in a multi-host environment with Docker 1.9 networking.

_Currently the only Hadoop version supported is 2.7.1_

The Images also support Portworx Persistent Storage layer for Docker. With Persistent Storage Volumes the Namenode and all Datanodes can be stopped, restarted, migrated and cloned while retaining all data.

_Volumes are currently configured for /hdfs/volume1_

# The Images

* **portworx/namenode**

  The namenode and secondary-namenode services run on this image. At startup it will configure itself with the hostname of the instance.

* **portworx/yarn**

  The Resource Manager and Job History Server run on this image.  Requires the HADOOP_HOST_NAMENODE environment variable to be set to the hostname of the Namenode instance at startup. Otherwise will set the short hostname to "namenode". 

* **portworx/datanode**

  The Datanode and NodeManager services run on this image.  At startup two environment variables are required: 1) HADOOP_HOST_NAMENODE environment variable to be set to the hostname of the Namenode; HADOOP_HOST_YARN variable set to the hostname of the Yarn (Resource Manager) instance.


# Pull the image

These images are all available from the Docker Hub automated build repository.

```
docker pull portworx/namenode:2.7.1
```

```
docker pull portworx/yarn:2.7.1
```

```
docker pull portworx/datanode:2.7.1
```


# Start a cluster

Make sure you have pulled all three images. Also make sure that SELinux is disabled on the host. 

_All of the images come bundled with the same set of SSH keys for user equivalence. Note that this setup isn't meant to be run in secure environments._

To create a cluster with a Namenode, Yarn server, and 3 Datanode’s issue the following docker run commands. 

```
docker network create hadoopnet
```

```
docker run -itd -p 50070:50070 --net=hadoopnet --name namenode portworx/namenode
```

```
docker run -itd -p 8088:8088 -p 19888:19888 --net=hadoopnet --name hadoop-yarn -e HADOOP_HOST_NAMENODE=[namenode] portworx/yarn
```

```
docker run -itd --net=hadoopnet --name hadoop-node1 -e HADOOP_HOST_NAMENODE=[namenode] -e HADOOP_HOST_YARN=[yarn] portworx/datanode
```

```
docker run -itd --net=hadoopnet --name hadoop-node2 -e HADOOP_HOST_NAMENODE=[namenode] -e HADOOP_HOST_YARN=[yarn] portworx/datanode
```

```
docker run -itd --net=hadoopnet --name hadoop-node3 -e HADOOP_HOST_NAMENODE=[namenode] -e HADOOP_HOST_YARN=[yarn] portworx/datanode
```



## Viewing Web Interfaces

```
Namenode - http://[namenode]:50070
```

```
Resource Manager - http://[yarn]:8088
```

```
Job History - http://[yarn]:19888
```



#Running Hadoop examples:

docker exec -it [yarn instance] /bin/bash

Set PATH=$PATH:/usr/local/hadoop/bin 
HADOOP_PREFIX=/usr/local/hadoop


```
cd $HADOOP_PREFIX
hadoop jar $HADOOP_PREFIX/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.1.jar teragen 10000 /teragen

hadoop jar $HADOOP_PREFIX/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.1.jar randomwriter randomout
# Check output
bin/hdfs dfs -cat randomout/*
```


## Acknowledgements & Support

Much of this work is based on the sequenceiq/hadoop-docker image. 

If you have questions or would like to contribute to this project send an email to garry@portworx.com

