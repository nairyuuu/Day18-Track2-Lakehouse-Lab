# Lakehouse Anti-Pattern Reflection

I am most vulnerable to **Anti-Pattern #3: Treating the External Vector DB as the System-of-Record** (and the resulting *Vector Lifecycle Bug*).

In my RAG and LLM agent projects, I ingest high-velocity document chunks and push embeddings into an external vector database alongside raw storage. When data updates or compliance erasure requests (e.g., GDPR or EU AI Act Art. 10) occur in the lakehouse, downstream vector synchronization is often one-directional (upserts only) or decoupled from transactional deletes.

As demonstrated in Notebook 7, soft or hard deletions in Delta Lake leave stale embeddings retrievable in the external vector index. This exposes the serving pipeline to severe data leakage and privacy compliance violations.

**Mitigation:** I must treat the Lakehouse as the single system-of-record, store vector embeddings directly in the table schema (with `int8` quantization), and enforce CDC-driven change data feeds (`load_cdf`) to guarantee atomic delete propagation across all derived vector indices.
