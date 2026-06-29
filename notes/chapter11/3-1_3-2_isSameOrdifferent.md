### 1) In Figures 3-1 and 3-2, are the Fanout Service and News Feed Service the same or different?

They are **different** services handling completely opposite directions of data flow:

-   **Fanout Service (The Write Flow - Fig 3-1):** This service is triggered when a user _creates_ a new post. Its job is to take that new post and push (fan out) the post ID to all of that user's friends so it shows up in their feeds. It is **write-heavy**.
    
-   **News Feed Service (The Read Flow - Fig 3-2):** This service is triggered when a user _opens_ their app to read their own feed. Its only job is to quickly fetch the list of pre-computed post IDs from the cache for that specific user. It is **read-heavy**.
    

They are separated to allow you to scale them independently (e.g., allocating more instances/pods to the Read service since users read feeds vastly more often than they post).
