# Distributed Caching - Detailed Interview Prep Notes

## User:

You are an expert solution architect working with distributed cache help
me understand th3 concept in detail with real-time world example to
clear faang level interview.

## Assistant:

Got it 👍 --- let's go **deep into distributed caching** like you'd need
for a **FAANG-level system design interview**.

------------------------------------------------------------------------

# 🔹 What is Distributed Cache?

A **cache** is a **temporary, high-speed data store** (often in memory)
that sits between your application and a slower data source (like a
database, file system, or API).

A **distributed cache** means this cache is:\
- **Shared across multiple servers/nodes** in a cluster\
- **Scalable** (can add/remove nodes)\
- **Consistent** (tries to give same data regardless of which node is
accessed)

Think of it as a **shared "fast lookup" memory** for your whole
application ecosystem.

------------------------------------------------------------------------

# 🔹 Why Do We Need Distributed Cache?

-   Databases are **slow** compared to in-memory lookups.\
-   Large-scale apps (Netflix, Amazon, Meta) have **millions of
    concurrent users**.\
-   Without caching, the DB becomes a bottleneck.\
-   Distributed cache ensures **low latency (\~ms)** and **high
    throughput** across **multiple servers**.

------------------------------------------------------------------------

# 🔹 Real-World Analogy

Imagine **Starbucks**:

-   The **main coffee kitchen (database)** is slow but has the source of
    truth.\
-   Each barista (app server) can prepare drinks, but if everyone goes
    back to the kitchen for every ingredient, it's slow.\
-   So, they set up **ready-to-go stations (cache)** stocked with
    popular items like milk, sugar, or syrups.\
-   When a customer (user request) arrives:
    -   If the item is at the station → quick service (cache hit ✅).\
    -   If not → barista goes to the kitchen, fetches it, and stocks the
        station for the next customer (cache miss ➡️ DB fetch ➡️ cache
        update).

In distributed caching:\
- Multiple Starbucks branches (servers) share a **central warehouse
(distributed cache cluster)**.\
- This keeps stations across branches consistent and fast.

------------------------------------------------------------------------

# 🔹 How Distributed Cache Works (Core Concepts)

### 1. **Cache Key-Value Store**

-   Data stored as **key → value**\

-   Example:

        Key: user:123:profile  
        Value: {id: 123, name: "Alice", premium: true}

### 2. **Cache Placement (Consistent Hashing)**

-   If you have multiple cache servers, which one stores the data?\
-   **Consistent hashing** distributes keys evenly, ensuring minimal
    rebalancing when nodes are added/removed.

### 3. **Cache Policies**

-   **Eviction**: What if memory is full?
    -   LRU (Least Recently Used), LFU (Least Frequently Used), FIFO\
-   **TTL (Time-to-Live)**: How long should data live in cache before
    expiring?

### 4. **Cache Invalidation Strategies** (VERY IMPORTANT in interviews 🚨)

-   **Write-through**: Data written to cache *and* DB at the same time.
    (Strong consistency, slower writes)\
-   **Write-around**: Data written to DB only, loaded to cache only on
    read. (Avoids polluting cache with rarely-used data)\
-   **Write-back / Write-behind**: Write to cache first, flush
    asynchronously to DB. (Fast, but risk of data loss if cache
    crashes)\
-   **Cache-aside / Lazy loading**: App checks cache first; if not
    found, fetch from DB and update cache. (Most common)

------------------------------------------------------------------------

# 🔹 Popular Distributed Caching Systems

-   **Redis** -- In-memory key-value store, supports persistence,
    pub/sub, clustering.\
-   **Memcached** -- Purely in-memory, simpler than Redis, no
    persistence.\
-   **Hazelcast / Apache Ignite** -- JVM-based distributed caching
    frameworks.

------------------------------------------------------------------------

# 🔹 Example at FAANG Scale

### Netflix (Distributed Caching Example)

-   **Problem**: Millions of users browsing shows → DB queries would
    explode.\
-   **Solution**:
    -   Popular metadata (movie titles, posters, recommendations) stored
        in Redis clusters.\
    -   User preferences (e.g., "continue watching list") cached
        regionally.\
    -   When a user opens Netflix:
        -   UI calls → distributed cache hit in \<1ms.\
        -   If cache miss → fetch from DB → update cache.\
-   **Result**: Smooth streaming experience, no DB overload.

------------------------------------------------------------------------

# 🔹 Key Interview Points (What FAANG will test you on)

✅ **When would you use a distributed cache?**\
- High read-heavy workloads (e.g., product catalog, session store,
feed).

✅ **When NOT to use it?**\
- When data changes extremely frequently (real-time stock prices → may
cause stale reads).\
- When strong consistency is absolutely required (banking transactions).

