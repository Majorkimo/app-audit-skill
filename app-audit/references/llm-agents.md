# LLM and agent features

Load when the app calls a model, runs an agent, exposes or consumes tools or MCP servers, or retrieves documents into a prompt.

The model is not a trust boundary. Treat everything the model reads as potentially attacker-written and everything it outputs as untrusted input to the next component. Security comes from what the surrounding code allows, never from the prompt asking the model to behave.

## Untrusted input reaching the prompt

Trace every source:

- Direct: the user's own text and files.
- Indirect: retrieved documents, web pages, emails, tickets, uploaded files, database fields other users can write, and results returned by tools.
- Tool metadata: tool names, descriptions, and schemas are read by the model too. A poisoned or changed description can steer the model toward another tool or leak data through arguments, without the poisoned tool ever being called.
- Memory: anything the system stores from one conversation and injects into later ones, including other users' conversations if memory is shared.

For each source, ask who can write to it. If someone other than the current user can, indirect prompt injection is in scope.

## What the model can do

Build a per-tool authority map:

`tool | handler file:line | intended operation | credential it runs under | effective permissions of that credential | where caller, tenant, and resource are checked`

- A tool named "read" or annotated read-only is a claim until the handler proves it.
- One service key shared across tools means one injected instruction owns everything that key can do. Note every shared credential and its blast radius.
- Authorization inside the tool handler must use the end user's identity and tenant, not the agent's service identity. A tool that runs as a privileged service and trusts an id from the model's arguments is broken object-level authorization with an extra step.
- Generic tools (run SQL, fetch URL, execute code, call API) are the widest authority in the system. Check what actually constrains them.
- Read and write separation: can content read by one tool trigger a write, a send, or a deletion by another without a human in between.

## Human approval

- Whether high-impact actions require approval, and whether the approval screen shows the real target and arguments, not a model-written summary of them.
- Approvals that persist: "always allow" granted once and reused after the tool or its description changed.
- MCP servers and third-party tools: pinned to a reviewed version, or free to change their definitions after approval.

## Output handling

- Model output rendered as HTML or markdown without sanitizing. Markdown images and links are a data-exfiltration channel: the model is told to embed secrets in a URL and the client fetches it.
- Model output passed to a shell, an interpreter, a SQL engine, a template, or a file path.
- Model output trusted as a decision: an approval, a role, a price, or a moderation verdict that the code acts on without a second check.
- Structured output parsed without validation.

## Retrieval

- Retrieval that respects tenant and user entitlements at query time. An index shared across tenants and filtered by the prompt is not filtered.
- Documents the user could not open directly but can extract through the model.
- Who can add to the index, and whether ingested content is treated as instructions.
- Deletion: removed documents still present in the index or embeddings.

## Secrets and data

- System prompts holding secrets, keys, or logic that must stay private. Assume the system prompt can be extracted.
- Provider API keys in the client or mobile bundle.
- Personal or regulated data sent to the model provider: whether it is known, intended, minimized, and covered by the provider terms the project relies on.
- Prompts and completions written to logs, analytics, or traces with personal data in them.

## Abuse and cost

- Rate, token, and spend limits per user and tenant. An unauthenticated or unlimited endpoint in front of a paid model is a billing attack.
- Agent loops with no step, time, or cost ceiling.
- Use of the app's model access for unrelated work through a thin wrapper.

## Safety of the output itself

- Where the product's users include minors or vulnerable groups, what filtering and escalation exist, and whether they run server-side.
- Advice in regulated domains (health, legal, financial) presented without the guardrails the product claims.

## Verification planning

Injection tests are planned, not run, unless authorized against a non-production environment: the same task with benign content and with planted instructions in each indirect source, observing which tools were called with which arguments and what left the system. A model ignoring one payload once is not proof of a control.
