<!-- file date: 2026-11-17 · committed before publication -->

📌 A Go service throttled by CFS at 25 percent CPU. More cores did not help. One env var did.

🔎 It is the GMP model. GOMAXPROCS bounds P, the processors that run goroutines. Not M, the OS threads. A thread in a syscall leaves its P and keeps burning CPU. The GC takes a quarter of P for mark workers and pulls allocating goroutines into assist. Kubernetes charges every thread of the cgroup to one quota, 400 ms per 100 ms period at a limit of 4.

🧠 Quiet second, 4 P a quarter busy, about one core. GC mark lands in a busy second, all 4 P fill up, a few threads return from syscalls on top, and the pod wants 4.5 cores in one 100 ms window. Quota gone, all threads wait for the next period. Mark runs every few seconds, so 1 percent of periods throttled at 25 percent load. Extra cores changed nothing, automaxprocs raises GOMAXPROCS with the limit and the burst grows with it.

💡 The lever is headroom. Keep the limit at 4, set GOMAXPROCS=3 in env, automaxprocs honours it. One P less costs nothing at 25 percent. After the deploy the throttling panel went flat. Bonus trap, the kafka consumer's 16 percent spike set the axis and hid the pods' 0.3 percent.

#golang #kubernetes #performance #gc

---

Gemini Banana prompt:

Cartoon illustration, clean outlines, colorful, slightly humorous, LinkedIn-professional style. The blue Go gopher is a juggler on a small stage with four spotlights labeled "4 CPU". It keeps three balls in the air labeled "P", "P", "P", and a small sign on its belt reads "GOMAXPROCS=3". The fourth spotlight is deliberately empty and labeled "headroom", and in it a tiny garbage truck labeled "GC" and two little runners labeled "syscall" squeeze through happily. Above the stage a big stopwatch reads "100 ms" with a thin red slice labeled "quota". On the left a crumpled version of the same stage shows a gopher juggling four balls, the garbage truck crashing into the stopwatch and a red sign "throttled". On the right a small scroll of YAML shows "cpu: 4" and "GOMAXPROCS: 3" with a green checkmark. In the far corner a wall monitor shows a tall red spike labeled "kafka consumer 16%" dwarfing a tiny flat line labeled "pods 0.3%". A small handwritten signature "ggranj" in the bottom-right corner. Warm lighting, friendly vibe, no text overlays except the labels described.
