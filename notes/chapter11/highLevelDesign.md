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
    
