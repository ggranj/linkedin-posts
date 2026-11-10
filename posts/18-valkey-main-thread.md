<!-- file date: 2026-11-11 · committed before publication -->

📌 A week of latency spikes every five minutes across three services. The managed Valkey behind them looked healthy on Grafana. It was not.

🔎 The panel showed cpu_user_main_thread at 65 to 70 percent. The cloud console showed system time next to it. Together the single main thread sat at 100 percent in peaks. Valkey is one thread per node, and no HPA on the consumers moves that wall, only shards do. One day later the data lived on a 4 by 3 sharded cluster and the feature service's p99 was a flat 10 ms line.

🧠 Before that we chased the wrong suspect. New analytics events, 500 per second, went out through the Istio sidecar, so we gave the sidecar more CPU. It helped by a third and taught a lesson. A sidecar request is part of the pod's total. Raise it and a percentage based HPA sees lower utilisation and scales less eagerly. Fix one container, change the autoscaling of the whole pod.

💡 Two habits came out of it. When the dashboard says fine and everyone is slow, open the provider's console. And grep the helm charts for who else reads that store. Three services did, nobody remembered.

#golang #valkey #redis #kubernetes #incident

---

Gemini Banana prompt:

Cartoon illustration, clean outlines, colorful, slightly humorous, LinkedIn-professional style. The blue Go gopher stands between two monitors. The left monitor is a calm green Grafana-style gauge reading "65%" labeled "user". The right monitor is a cloud console with a red gauge at "100%" labeled "user + system", the gopher's eyes wide. Behind the monitors a single-lane road labeled "main thread" is jammed with tiny delivery trucks labeled "GET", while next to it a fresh four-lane highway labeled "4 x 3 shards" flows freely. In the corner a small box labeled "istio-proxy" lifts a dumbbell labeled "CPU request" and a balance scale labeled "HPA" tips to one side. A small handwritten signature "ggranj" in the bottom-right corner. Warm lighting, friendly vibe, no text overlays except the labels described.
