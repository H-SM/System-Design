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
- Horizontal Scaliing (Multiple servers) - buy more machines
- Vertical Scaliing (Single server) - buy bigger machines

Horizontal Scaliing (Multiple servers) | Vertical Scaliing (Single server) |
--- | --- | 
Load Balancing Required | N/A |
*Resilient* | Single point of failure |
Network call (RPC - Remote Procedure Call) - **slow** | *Inter process communication* - **fast** |
Data Inconsistency | *Consistent* |
*Scales well as users increase* | Hardware Limit |

*Properties that we take* - over on the real world case of having both simultaneously over a distributed server. 
