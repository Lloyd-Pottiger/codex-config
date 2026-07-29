---
name: research
description: Investigate a question against high-trust primary sources and capture the findings as a Markdown file in the repo. Use when the user wants a topic researched, docs or API facts gathered, or reading legwork delegated to a background agent.
---

# Research

Delegate a bounded research question to a background agent so independent work can continue in parallel.

1. Define the exact question, scope, and output path before delegation. Follow the repository's existing research-note convention; if none exists, choose a concise repo-local path and report it.
2. Spawn one background agent with the question, relevant repository context, output path, and the evidence contract below. Give it the research task directly; do not ask it to invoke `$research` again.
3. Require the background agent to:
   - use high-trust primary sources such as official documentation, source code, specifications, and first-party APIs;
   - trace material claims to the source that owns them, distinguish evidence from inference, and expose conflicting or missing evidence;
   - write one self-contained Markdown file with findings and citations at the assigned path;
   - avoid modifying implementation, tracker state, or unrelated files.
4. Continue independent work while the agent researches. Before reporting the research complete, wait for it, inspect the artifact, and verify that the question is answered and material claims are source-backed.

For multiple independent questions, use one agent and one non-conflicting artifact per question, parallelized within the environment's available agent capacity.
