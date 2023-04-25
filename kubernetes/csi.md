# CSI

## Ceph
### OSD
OSD stands for Object Storage Device in Ceph. It is a key component of the Ceph distributed storage system, responsible for storing and retrieving data. Each OSD is responsible for managing a specific set of objects and provides data replication and recovery services.

When a client writes data to Ceph, the data is broken down into objects and distributed across multiple OSDs. The OSDs replicate the data to ensure that there are multiple copies of each object, providing high availability and data durability. The OSDs also ensure that data is distributed evenly across the cluster, optimizing performance.


