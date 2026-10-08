<!-- file date: 2026-02-13 · published on LinkedIn: (add link) -->

🤔 Ever wondered how a system can turn a user's browsing history into spot-on recommendations, seemingly out of thin air?

That's the kind of magic I dove into with my last project: integrating a real-time ONNX recommendation model into Go-based ranking service for personalized search results.

⚙️ Technical feats:
- Cached Tensor Specs & On-the-Fly Creation Cache output metadata (shapes, types) once per load for quick, type-safe tensor allocation-handling dynamic batches effortlessly.
- Streamlined Tensor Prep & Pooled Inference Convert string IDs to padded tensors/masks, run via DynamicAdvancedSession in channels. Parse ints back to IDs, all concurrency-safe.
- Leak-Proof Cleanup & Validation Deferred destroys, timeouts, and pre-swap metadata checks prevent issues like mismatched tensors.
- Atomic Session Pool Swapping Lock-free hot reloads via atomic.Pointer. Old pools drain safely with ref-counting, keeping inference uninterrupted during updates.
- S3 + Dynamic Configurator Models pull from S3, with metadata like output tensor names from a configurator. Fallback defaults and a reload watcher ensure seamless swaps without restarts.

👉 This is my go-to practice for flexibility - integrating S3 and configurators wherever possible, especially with models, wrappers, and cross-team interactions where everyone needs something from everyone else.

📊 Outcome?
Handles ~1k rps with <50ms latency, generating recs on the fly while staying robust to outages-pure Go with onnxruntime_go.

📝 Next up?
A/B tests, GPU boosts. It's a game-changer for ML-driven search.

Tackled ONNX in Go? How do you handle live model updates?
