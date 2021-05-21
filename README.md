# CockroachDB-notes

### Type of Database:
- Distributed
- Scaled Horizontally 

### Data Storage Mechanism
- Stores the entire user and system data in giant sorted maps of key value pairs

### Important Terms
#### 1. Cluster
The group of deployment which acts as a single logical application

#### 2. Node
Each individual system of the cluster forms the node

#### 3. Range 
- The entire key space is divided into chunks of 64 MB ranges
- Ranges are small enough to be split into be moves/split quickly
- Ranges are large enough to amortize indexing overhead
- The data in the ranges is ordered and the ranges themselves are ordered

#### 4. Range Indexing
To maintain the order between ranges, we need to index them. Thus, an index structure is used to locate ranges (similar to a B-tree).

### Architecture - Layered
#### Last layer - Distributed, replicated, transactional Key Value Pairs
- Keys and Values are strings (lexiographically ordered by key)
- Multi Version Concurrency Control (MVCC) - Entries are not replaced but new versions are added and pointer to current version updates kinda like version control.
- Monolithic Key Space - The entire set of key-value pairs is ordered and stored in one space. (Ordered becaus ethe SQL layer on top needs ordered data for certain queries)
- This entire space is divided into chunks of 64 MB ranges
- The Key Space is empty in the beginning and as the database is populated, the key space is divided into the 64MB ranges. When the size of teh database increases, the ranges increase and when the size decreases, the ranges are merged into one

### Replica
#### Replication
- CockroachDb maintains 3 replicas by default though this is modifiable
- Each ranges uses an individual raft (A distributed concensus protocol) to keep its replicas in sync
- This implies we replicate at the ;evel of ranges and not at the level of nodes
- Raft provides atomic replication of commands
- Replicas elect a leader (the copy which maintains the latest changes)
-  Commands which change the database are proposed by a leader and distributed to followers. These commands are only acknowledged when a moajority of followers vote that they have made the change. For e.g. if a write command is proposed by a leader, all replicas will asked to write the change. The client will be notifies that the change is made only when a majority of the replicas have made the change. this ensure resiliancy in case some the replicas fail.  
-  We cannot read from any replica as they are not always up to date. Only leader is. therefore reads are always performed at the leader also called #### Leaseholder. 
-  Reads without Consesus. 
-  One replica is chosen as the leaseholder. 
    1. Performs read
    2. Co-ordinates writes (roposal, key-locking and deadlock dedection)

### Replica Placement
- Each range is a raft state machine having one or more replicas
- These replicas need to be placed taking into account the following
  1. Space
  2. Diversity
  3. Load
  4. Latency
 
#### Replica Placement: Diversity
- Optimizes lacement of replicas across "Failure Domains"
- Spread replicas across as many failure domains as possible
  Failure Domains:
  1. Disk
  2. Single Machine
  3. Rack
  4. Datacenter
  5. Region

#### Replica Placement: Load
- Balances placement using heuristics that consider real-time metrics of the data itself
- A range is said to have high load if it is accessed more than others

#### Replica Placement: Latency and Geo-Partitioning
- We apply a constraint which indicates regional placement so that we can ensure low latency access or jurisdictional control of data 

### Transactions

- Follow ACID semantics. (Atomicilty, Consistency, Isolation and Durability)
- Serialisable isolation
- Transactions can span arbitrary ranges
- Conversational => The full set of operations is not required up front
- Raft provides atomic writes to individual changes
- Bootstrap transaction atomicity using Raft atomic writes
- Transactipn record automatically flipped from pending to commit when a moajority of replicas ack
- Leader committed first and then followers comitted. ACK for commiting also sent
- gateway ACK client

### SQL Data in KV World
- The SQL data model needs to be mapped to KV data
- ID id key. Everything else is comma separated value
- Or /TableName/Index/ID is key.
    
### Distributed SQL Execution
- Execute the given query at each node and execute the group part of the query again at the leader node

### Distributed SQL Optimisation
- An optimizer considers all the possible plans which are logically equivalent of the given query and chooses the best one
The different transformations of preparing the query and data are:
1. Fold constants
2. Check types
3. Resolve names
4. Report Semantic errors
5. Compute properties
6. Retrieve and attach stats
7. Cost-independent transformations

#### 1. SQL Optimisation: Cost Independent Transformations
- Some transformations always make sense. e.g. constant folding, filter push-down, etc. These transformations are cost independent. This means if it can be applied to query, it is applied
- Domain specific Language for transformations (DSL) s complied down to code which efficiently matches the memo. 

#### 2. SQL Optimisation: Cost Dependent Transformations
- Some transformations are not universally good. E.g. index selection, join reordering, etc. These transformations are called cost based.
- Need to try both paths of cost based transformations and need to maintain both original and transformed query
- State explosion: thousand of possible query plans. Memo data structure maintains a forest of query plans 
- Estimate cost of each query and select query with the lowest cost
- Costing is based on table statistics and estimating the cardinality of inputs to relational expressions

#### 3. SQL Optimisation: Cost-based Index Transformations
The index to use for a query is affected by multiple factors
- ilters and join conditions
- equired ordering (Order By)
- Implicit ordering (Group By)
- Covering vs Non covering (i.e. an index join is required)
- Locality

for e.g. sorting is expensive when there is lots of data but cheap when small amount of data. 
Instead of scanning the entire data where x belongs to \[10,inf) and then sorting on y (order by y)
Scan all y and filter entries where x>10
^^ most optimal solution

#### 4. Locality aware SQL Optimisation
- Network latencies and throughput are important
- uplicate eread-,ostly data in each locality
- Plan queries to use from the same locality 


Ref: https://youtu.be/OJySfiMKXLs
