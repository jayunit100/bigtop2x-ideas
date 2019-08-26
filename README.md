# Problem

Installation of things has been commoditized by containers and K8s.  The more important
problems we have nowadays are around interoperation, learning, and integration of different
tools for different problems in the analytics space.

Modern data scientists need 'batteries included' frameworks that can be used to model and
address different types of analytics problems over time, which can replicate the integrated
functionality of AWS, GCP, and so on.

# Current Status

This repository currently integrates installation of a full analytics stack for kubernetes
with batteries included, including storage.

```
                       +----------------+
                       |                |    XXX           XXX          XXXXXX
                       |    NIFI        |XXXXX  XXX       XX  XXX     XXX    XX
                       |                |         XX    XXX     XX    X       XX
                       |                |          XXXXXX        XXXXXX        X
                       +-----+----------+                                     X
+-------------+              |                                                X
|             |              |                                                XXXXXX
|    Kafka    |              |                                                      XXXX
|             |              |                         +----------------+           XXXX
+-----+-------+              |                         |                |     XXXXXXX
      |                      |                         |  Zepplin       |    XX
      |               +------v------+                  |                |    XXXXXX
      +-------------->+             |                  |                |         X
                      |    Zookeeper+-------+          +-----------+----+         X
                      |             |       |                      |           X  X  XX
                      +-------------+       |                      |           XX X XX
                                            |                      |            XXXXX
                                            |                      |
                                            |                      |  +--------v------+
                                            v                      +> | Spark         |
                                    +-------+----------+---+          |               |
                                    |                  |   |          |               |
                                    |    Volume PRovisioner|          +---------------+
                                    |    (NFS or hostpath) |
                                    |                  |   |
                                    +-------------^----+---+ .            (Presto)
                                                  ^                          |
                                                  |                          |
                                                  |                          V
                                                  |                +---------------+
                                                  |                |               |
                                                  |                |               |
                                                  +----------------+   Minio       |
                                                                   |               |
                                                                   +---------------+
```

If all services are deployed succesfully, you ultimately will have an inventory looking like this:


```
$> kubectl get pods -n bigdata
NAME                                          READY   STATUS    RESTARTS   AGE
coordinator-56956c8d84-hgxvc                  1/1     Running   0          34s
fantastic-chipmunk-livy-5856779cf8-w8wlr      1/1     Running   0          3d1h
fantastic-chipmunk-master-55f5945997-mbvbm    1/1     Running   0          3d
fantastic-chipmunk-worker-5f7f468b8f-mwnmg    1/1     Running   1          3d1h
fantastic-chipmunk-worker-5f7f468b8f-zkbrw    1/1     Running   0          3d1h
fantastic-chipmunk-zeppelin-7958b9477-vv25d   1/1     Running   0          3d1h
hbase-hbase-master-0                          1/1     Running   0          4h4m
hbase-hbase-rs-0                              1/1     Running   2          4h7m
hbase-hbase-rs-1                              1/1     Running   1          4h5m
hbase-hbase-rs-2                              1/1     Running   0          4h4m
hbase-hdfs-dn-0                               1/1     Running   1          4h7m
hbase-hdfs-dn-1                               1/1     Running   0          4h5m
hbase-hdfs-dn-2                               1/1     Running   0          4h5m
hbase-hdfs-nn-0                               1/1     Running   0          4h7m
minio-7bf4678799-cd8qz                        1/1     Running   0          3d22h
my-kafka-0                                    1/1     Running   0          27h
my-kafka-1                                    1/1     Running   0          27h
my-kafka-2                                    1/1     Running   0          27h
nifi-0                                        4/4     Running   0          2d3h
nifi-zookeeper-0                              1/1     Running   0          2d3h
nifi-zookeeper-1                              1/1     Running   0          2d3h
nifi-zookeeper-2                              1/1     Running   0          2d3h
worker-565c7c858-pjlpg                        1/1     Running   0          34s
```

## Instructions!

1. First , install an NFS volume provisioner from the instructions storage/ directory
2. Then follow the other instructions in the storage README
3. Now, install components one by one from the README.md files in the processing/ directory.

This will yield the following analytics distro, all running in the bigdata namespace (make sure to use
`--namespace=bigdata` or similar on all `helm install` or `kubectl create` directives).  IF you mess anything up
do `helm list` (find your installation, i.e. XYZ) followed by `helm delete XYZ`  to clear out your components.

In particular, this repo modifies stock helm charts in a variety of ways to make things work together.

1. We don't use stable/spark because its *old*.  Instead we use microsofts spark, which comes integrated
with zepplin properly.
2. We use configmaps for configuration of *spark*.  For spark, this allows us to inject
different types of configuration stuff from the kuberentes level, rather then baking them into the image (note that 
you cant just inject a single file from a config map, b/c it overwrites the whole directory).  This allows us
to inject minio access properties into spark itself, while also injecting other config.
3. For Kafka, we config map the environment variables so that we can use the same zookeeper instance as 
NiFi.  
4. For Presto, the configuration parameters for workers/masters are all injected also via config map.  We use
a fork of https://github.com/dharmeshkakadia/presto-kubernetes for this change (PR's are submitted to make this upstream).
5. For minio there arent any major changes needed out of the box, except using emptyDir for storage if you dont have
a volume provisioner.

============================================ 

Notes and Ideas
 
# Inspiration 

Recently saw https://github.com/dacort/damons-data-lake.
- A problem set that is increasingly relevant: lots of sources, real time, unstructured warehouse/lake.
- No upstream plug-and-play alternative to cloud native services stack.
- Infrastructure, storage, networking is the hardest part.


