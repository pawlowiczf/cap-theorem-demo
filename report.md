## Exercise 1
CP - Consistency, Partition tolerance

After partitioning the cluster (isolating the Node 3), we still do have the quorum with remaining two nodes: $ Q=\frac{n+1}{2}=\frac{3+1}{2}=2 $. It means, that the group with two nodes can still operate and handle the requests I send. Increasing the counter on the Node 1 and Node 2 works. We do not receive any errors nor timeouts. The quorum and the consistency are maintained - the failing node does not influence the system behaviour. On the other hand, increasing the counter on the Node 3 is unsuccessful - we receive the timeout.

We couldn't receive response from Node 3, because the system cannot guarantee consistent data there (no quorum). On the other hand, Nodes 1 and 2 can continue serving requests, because they still form a quorum. Data remains consistent between these two nodes. In conclusion, in CP systems, a node can respond only if it can ensure consistency. Nodes 1 and 2 can communicate and achieve quorum, so they are allowed to serve requests. Node 3 cannot verify its data against the quorum, so it is blocked to avoid returning obsolete and inconsistent data.

After healing the network connections, the latest counter value is available on Node 3. Node 3 synchronizes its state with the cluster (thanks to Raft protocol).

## Exercise 2
CP - Consistency, Partition tolerance

In this case, the connection between all nodes is lost and the cluster becomes unavailable. Therefore, there is not a sufficient number of connected nodes to form a quorum (2). As described above, in this CP scenario, a node rejects the operation (counter incrementation or read) and returns the error "no quorum". This is expected behavior - the node cannot verify its data against the quorum and therefore does not perform the operation.

After the network connections are restored, the nodes synchronize their state with the cluster. However, because there was no quorum during the "blackout", no counter increments were performed, and the counter value remains the same as it was before the network partition.

## Exercise 3.
AP - Availability, Partition tolerance

This scenario differs from scenarios 1 and 2. Here, availability is prioritized over consistency. This means that all nodes serve requests, but the returned data may be obsolete or outdated. The data eventually converges to the latest state, but not immediately (eventual consistency). Increasing the counter value on one node becomes visible on the others after a brief period. After network partition, all nodes still serve requests (increasing the counter is possible on all nodes, even Node 3). As a result, different nodes may accept updates, which are resolved later when the network is restored. Once the network partition is resolved, the nodes exchange their updates and synchronize their states. After synchronization and conflict resolution, all nodes converge to the same final value. In our case, conflict resolution is performed with CRDT data structure, which provides eventual consistency.

## Exercise 4.
Which data structures -- AP or CP -- require the Raft algorithm? Why is this algorithm needed? After partition heal, what is the role of Raft?

Raft algorithm is required only on CP systems - it maintains strong consistency, even when some of the nodes are failing. After partition healing, Raft reconciles logs by making all nodes follow the leader's log, discarding any conflicting entries, and replicating the correct entries until every node has the same history. Without Raft (without consensus algorithm), soon the nodes in the system would have different, conflicting state. There would be no way to commit operations, elect new leader, replicate the correct entries etc. Raft takes care of consistency.

## Exercise 5.
Explain what it means that the PN Counter in Hazelcast provides Read-Your-Writes (RYW) and Monotonic reads guarantees and why they are session guarantees.

PN Counter operates in a weakly consistent, replicated environment where updates are propagated to other replicas asynchronously, meaning that at any given moment different nodes may hold different states of the counter. To address this inconsistency, the PN Counter provides two session guarantees as defined in "Session Guarantees for Weakly Consistent Replicated Data": Read-Your-Writes (RYW) and Monotonic Reads.

The RYW guarantee ensures that any read operation performed within a session will be directed to a replica that has already processed all writes made earlier in that same session, so a client will never observe a state that appears to predate its own updates. Monotonic Reads guarantee that successive reads within a session are always served by replicas that are at least as up-to-date as the replica used in the previous read.

Hazelcast enforces both guarantees by maintaining a vector clock on the client side, tracking the observed state after each operation and using it to select only sufficiently up-to-date replicas for subsequent calls. These are called session guarantees because they apply only within a single sequence of operations and hold only as long as a sufficiently up-to-date replica remains reachable.