<!-- file date: 2026-06-26 · published on LinkedIn: (add link) -->

📌 Our Redis cluster collapsed under MULTI/EXEC transactions. We never wrote any.

A/B rolling our Go ranking service to 50%. The feature-store Valkey starts choking. Commandstats panel: EXEC ≈ PTTL ≈ HGETALL — all 87k/s, matching digit-by-digit on a single replica.

But our code has zero `MULTI`/`EXEC`. So who's writing them?

🧠 Read valkey-go 1.0.72 source, `pipe.go`. The `.Cache()` builder method — client-side caching — wraps every CSC miss in:

`CLIENT CACHING ON → MULTI → <cmd> → PTTL → EXEC`

Four server ops per miss instead of one. Beautiful when CSC hits — fatal when it doesn't.

💡 Our trigger: a Kafka consumer burst-writes features. Every write invalidates client trackers. Next reads miss → 4× amplification × hundreds of pods. Each hit is free, every miss is a paper cut at scale.

📊 Fix: drop `.Cache()` where reuse is low (user features — touched once per request). Keep it where reuse is high (catalog items — touched 500× per request). 28 lines of diff. Server load back to baseline.

🔧 Takeaway: client-side caching isn't free. Under heavy invalidations it's worse than no cache. When you see ops you didn't write, read the library source — not the docs.

#golang #backend #redis #valkey #caching #incident #postmortem

---

Gemini Banana prompt:

Cartoon illustration, clean outlines, colorful, slightly humorous, LinkedIn-professional style. A detective scene in a server room. The blue Go gopher (center) wears a Sherlock-style trench coat and holds a magnifying glass, leaning over a thick open book on a desk labeled "valkey-go / pipe.go". Floating up from the book is a ghostly translucent command chain: "MULTI → HGETALL → PTTL → EXEC", each command with a tiny confused face whispering "but we didn't write me!". In the background, a friendly Valkey/Redis server cube character sweats and clutches its chest, drowning in a downpour of small Kafka envelopes stamped "HSET" pouring from a cloud above. To the side, a chalkboard shows ".Cache()" with an arrow into the ghostly chain, labeled "client-side caching: 4 ops per miss". A small wall calendar in the corner is circled in red: "A/B 50% rollout day". Warm investigative lighting, sleuth/mystery vibe, no text overlays except the labels described.
