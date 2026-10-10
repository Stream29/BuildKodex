# Task Tree

- `Separate retained-object evidence from committed heap/RSS`() // complete
- `Measure actual heap use and standard periodic concurrent collection`() // complete; no accepted fix
- `Reject adoption without functional and navigation gates`() // rejected

# Details

- Parent: [full GUI repair](../executable/2026-10-08-repair-full-model-idea-import-oom.md).
  R20's normal compiler exit is insufficient; IDEA receive-stage memory
  still triggers the original reserve before external success.
- R18 MAT groups concern reachable objects at a particular earlier phase.
  They cannot prove the amount of unused committed heap at the later guard.
  Add passive `jstat -gc` sampling every10s, never a requested collection.
- Private R21 keeps the exact V3 plugin, idle60 policy, actual Native2.4.0
  cache, all targets, Gradle4GiB /IDE3GiB /compiler2GiB and4096MiB reserve.
  Add only `-XX:G1PeriodicGCInterval=30000` and numeric private GC logging.
- [JDK25 G1 documentation](https://docs.oracle.com/en/java/javase/25/gctuning/garbage-first-garbage-collector-tuning.html)
  specifies that periodic collection is disabled by default and normally uses
  concurrent marking. No external `System.gc`, Full-GC command, process kill
  or lower safety threshold. Confirm the actual collector and cycle cause
  from the GC log; do not infer released RSS from object counts.
- Preserve R20, including its failed acceptance. Fresh private inputs and
  complete four-import native observations remain mandatory. A lower RSS
  alone does not close the repair or justify production JVM tuning.
- R21 stops at the unchanged host reserve before external success, after the
  Gradle action completes. Numeric GC log confirms G1 and a natural allocation
  failure Full GC:4094MiB to3140MiB, with4096MiB committed. No `System.gc()`;
  the continuous allocation/collection phase does not reach a periodic cycle.
  The final sampled used heap is3,271,203KiB, not merely an idle empty heap.
- Reject this tuning as the current repair. Do not keep rerunning it or
  carry the unused periodic setting into the next candidate. Preserve numeric
  sampling, GC log and failed acceptance; original reserve/budgets remain.
  Continue [the explicit test-owner combination](../executable/2026-10-10-experiment-v3-with-real-test-owners.md).
