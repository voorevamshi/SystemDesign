### The Full EC2 Sizing Matrix (For Spring Boot Apps)

When designing a large-scale system like a News Feed, senior engineers categorize application server tiers based on their compute, memory, and network capabilities.

Here is the complete tier table mapping out how different AWS instances perform when running a standard Spring Boot / Tomcat microservice:

## AWS Instance Class Comparison for Spring Boot Microservices

| Tier | AWS Instance Class | Specs (vCPU / RAM) | Average Spring Boot QPS | ELI5: What is this instance like? | Best Used For |
|---|---|---|---|---|---|
| **Micro Tier** | **t3.micro** | 2 vCPU / 1 GB | ~150 QPS | 🚲 **A tiny bicycle**. It gets you down the street, but it will break if you try to haul heavy boxes. | Proof of concepts, local development testing, or small personal side projects. |
| **Small Tier** | **t3.small** | 2 vCPU / 2 GB | ~400 - 600 QPS | 🛴 **A basic scooter**. Can carry a bit more weight, but will struggle on steep highways. | Internal tools, low-traffic staging environments, or lightweight utility microservices. |
| **Medium Tier** | **t3.medium** | 2 vCPU / 4 GB | ~800 - 1,200 QPS | 🚗 **A standard sedan car**. Safe and reliable for everyday driving, but not built for racing. | Moderate production workloads; good for microservices with predictable, steady read traffic. |
| **Production Tier (Compute Optimized)** | **c6i.xlarge** | 4 vCPU / 8 GB | ~3,000 - 5,000 QPS | 🚚 **A heavy-duty delivery truck**. It can move massive amounts of cargo at high speed without breaking a sweat. | Core production microservices. Ideal for Fanout Workers or News Feed Services. |
| **High-Performance Tier** | **c6i.4xlarge** | 16 vCPU / 32 GB | ~15,000 - 20,000 QPS | 🚆 **A massive freight train**. Moves incredible volume, but requires significant space and cost to run. | Monolith architectures handling blended workloads or heavy high-throughput data processing engines. |


### Deep Dive: How to Choose a Tier for 10M Users

As a senior engineer, you don't just guess these tiers—you calculate them based on your bottlenecks:

1.  **Memory Bottleneck (Why 1GB is dangerous, but 4GB+ is safe):** A standard Spring Boot application requires about 500MB of RAM just to boot up. If you use a `t3.micro` (1GB), you only have 512MB left for actual data processing. If your Fanout Service pulls a list of 5,000 friend IDs from the database into memory as Java Objects, your JVM memory usage will instantly spike. A `t3.medium` (4GB) or `c6i.xlarge` (8GB) gives the JVM plenty of room to allocate memory for incoming payloads without triggering garbage collection pauses or crashing due to `OutOfMemoryError`.
    
2.  **The Burstable Trap (`t3` series):** The `t` in `t3` stands for **burstable**. These instances use a "CPU credit" system. If traffic is low, you accumulate credits. If a sudden flash sale or viral post happens, the instance consumes credits to handle the spike. If you run out of credits, AWS intentionally throttles your CPU speed to a crawl, and your QPS will instantly tank. For high-volume systems processing millions of fanouts, you want the `c` series (Compute Optimized) because their performance is 100% dedicated and never throttled.
    
Does the difference between the burstable tiers (`t3`) and production-grade tiers (`c6i`) make sense for your Spring Boot applications?
