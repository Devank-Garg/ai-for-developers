# Resources: Chatbots

Curated external resources for going deeper on building conversational LLM applications.

---

## Official guides

**[Anthropic — System prompt best practices](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/system-prompts)**
https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/system-prompts
Anthropic's guidance on structuring system prompts: tone, scope, formatting instructions, and persona design. The examples are Claude-specific but the principles apply across providers.

**[Microsoft — Prompt engineering techniques](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/prompt-engineering)**
https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/prompt-engineering
Microsoft Azure's prompt engineering guide, written from a production deployment perspective. Covers system messages, few-shot examples, and output formatting — with emphasis on reliability at scale.

---

## Reference code

**[Anthropic Cookbook — Customer support agent](https://github.com/anthropics/anthropic-cookbook/tree/main/patterns/agents)**
https://github.com/anthropics/anthropic-cookbook/tree/main/patterns/agents
End-to-end worked examples of multi-turn chatbot patterns from Anthropic's team. Shows session management, tool use integration, and the conversation array structure in practice.

---

## Engineering blogs

**[How Discord stores trillions of messages](https://discord.com/blog/how-discord-stores-trillions-of-messages)**
https://discord.com/blog/how-discord-stores-trillions-of-messages
Discord's engineering deep dive on message persistence at scale. Not LLM-specific, but directly relevant to the challenge of storing and retrieving conversation history efficiently — the same problem you face with chatbot session storage.

---

## Video

**[Building a production chatbot — Google for Developers](https://www.youtube.com/watch?v=hArMx7A89YE)**
https://www.youtube.com/watch?v=hArMx7A89YE
Google's walkthrough of production chatbot architecture: session handling, context management, and safety considerations. Covers the gap between a working demo and a deployable product.
