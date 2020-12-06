# 3. Managing documents

## Updating documents
Create doc:
```json
PUT /pages/_doc/100
{
  "name": "bandage",
  "sold": 0
}
```

Update doc:
```json
POST /pages/_update/100
{
  "doc": {
    "sold": 2
  }
}
```
`doc` object holds the key-value pairs of the doc you want to update.

Check the value:
```json
GET /pages/_doc/100
```
Output:
```json
{
  "_index" : "pages",
  "_type" : "_doc",
  "_id" : "100",
  "_version" : 2,
  "_seq_no" : 1,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "bandage",
    "sold" : 2
  }
}
```
*Note*: Es docs are immutable, an _update action actually overwrites the old doc with the new one.

## Scripted updates
Use scripts to perform actions on a document's values.  
Example: increase number of products sold by one.
```json
POST /pages/_update/100
{
  "script": {
    "source": "ctx._source.sold++"
  }
}
```
`source`: set the script as its value.
`ctx._source.sold--`: `ctx` is the current context, from which we get the `_source` field, and from that the `sold` field. `--` increments the value of `sold` by one.  
Can also set the value, e.g. `ctx._source.sold = 10`.

### Update with a parameter
```json
POST /pages/_update/100
{
  "script": {
    "source": "ctx._source.sold += params.quantity",
    "params": {
      "quantity": 4
    }
  }
```
### Conditional update
Ignore the docs that are at zero sold items, otherwise subtract one. In the line `ctx.op = 'noop'` the operation on those docs is set to *no operation*.
```json
POST /pages/_update/100
{
  "script": {
    "source": """
      if (ctx._source.sold == 0) {
        ctx.op = 'noop';
      }
    ctx._source.sold--;
    """
  }
}
```
This version *doesn't* update the doc, which you can see from the `_version` number staying the same. In the following alternative, the doc is always updated, even when the value doesn't change:
```json
POST /pages/_update/100
{
  "script": {
    "source": """
      if (ctx._source.sold > 0) {
        ctx._source.sold--;
      }
    """
  }
}
```
The first version with `noop` can be used if you want to be able to detect when nothing has changed.

You can also use a **conditional doc deletion** `ctx.op = 'delete'`, at your own discretion.

## Upserts
Run the script if the doc already exists. If it doesn't then use the contents of `upsert` to create a new doc.
```json
POST /pages/_update/101
{
  "script": {
    "source": "ctx._source.sold++"
  },
  "upsert": {
    "name": "tape",
    "sold": 5
  }
}
```
If the doc didn't exist yet, you see `"result": "created"` returned, otherwise you see `"result": "updated"`.

## Deleting documents
```json
DELETE /pages/_doc/101
```

## Understanding routing
How does Es know on which shard to store or find a doc? **Routing**: the process of resolving a shard for a doc.  
Es uses a routing formula:
```
shard_num = hash(_routing) % num_primary_shards
```
`_routing` by default equals the doc's id. Docs are stored and retrieved with this same formula. Thus, it can find the right shard from the doc id.  

*This also explains why you can't change the number of shards after your index has been created: it would try to retrieve docs from the wrong shard.*

## How Es reads data
1. Coordinating node receives a read request
2. Node resolves to the right shard through routing
3. A shard is chosen from the replication group (otherwise the primary shard would always get all the request, giving uneven load) - **adaptive replica selection** (ARS)
4. Coordinating node sends request to shard
5. Coordinating node receives reply and sends it to the client

## How Es writes data
1. Same **routing** process as above
2. Write request is always sent to primary shard
3. Primary shard **validates** request (structure, field values)
4. Primary shard performs write operation
5. Primary shard forwards operation to the replica shards (in parallel)

### How is failure dealt with in this async architecture?
E.g. Primary shard sends operation to replica 1, but goes down before it send to replica 2.
=> Es goes into **recovery process**. A replica shard is promoted to primary shard.
Say replica 2 is promoted to primary shard. **Problem**: the replica 1 is now not in the same state as replica 2.

Solution through **primary terms** and **sequence numbers**.

#### Primary terms
`_primary_term`
* Counter for how many times a primary shard has changed
* A way to distinguish between old and new primary shards
* Primary term is appended to write operations

#### Sequence numbers
`_seq_no`
* Counter for each write operation
* Primary shard increases the sequence number
* Sequence number is appended to write operations
* Enables Es to order write operations

Combined, primary terms and sequence numbers allow Es to track which write operations have been performed yet, instead of having to compare all data on disc between shards.
For large indices, this can still be very expensive. For this, Es uses **global and local checkpoints**.

#### Global and local checkpoints
* Essentially sequence numbers
* Each replication group has a **global checkpoint**
* Each replica shard has a **local checkpoint**
* **Global checkpoint**:
    - All active shards in a group have been aligned to *at least this sequence number*
* **Local checkpoint**:
    - Sequence number for last write operation  

Thus: Es only needs to **compare global and local checkpoints**, and perform the write operations that a particular shard missed.

## Understanding doc versioning
### Internal versioning
* Doc version number, note: *not* a revision history
* Value retained for 60 s after doc deletion
* Can instruct to return version field on query

### External versioning
* For when docs are also stored in a RDB (e.g. SQL db apart from Es)

This versioning is generally not that useful for recovery ("optimistic concurrency control"). This just explains the presence of the field.

## Optimistic concurrency control
* If write operations arrive out of sync, prevent overwriting documents with older versions
