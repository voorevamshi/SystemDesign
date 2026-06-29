
### High-level design
 The design is divided into two flows: feed publishing and news feed building. 
 * Feed publishing: *  when a user publishes a post, corresponding data is written into cache and database. A post is populated to her friends’ news feed. 
 * Newsfeed building:*  for simplicity, let us assume the news feed is built by aggregating friends’ posts in reverse chronological order.

Figure 3-1(11-2) and Figure 3-2(11-3) present high-level designs for feed publishing and news feed building flows, respectively.

### Feed publishing:  
User makes a post with content "Hello" through API.

<img width="500" height="533" alt="image" src="https://github.com/user-attachments/assets/87aa66a5-4fe1-473c-a8e3-d1a75fa9bd8c" />

### Newsfeed building:
A user sends a reuest to retrive her news feed

<img width="450" height="495" alt="image" src="https://github.com/user-attachments/assets/f53fb1e6-ea40-4a4a-8389-cc8f72547db6" />

In the high-level design phase, the system looks deceptively simple:

-   A client requests a feed (`/v1/me/feed`) through a **Load Balancer**.
    
-   Traffic is routed to a monolithic or singular **News Feed Service**.
    
-   The data is pulled directly out of a **News Feed Cache**.
    

### Feed publishing deep dive (Figure 3-3(11-4))

<img width="519" height="546" alt="image" src="https://github.com/user-attachments/assets/ffb63d98-7b2f-40f4-9f80-58c6ee4698ec" />

### Newsfeed retrival deep dive (Figure 3-4(11-7))

<img width="505" height="433" alt="image" src="https://github.com/user-attachments/assets/e30bb615-0149-4712-911f-c9959a2b0f07" />


### Questions
[1) In Figures 3-1 and 3-2, are the Fanout Service and News Feed Service the same or different?](3-1_3-2_isSameOrdifferent.md)
### Analyzing Feed publishing  Figure 3-3: The Endpoint & The Cache
[2 (i) Is the News Feed Cache a Redis cache?](isNewsFeedCacheaRediscache.md)
[2 (ii) Explain Fanout Workers like I'm 5 (ELI5)](explainFanoutWorksers.md)
[2 (iii) Why route through a Message Queue and Workers instead of writing directly from the Fanout Service to the Redis Cache?](whyFanoutNotDirectlyWriteToCache.md)
[2 (iv) Why a t3.micro Struggles with Spring Boot](whyT3microStrugglesWithSpringBoot.md)
[2 (v) The Full EC2 Sizing Matrix (For Spring Boot Apps)](fullEc3Metrix.md)

        
