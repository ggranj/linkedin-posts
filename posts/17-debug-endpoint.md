<!-- file date: 2026-10-27 · committed before publication; published on LinkedIn: (add link) -->

📌 10% of products came back with empty features. No logs, no metrics, two hours.

We were restoring ranking after an outage and ML asked which products had no features in the store. I had no access to logs. A metric with product ids would blow up cardinality. Grafana is not for lists.

🧠 So the service counted it itself. A map of product id to miss count in pod memory, capped at 5000 entries, about 0.5 MB. Exposed on the existing debug ingress as GET /debug/empty-goods, with DELETE to reset before the next data load. Add costs 3.6 µs per request with 300 misses, Report takes 0.55 ms at 5000 entries, sorting runs outside the lock. The ranker already cuts input to 150 products, so a second pass over the map is free. For the catalog service with 10k products it would not be, that one needs a single pass and sampling.

💡 An hour after rollout the lead pulled a JSON with 1351 unique ids and handed it to ML to join with their pipeline. From idea to production in two hours.

📊 When the right tool is missing, a temporary one inside the service beats waiting for access. Mark it with the ticket id and delete it once the question is answered.

#golang #backend #observability #incident

---

Gemini Banana prompt:

Cartoon illustration, clean outlines, colorful, slightly humorous, LinkedIn-professional style. Inside a server room, the blue Go gopher in a detective trench coat stands at a small wooden door in the side of a big server rack, the door labeled "GET /debug/empty-goods". Through the open door a tidy little notebook is visible with a tally list "product id - misses", a sticker on it reads "cap 5000". Next to the rack, a locked heavy vault door labeled "LOGS" with a sign "access pending" and cobwebs, and a Grafana-style dashboard on the wall showing a confused shrug emoji and the words "not for lists". On the floor, a conveyor belt carries product boxes, every tenth box is empty and transparent. A clock on the wall shows "2 hours". A small handwritten signature "ggranj" in the bottom-right corner. Warm lighting, friendly vibe, no text overlays except the labels described.
