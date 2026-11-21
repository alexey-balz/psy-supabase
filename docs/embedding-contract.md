# Canonical embedding contract

All Psy-Supabase embeddings use exactly one model for every operation:

- query embeddings
- interaction embeddings
- knowledge/document embeddings
- cached embeddings
- batch enrichment
- associative memory

Canonical model: `sentence-transformers/all-MiniLM-L6-v2`
Canonical dimension: `384`

The database schema should use `vector(384)` for every embedding column.
