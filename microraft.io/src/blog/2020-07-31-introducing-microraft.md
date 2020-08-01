
# Introducing MicroRaft

July 31, 2020 | Ensar Basri Kahveci

___Disclaimer: MicroRaft is a project I develop in my free time. It is not
affiliated, associated, endorsed by, or in any way officially connected with my
current employer Facebook, or any of its subsidiaries or its affiliates.___

I am pleased to announce the first public release of MicroRaft! MicroRaft is a
feature-complete, stable, production-grade and, most importantly, open-source
implementation of the <a href="https://raft.github.io/" target="_blank">Raft
consensus algorithm</a> in Java. You can use MicroRaft to build highly available
and strongly consistent data, metadata and coordination services. The source
code is available at <a href="https://github.com/MicroRaft/MicroRaft">Github</a>
with the Apache 2 License.

MicroRaft is a complete implementation of the Raft consensus algorithm. It
implements the leader election, log replication, log compaction (snapshotting),
and cluster membership change components. Additionally, it realizes <a
href="https://microraft.io/#features" target="_blank">a rich set of
optimizations and enhancements</a> to allow developers to run Raft clusters in a
reliable and performant manner, and tune its behaviour based on their needs.

MicroRaft works on top of a minimalistic and modular design. It is a single
lightweight JAR with a few hundred KBs of size and only a logging dependency. It
contains an isolated implementation of the Raft consensus algorithm, and a set
of accompanying interfaces to run the algorithm in a multi-threaded and
distributed environment. These interfaces surround the Raft consensus algorithm,
and abstract away the concerns of persistence, thread-safety, serialization,
networking and actual state machine logic. Developers are required to implement
these interfaces to build *CP* distributed systems on top of MicroRaft.

## Use cases

You can use MicroRaft to build highly available and strongly consistent data,
metadata and coordination services. Here are some common examples:

* distributed key-value store where each partition / shard is maintained by a
  separate Raft cluster
* control plane or coordination cluster
* leader election mechanisms
* group membership management systems
* distributed locks
* distributed transaction managers
* distributed resource schedulers MicroRaft contains the Raft consensus
  algorithm behind well-defined and minimal abstractions so that these systems
  can be built without intertwining their high-level logic with Raft logic.

## Main Abstractions

The following figure depicts MicroRaft's main abstractions. `RaftNode` runs the
Raft consensus algorithm with the Actor model. Clients talk to the leader
`RaftNode` in order to replicate their operations. They can talk to either the
leader or follower `RaftNode`s to run queries with different consistency
guarantees. `RaftNode` uses `RaftModelFactory` to create Raft log entries,
snapshot entries, and Raft RPC request and response objects. These objects are
defined under an interface, `RaftModel`, so that developers can implement them
using their favorite serialization framework, such as Protocol Buffers, Avro, or
Thrift. Each `RaftNode` talks to its own `Transport` object to communicate with
the other `RaftNode`s. For instance, if the `RaftModel` interfaces are
implemented with Protocol buffers, a `Transport` implementation can internally
use gRPC to communicate with the other `RaftNode`s. `RaftNode` also uses
`RaftStore` to persist Raft log entries and `StateMachine` snapshots to stable
storage. `StateMachine` enables users to implement arbitrary services, such as
an atomic register or a key-value store, and execute operations on them. If it
is a key-value store, then clients can replicate operations like `get`, `set`,
`delete`, etc. Once a log entry is committed, i.e, successfully replicated to
the majority of the Raft group, the operation it contains is passed to
`StateMachine` for execution andthe leader `RaftNode` returns the output of the
execution to the client.

![Architectural overview of a Raft group](/img/microraft_architectural_overview.png){: style="height:592px;width:800px"}

## Getting started

Run the following command on your terminal for a sneak peek at MicroRaft. It
starts a 3-node local Raft group (a Raft cluster in MicroRaft terms), elects a
leader, and commits a number of operations.

```shell
$ git clone https://github.com/MicroRaft/MicroRaft.git && cd MicroRaft && ./mvnw clean test -Dtest=io.microraft.tutorial.OperationCommitTest -DfailIfNoTests=false -Ptutorial
```

Follow the [tutorial](/docs/tutorial-building-an-atomic-register) to learn how
to build an atomic register on top of MicroRaft and for full details, check out
MicroRaft's [APIs and main abstractions](/docs/apis-and-main-abstractions).

## Ode to open source

I couldn't have brought MicroRaft into the daylight without the power of open
source. MicroRaft originates from [the Raft
implementation](https://github.com/hazelcast/hazelcast/tree/master/hazelcast/src/main/java/com/hazelcast/cp/internal/raft)
that powers Hazelcast IMDG's CP Subsystem module. I was one of the main
contributors of the CP Subsystem and was thinking about converting its Raft code
into a separate library back in 2019. However, we were very busy with developing
new features and I didn't have the time to try my idea. I left Hazelcast in
February, 2020. My plan was to chill for a couple of weeks before I start my new
job at Facebook in April, 2020. However, life happened and I had to push back my
relocation to London because of the global Covid-19 crisis. I started to think
about some side projects to amuse myself during my time at home and decided to
give this project a try.

<a href="https://twitter.com/mmdogan" target="_blank">Mehmet Dogan</a> and I
developed Hazelcast CP Subsystem's Raft code inside Hazelcast codebase, but we
isolated it from the rest of the Hazelcast code. It depends on Hazelcast for
networking, logging and testing. So I started by moving out the Raft code and
defining abstractions for the parts depending on Hazelcast. Then I implemented a
number of enhancements and improvements that you can see at the <a
href="https://github.com/MicroRaft/MicroRaft/commits/master"
target="_blank">commit history</a>.

MicroRaft carries on Hazelcast's heritage and is released with the Apache 2
License.

## What is next

I wrote down <a href="https://microraft.io/docs/roadmap/" target="_blank">a list
of future work</a> on MicroRaft. I am planning to work on them in my free time.
The list is tentative. I can change it based on the feedback and engagement of
developers who want to use MicroRaft.

MicroRaft is a new open source project. Any kind of contribution and feedback is
welcome! The development happens on <a
href="https://github.com/MicroRaft/MicroRaft" target="_blank">Github</a>. There
is also a <a
href="https://join.slack.com/t/microraft/shared_invite/zt-dc6utpfk-84P0VbK7EcrD3lIme2IaaQ"
target="_blank">Slack group</a> for discussions and questions. Last, you can
follow <a href="https://twitter.com/microraft" target="_blank">@MicroRaft</a> on
Twitter for announcements.