## Overview

### Spark Cluster in High Availability
Apache Spark™ is a fast and general purpose engine for large-scale data
processing. This bundle deploys Apache Spark in high availability standalone mode.

Two types of services are involved in a Spark deployment a) Workers b) Masters.
By default, a Spark cluster in standalone mode is resilient to Worker failures
(should a job fails due to a Worker failure, Spark will resubmit the same job to a
different Worker)

Master node failures need extra care and are not handled by Spark alone.
Spark employes Apache Zookeeper for leader election and some state storage.
In a cluster with multiple masters connected to the same Zookeeper deployment,
one Spark master will be elected as “leader” and the others will remain in standby mode.
In case the leader master fails, another master will take over then resume job scheduling.

### Usage

This bundle deploys three units of Apache Zookeeper (in HA) and three Spark units.
Every Spark unit is a potential master leader. 
Relating Zookeeper to Spark results in the later becoming highly available.

    juju deploy cs:bundle/apache-spark-ha

After deploying this bundle you can scale the Spark cluster to your needs by
adding and removing units:

    juju add-unit spark
    juju remove-unit <spark_unit_to_remove>

Please refer to [apache-spark](https://jujucharms.com/apache-spark/)
for instructions on how to submit jobs to the Spark cluster.


### Smoke-test HA

Spark units report "Ready (standalone HA)" in their status so if you need to
identify the node acting as master you need to query the Zookeeper deployment.

    juju ssh zookeeper/0
    zkCli.sh get /spark/master_status  

Navigate to http://{spark_master_ip_address}:8080 and note that the master
on this unit is marked as "ALIVE". Proceed to removing that unit:

    juju remove-unit <spark_master_unit>

Within two minutes a new master leader will be selected. By querying Zookeeper
again we can find which unit acts as the new leader.


## Contact Information

- <bigdata@lists.ubuntu.com>


## Help

- [Juju mailing list](https://lists.ubuntu.com/mailman/listinfo/juju)
- [Juju community](https://jujucharms.com/community)
