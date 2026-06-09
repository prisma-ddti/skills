---
name: documentation-lookup
description: Fetch current library, framework, and API documentation via the Context7 MCP instead of relying on training data. Use this skill whenever the user asks a question that depends on accurate, up-to-date behavior of any library, framework, SDK, or API — including setup/configuration questions, code examples, API references, migration guides, or version-specific behavior. Trigger even if the user doesn't name the skill explicitly but clearly needs current docs to get a correct answer.
---

# Documentation Lookup (Context7)

Answer library/framework/API questions using live documentation from the Context7 MCP, not training data. Two tools are available: **resolve-library-id** (find the Context7 library ID) and **query-docs** (fetch docs for that ID).

## Workflow

### 1. Resolve the library ID

Call **resolve-library-id** with:

- **libraryName**: the library or product name from the user's question.
- **query**: the user's full question (improves relevance ranking).

This returns candidate library IDs. Pick the best match by considering exact name match, benchmark score (higher = better docs quality), source reputation, and version alignment if the user specified one.

A valid library ID is required before calling query-docs. If resolution returns no results, tell the user the library wasn't found in Context7 and offer to help via web search or training knowledge instead.

### 2. Fetch the documentation

Call **query-docs** with:

- **libraryId**: the ID selected in step 1.
- **query**: the user's specific question. Be precise — vague queries return vague results.

If the returned content doesn't answer the question, try one more query-docs call with a rephrased question before falling back. State uncertainty clearly rather than guessing.

### 3. Answer using the fetched docs

Respond using the current documentation. Include relevant code examples from the docs when helpful, and cite the library version when it matters.

## Constraints

- **Max 3 MCP calls per question** (combined resolve + query). Each call adds latency and token cost; keep it efficient. If 3 calls aren't enough, use the best information gathered so far and flag gaps honestly.
- **No secrets in queries**: strip API keys, passwords, tokens, and other credentials from any text sent to resolve-library-id or query-docs.

## Failure handling

- **No resolution results**: the library may not be indexed in Context7. Say so and offer alternative approaches (web search, training knowledge).
- **Irrelevant docs returned**: rephrase the query once. If still off-target, answer with what you have and note the limitation.
- **User didn't specify a library**: if the question implies a library dependency but doesn't name one, ask which library they're using before calling resolve.

## Example

User asks how to configure middleware in a web framework.

1. Call **resolve-library-id** with the framework name and the user's question.
2. From results, pick the official library ID (best name match + highest benchmark score). If the user mentioned a version, prefer a version-specific ID if available.
3. Call **query-docs** with that ID and the question.
4. Answer using the returned docs, including a minimal code example if the docs provide one.
