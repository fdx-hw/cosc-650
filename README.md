# COSC 650: Applied LLMs

A hands-on, project-based repository for building and reasoning about LLM systems, from tokenization through prompt engineering, tool use, retrieval, and fine-tuning.

## Course Context

This repository supports **COSC 650: Applied LLMs** at Maryville University, an 8-week course that moves from the input layer (how text becomes tokens) through the processing layer (how transformers turn tokens into predictions) to the practitioner's interface (prompt engineering as code), and then into giving models the ability to act, function calling, retrieval-augmented generation, fine-tuning, and the production-readiness questions that decide whether any of it survives contact with real users. Assignments emphasize treating prompts as versioned, testable artifacts and documenting experiments as GitHub Issues, so the repo builds toward a small applied-LLM portfolio rather than a folder of disconnected homework.

## Repository Structure

```
.
├── week-01/     # Tokenization Analysis
├── week-02/     # Inference and Sampling
├── week-03/     # Prompts as Engineering Artifacts
├── week-04/     # Multi-Tool Assistant
├── week-05/     # RAG Pipeline with Retrieval Evaluation
├── week-06/     # Advanced Retrieval, Transform and Measure
├── week-07/     # The Adaptation Decision and a Production Dataset
├── week-08/     # Integrated LLM System
└── notes/       # Reading notes, experiment logs, and reference material
```

Each `week-XX/` directory contains that week's exercises, code, and a short writeup.

## Technologies

- **Python** — primary language for all exercises and projects
- **Jupyter** — notebooks for exploratory work and walkthroughs
- **tiktoken** — tokenization
- **Anthropic SDK** — model access, function calling, and API-based experimentation
- Additional libraries introduced as needed (e.g., for retrieval, evaluation, and fine-tuning) in later weeks

