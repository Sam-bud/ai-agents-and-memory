# AI, Agents and Memory: how these systems are built and run

*My understanding, in my own words.*

Let it be Agentic AI, where multiple agents work in a system, or one AI model; the basic working remains the same. It is like a thread handling a single task. A task that has multiple tasks is handled by a thread for each (each agent is a separate model call with its own role and instructions, called a prompt), coordinated by an orchestrator, and at the end it produces the result.

## The foundation

What is the basic here then? The basic here, in any AI, is a configuration of weights (parameters) arranged in layers of neurons (a neural network) that defines the behaviour of the model. The heart or the brain is formed by training this configuration. The training data is converted into tokens and vectors (embeddings), and training keeps adjusting the weights until the behaviour is right.

When the model is to be trained, the foundation should be strong: its architecture and how the input is represented (tokenization and embeddings; in classical ML this is the column or features definition). If this is not strong, the foundation is not strong, and the training data would be the pillars that form the structure.

## RAG and ingestion

RAG (Retrieval-Augmented Generation) is the architecture that guides the construction. What is RAG? It can be called the context based on which the LLM (Large Language Model) or AI model needs to respond. RAG does not change the model's weights; it works at request time. The right pieces of business or product knowledge are retrieved for the question and fed to the LLM as context (grounding), so it answers from the business or product facts.

But RAG is only as good as what is fed into it, so ingestion (the pipeline that prepares knowledge for retrieval) is a stage on its own. It starts with the original documents (the source of truth: manuals, tickets, runbooks, policies, past resolutions). These are cleaned and parsed, then split into text chunks (small pieces, each carrying one idea, so the right piece is retrieved rather than a whole document). Each chunk is stored with metadata and, where vector search is used, an embedding (the vector form of the text). Every chunk keeps a reference back to its original document, so an answer can be traced to its source, and when a document changes only its chunks are re-ingested.

This is where the schema (the fixed structure every record follows) matters. A simple chunk schema:

| Field | Purpose |
|---|---|
| chunk_id | unique id of the chunk |
| doc_id | reference to the original document |
| text | the chunk content |
| embedding | vector form of the text (only if vector search is used) |
| source, version | where it came from, which version |
| tags | filters such as service or error type |
| approval_status | whether a human approved it |
| ingested_at, valid_until | when it was added, when it should expire or be reviewed |

The same discipline applies on the way out: agents should return structured output (for example JSON) that code validates against a schema, so one agent's output can be safely trusted by the next.

## Guardrails and agents

How do we use models or agents in the design? As we are depending on a machine for intelligence, guardrails become the most important. Without HITL (Human-in-the-Loop) it will become chaos, as the machine can run blindly believing everything is correct (this is hallucination: confident but wrong output).

Agents or models are designed in such a way that one can work to bring the RAG (retrieval agent) and another can validate the returned data (validation agent), as that makes the system more reliable. A low confidence score can be redirected to verification (confidence-based routing). Also, agents or models can be used to verify the result of the LLM or main model (a verifier or judge model), which strengthens verification and makes the system more reliable.

## Memory: naming each type for clarity

Memory is used in different ways in these systems, so each type should be named to be clear about its usage:

- **Parametric memory:** knowledge stored in the model's weights during training. Strong for general language and reasoning, but fixed at training time, so it can be outdated and is hard to edit.
- **Working (short-term) memory:** what the model holds right now, which is the context window: prompt, conversation, retrieved chunks, tool results, intermediate steps. It is limited by tokens and gone when the task ends.
- **Episodic memory:** records of specific past events: what happened, when, what was done, and the outcome (past cases and how they were resolved).
- **Semantic long-term memory:** general facts, concepts, and rules distilled from experience, independent of any single event (how services depend on each other, typical fix patterns, domain knowledge).
- **RAG (external memory):** the mechanism that keeps episodic and semantic knowledge outside the model and brings the relevant part into working memory when needed. This is how a model gets fresh, business-specific knowledge without retraining.

In simple terms: parametric memory is the brain, working memory is the desk, and external memory is the library.

## The life cycle, and drift

This is the basic life cycle: ingest, store, retrieve, reason, verify, human approval, and learn back. Approved outcomes are written back as episodic memory, and repeated patterns can be distilled into semantic memory.

Systems change after go-live, so monitoring is part of the design. These terms come from ML monitoring and apply here as well:

- **Data drift:** the input changes (new services, new log formats, new vocabulary) compared with what the system was built on.
- **Concept drift:** the meaning of "correct" changes. The same input now needs a different answer, for example after an architecture change the same error has a different root cause and the old fix no longer works.
- **Prediction drift:** the pattern of the model's outputs shifts over time (more low-confidence results, a different classification mix). It is often the earliest visible signal, because the correct labels arrive late.

To handle drift: monitor all three, keep a golden test set (fixed cases with known good answers) and re-run it after every change, refresh and expire chunks through ingestion with human approval, and retrain or fine-tune only when parametric memory itself has become stale. RAG helps because knowledge updates through ingestion, not retraining. Stale chunks are also a form of drift, so version them and give them an expiry.

The level and complexity will differ, but the basics remain the same.