✅ **How do you handle cache stampede (thundering herd problem)?**\
- Use **lock + single DB fetch** to prevent many requests hitting DB
simultaneously.\
- Techniques: **Request coalescing, randomized TTL,
stale-while-revalidate.**

✅ **How do you scale cache clusters?**\
- Sharding + consistent hashing.\
- Replication for fault tolerance.

✅ **What happens if cache goes down?**\
- Fallback to DB (but ensure DB can handle sudden surge).\
- Use circuit breakers and rate limiting.

------------------------------------------------------------------------

# 🔹 Interview-Ready Summary

**Distributed cache** is a **clustered, scalable, in-memory data store**
that:\
- Reduces latency and DB load.\
- Uses **keys, eviction policies, TTL, and replication**.\
- Must handle **consistency, failures, and scaling**.\
- Real-world: Netflix, Amazon, Facebook all use it for **fast retrieval
of frequently accessed data**.

------------------------------------------------------------------------

...

# 📝 **Distributed Caching -- Interview Cheat Sheet**

------------------------------------------------------------------------

## 🔹 1. What is it?

-   **Distributed cache = in-memory key--value store across nodes**
    (Redis, Memcached, EVCache).\
-   Purpose: **reduce DB load, improve latency, scale reads**.\
-   DB = source of truth; cache = optimization.

------------------------------------------------------------------------

## 🔹 2. Where to use?

✅ High read-heavy workloads (product pages, timelines, recs).\
❌ Not for critical strongly consistent data (bank balances).

------------------------------------------------------------------------

## 🔹 3. Architecture

-   **L1 cache**: in-process, per app server (Caffeine, Guava).\
-   **L2 cache**: shared distributed cluster (Redis, Memcached).\
-   **DB**: authoritative, persistence layer.\
-   Sync L1 + L2 via **pub/sub or invalidation**.

------------------------------------------------------------------------

## 🔹 4. Cache Strategies

-   **Cache-aside (lazy load):** App fetches DB on miss, writes back.
    (most common)\
-   **Write-through:** Write DB + cache simultaneously (strong
    consistency, slower writes).\
-   **Write-around:** Write DB only, cache on read.\
-   **Write-back:** Write to cache, async flush to DB (fast, but risk of
    loss).

------------------------------------------------------------------------

## 🔹 5. Key Problems + Fixes

-   **Cache Stampede:** Many requests miss at once.
    -   🔧 Single-flight lock, stale-while-revalidate (SWR), jittered
        TTL.\
-   **Cache Avalanche:** Many keys expire at same time.
    -   🔧 Random TTL, background refresh.\
-   **Cache Penetration:** Invalid keys hammer DB.
    -   🔧 Negative caching, bloom filter.\
-   **Hot Keys:** One key gets all traffic.
    -   🔧 Replication, sharding, inline partial values.\
-   **Cache Consistency:** DB vs cache mismatch.
    -   🔧 Write-through, event-driven invalidation.

------------------------------------------------------------------------

## 🔹 6. Failure Modes

-   **Banking:** If cache fails → bypass to DB (never serve stale).\
-   **Netflix/Twitter:** If cache fails → serve stale/defaults
    ("Trending now") until recovery.\
-   **Recovery risk:** DB storm when cache comes back.
    -   🔧 Stagger warm-ups, throttle rebuilds, request coalescing.

------------------------------------------------------------------------

## 🔹 7. Real-World Examples

-   **Amazon:** Product metadata cached with DynamoDB DAX + Redis
    (write-through for correctness).\
-   **Netflix:** EVCache (Memcached) for recommendations & metadata,
    tolerant to staleness (SWR).\
-   **Uber:** Redis cluster for driver locations & surge pricing (short
    TTLs + jitter).

------------------------------------------------------------------------

## 🔹 8. Interview Soundbites

-   "Cache is an optimization layer, not source of truth."\
-   "We trade off a bit of freshness for massive latency + scale
    gains."\
-   "I'd use SWR for Twitter, background refresh for Amazon pricing, and
    write-through for banking."\
-   "The real danger isn't cache miss, it's cache recovery stampedes.
    That's why I'd use staggered warmups and coalescing."

------------------------------------------------------------------------

## 🔹 9. Timeline Example (Twitter/Instagram feed)

-   **Cache pages** of tweet IDs, not full timeline.\
-   TTL: 60--120s with jitter.\
-   SWR + background refresh for hot users.\
-   Engagement counters cached separately with short TTL.\
-   Deletes/blocks → event-driven invalidation (Kafka).

------------------------------------------------------------------------

✅ With this sheet, you can answer: *what it is, when to use it, key
problems, strategies, failures, real-world examples*. That's 90% of
cache-related interview questions.
