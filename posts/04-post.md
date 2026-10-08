<!-- file date: 2026-02-05 · published on LinkedIn: (add link) -->

📌 Canary release, part 2: The replica scaling trap

Latency spiked from 50ms to 5 seconds during canary rollout. Here's the fix.

(If you missed part 1 about the "initial: true" flag trap - check my previous post for the full picture)

🧐 What went wrong:

Canary config looked correct: fixed 2 replicas, weights 10% -> 30% -> 50% -> 70% -> 90% -> 100%. Worked fine up to 70-90% weight.

🧠 Root cause:

With ~800 req/s at 90% weight:
- 720 req/s -> 2 canary pods = 360 req/s per pod
- 80 req/s -> 4-16 stable pods = 5-20 req/s per pod

That's up to 72x load imbalance. Canary pods were drowning while overloaded while stable pods remained mostly idle.

🛠️ The fix:

Scale canary replicas dynamically with traffic:
- 10-30% weight -> 4 replicas
- 50-70% weight -> 6 replicas
- 90-100% weight -> 8 replicas

This keeps ~60-90 req/s per pod throughout the rollout.

💡Takeaway:

Always calculate: (traffic x weight) / replicas at every step. What handles 10% traffic might collapse at 90%.

Have you hit similar canary rollout difficulties?

hashtag#backend hashtag#kubernetes hashtag#ArgoRollouts hashtag#canary hashtag#devops hashtag#golang
