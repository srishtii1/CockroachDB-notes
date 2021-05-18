# CockroachDB-notes

### Type of Database:
- Distributed
- Scaled Horizontally 

### Data Storage Mechanism
- STores the entire user and system data in giant sorted maps of key value pairs
- 

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

#### Range Indexing
To maintain the order between ranges, we need to index them. Thus, an index structure is used to locate ranges (similar to a B-tree).

### Architecture - Layered
#### Last layer - Distributed, replicated, transactional Key Value Pairs
- Keys and Values are strings (lexiographically ordered by key)
- Multi Version Concurrency Control (MVCC) - Entries are not replaced but new versions are added and pointer to current version updates kinda like version control.
- Monolithic Key Space - The entire set of key-value pairs is ordered and stored in one space. (Ordered becaus ethe SQL layer on top needs ordered data for certain queries)
- This entire space is divided into chunks of 64 MB ranges
- The Key Space is empty in the beginning and as the database is populated, the key space is divided into the 64MB ranges. When the size of teh database increases, the ranges increase and when the size decreases, the ranges are merged into one
- 
