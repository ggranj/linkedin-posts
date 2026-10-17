<!-- file date: 2026-10-17 · committed before publication; published on LinkedIn 2026-10-17: https://www.linkedin.com/feed/update/urn:li:activity:7517195420394151937 -->

📌 The experiment ended a week ago. Its network call didn't.

Bad platform day: our Go ranking service got slow, redeploy didn't help. The cause: a synchronous gRPC call to the A/B service on every request with a user id - for an experiment that had already finished.

🧠 Why nobody saw it: the call ran in an errgroup next to feature enrichment. While A/B was faster, it was invisible; the day it got slower, its latency became the request's. Client timeout 175 ms, inside a 300 ms budget from our caller. The LRU in front (100k, 30 min) missed new devices and fresh pods.

💡 Fix: one env line. The boost sat behind a flag - PERS_BOOST_ENABLED=false dropped the A/B and feature-store calls at once. No code, no pipeline; no boost until the dependency is gone.

Takeaways:
- A parallel call is a hidden critical path - until it becomes the slowest.
- Experiment over → gate out of the code the same week.
- The sister service dropped the same gate months earlier: p99 48 → 27 ms. A lesson not carried over is half a lesson.

📊 Follow-up: one page per service - dependency → switch → cost. Ten seconds to read; mid-incident, finding the switch took ten minutes.

#golang #backend #incident #abtesting

---

Gemini Banana prompt:

Cartoon illustration, clean outlines, colorful, slightly humorous, LinkedIn-professional style. A relay race on a running track inside a server room. Three runners carry the same baton labeled "request - 300 ms": a fast runner "feature enrichment", and next to him a slow, sweating runner in an old-fashioned lab coat labeled "A/B check" who holds a dusty ticket reading "experiment: FINISHED" with cobwebs on it, and a wristwatch showing "175 ms". The finish line judge, an impatient envelope character labeled "caller", taps a big stopwatch. On the side of the track, the blue Go gopher in a detective trench coat and deerstalker hat examines a wall panel of toggle switches through a magnifying glass; one switch is labeled "PERS_BOOST_ENABLED" and his paw is flipping it to OFF. A small sign on the panel reads "one page: dependency → switch → cost". Warm lighting, friendly vibe, no text overlays except the labels described.
