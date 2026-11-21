# Canonical embedding contract

All Psy-Supabase embeddings use one model for:

- query embeddings
- interaction embeddings
- knowledge/document embeddings
- cached embeddings
- batch enrichment
- in-process associative memory

The canonical model is `sentence-transformers/all-MiniLM-L6-v2` and produces 384-dimensional vectors.

Database vector columns must use `vector(384)`. Existing `vector(2048)` columns must be migrated before deployment.
