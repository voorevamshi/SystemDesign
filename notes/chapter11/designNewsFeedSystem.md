
### High-level design
 The design is divided into two flows: feed publishing and news feed building. 
 * Feed publishing: *  when a user publishes a post, corresponding data is written into cache and database. A post is populated to her friends’ news feed. 
 * Newsfeed building:*  for simplicity, let us assume the news feed is built by aggregating friends’ posts in reverse chronological order.

- [High level Design](highLevelDesign.md)
- [Q & A](q&a.md)

### Feed publishing deep dive (Figure 3-3(11-4))

<img width="519" height="546" alt="image" src="https://github.com/user-attachments/assets/ffb63d98-7b2f-40f4-9f80-58c6ee4698ec" />

### Newsfeed retrival deep dive (Figure 3-4(11-7))

<img width="505" height="433" alt="image" src="https://github.com/user-attachments/assets/e30bb615-0149-4712-911f-c9959a2b0f07" />

