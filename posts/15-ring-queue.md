<!-- file date: 2026-10-08 · committed before publication; published on LinkedIn: (add link) -->

📌 Your timeout covers waiting for an answer. It doesn't cover waiting for a place in the queue.

Load test on a Go ranking service: cache budget 350 ms, yet handlers sat 15-22 seconds inside a single DoMulti. Deadline set, honored nowhere.

🧠 Read the client source (valkey-go, ring.go). One TCP connection per node, a ring buffer of 1024 slots in front of it - and a slot is a whole pipeline, up to 500 GETs for us. Commands enter through:

func (r *ring) PutMulti(_ context.Context, ...)

The context is `_`. If the ring is full, it parks on a sync.Cond until a slot frees. The deadline check sits after this call returns - correct, instant, and only once you already hold a slot.

Two different waits. Your timeout owns one of them.

💡 Fix: a weighted semaphore in front of the ring, capped well below 1024, taken with Acquire(ctx) - the context-honest wait the library doesn't give you. Two details matter:
- release the slot when the pipeline completes, not when the caller gives up - otherwise the next one walks into a still-full ring
- background writes use TryAcquire: no slot, drop the write. A best-effort cache never queues ahead of the critical path

📊 Plus a test that asserts the library's behaviour. If upstream ever fixes PutMulti, the test fails - and the failure means the gate can go.

#golang #backend #valkey #redis #highload #incident

---

Gemini Banana prompt:

Cartoon illustration, clean outlines, colorful, slightly humorous, LinkedIn-professional style. A nightclub entrance in a server room. The door is labeled "ring buffer - 1024 slots" with a velvet rope. Behind the rope stands a long queue of envelope characters, each stamped "pipeline - 500 GETs", looking tired; one of them holds a small alarm clock showing "350 ms" that is clearly ignored. At the door sits a sleeping bouncer with a name tag "PutMulti(_ ctx)" - his ears are literally plugged, a little speech bubble from the clock says "hello? deadline?" and he doesn't react. In front of the rope, the blue Go gopher in a smart vest holds a clipboard labeled "Acquire(ctx)" and lets exactly a few envelopes through, politely turning one away toward a small bin labeled "TryAcquire: dropped" for background writes. On the wall, a framed certificate reads "Test: fails when the library gets better". Warm neon lighting, friendly vibe, no text overlays except the labels described.
