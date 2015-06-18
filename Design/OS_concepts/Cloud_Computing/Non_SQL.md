* Why we need Non_SQL
    * 1. Data Large and unstructred 
    * 2. Lots of random reads and writes
    * 3. Sometimes write-heavy, but Relational SQL is read-heavy
    * 4. Foreign keys rarely needed
    * 5. Joins infrequent

* Needs recently
    * Speed
    * Avoid Single point of Failure (SPoF)
    * Low TCO (Total cost of operation)
    * Fewer system adminstrators
    * Incremental Scalability
    * Scale out than scale up

* NoSQL = "Not Only SQL"
    * Necessary API operations:
      * get(key)
      * put(key, value)
    * Unstructured
    * No schema imposed, maybe missing columns or some Rows
    * No foreign keys, joins may not be supported 

* Column-Oriented Storage
    * RDBMS store an entire row together
    * NoSQL systems typically store a column together 
    * Why useful ?
        * Queries that touch only a few columns are faster than those in a row-oriented storage
        * It enables faster range searches using one column
        * It prevents the entire table from being read for a query 
        * example: get the blog_ids fromt he blog table, for the RDBMS, need to fetch
        all the table and get each row, but for NoSQL, we only need to fetch that column 
        do not need other column at all

* Cassandra
  * a distributed key-value store
     * Based on the Distributed Hash Table (DHT), each data center keep a ring store servers and keys
     * Coordinator could be per query/client/data center based 
     * Each data center will have a ring 
     * there is no finger/routing tables in Cassandra ring
     * Instead of using the finger table, Coordinator need to know all the keys store in which nodes
       (include the duplicate nodes), 
   
   * Partitionar: which find the servers to the particular key in the coordinator   

  * Replication Strategy (two options)
      * A. SimpleStrategy
      * B. NetworkTopologyStrategy
      
      * SimpleStrategy: uses the Partitioner, of which there are two kinds
         * 1. RandomPartitioner: Chord-like hash partitioning
         * 2. ByteOrderedPartitioner: Assigns ranges of keys to servers.
               * maintain a table in Cassandra, preserve the ordering of the keys in there, like timestamp 
               * Easier for range queries (e.g., get me all twitter users starting with [a-b])
               * it's not based on the hash, just map the key value to the ring, so the order of keys in the real spaces 
                 is the same in the ring space 

      * NetworkTopologyStrategy: for multi-DC deployments
         * Two replicas per DC
         * Three replicas per DC
         * Per DC
            * How to set the replicas in per DC
               * First replica placed according to Partitioner
               * Then go clockwise around ring until you hit a different rack, rack failure tolerance 
               
   * Snitches:
      * Maps IPs to racks and DCs,  Configured in cassandra.yaml config file
         * SimpleSnitch: Unaware of Topology (Rack-unaware)
         * RackInferring: Assumes topology of network by octet of server’s IP address
            * 101.201.301.401 = x.<DC octet>.<rackoctet>.<node octet>
         * PropertyFileSnitch: uses a config file
         * EC2Snitch: uses EC2
   
  * Writes (Check the slides, important !) 
      * The basic sequence: 
         * Client sends write to one coordinator node in Cassandra cluster
               * Coordinator may be per-key, per-client, or perquery 
               * Per-key Coordinator ensures writes for the key are serialized
         * Coordinator uses Partitioner to send query to all replica nodes responsible for key
         * When X replicas respond, coordinator returns an acknowledgement to the client
         
      * Hinted Handoff mechanism 
          * If any replica is down, the coordinator writes to all other replicas, and keeps the write locally
