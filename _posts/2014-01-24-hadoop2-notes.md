---
layout: default
title: Hadoop 2 Notes
---
#Hadoop 2 Notes

## New Software module layout
* Hadoop Commons
* HDFS
* YARN
* MapReduce

## Updated/New Config files
* core-site.xml  
* hdfs-site.xml   
* mapred-site.xml
* yarn-site.xml
* capacity-scheduler.xml  
* hadoop-policy.xml 
* Many Properties have been renamed (with old names deprecated).  Here is the full list:
  http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/DeprecatedProperties.html

## Command line tools updates
* hadoop fs  (many deprecated)
* hdfs dfs 
* hdfs admin
* hdfs haadmin

## HadoopV2 software components
* **NameNode** (same, but can be HA or Federated)
* **ResourceManager** (replaces part of JobTracker)
* **DataNode** (same)
* **NodeManager** (replace TaskTrackers)

## High Availability Name Node Components:
* Zookeeper process to detect NN failure and election process
* Primary NameNode (elected) -- writes to Journal nodes.
* Passive/Standby NameNode --  reads from Journal Nodes and maintains state/checkpoint/etc.
* Journal Manager Nodes (QJM) -- edits written to a quorum of journal nodes (not ZK, but can be Bookeeper).
* Automatic (via Zookeeper failover controller ZKFC) or manual via 'hdfs haadmin' command
* Journal Nodes enforce only one NN writer (using epoch time of when the NN was elected primary). If both NNs think they are active, the QJM will instruct one to stop.
* DataNodes send block reports to both NNs.

## HDFS Federation (horizontal NN scaling): 
* Multiple NameNodes in a cluster where each is responsible for different namespaces.  Create blockpool (BP-) on each datanode storage directories.  Oddly, this is not HA, and seems kinda difficult to make it so...
* The datanodes will register with each NN and send the appropriate block reports. 
* The datanodes need to know which NN to send blocks to.
* ViewFS used to provide a single logical Namespace for clients. 

## YARN / Next Gen data processing platform.
YARN is a cluster resource management framework that allows for the execution of a wide range of cluster applications, including batch (MRv2), interactive (TEZ), online (HBASE), streaming (Storm), etc.

###YARN Components:
* **ResourceManager** (master/SPOF -- performs the scheduling side of the "old" JT) -- Application management, scheduling, and security.
* **NodeManager** (runs on DNs) -- Provides "Container" resources to lease (containers to do the work).
* **ApplicationMaster** (runs within a container in the NodeManager -- performs the job execution management side of the "old" JT) -- A special Container to manage the execution of a single job.  ApplicationManagers are dynamically created (by the ResourceManager) at the start of a job and ensure the job gets its resources (from the ResourceManager), and handles container failures.

###YARN application Lifecycle/Workflow
1. Client Submits application request to the ResourceManager.
2. ResourceManager responds with ApplicationID to Client
3. The Client submits the application details to the ResourceManager (including user, queue, dependencies, resource requests + container launch context including resources req, lanuch commands, delegation tokens, etc)
4. The ResourceManager starts the ApplicationMaster (on a given Node Manager).
5. The ApplicationMaster requests resources/capabilities from the ResourceManager.
6. The ResourceManager provides/schedules container resources
7. The ApplicationManager issues container launch requests on the various NodeManagers. (rinse and repeat)
8. Once all tasks are complete, the ApplicationManager persists job info to HDFS and terminates.
 
 Note: The ApplicationManager is responsible for handing container failures/retries.  The ResourceManager will restart the ApplicationManager if it fails (The job will resume)

###YARN Misc notes:
* No longer specify # of map / reduce slots on a given server.  The server's resources are dynamically allocated based on the configured size of a given application container.  No more "reserved" map / reduce slot.
* Performance characteristics vs MRv1. -- What is the overhead of the container-based management when running primarily MapReduce applications?  Also, what is the impact of no longer having a defined set of map and reduce slots.
* Other container applications that may be useful to us -- TEZ (interactive), Storm (Streaming)
* New ResourceManager GUI, which proxies to NodeManagers (ApplicationMaster) for job details, counters, etc...

## Backups/Recovery updates
* HDFS snapshot ( need to administratively allow directory snapshot, and then you make the snapshot)
* Using Oozie and/or Falcon to orchestrate backup/recovery workflow. 

## Misc Notes:
* Security features -- Encryption in transit, Kerberos, etc...
* Oozie is a workflow scheduler that can coordianate the execution of various sequential task including map reduce, streaming, pig/hive, distcp, and other 
* HDFS Web Services endpoints on various components, JMX stats, etc...
* HDFS NFS Gateway -- Supports nfs3 and can be use to browse, upload, download, and stream files into HDFS -- No append mode.

## Resources:
* http://hadoop.apache.org/docs/stable/
* http://blog.cloudera.com/blog/2013/10/apache-hadoop-2-is-here-and-will-transform-the-ecosystem/
* http://blog.cloudera.com/blog/2013/11/migrating-to-mapreduce-2-on-yarn-for-users/
* http://blog.cloudera.com/blog/2013/11/migrating-to-mapreduce-2-on-yarn-for-operators/

