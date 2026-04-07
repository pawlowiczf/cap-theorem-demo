## Exercise 1
CP - Consistency, Partition tolerance

After partitioning the cluster (isolating the Node 3), we still do have the quorum with remaining two nodes: $ Q=\frac{n+1}{2}=\frac{3+1}{2}=2 $.
It means, that the group with two nodes can still operate and handle the requests I send. Increasing the counter on the Node 1 and Node 2 works.
We do not receive any errors nor timeouts. The quorum and the consistency are maintained - the failing node does not influence the system behaviour.
On the other hand, increasing the counter on the Node 3 is unsuccessful - we receive the timeout. This is because the node is isolated - sent requests do not reach it.

## Exercise 2
CP - Consistency, Partition tolerance