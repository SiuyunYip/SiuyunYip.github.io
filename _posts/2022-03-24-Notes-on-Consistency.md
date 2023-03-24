---

layout:     post

title:      Notes on Consistency

subtitle: 	Consistency in client & server side 

date:       2022-03-24

author:     Siuyun

header-img: img/consensus_algo.png

catalog: true

tags:

    - Distributed system

---





> There are two ways of looking at consistency:
>
> One is from the developer / client point of view; how they observe data updates. The second way is from the server side; how updates flow through the system and what guarantees systems can give with respect to updates.



# Client side consistency

Consider a storage system that is perceived as a blackbox and need to be shared by three processes (A & B & C). Theses processes are independent of each other and need to communicate the share information. The client side consistency is all about "how and when an observer (i.e., process A & B & C) sees updates made to a data object in the storage systems".



## Degree of Consistency

### Strong consistency

After the update completes, any subsequent access (by A, B or C) will return the updated value.

### Weak consistency

The system doesn't guarantee that the data subsequently accessed by (by A, B or C) is the updated one.

### Eventual consistency

> The storage system guarantees that if no new updates are made to the object eventually (after the inconsistency window closes) all accesses will return the last updated value.  



## Variations of Eventual consistency model

### Causal consistency

> If process A has communicated to process B that it has updated a data item, a subsequent access by process B will return the updated value and a write is guaranteed to supersede the earlier write. Access by process C that has no causal relationship to process A is subject to the normal eventual consistency rules.  

### Read-your-writes consistency

> Process A after it has updated a data item always accesses the updated value and never will see an older value. This is a special case of the causal consistency model.  

### Session consistency

> A process accesses the storage system in the context of a session. As long as the session exists, the system guarantees read-your-writes consistency. If the session terminates because of certain failure scenarios a new session needs to be created, and the guarantees do not overlap the sessions.  

### Monotonic read consistency

> If a process has seen a particular value for the object, any subsequent accesses will never return any previous values. 

### Monotonic write consistency

> The system guarantees to serialize the writes by the same process. Systems that do not guarantee this level of consistency are notoriously hard to program.  



# Server side consistency

## What is quorum protocol

> A quorum protocol is a set of rules that determine the minimum number of members of a group that must be present for a decision to be made. 

A subbranch of quorum protocl is Simple Majority. This protocol requires that more than half of the members of a group be present for a decision to be made. For example, if there are 10 members in a group, at least 6 members must be present for a decision to be made.



## Quorum protocols in Distributed data storage system

The quorum-based replication protocol is usually used in the case of data stored on multiple nodes (or replicas) for redundancy and fault tolerance purposes.

In this system, there are three parameters that are defined:

* N: the total number of nodes that store replicas of the data
* W: the number of replicas that need to acknowledge the receipt of an update (or write operation) before the update is considered complete
* R: the number of replicas that need to be accessed (or read from) when a data object is accessed through a read operation.

For example, if N is 5, W is 3, and R is 2, it means that there are 5 nodes storing replicas of the data, and when update is made to the data, at least 3 of those replicas need to acknowledge the receipt of the update before the update is considered complete. When a read operation is performed on the data, at least 2 of the replicas need to be accessed in order to retrieve the data.

These parameters can be adjusted to balance the trade-off between data consistency and system performance in a distributed data storage system.

### W+R > N

The condition W+R > N is often used in quorum-based replication protocols to ensure strong consistency. This condition means that the total number of replicas involved in a write or read operation (i.e., W+R) is greater than the total number of replicas storing the data (i.e., N). This ensures that there is an overlap in the replicas involved in different operations, which in turn ensures that all replicas eventually receive all updates made to the data.

Consider the following scenarios:

* Write operation: When a write operation is performed, at least W replicas need to be updated before the operation is considered successful. Since W+R > N, there must be some overlap between the set of replicas involved in this write operation and the set of replicas that may be involved in future read operations. This ensures that any subsequent read operation will access at least one replica that has been updated by the write operation.
* Read operation: Vice versa.

### R=1 & N=W

Optimized for the read case

### W=1 & R=N 

Optimized for the write case. However the durability is not guaranteed in the presence of failures.

### W < (N+1)/2

There is the possibility of conflicting writes. Conflicting writes occur when two or more clients attempt to write to the same piece of data at the same time, and the replicas involved in each write operation do not have a sufficient overlap. For example, suppose there are 5 replicas of a piece of data, and W is set to 2. If two clients attempt to write to the same piece of data at the same time, and each write operation involves a different set of two replicas, then it is possible for the data to become inconsistent, as different replication may contain different versions of the data.

### W+R <= N

It's vulnerable to reading from nodes that have not yet received the updates
