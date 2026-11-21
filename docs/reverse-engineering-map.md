# Reverse-Engineering Concept Map

This document translates repository structures into maintainable concepts. It uses source and tests as evidence while marking interpretation and uncertainty.

## System nouns

### Question

A user's text supplied to `/chat`.

Implementation entry: `main.chat()` extracts `request.json["question"]`.

Derived objects include validated input, a toxicity result, an embedding, topic/emotion inference, pain-point results, and generation context.

### Interaction

A question and generated answer with context and metadata.

Implementation: `RAGProcessor.generate_response()` invokes `DatabaseManager.save_interaction()`.

Metadata may include session ID, topic, emotion, template, pain-point details, and context sources.

### Embedding

A vector representation of text used to find semantic similarity.

Implementation:

- primary generation through `ModelManager` / `TextGenerator`;
- fallback SentenceTransformer generation;
- database-side caching and pgvector lookup.

### Knowledge context

Text from documents considered semantically relevant to the question.

Implementation:

- `RAGProcessor._enhance_context_with_relevant_documents()`;
- `DatabaseManager.find_similar_documents()`;
- configurable similarity thresholds and character limits.

### Conversation context

Recent or semantically similar prior interactions used while generating a response.

Implementation:

- chronological retrieval through context-determination helpers;
- `create_context_from_similar_interactions()`;
- conversation-history retrieval in the database layer.

### Pain point

A product-level label for a repeated or semantically persistent concern.

Implementation:

- `PainPointDetector` analyzes history and chunked text;
- `RAGProcessor.detect_pain_points_from_embedding()` asks the database for a candidate;
- session monitoring compares message similarity and themes.

This term does not itself establish diagnosis or clinical severity.

### Approach

A machine-readable response strategy, such as anxiety exploration or gentle refocus.

Implementation:

- pain-point data supplies an `approach_type`;
- `utils_mapping.map_approach_to_template()` translates it to a template;
- `PromptSelector` / `ResponseGenerator` use the resulting context.

### Response

The final answer returned by `/chat`.

Implementation:

- generated as a string;
- may include optional monitoring metadata;
- saved with the question and context.

## Component map

| Component | Inputs | Decisions | Outputs | Evidence strength |
|---|---|---|---|---|
| Flask request hooks | Headers, JSON, environment | User context and initialization | `g.db_manager`, errors | Direct |
| `RAGProcessor` | Question, session ID, database, generator | Pipeline coordination | Response and metadata | Direct |
| `DatabaseManager` | User ID, text, vectors | Storage and SQL/RPC operations | Records and similarity results | Direct |
| `EmbeddingProviderAdapter` | Text, configured model | Primary/fallback embedding choice | Vector or zero fallback | Direct |
| `PainPointDetector` | History, thresholds | Repetition or similarity classification | Pain-point data | Implemented heuristic; clinical validity unverified |
| `PromptSelector` | Question text | Topic/emotion/approach context | Prompt-selection metadata | Implemented heuristic |
| `ResponseGenerator` | Context and pain-point results | Template assembly | Prompted generation request | Direct |
| `TextGenerator` | Prompt and settings | Local model output | Text | Direct |
| Monitoring session | Messages and responses | Similarity, themes, intensity | Dashboard and status data | Heuristic |

## Separation of evidence and interpretation

### Evidence

- The code made a database call.
- The database returned a vector match with a similarity score.
- The text generator returned a string.
- The interaction was persisted.
- A test asserted a branch or output shape.

### Interpretation

- The retrieved document is relevant.
- The question indicates anxiety.
- A repeated topic is a pain point.
- The approach template is appropriate.
- The answer is therapeutically helpful.

The repository contains mechanisms for interpretation. It does not establish every interpretation's clinical correctness.

## Test as specification

- **Existence and initialization:** classes and configuration load correctly.
- **Happy paths:** embeddings, retrieval, and response generation work with mocks.
- **Edge cases:** empty history, insufficient history, malformed input, and database errors.
- **Integration shape:** components are wired together and receive expected arguments.
- **Behavioral branches:** pain-point approaches and repeated-theme behavior change in controlled scenarios.
- **Safety-adjacent behavior:** toxicity, crisis, error recovery, and fallback branches are represented.

Machine verification confirms software behavior. It does not certify the quality of medical or psychological advice.

## Memory-like mechanisms

The repository has several different memory mechanisms:

- database conversation history;
- embedding cache;
- associative memory;
- theme clusters;
- emotional trajectories;
- session-level in-memory monitoring.

These should not be presented as one uniform store: each has a different lifetime and implementation mechanism.

## Retrieval

The system uses PostgreSQL/Supabase and pgvector-style similarity search. Retrieval is chronological and semantic:

- **chronological:** what happened recently;
- **semantic:** what resembles the current question;
- **knowledge:** what appears relevant in documents.

## Safe rewrite language

Prefer statements such as:

- “The request pipeline adds retrieval context before generation.”
- “A classifier or heuristic determines a pain-point label.”
- “A database RPC performs vector similarity search.”

Avoid unsupported claims such as:

- “The system understands the user.”
- “The system diagnoses anxiety.”
- “The system provides safe therapy.”
- “Test coverage proves clinical correctness.”

## Suggested next outputs

1. A component dependency graph.
2. A database/Supabase schema and RPC inventory.
3. A prompt and template inventory.
4. A privacy and data-retention matrix.
5. A clinically reviewed safety scenario catalog.
6. A test-evidence matrix separating unit, mock integration, and real-system tests.
7. A public API reference for `/chat`, monitoring, and health endpoints.
