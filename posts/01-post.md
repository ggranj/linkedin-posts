<!-- file date: 2026-01-23 · published on LinkedIn: (add link) -->

📌 Why Our Team is Considering Switching from Redis to Valkey: Real Performance Data and Perspectives

Our backend development team has been looking at Valkey as an alternative to Redis for quite some time, and I recently studied a detailed comparison of these technologies that I'd like to share.

❓What is Valkey and why does it matter?

Valkey is a fork of Redis that maintained the BSD 3-Clause license after Redis changed its licensing model in spring 2024. The key difference: Valkey remains fully open-source, without an enterprise version or commercial restrictions.

❓What did the benchmarks show?

Tests based on YCSB demonstrated that Valkey 8.0 outperforms both Redis 6.2 and Redis 7.2 in all load scenarios. Particularly interesting is that Redis 7.2 often performs worse than version 6.2, confirming the experience of many teams who encountered regressions when upgrading.

⚙️ Technical improvements that impressed us:

- Innovative approach to I/O threads: In Redis, I/O threads and the main thread often "wait" for each other. In Valkey, this is fixed — threads can work in parallel thanks to the use of atomics and task queues for each I/O thread.
- Hash table optimization: Improved processor cache utilization through prefetch, significantly reducing data access latency.
- Dual-channel replication: Moving the change buffer to the replica, reducing the risk of Out Of Memory on the primary during intensive workloads.
- Key embedding: Storing short keys directly in the structure, saving 10-20% of RAM.
- Extended metrics: New monitoring capabilities, including tracking load on individual cluster slots.

📝 What's next?

The active developer community around Valkey promises more dynamic project development. We plan to conduct our own testing in our infrastructure before making a final decision, but the initial data looks promising.

Has anyone already switched to Valkey in production? I'd love to hear about your experience in the comments!
hashtag#backend hashtag#golang hashtag#redis hashtag#valkey hashtag#HighLoad hashtag#OpenSource
