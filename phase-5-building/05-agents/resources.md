# Resources: Agents

Curated external resources for going deeper on LLM agents and tool-use systems.

---

## Papers

**[ReAct: Synergizing Reasoning and Acting in Language Models](https://arxiv.org/abs/2210.03629)**
https://arxiv.org/abs/2210.03629
The paper that formalized the observe → think → act loop used in modern agents. Short, readable, and directly maps to the execution loop component card. The pseudocode in this module is a practical implementation of the ReAct pattern described here.

---

## Reference code

**[AutoGPT](https://github.com/Significant-Gravitas/AutoGPT)**
https://github.com/Significant-Gravitas/AutoGPT
One of the first widely-used autonomous agent implementations. Worth reading not for the code itself, but to see the failure modes that occur in real agentic loops — infinite loops, hallucinated steps, and lack of human checkpoints — all documented in its issue tracker. A useful study in what to build against.

**[LangGraph](https://github.com/langchain-ai/langgraph)**
https://github.com/langchain-ai/langgraph
LangChain's framework for building stateful, multi-step agent workflows as graphs. The graph structure enforces explicit control flow — the opposite of an unconstrained agent loop. Useful reference for how production teams add structure and checkpoints to agent execution.

---

## Blog posts

**[Simon Willison — Prompt injection and what the industry needs to do about it](https://simonwillison.net/2023/Apr/14/worst-that-could-happen/)**
https://simonwillison.net/2023/Apr/14/worst-that-could-happen/
The most cited analysis of prompt injection risks in agentic systems. Covers direct injection (user-controlled), indirect injection (via retrieved content), and why current defenses are limited. Required reading before shipping any agent that processes external data.

---

## Video

**[Anthropic — Building effective agents (YouTube)](https://www.youtube.com/watch?v=TiudNBhGPPA)**
https://www.youtube.com/watch?v=TiudNBhGPPA
Anthropic engineers walk through the architecture of production agentic systems: when to use agents vs. simpler approaches, how to structure tool definitions for reliable selection, and real failure modes observed in deployed systems. Closely aligned with the component cards in this module.
