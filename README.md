# SYSTEM DESIGN NOTES

## HOW TO MAKE A SCABLE SYSTEM (HIGH LEVEL DESIGN)

1. vertical scaling: optimise precision and increase through put with the same resources 
2. preprossing (e.g cron job) : prepare before hand during non pick hours 
3. Backups: keep backups and avoid single point of failure 
4. horizontal scaling: get more resources 
5. micro service architecture 
6. distributed system (partioning)
7. load distribution 
8. Decoupling 
9. Logging 
10. extensible

## LOW LEVEL DESIGN 
Its overlooking how we make the above steps, how we actually code it down in a clear and proper manageable codebase. 
Naming all the classes and how do we interepret the aspects (decoupled attributes for that section). 

## SCALING A SYSTEM - (horizontal vs Vertical scaling)
Issues based on horizontal and vertical scaling.
- Horizontal Scaling (Multiple servers) - buy more machines
- Vertical Scaling (Single server) - buy bigger machines

Horizontal Scaling (Multiple servers) | Vertical Scaling (Single server) |
--- | --- | 
Load Balancing Required | N/A |
*Resilient* | Single point of failure |
Network call (RPC - Remote Procedure Call) - **slow** | *Inter process communication* - **fast** |
Data Inconsistency | *Consistent* |
*Scales well as users increase* | Hardware Limit |

*Properties that we take* - over on the real world case of having both simultaneously over a distributed server. 

## LOAD BALANCING

Overlooking the topic for **consistent hashing**. talking about the case for *vertical scaling* over the server, how do we manage out the extent and the generic mapping to the allinged servers for that task in hand. How do we direct the request to the multiple instances for that server call in order to make out a response? 
over on the **N servers** we will need something to balance out the load dependent on the amount of calls over on the servers. This is **Load balancing**. 
Consistent hashing is one such balancer, We will get a random requestID (0 - M-1), we can take this requestID and hash it that can be mapped - 
```bash
h(r1) -> m1 % N

r1 - requestID 
m1 - mapped value for the hash 
N - number of servers

eg -> 
NOTE - the servers are s0, s1, s2, s3 
h(10) -> 3 % 4 -> 4rd server
h(20) -> 15 % 4 -> 4rd server
h(35) -> 12 % 4 -> 1st server

load factor -> 1/N
```


What if we need to add more servers here? now the load balancing will be different over on the probablity for it to come over on a server being lesser as the number of server increase (load factor increase -> 1/N), eg., we had 4 servers before so there was a 25% prob for the call being in that server, but when we jump to 5 servers that comes down to 20% (reflecting upon the change in the buckets of calls, **A SHIFT IN THE CALLS BUCKETS**). The cost of this CHANGE IS HUGE (imagine having 100 buckets before) we had to shift out mostly 100 buckets at together to work with the new hashing system. 

![Bucket Diagram](/image.png)

Imagine having a cache here, that gets completely dumped. A more general idea to do this is having a small shift from the ends of each servers that overlooks the calls on the new server and cause a minimal dump of information. 

![Generic Bucket Diagram](/image-2.png)

| **how does the consistent hashing overlooks this change in the bucket in the optimal way like above?**

Imagine having the request as a circle over the possible hashes, and the servers to have mapped hash over the similar function and further MOD it out to the N (the total servers we have) and assign the servers in the ring. Imagine being the worst case for that server to handle it.. We simply map the request to the nearest server in the **ring**. The load the the distance between the servers are uniformly random, thus the architecture to be the best for the case. 

![Consistent Hashing](/image-3.png)


```
The expected load factor (average) -> 1/N
```

In this architecture if we add a new server now, the new server feeds the load the hash function assigns to take. Or simply it governs the set area an old server was already handling. making the change much less in order for minimizing the aspect that affected previously over the call buckets. **but, practically we can have skewwed distributions**, if any server fails and the next fallback is comparatively away. (look the diagram to imagine that well), if the S1 server failed the S4 has the half of teh capacity. (This may be more prone in less number of servers). 

**Now how to solve the above?** We can make multiple virtual servers in the grid (not buy them, but mentally map them). Having multiple hash functions (h1, h2, h3 for example) **so the same server would map to 3 points (as an example)**, Doing it at log(M) the load never becomes skewwed. 

| Example code -> [Consistent Hashing](/consistentHashing.java)

## MESSAGE/TASK QUEUE

Having a system to relieve the client from having an immediate response (while your ordering food they ask "please wait for some time"), being a confirmation over the action done from the client. This gets pushed to a queue (of orders) of messages awaiting the fulfillment by the subscriber (the end maker). After which the fulfillment is handled and the required (the expected) response is given back the the client with what they want. This makes the client happy (even thou they need to wait), the server to take its time in order to process the request (imagine being an expensive request), being **completely async!** The messages could be pushed in set priority (time, expected params or simplt a FIFO). The client may work on other aspects, like placing new requests that doesn't need to wait for the time of fulfillment for the main request. **The other advantage being we can have multiple subscribers, or the servers to fulfill the request and be more resilient over the request handing** If one servers just burns, the next one can handle it out. (this is load balacing in the back, wioth some heart-beat machanism on the servers). Examples - RabbitMQ or Apache Kafka.

| how is is different from Pub/Sub? 

Pub/Sub is not a traditional message queue; it is a broadcast messaging pattern, whereas a message queue is a point-to-point task distribution pattern.

Key Differences
• Message Queue (Point-to-Point)
	• Pattern: One-to-one.
	• Behavior: A single message goes to only one consumer. Once a worker processes and acknowledges the message, it is removed from the queue.
	• Best For: Distributing background jobs, balancing workloads, and ensuring a task is done only once (e.g., processing a payment).
	• Example: Amazon SQS (Standard queues).
• Pub/Sub (Publish-Subscribe)
	• Pattern: One-to-many (Fan-out).
	• Behavior: A publisher sends a message to a topic, and every subscriber receives an independent copy of that message.
	• Best For: Broadcasting events, system-wide notifications, and event-driven architectures where multiple services need to react to the same action.
	• Example: Google Cloud Pub/Sub or AWS SNS.
The Overlap
Modern messaging tools like RabbitMQ or Apache Kafka can act as both, depending on how you configure your consumers and subscriptions. For instance, if you attach a single subscription/consumer group to a topic, a Pub/Sub system can behave like a work queue.


## Monolith v/s Microservices

Monolith | Microservices |
--- | --- | 
 **Myth** A huge machine running the entire system | **Myth** 1 function or group of features in bunch of services (systems) being interconnected |
A single source of truth for the entire business logic | chunked out into business units (connected via a gateway like ep-service) |
Adv - Scales out into multiple servers over load | Scability (easier to design) |
Adv - less moving parts (no need to think about breaking into services and easy to operate) | comparatively needs less context (to that service) |
Adv - less duplications | less paralled dependency between teams |
Adv - FAST (new internal RPC calls between servers) | **easier to reason and scale about (more clear picture in the content of service)** |
DisAdv - new team members needs a lot of context (like loop-backend) | not easy to design (could have far more parts) |
DisAdv - complicated deployments (any change require a new full deployment) |  |
DisAdv - SINGLE POINT OF FAILURE |  |
small team | large team (with different logics to cover) |
