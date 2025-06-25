# Java Performance Tuning and GC- Algorithms and SWAP

## JVM Memory

<img height="400" src="img/memory_model.png" width="600"/>

## Garbage Collector

.. cleans up **unreachable** references

Generally follows the principle of:

1. mark/promote
2. sweep
3. compact

### Marking

<img height="300" src="/img/Marking.png" width="500"/>

### Promoting

<img height="300" src="/img/Promotion.png" width="500"/>

### Compacting

<img height="300" src="/img/Compaction.png" width="500"/>

### Types

| GC Name                        | Description                                         | Key Tuning Flags                                                                                                                      |
|--------------------------------|-----------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------|
| Serial GC                      | Single-threaded STW; best for small heaps or tests  | -XX:+UseSerialGC<br>-XX:MaxHeapFreeRatio=<…><br>-XX:MinHeapFreeRatio=<…>                                                              |
| Parallel (Throughput) GC       | Multi-threaded STW; maximizes overall throughput    | -XX:+UseParallelGC<br>-XX:ParallelGCThreads=<…><br>-XX:MaxGCPauseMillis=<…><br>-XX:GCTimeRatio=<…>                                    |
| CMS (Concurrent Mark-Sweep) GC | Concurrent low-pause STW; deprecated in JDK14+      | -XX:+UseConcMarkSweepGC<br>-XX:CMSInitiatingOccupancyFraction=<…><br>-XX:+UseCMSInitiatingOccupancyOnly<br>-XX:ParallelCMSThreads=<…> |
| G1 (Garbage-First) GC          | Region-based; pause-predictable; default since JDK9 | -XX:+UseG1GC<br>-XX:MaxGCPauseMillis=<…><br>-XX:InitiatingHeapOccupancyPercent=<…><br>-XX:G1ReservePercent=<…>                        |
| ZGC                            | Scalable, concurrent, region-based; low-latency     | -XX:+UseZGC<br>-XX:ZCollectionInterval=<…><br>-XX:ZUncommitDelay=<…><br>-XX:ZFragmentationLimit=<…>                                   |
| Shenandoah GC                  | Concurrent, region-based; ultra low-pause           | -XX:+UseShenandoahGC<br>-XX:ShenandoahGCThreads=<…><br>-XX:ShenandoahHeapRegionSize=<…><br>-XX:ShenandoahUncommitDelay=<…>            |

Note: The parallel collectors only provide speedup if the system is not CPU-bound. If the GC is not
able to keep up this triggers a full(non-parallel GC)

## Out of memory errors

### Out of native memory

32-bit systems are constrained by ~3.5GB. If RAM and virtual-RAM are
fully spent and the J

### JNA allocated memory

Hard to track since JVM cannot manage JNA(that allocate native memory)

### GC- overhead limit reached

JVM throws an out of memory error when it determines that the application spends
too much time performing GC.

All the following conditions must be true

| # | Description                                                     | Flag                                        | Default       |
|---|-----------------------------------------------------------------|---------------------------------------------|---------------|
| 1 | % of total time spent in GC exceeds threshold                   | ‑XX:GCTimeLimit=N                           | 98 (i.e. 98%) |
| 2 | % of heap freed by GC is below threshold                        | ‑XX:GCHeapFreeLimit=N                       | 2             |
| 3 | Conditions (1) & (2) have held for this many consecutive cycles | (no dedicated flag)                         | 5 cycles      |
| 4 | Turn on the GC‐overhead‐limit check (terminates JVM when hit)   | ‑XX:+UseGCOverheadLimit/-UseGCOverheadLimit | Enabled (`+`) |

Note: in the 4th run the JVM removes all soft-references

## Common patterns ([source](https://blog.gceasy.io/interesting-garbage-collection-patterns/))
<img height="300" src="/img/x/img_0.png" width="500"/>
<img height="300" src="/img/x/img_1.png" width="500"/>
<img height="300" src="/img/x/img_3.png" width="500"/>

## Tools

For this we will mostly look at JDK tools.

| Tool          | Desciption                                     |
|---------------|------------------------------------------------|
| **jcmd**      | prints basic class, threat, JVM information    |
| jconsole      | GUI-profiling tool                             |
| jmap          | creates heap dump and memory usage information |
| **jinfo**     | prints system properties                       |
| jstack        | dumps the stack of Java processes              |
| jstat         | provides information about GC                  |
| **jvisualvm** | GUI-profiling tool                             |

## Misc

'-XX:+UseLargePages' - lager pages means less context/page swapping. This will generally improve
performance if supported by os

## Swapping

The os provides virtually more RAM than it has by using the underlying persistence medium(eg SSD).
This will always be slow. The only way to prevent this is to not allocate beyond the available RAM.