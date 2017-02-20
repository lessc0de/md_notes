# [Zookeeper](http://zookeeper.apache.org/doc/r3.4.9/zookeeperOver.html)

## Zookeeper: A Distributed Coordination Service for Distributed Applications

Zookeeper is a distributed __coordination service__ for distributed applications. It exposes a set of primitives to implement services for:

* synchronization
* configuration maintenance
* groups
* naming

## Design goals

### Simple

Distributed processes coordinate with each other through a __shared hierarchical namespace__ which is organized similarly to a standard file system. Files and directories correspond to `znode`s (in Zookeeper parlance). Unlike a typical file system, the namespace is kept in-memory to achieve high throughput & low latency.

### Replicated

Like the distributed processes it coordinates, Zk itself is intended to be replicated over a set of hosts called an _ensemble_.

![](http://zookeeper.apache.org/doc/r3.4.9/images/zkservice.jpg)

The servers that make up the _Zookeeper service_ itself must all know about each other. They maintain an in-memory image of state, along with transaction logs and snapshots in a persistent store. As long as a majority of the servers are available (quorum), the Zk service will be available.

Clients connect to a single Zk server. The client maintains a TCP connection to:

* send requests
* get responses
* watch events
* send heart beats

If the TCP connection breaks, the client will connect to a different server.

### Ordered

Each _update_ is stamped that reflects the order of all Zk transactions. This order can be used to implement synchronization primitives.

### Fast & Scalable

In "read-dominant" workloads, Zk apps can run on thousands of machines. It performs best when there are more reads than writes, at ratios of around 10:1.

## Data model and the hierarchical namespace

A __name__ is a sequence of path elements separated by `/`, e.g. `/app1/p_1`

![](http://zookeeper.apache.org/doc/r3.4.9/images/zknamespace.jpg)

### Nodes and ephemeral nodes

`znode`s (also referred to as just "nodes") correspond to "files" or "directories" in the Zk namespace. Unlike in standard file systems, a node can have data associated with it. It can also have children. It's like how a file can also be a directory.

`znode`s maintain a stat structure that includes __version numbers__ for

* data changes
* ACL changes
* timestamps

to allow cache validations and coordination updates. The __version number__ increases each time a `znode`'s data changes. For instance, a whenever a client retrieves data, it also receives the version of the data.

Data stored in the `znode` is read and written atomically.

Zk also has a notion of __ephemeral nodes__. These `znodes` exist so long as the (client) session that created it is active; the node is deleted when the session ends.

#### [Example usage of ephemeral nodes](http://www.palladiumconsulting.com/2014/05/zookeeper-1-ephemeral-nodes/)

When a data server (our Zk client) creates a node, it can create it as ephemeral. So long as that client is alive (as determined by a heartbeat built into the Zookeeper protocol), the node is kept alive. If the Zk server ever loses the heartbeat, it will delete the relevant node. Now without any extra work on your part, the service registry is kept up to date, even in the face of recalcitrant servers.

### Conditional updates and watches

Clients can set a __watch__ on a `znode`. A watch will be triggered and removed when the `znode` changes (i.e. data changes). When a watch is triggered, the client is notified about the change. Also, if the client session to Zk abruptly ends, the client will receive a notification.

### Guarantees

Since Zookeeper's goal is to be a basis for the construction of more complicated services, such as synchronization, it provides a set of guarantees.

* Sequential Consistency - _Updates from a client will be applied in the order that they were sent._
* Atomicity - Updates either succeed or fail. No partial results.
* Single System Image - A client will see the same view of the service regardless of the server that it connects to.
* Reliability - Once an update has been applied, it will persist from that time forward until a client overwrites the update.
* Timeliness - The clients' view of the system is guaranteed to be up-to-date within a certain time bound.

### Simple API

* `create`: creates a node at a location in the tree
* `delete`: deletes a node
* `exists`: tests if a node exists at a location
* `get data`: reads the data from a node
* `set data`: writes data to a node
* `get children`: retrieves a list of children of a node
* `sync`: waits for data to be propagated