until down replica comes back up. When all replicas are down, the Coordinator (front end) buffers writes (for up to a few hours).   
          * When all replicas are down, the Coordinator (front end) buffers writes (for up to a few hours).
          * Coordinatior has the time-out of store 
      * One ring per datacenter
          * Per-DC coordinator elected to coordinate with other DCs, this coordinator is different with the coordinator
            used in the query 
          * Election of coordinate per datacenter using ZooKeeper
      
      * What's the replic do when the coordinator forwards the write ?
          * On receiving a write
            * 1. Log it in disk commit log (for failure recovery), if there is any failure for the replic, after it recovered,
                 it could check this disk commit log and find if there are some writes finished partially, will ask for 
                 coordinator to get the informaiton 

            * 2. Make changes to appropriate memtables
               * Memtable = In-memory representation of multiple keyvalue pairs
               * Cache that can be searched by key
               * Write-back cache(stored intentatively in memory) as opposed to write-through (write directly to the disk called write-through cache)
               
            * Later, when memtable is full or old, flush to disk, so SSTable, Index table and bloom filter are in disk
               * Data file: An SSTable (Sorted String Table) – list of key-value pairs, sorted by key
               * Index file: An SSTable of (key, position in data sstable) pairs, which is reduce the lookup time in SSTable
               * sometimes we need to check the exist key in the table, but key not in memtable, how we check the SSTable ?
                 if we use the binary search, very slow (O(logN))
               * And a Bloom filter (for efficient search) 
            
            * Bloom Filter 
               * use k hash functions 
               * for each key-k, use k hash functions to map to the bit map and set as 1
               * false positives 
               * never false negatives 
               
      * Compaction
         * Data updates accumulate over time and SStables and logs need to be compacted
               * The process of compaction merges SSTables, i.e., by merging updates for a key
               * Run periodically and locally at each server
      * Delete
         * Delete: don’t delete item right away
            * Add a tombstone to the log
            * Eventually, when compaction encounters tombstone it will delete item
            
      * Read
         * 

* P2P Systems
    * A. Gnutella
      * little-endian format for the message structure and protocol 
      * To avoid duplicate transmissions, each peer maintains a list of recently received messages
      * Query forwarded to all neighbors except peer from which received
      * Each Query (identified by DescriptorID) forwarded only once
      * QueryHit routed back only to peer from which Query received with same DescriptorID
      * Duplicates with same DescriptorID and Payload descriptor (msg type) are dropped
      * QueryHit with DescriptorID for which Query not seen is dropped
      
    * Five main messages:
      * Query (search)
        * when the nodes receive the query message, it will first check the local whether hit the keyword
        * then flood the query messages to all the neighbors 
        * if receive the duplicate, will not send the msgs second time
      * QueryHit (response to query)
      * Ping (to probe network for other peers)
        * keep set of the neighbors are fresh 
      * Pong (reply to ping, contains address of another peer)
      * Push (used to initiate file transfer)
          * if there is the response behind the requestor, then we could use the push messages since it already connected
            to peers, which go through the query path
      
    * B. Chord
      * Distributed Hash Table
        A distributed hash table allows you to do the same in a distributed setting (objects=files)
        * Performance concerns:
          * Load balancing
          * Fault-tolerance
          * Efficiency of lookups and inserts
          * Locality
      * Chord:
        * Memory: O(logN)
        * Lookup Latency: O(logN)
        * Messages for a lookup: O(logN)
      * How we place the nodes(peers) in the ring ?
          * Uses Consistent Hashing on node’s (peer’s) address
          * SHA-1(ip_address,port) -> 160 bit string
          * Truncated to m bits
          * Called peer id (number between 0 and 2^m - 1)
          * Not unique but id conflicts very unlikely
          * Can then map peers to one of 2^m logical points on a circle
          * kinds of the peer pointers(types of neighbors): 
              * 1) successors 
              * 2) Predecessors
              * 2) Finger Table:
                  * for example: m = 7, the fingle table i from 0 to 6
                  * for each node id, let's say the peer with id 33, the first peer id is 
                   id >= n + 2^i (mod 2^m)
                  * details see the video, charpter 3.5 P2P
      * How we place the files?
         * SHA-1(filename) 160 bit string (key)
         * File is stored at first peer with id greater than its key (mod 2^m)
      * How the search works ?
         * At node n, send query for key k to largest successor/finger entry <= k
         * if none exist, send query to successor(n)
      
      * How we handle the failures ? 
         * we may not find the node which store the data 
            * Save r successors in table
            * Choose r = 2log(N) suffices to maintain lookup
         * if the node which save the data has failure 
            * duplicate the key in the Predecessors and successors
      
      * When the new node inserted
         * update the successor and predesessor
         * talks to neighbors to update finger table (Stablization protocol used by each node)
         * a new peer affects O(logN) other finger entries in the system 
         * Stabilization Protocol 
      
      * Virtual Nodes
         * ensure load balance
        
      
      
      
      
      