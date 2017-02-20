# [Strong consistency models](https://aphyr.com/posts/313-strong-consistency-models)

Network partitions are bound to happen.

* Switches
* NICs
* host hardware
* OS
* disk
* virtualization layer
* language runtime exception

These all conspire to

* delay
* drop
* duplicate
* re-order

our messages.

In an uncertain world, we want to maintain a sense of _intuitive correctness_.

### Correctness

A __system__ is comprised of __state__. Some __operations__ applied to that state transform that state. As the system runs, it moves from state to state through a history of operations.

For instance, our __state__ might be a variable and the reads/writes be the __operations__.

```ruby
x = "a"; puts x; puts x
x = "b"; puts x
x = "c"
x = "d"; puts x
```

![](https://aphyr.com/data/posts/313/uniprocessor-history.jpg)

Let's call this sytem of a single variable a __register__.

This hints at a definition of _correctness_: given some _rules_ which relate the operations and state, the history of operations in the system should always _follow those rules_. We call those rules a __consistency model__.

### Concurrent histories

Imagine a concurrent program. There are multiple logical threads of control, called __processes__.

![](https://aphyr.com/data/posts/313/multiprocessor-history.jpg)

_Two processes reading/writing to a variable concurrently; one process at the top, another at the bottom._

### Light cones

However, this is not the full story. Processes are _distant_ from each other. An operation by a client on a server optionally involve(s)

* a request
* processing on the server
* a response

Each of these takes an unknown (but possibly bounded) amount of time. Basically, operations are not instantaneous.

![](https://aphyr.com/data/posts/313/lightcone-history.jpg)

_The top process (X) updates the variable from `a` to `b`, then reads it. The bottom process (Y) sends a read request after X's write finishes, so Y reads value `b`._

![](https://aphyr.com/data/posts/313/concurrent-read.jpg)

_The top process (X) updates the variable from `a` to `b`, then reads it. The bottom process (Y) sends a read request before X sends a write request, but Y reads `b` instead of `a` b/c its request arrives later than X's write request._

We must relax our consistency model b/c operations are no longer instantaneous.

## Linearizability

Suppose there is a single global state each process talks to. If we assume that operations on that state take place __atomically__, we know that each operation appears to take effect atomically at some point between its invocation and completion. __This means that even though operations are concurrent and take time, it appears as though only one operation takes place at any point in time. They occur in a nice linear order (hence "linearizability")__

### Consequences

#### 1. Operations concurrent with any write operation may see an old value. However, operations that start after a write operation's completion must see the written value (or a newer value written by another write that completed before).

Once an operation is complete (request, processing, and response are done), _everyone_ must see it (or some later state).

![](https://aphyr.com/data/posts/313/linearizability-complete-visibility.jpg)

_Y's first read returns `a`, but this okay b/c this operation didn't start after X's write: Y's 1st read was concurrent with X's write. Y's second read replies with `b`._

#### 2. State can be mutated safely under linearizability conditions. This is achieved by mutexes, semaphores, channels, and etc.

Linearizability also allows you to mutate state safely. You can define an operation like `compare-and-set`, which sets the value of the register to a new value iff the register currently has some other value. If the value had been updated by another process in the meantime, the write would fail. `compare-and-set` is used as the basis for

* mutex
* semaphore
* channel
* shared counter
* shared list
* shared set
* shared map
* shared tree

So even though operations are concurrent, mutexes enable atomicity and therefore linearizability.

#### 3. Even though operations concurrent with a write may see old values, stale values are not seen.

A stale value is one set before the operation even begins (e.g. before the request is even sent). An old value, on the other hand, is one that is set to a new value after the operation starts but also before the operation finishes:

```
time_operation_start < time_old_value_change_to_new_value < time_operation_finish
```

## Sequential consistency

__Let's allow processes to _skew_ in time, such that their operations can take effect _before_ invocation or _after_ completion. However, operations from any given process must take place in the same relative order as that process' order.__ Operations are still atomic.

![](https://aphyr.com/data/posts/313/sequential-history.jpg)

_(Ignore the "lines can't cross" sentence for now...)_

To illustrate this, think of a queue that YouTube uses. When a user uploads a video, YouTube puts that video into a _queue_ for processing, then returns a web page for that video right away. We can actually _watch_ the video at that point; the video upload _takes effect_ a few minutes later, when it's finally fully processed. Queues remove synchronous behavior while preserving order.

To futher illustrate this example, a user sees their YouTube video go through 3 states:

1. Not existing
2. Processing
3. Watchable

Other users see:

1. Not existing
2. Watchable

Because both hold the same order, this kind of system is sequentially consistent. If a user submitted their video for processing, refreshed their page, and still did not see their video as not existing, then this will violate sequential consistency; but this should only happen b/c of caching, new sessions, incognito mode, DNS, etc.

Caches themselves also behave like sequentially consistent systems. If I write a tweet on Twitter, it takes time to percolate through layers of caching systems. Different users will see my message at different times, but each user will see my operations _in order_. Also, once a post is seen, it shouldn't disappear. Similarly, if I write multiple comments, they'll become visible sequentially, not out of order.

## Causal consistency

You know, we don't have to enforce the order of _every_ operation from a process. Perhaps, only __causally related__ operations must occur in order. Operations are still atomic.

We might say, for instance, that all comments on a blog post must apear in the same order for everyone, and insist that _any reply_ be visible to a process _only after the post it replies to_ is visible. If we encode those causal relationships like "I depend on operation `X`" as an explicit part of each operation, the database can delay making operations visible until it has all the operation's dependencies.

__This is weaker than sequential consistency - not all operations submitted by a process will happen in the order submitted by that process. However, causally related operations will occur in the order submitted by the process.__

## Serializable consistency

__Serializability means that processes are allowed to _skew_ in time, as in sequential consistency. Unlike sequential consistency, however, there is no guarantee that operations from any given process appear in the same relative order as that process' order. Good thing is, operations still occur atomically.__

__Strong serializability (refered to `SERIALIZABLE` in _some_ SQL databases) gives time bounds like linearizability.__


# Weak consistency models

| Isolation level | Dirty reads | Non-repeatable reads | Phantom reads |
| --------------- | ----------- | -------------------- | ------------- |
| Read Uncommitted | may occur | may occur | may occur |
| Read Committed | - | may occur | may occur |
| Repeatable Read | - | - | may occur |
| Serializable | - | - | - |

* Read Uncommitted - No locks are used in a transaction.
* Read Committed - Write locks are used in a transaction.
* Repeatable Read - Read and write locks are used in a transaction, but read locks are unlocked when a `SELECT` operation is performed.
* Serializable - Read, write, and range locks are used in a transaction.
* Snapshot Isolation - Don't use a lock. Instead, if there's a write-write conflict, abort and retry: this sort of achieves atomicity. However, write skew can happen. Both transactions take a private (and identical) snapshot of the database, then do a conflicting write. You can workaround write skew with (a) Materialize the conflict and (b) read -> write promotion [to upgrade to write-write conflict].

## Dirty Reads (Fix with Read Committed)

```sql
-- Transaction 1 --                     -- Transaction 2 --

/* Query 1 */
SELECT age FROM users WHERE id = 1;
/* will read 20 */


                                        /* Query 2 */
										UPDATE users SET age = 21 WHERE id = 1;
										/* No commit here */


/* Query 1 */
SELECT age FROM users WHERE id = 1;
/* will read 21 */


										ROLLBACK; /* lock-based DIRTY READ */

```

A dirty read (aka uncommitted dependency) occurs when a transaction is allowed to read data from a row that has been modified by another running transaction and not yet committed. In this case no row exists that has an id of 1 and an age of 21.

## Non-repeatable reads (Fix with Repeatable Read)

```sql
-- Transaction 1 --                     -- Transaction 2 --

/* Query 1 */
SELECT * FROM users WHERE id = 1;


                                        /* Query 2 */
										UPDATE users SET age = 21 WHERE id = 1;
										COMMIT; /* in multiversion concurrency
										           control, or lock-based
                                                   READ COMMITTED */


/* Query 1 */
SELECT * FROM users WHERE id = 1;
COMMIT; /* lock-based REPEATABLE READ */

```

A non-repeatable read occurs, when during the course of a transaction, a row is retrieved twice and the values within the row differ between reads.

## Phantom reads (Fix with Serializable)

```sql
-- Transaction 1 --                     -- Transaction 2 --

/* Query 1 */
SELECT * FROM users
WHERE age BETWEEN 10 AND 30;


										/* Query 2 */
										INSERT INTO users(id,name,age) VALUES (
                                          3, 'Bob', 27
                                        );
										COMMIT;


/* Query 1 */
SELECT * FROM users
WHERE age BETWEEN 10 AND 30;
COMMIT;

```

A phantom read occurs when, in the course of a transaction, two identical queries are executed, and the collection of rows returned by the second query is different from the first.

## Read skew

Read skew is similar to non-repeatable read (or even dirty read), in that the second read is after another transaction's write that modifies some other value. The second reads that same other value, but the transaction keeps the old value of the first read, so in total, the values are inconsistent (i.e. not allowable under serializability)!


## Write skew

A write skew occurs when two transactions read the same two rows, perform a consistency check (C in ACID), then perform conflicting updates to both rows, resulting in an inconsistent (i.e. unserialized) result.

## Read your writes

Many applications let the user submit some data, and then view what they have submitted. This might be a record in a customer database, or a comment on a discussion thread, or something of that sort. When new data is submitted, it must be sent to the leader, but when the user views the data, it can be read from a follower. This is especially appropriate if data is frequently viewed, but only occasionally written.

With asynchronous replication, there is a problem: if the user views the data shortly after making a write, the new data may have not yet reached the replica. To the user, it looks as though the data they submitted was lost, so they will be understandably unhappy.

![](https://martin.kleppmann.com/2015/05/linearizability.png)

In this situation, we need read-after-write consistency, also known as read-your-writes consistency. This is a guarantee that if the user reloads the page, they will always see any updates they submitted themselves. It makes no promises about other users: other users’ updates may not be visible until some later time. However, it reassures the user that their own input has been saved correctly.

How can we implement read-after-write consistency in a system with leader-based replication? There are various possible techniques, to mention a few:

* When reading something that the user may have modified, read it from the leader, otherwise read it from a follower. This requires that you have some way of knowing whether something might have been modified, without actually querying it. For example, user profile information on a social network is normally only editable by the only owner of the profile, not by anybody else. Thus, a simple rule is: always read the user’s own profile from the leader, and any other users’ profiles from a follower.
* If most things in the application are potentially editable by the user, that approach won’t be effective, as most things would have to be read from the leader (negating the benefit of read scaling). In that case, other criteria may be used to decide whether to read from the leader. For example, you could track the time of the last update; for one minute after the last update, all reads are made from the leader. You could also monitor the replication lag on followers, and prevent queries on any follower that is more than one minute behind the leader.
* Another approach: the client can remember the timestamp of its most recent write—then the system can ensure that the replica serving any reads for that user reflects updates at least until that timestamp. If a replica is not sufficiently up-to-date, the read can either be handled by another replica, or the query can wait until the replica has caught up. The timestamp could be a logical timestamp (something that indicates ordering of writes, such as the log sequence number), or the actual system clock (in which case clock synchronization becomes critical).
* If your replicas are distributed across multiple datacenters (for geographical proximity to users or for availability), there is additional complexity. Any request that needs to be served by the leader must be routed to the datacenter that contains the leader.

Another complication arises when the same user is accessing your service from multiple devices, for example a desktop web browser and a mobile app. In this case you may want to provide cross-device read-after-write consistency: if the user enters some information on one device, and then views it on another device, they should see the information they just entered.

In this case, there are some additional issues to consider:

* Approaches which require remembering a timestamp of the user’s last update become more difficult, because the code running on one device doesn’t know what updates have happened on the other device. This metadata would need to be centralized.
* If your replicas are distributed across different datacenters, there is no guarantee that connections from different devices are routed to the same datacenter. (For example, if the desktop computer uses the home broadband connection and the mobile device uses the cellular data network, their network routes may be completely different.) If your approach requires reading from the leader, you may first need to route requests from all of a user’s devices to the same datacenter.

## Monotonic reads

If you read a value and then perform another read, it may be possible that this next read value gives an older value. This happens if when you have asynchronous replication, you read first from one follower replica (that has committed) and then read from another follower replica (that hasn't committed yet).

Monotonic reads is a guarantee that this kind of anomaly does not happen. It’s a lesser guarantee than strong consistency, but a stronger guarantee than eventual consistency. When you read data, you may see an old value; monotonic reads only means that if one user makes several reads in sequence, they will not see time go backwards, i.e. they will not read older data after having previously read newer data.

One way of achieving monotonic reads is to make sure that each user always makes their reads from the same replica (different users can read from different replicas). For example, the replica can be chosen based on a hash of their user ID, rather than randomly.

## Consistent prefix reads

If you have a weak consistency model that isn't causally consistent, you may see anomalies that, of course, violates causality, like a reply comment that shows up on a website, but not the comment that it replies to.

Preventing this kind of anomaly requires another type of guarantee: consistent prefix reads. This guarantee says that if a sequence of writes happens in a certain order, then anyone reading those writes will see them appear in the same order.

Consistent prefix reads are easy to achieve in a single partition (leader-based replication normally applies writes in the same order to each follower), but are difficult if there are multiple independent partitions (shards). This guarantee is similar to snapshot isolation.
