To give you a definitive answer: **an AWS `t3.micro` instance cannot safely handle 50,000 requests per second** (the fanout volume generated if 10 users with 5,000 friends post simultaneously), and it will struggle significantly with a standard Spring Boot application under any moderate load.

As a senior engineer, you know that a Spring Boot application carries non-trivial baseline overhead. Let's look at why `t3.micro` is severely constrained, and then map out the exact capacity matrix you need to calculate the infrastructure required for 10 million users and 5,000-friend fanouts.

### Why a `t3.micro` Struggles with Spring Boot

An AWS `t3.micro` gives you:

-   **2 vCPUs** (burstable, meaning you have a limited balance of CPU credits).
    
-   **1 GB of RAM**.
    

#### The Memory Bottleneck

The default Java 17/21 HotSpot JVM footprint combined with the Spring Framework class-loading overhead typically consumes **400MB to 600MB of RAM** just sitting idle.

-   If you run a standard Embedded Tomcat configuration (which defaults to a thread pool of **200 worker threads**), each active thread allocates its own stack memory (usually 1MB per thread via `-Xss`).
    
-   If your application experiences a sudden traffic spike, your JVM will instantly hit the 1GB hardware limit, trigger heavy operating system swapping, or trigger the Linux **OOM (Out of Memory) Killer**, crashing your service.
    

#### The Reality of Throughput

-   **With standard blocking I/O (Spring WebMVC):** A `t3.micro` can comfortably handle roughly **100 to 200 concurrent HTTP requests per second (QPS)**, assuming your backend logic is fast (e.g., a database lookup taking $< 20\text{ms}$).
    
-   **With reactive I/O (Spring WebFlux / Netty Engine):** You can bypass the thread-per-request limitation, pushing the capacity up to **500 to 1,000 QPS**, but you are still tightly throttled by the 2 vCPUs and limited network bandwidth.
    

### The Infrastructure Calculation Table (Scale Matrix)

To calculate the capacity required for your user base (1,000 users or your ultimate target of 10 million users with 5,000 friends each), use the architectural metrics below. This table contrasts a weak `t3.micro` against a production-grade instance typically used for a Spring Boot tier (like a compute-optimized `c6i.xlarge`).

## AWS Instance Comparison for News Feed / Fanout Architecture

| Architectural Metric | AWS t3.micro (1GB RAM, 2 vCPU) | AWS c6i.xlarge (8GB RAM, 4 vCPU) | Impact on News Feed / Fanout Calculation |
|---|---|---|---|
| **Max Safe Memory Allocation (-Xmx)** | 512 MB (leaving room for OS) | 6 GB (plenty of room for heavy JVM heap) | Determines how many incoming user payloads can be held in memory simultaneously. |
| **Average Spring Boot QPS Capacity** | ~150 QPS | ~3,000 - 5,000 QPS | Determines the number of incoming **"Post Message"** requests the web tier can parse and write to Kafka per second. |
| **Network Bandwidth** | Up to 5 Gbps (Burstable, low baseline) | Up to 12.5 Gbps (Sustained) | Crucial for fanout. Fetching 5,000 friend IDs from a Graph DB requires large network I/O packets. |
| **Recommended Use Case** | Small pet projects, development testing | Production Spring Boot Microservice node | Do not use t3.micro for critical system components at scale. |


### Let's Do the Math: 1,000 Users with 5,000 Friends

Let's apply Alex Xu's Back-of-the-Envelope estimation framework.

#### Scenario: 1,000 active users decide to publish a post within the exact same minute.

1.  **Incoming Write Volume:** $1,000 \text{ posts} / 60 \text{ seconds} = 16.6 \text{ QPS}$ hitting your frontend Web Servers. A single `t3.micro` _could_ easily ingest this low traffic and drop the 1,000 events into Apache Kafka.
    
2.  **The Fanout Amplification:** Once those posts hit the **Fanout Workers**, the math multiplies drastically.
    
    $$\text{Total Cache Write Operations} = 1,000 \text{ posts} \times 5,000 \text{ friends} = 5,000,000 \text{ write tasks}$$
    
3.  **The Processing Target:** If your functional requirement says a feed update must appear in a friend's feed within **5 seconds** of posting:
    
    $$\text{Required Fanout Processing Speed} = \frac{5,000,000 \text{ tasks}}{5 \text{ seconds}} = 1,000,000 \text{ writes/sec}$$
    

#### Conclusion for Your Architecture:

To handle **1,000,000 Redis cache writes per second** during that spike:

-   Your **Fanout Service / Worker tier** needs enough CPU power to parse millions of queue messages. If a single production-grade `c6i.xlarge` Spring Boot instance handles 5,000 concurrent processing tasks, you would need a cluster of roughly **200 worker pods** during peak burst, or you let the Kafka queue absorb the delay, allowing fewer workers to drain it over a longer period (e.g., 60 seconds instead of 5).
    
-   Your **News Feed Cache (Redis)** must be scaled horizontally as a Redis Cluster with multiple shards to support a million write operations per second.
    

This perfectly illustrates why Alex Xu's design relies entirely on the **Message Queue**; if your Fanout Workers were deployed on a single `t3.micro` without a queue, the server would immediately experience CPU exhaustion and run out of memory.
