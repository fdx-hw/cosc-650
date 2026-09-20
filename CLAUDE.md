# CLAUDE.md

## Project
Course repository for COSC 650: Applied LLM Systems (Maryville University).
8-week graduate course covering tokenization, transformer architecture,
prompt engineering, function calling, retrieval-augmented generation,
fine-tuning, and evaluation.

## Structure
- week-01/ through week-08/ : weekly assignments and notebooks
- notes/ : research notes and reading annotations
- project/ : final project code and documentation
- CLAUDE.md : this file
- README.md : human-facing project description

## Conventions
- Notebooks are saved from Google Colab via File > Save a copy in GitHub
- All code is Python 3.11+
- tiktoken is used for tokenization experiments
- Commits use descriptive messages, not "update" or "fix"
- Analysis and explanations go in Markdown cells, never in `#` code comments
- Commit incrementally as work progresses, not as one final "submission" commit
- transformers and torch (CPU) are used for local model experiments (e.g. distilgpt2)
- Live calls use the Anthropic API (ANTHROPIC_API_KEY); cheapest model that fits the task
- In Colab, read secrets via google.colab.userdata, not hardcoded
- Notebooks needing extra packages (anthropic, sentence-transformers) install them in a %pip cell, not just a note
- Semantic similarity: sentence-transformers (all-MiniLM-L6-v2), paired with exact-match, never alone
- Prompt versions are separate files, not inline strings
- Tool JSON schemas use enums for closed value sets and required fields, same discipline as prompt files
- Guarded code-runners get three checks: an AST allowlist, a pre-execution size/magnitude check, and a time limit as a secondary layer -- a thread-based timeout alone can't preempt a single expensive built-in call (e.g. a large exponentiation), since CPython doesn't release the GIL mid-computation
- A live tool-use loop's mechanics (message shape, tool_result routing, retry handling, max-turns cap) get verified against a scripted fake client before running against the real API
- Tool descriptions state any semantic constraint the JSON schema can't express (units, currency, value ranges), since the schema only checks shape, not meaning

## Do Not
- Delete files or directories without confirming first
- Push to main without checking what is staged
- Commit API keys or any file in .env
- Treat a matching (or moved) aggregate score as proof an edit had (or didn't have) an effect without checking per-case results
- Treat a schema-valid, exception-free tool call as a correct one -- check what it actually computed, not just whether it validated
- Conclude a fix works from a single live re-run -- confirm with more than one, especially when temperature isn't pinned to 0
