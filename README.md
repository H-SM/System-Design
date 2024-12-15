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