```
GET /

POST test/_doc

GET _cat/nodes?v
```

shards should be around 50 GBs

djb2 is buggy, if you have 33 shards, some of shards will ton of docs and some other 0 docs; now murmur is used

multiple primary shards can be on the same node (node contains multiple shards - primary and secondary)

How to parallelize writes: add more shards
How to parallelize reads: add more replicas

```
GET
POST test/_search {
  "query": {
    "match": {
      "name": "Nenad"
    }
  }
}
```

Scatter Gather
Search returns top 10 results by default

Two phases of query:

- fetch top 10 results from every primary shard - only \_id and \_score
- get top 10 results of the whole set of documents retrieved from all shards and select top 10
- fetch these results by id

Apache Lucene write:
Document is written into a buffer (in-memory), but also in translog which is stored ont disk (cheap to write, sequential writes). That value is also written into into an immutable Lucene segment. So, all write operations will go to that segment and once this passes, create a new segment. (the higher write throughput, the bigger the refresh rate should be - the downside is the time we're waiting for the documents to be queryable)
Once per second (the default frequency of the refresh operation) segments are flushed to a disk.
When that segment is flushed to disk with fsync, translog is emptied.

Small updates are very expensive.
Segments are merged and then the deleted documents are removed (they are generally makred as deleted with bitmap). So, deleting documents can increase the index, but adding new documents can reduce its size. Keep that in mind when storing rapidly changed things, like counters.

Lucene is good at compressing the data.

NoSQL databases have an optimistic way of handling transactions - `_version` (update the document only if it has version x, otherwise raise an error to a client that should handle such update)

`_seq_no` - number of writes in that shard
`_primary_term` - the counter of failovers from master to some other node

If you write a document, it will be queryable once the document is flushed to disk (the refresh interval period); but

- if you get it by id, it will be returned
- if you set `PUT databases/_doc/1?refresh=wait_for`, it will block until the refresh is done, i.e. the document update is reflected to disk (useful for unit/integration tests)

Good optimization: if you do not query ES for some time (30secs maybe), no new small segments will be made, but once you query the ES, it will immediately create the segment and flush all updates. The bigger segments are better (in terms of storage and write throughput).

- filesystem caching (memory mapped) - the file is stored on disk, but loaded into memory for caching; it is recommended to give 50% or memory to heap (now less) and the default value.

ILM - Index Lifecycle management (defines when to move data from one tier to the next tier)
Hot data - freshly written, queried intensively
Warm data - queried ocassionaly, rarely updated
Cold and frozen data - stored for compliance in case someone needs it, not queried

---

Storing logs - good idea is to have one index per day (good performance and the storage costs are pretty much the same) and it's easy to delete such data, just remove the index, which is efficient
it is important to plan ahead in terms of number of shards, cause shards cannot be split ---> hmmm The split index API allows you to split an existing index into a new index, where each original primary shard is split into two or more primary shards in the new index.

cluster config data:
mappings
shard routing table
master node...

Queries are not cached
Filters are better than queries
