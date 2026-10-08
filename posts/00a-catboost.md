<!-- file date: 2026-02-13 · published on LinkedIn: (add link) -->

👋 Hi! Today I want to share my significant project that I'm undoubtedly very proud of and glad to be working on.

❓ What problem were we solving?
Ranking products in Elasticsearch had its limits: feature restrictions, lack of flexibility and so on. The solution was to move ML-ranking to a separate Go service with CatBoost.

⚙️ Technical solutions I'm proud of:
1. Atomic Model Pool Swapping
Instead of locks atomic.Pointer for lock-free pool replacement. The old pool is cleaned up only when all active predictions are completed (reference counting via atomic.Int32). Zero-downtime reload works like clockwork.

2. S3 + External Configurator
Models are versioned in S3, and the configurator provides metadata (which model to use, which features). If the configurator fails - the service runs on default settings. Independent deployment and rollback of models without restarting the service.

3. Smart Cache on Valkey
Pipeline operations for bulk requests (one round-trip instead of N). Asynchronous cache writes via fire-and-forget pattern-cache failure doesn't block ranking. Cache is an accelerator, not a dependency.

4. Parallel Batches with Concurrency Control
Requests to the external feature service go in batches of 200 items with a limit of 10 concurrent goroutines. Errgroup with SetLimit() prevents overload while speeding up processing by ~10x.

5. Model Validation Before Swap
Each new model undergoes tests on control samples, checks feature order and types. If something's wrong-rollback to the old model. Silent ML model bugs (like swapped features) won't slip through.

📊 What’s the result?
The service handles ~1k rps, reloads models without downtime, operates during partial infrastructure failures, and has transparent observability via OpenTelemetry. All in Go with sensible use of atomic operations, errgroup, and contexts.

📝 What’s next?
Ahead: A/B tests of new models, online learning, and further latency optimization. But even now, I'm proud of how we approached the task - every decision thought through for fault-tolerance and production readiness.

Have you had similar things in your projects? How do you handle hot reload of ML models? I'd love to discuss in the comments!

hashtag#backend hashtag#golang hashtag#machinelearning hashtag#catboost hashtag#search hashtag#ranking hashtag#highload hashtag#microservices hashtag#s3 hashtag#valkey
