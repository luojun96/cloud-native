# CSI

## Ceph
Ceph can be used to deploy a Ceph File System. All Ceph Storage Cluster depoloyments begin with setting up each Ceph Node and then setting up the network.

A Ceph Storage Cluster requires the following:

- **Monitors:** A Ceph Monitor (ceph-mon) maintains maps of the cluster state, including the monitor map, manager map, the OSD map, the MDS map, and the CRUSH map. These maps are critical cluster state required for Ceph daemons to coordinate with each other. Monitors are also responsible for managing authentication between daemons and clients. At least three monitors are normally required for redundancy and high availability.
- **Managers:** A Ceph Manager daemon (ceph-mgr) is responsible for keeping track of runtime metrics and the current state of the Ceph cluster, including storage utilization, current performance metrics, and system load. The Ceph Manager daemons also host python-based modules to manage and expose Ceph cluster information, including a web-based Ceph Dashboard and REST API. At least two managers are normally required for high availability.
- **Ceph OSDs:** An Object Storage Daemon (Ceph OSD, ceph-osd) stores data, handles data replication, recovery, rebalancing, and provides some monitoring information to Ceph Monitors and Managers by checking other Ceph OSD Daemons for a heartbeat. At least three Ceph OSDs are normally required for redundancy and high availability.
- **MDSs:** A Ceph Metadata Server (MDS, ceph-mds) stores metadata on behalf of the Ceph File System (i.e., Ceph Block Devices and Ceph Object Storage do not use MDS). Ceph Metadata Servers allow POSIX file system users to execute basic commands (like ls, find, etc.) without placing an enormous burden on the Ceph Storage Cluster.

Ceph stores data as objects within logical storage pools. Using the **CRUSH**(Controlled Replication Under Scalable Hashing) algorithm, Ceph calculates which placement group (PG) should contain the object, and which OSD should store the placement group. The CRUSH algorithm enables the Ceph Storage Cluster to scale, rebalance, and recover dynamically.
### OSD
OSD stands for Object Storage Device in Ceph. It is a key component of the Ceph distributed storage system, responsible for storing and retrieving data. Each OSD is responsible for managing a specific set of objects and provides data replication and recovery services.

When a client writes data to Ceph, the data is broken down into objects and distributed across multiple OSDs. The OSDs replicate the data to ensure that there are multiple copies of each object, providing high availability and data durability. The OSDs also ensure that data is distributed evenly across the cluster, optimizing performance.


