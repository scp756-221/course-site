## Reading 9
1. [Consistency Tradeoffs in Modern Distributed Database System Design](http://www.cs.umd.edu/~abadi/papers/abadi-pacelc.pdf)

## Overview
The paper: Consistency Tradeoffs in Modern Distributed Database System Design: CAP is Only Part of the Story

This paper by Daniel Abadi responds to a dozen years worth of writing and implementing databases after Eric Brewer gave his famous talk on the relation between consistency, availability, and partitions, which is generally (if misleadingly) referred to as the “CAP theorem”.

For our purposes, the implications of Brewer’s talk are straightforward: Given that network partitions will unavoidably occur, a system that stores the same mutable data item in multiple places must be designed to emphasize either consistency or availability in the event of a partition.

Abadi’s article extends this idea with an essential point: Although network partitions do occur, they are rare and most of the time a system will be connected. In this far more common case, the design must trade off consistency against latency. Abadi combines the two conditions, partitioned and connected, via a single acronym, PACELC.

## Part 1: pp. 37–40, (not including “Tradeoff Examples”)
Abadi frames his article in terms of distributed databases (DDBS) but the principles at work apply to any system of distributed persistent storage (including key-value stores) or systems built up on multiple datastores/databases. Although his introduction emphasize database-oriented concepts such as ACID, this isn’t important to how we will use his ideas in this class. The examples he gives are not important either. You may recognize Cassandra from your coursework as it remains commercially relevant; the Dynamo that is mentioned here is the predecessor of Amazon's DynamoDB that you have been using in the exercises and term project.

### CAP is for failures
The main point of this section is that designers make tradeoffs for both the partitioned case (“CAP”) and the far more common connected case. Abadi wants to highlight the tradeoffs in the connected case.

### Consistency / Latency tradeoff
The main point of this section is that to ensure high availability, designers must replicate data at least to another availability zone (AZ) and perhaps even to a different region. Abadi’s phrase, “over a WAN (Wide Area Network)” means “to another AZ or region”.

The table of latency magnitudes shows that transferring data to another AZ has 10 times greater latency than transferring within an AZ/datacentre, while transferring data to a distant region (say, western North America to Europe) is 1000 times greater. This size of these latencies drives the design choices in the next section.

### Data replication
This section is the heart of the paper. There are just three possible designs for replicating data. Each makes a distinct tradeoff between degree of consistency and latency.

The key point: There is no way to escape making a consistency-latency tradeoff if you want to maintain high availability. The tradeoff occurs even when the system is fully connected.

## Part 2: pp. 40 (beginning at “Tradeoff examples)–42
In the rest of the article, Abadi considers the implications of the necessary tradeoff he described in Part 1.

### Tradeoff examples
This section features detailed analyses of the designs of four systems. (The paper may not describe the current Cassandra design, which has likely developed considerably since the paper was written.)

Key conclusion:

> Ignoring the consistency/latency tradeoff of replicated systems is a major oversight,
> as it is present at all times during system operation, whereas CAP is only relevant
> in the arguably rare case of a network partition. In fact, the former tradeoff could
> be more influential because it has a more direct effect on the systems’ baseline
> operations. (p. 41)

### PACELC
Abadi summarizes this broader tradeoff by the acronym PACELC, pronounced “pass-elk”. This acronym and the principle it summarizes will be important in the course. The details of the specific systems are not as important but as to how he analyzes them.

## Your Turn

   Take 30 minutes for [Quiz 9](https://coursys.sfu.ca/2022sp-cmpt-756-g1/+q8/). 
