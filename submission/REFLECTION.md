# Lakehouse Anti-Pattern Reflection

Our team is most vulnerable to **Anti-Pattern #3: Treating the External Vector DB as the System-of-Record** (and the resulting *Vector Lifecycle Bug*).

In our RAG and LLM agent pipelines, we ingest high-velocity document chunks and push embeddings into an external vector database alongside our raw data store. When data updates or compliance erasure requests (GDPR / EU AI Act Art. 10) occur in the lakehouse, downstream vector synchronization is often one-directional (upserts only) or decoupled from transactional deletes. 

As demonstrated in Notebook 7, soft or hard deletions in Delta Lake leave stale embeddings retrievable in the external vector index. This exposes our serving pipeline to severe data leakage and privacy compliance violations.

**Mitigation:** We must treat the Lakehouse as the single system-of-record, store vector embeddings directly in the table schema (with `int8` quantization), and enforce CDC-driven change data feeds (`load_cdf`) to guarantee atomic delete propagation across all derived vector indices.
