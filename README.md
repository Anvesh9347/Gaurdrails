# LLM Guardrails

1. Deterministic (keyword) filtering: fast, free, predictable rules.
2. Model-based filtering: an LLM classifies each input as `safe` or `unsafe`.
3. PII protection middleware: automatically redacts, masks or blocks sensitive data before it reaches the model.

## Why guardrails?

LLM apps receive untrusted user input. Guardrails sit between the user and the model to:

- refuse harmful requests (hacking, weapons, malware),
- keep sensitive data (emails, card numbers, API keys) out of prompts and logs,
- make the system's behavior more predictable and auditable.

No single technique is enough. This notebook compares them so you can see the trade-offs and combine them.

## What the notebook does

### 1. Setup
Loads `OPENAI_API_KEY` from a local `.env` file using `python-dotenv`. The key is never written in the notebook.

### 2. Deterministic approach
A function checks whether the text contains any banned keyword (`hack`, `malware`, `virus`, `exploit`, `attack`, `bomb`).

| Input | Result |
|---|---|
| How to hack a computer system | blocked |
| How to make a cake | allowed |
| How to make a bomb | blocked |

- **Pros:** instant, no cost, fully predictable.
- **Cons:** easy to evade (misspellings, rephrasing) and prone to false positives (e.g. "heart attack" contains "attack").

### 3. Model-based approach
Sends the input to GPT-4 (`temperature=0`) with a prompt asking it to answer with exactly one word: `safe` or `unsafe`.

Same three test inputs, same outcomes: the hack and bomb requests are `unsafe`, the cake request is `safe`.

- **Pros:** understands meaning and context, harder to bypass with rewording.
- **Cons:** adds latency and cost per request, and its output can still occasionally be wrong or inconsistent.

### 4. PII protection with LangChain middleware
Builds an agent with `create_agent` and three `PIIMiddleware` layers applied to user input:

| PII type | Strategy | Effect |
|---|---|---|
| Email | `redact` | replaced with `[REDACTED_EMAIL]` |
| Credit card | `mask` | shown as `****-****-****-1111` |
| API key (custom regex `sk-[a-zA-Z0-9]{48,}`) | `block` | request is rejected |

The agent has a dummy `lookup customer` tool so you can see the tool call.

### 5. Test and inspection
A prompt containing `john.doe@example.com` and card `4111-1111-1111-1111` is sent to the agent. Printing the full message history shows that the **model only ever saw** the redacted email and masked card, and the tool was called with the sanitized values. This proves the protection happens before the data reaches the LLM.

## Key takeaways

- Use deterministic rules for cheap, obvious blocks.
- Use a model-based check for nuanced or ambiguous content.
- Use PII middleware so sensitive data never reaches the model or the tools.
- Layer them: rules first, then model check, then PII handling.



