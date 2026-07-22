+++
date = '2026-07-21T16:10:43-04:00'
draft = false
title = 'Road to Forward Deployed Engineer -- An Intro'
+++

This is the first post in a series I called _"Road to Forward Deployed Engineer"_. After my last job as a Software Technical Program Manager at Oxide, I've been weighing what's next. It is the first time in 16 years that I have a break, which has been energizing in so many aspects: I had time to go outdoors in several camping trips -- summer is too short in the northern hemisphere! --, playing with synthesizers again, reading a lot, practicing my communication skills, and catching up with the crazy pace of AI.

I've spent the last 8 years in management roles, and while I enjoy them, and had the chance to stay fairly technical, I want to go back to deep hands-on work, mixed with customer engagement. The Forward Deployed Engineer role has made waves in the past couple of years across tech circles, even though it's not a new concept, and some would even argue that it's _consultancy work_ in disguise -- which I don't think it's true after spending 10+ years in a consultancy/staff augmentation company. The FDE role's intersection between product and customer work energizes me, and I feel it's a natural next step for me.

My technical background is heavy on backend systems and data integration, via custom components (services) built on Java, Scala, or Python, and early in my career, using [tools no one even mention anymore](https://en.wikipedia.org/wiki/Enterprise_service_bus). While I use AI in my development workflow, I haven't built a production-like system that leverages AI at its core, so the question is: _what do I need to learn to close the gap?_ There are lots of courses and tutorials out there, and I could just simply follow them and learn the concepts, but I have figured that I am bad at learning by watching others do things, even if there is an "assignment" after the lesson, and AI Engineering roadmaps [like this](https://roadmap.sh/ai-engineer) cover a broader surface area that I don't see practical for me at this stage.

## LLMs as learning partners

I recently watched [an interview](https://www.youtube.com/watch?v=D_4AYLr_hFI) with [Simon Eskildsen](https://sirupsen.com/about) (Turbopuffer's co-founder) in where he describes how he uses LLMs to become _a learning machine_, which deeply resonated with the way I've been using them to navigate through complex codebases, understand concepts from first principles, and brainstorm ideas. This case is not different, so I fired up Claude and prompted the following:

> I will pivot to Forward Deployed Engineer, but I have to close my AI gap. I was chatting with you yesterday about following the AI Engineer roadmap here: https://roadmap.sh/ai-engineer, but I don't find it actionable for my context. I am a software developer, and have most of the foundations in place, but I want to close the gap in understanding how to implement LLM-backed systems in production.
>
>  I've created this project because I have the following idea: what if we build a study plan in a way that you lay out the topics, with a practical exercise that I implement in Python, then you grade it, give me feedback, and move onto the next topic? Ideally building on top of the previous implementation.
>
>  I am imagining something like this: an enterprise use case in where we need to work with information from several systems and implement workflows, and automations. Our mission is to shape the program in a way that I can learn these concepts in, at most, two weeks, devoting 4-6 hours daily to study. Let's brainstorm, and ask as many clarification questions as you want about me, my background, my goals, the idea, or anything you think it's relevant

That's a far from perfect prompt, but Claude came up with this answer:

> This is a good idea, and it fits how you learn — you build it, first principles, I grade. Let me react, drop a straw-man on the table, then ask a few things to lock it in. Where your real gap is
> 
> Forward-deployed = you parachute into a customer and, in days, stand up a working LLM system on their messy real data and systems, prove it's reliable, and explain it to non-engineers. Your distributed-systems / data / backend foundation already covers ~70% of that. The missing 30% is a specific, learnable layer:
> 
> structured LLM output → retrieval over enterprise data → tool/function calling against real systems → agentic workflow orchestration → evals + tracing to prove it works.
>
> That layer is exactly what we drill. Nothing on the roadmap.sh list about training models, GPUs, or fine-tuning matters for FDE — we skip it.
>
> The shape: one enterprise agent that grows every day
>
> Rather than disconnected exercises, we build a single "enterprise assistant" in a domain you pick and add one capability per day, each graded before moving on. By Day 10 you have a demo-able repo plus a talk track — which is itself an FDE interview asset.
>
> Daily loop:
> 1. I give you the concept (tight) + a spec: problem statement, the interface/contract, acceptance criteria.
> 2. You implement in Python, solo.
> 3. I grade on a rubric — correctness, LLM-system design choices, reliability/eval thinking, code quality, and "could you demo and explain this to a customer."
> 4. Feedback → the next topic builds on your code.
>
> Straw-man 10-day arc (I'll reshape from your answers):
>
> | **Day** | **Topic**                     | **You build**                                                              |
> |---------|-------------------------------|----------------------------------------------------------------------------|
> | 1       | LLM calls + structured output | Typed extraction from messy tickets/emails (Pydantic, validation, retries) |
> | 2       | Prompt reliability            | Routing/classification + a tiny eval                                       |
> | 3       | Embeddings + retrieval        | Index the enterprise KB in pgvector (reuse your setup)                     |
> | 4       | RAG end-to-end                | Q&A with citations + "I don't know" handling                               |
> | 5       | Evals for RAG                 | Eval set, LLM-as-judge, recall@k, regression harness                       |
> | 6       | Tool / function calling       | Agent hits mock enterprise APIs (CRM/tickets/orders)                       |
> | 7       | Agentic workflow              | Multi-step "resolve this case" across systems; workflow-vs-agent judgment  |
> | 8       | Observability + reliability   | Langfuse tracing, guardrails, cost/latency                                 |
> | 9       | Productionize                 | Wrap as an MCP server / FastAPI (your capstone idea), streaming            |
> | 10      | Capstone + demo               | Polish, recorded demo, "how I'd forward-deploy this" write-up              |

That's a pretty cool plan in my opinion. After I gave Claude the thumbs up, it came up with our fictional customer and their use case:

> The customer you're "deploying into"
>
> To make this concrete, here's the fictional account we build for — reskin it if you want:
> 
> Meridian — a B2B SaaS data-integration platform (connectors, syncs, pipelines). I picked dev-infra deliberately: the support tickets are technical (sync failures, connector auth, API rate limits, plan entitlements), so you can actually judge whether the agent's answers are correct — which you'll need when you grade your own RAG and evals.
>
> Its "systems," which your agent will read from and act on:
> - Tickets (Zendesk-like) — subject, body, customer, status, priority, history
> - CRM (Salesforce-like) — account, plan tier, entitlements, renewal date, health
> - Knowledge base — help docs, runbooks, policies (your RAG corpus)
> - Billing/usage — plan limits, current usage 

And it gets better:

> What I provide vs. what you build
>
> This mirrors real FDE work: the customer's data and systems are given — you build the LLM layer on top.
>
> - I provide (fixtures): synthetic seed data + thin mock system stubs (customer_env/ — functions like get_account(), search_kb(), update_ticket() backed by JSON/SQLite). Generating fake enterprise data isn't the skill we're training.
> - You build (agent/): everything LLM — prompts, schemas, retrieval, tools, orchestration, evals, tracing. This is 100% your code, from first principles. No finished code from me.
> How I'll grade (same rubric every day, so you can aim at it)
> 
> Each exercise scored 0–4 on five axes, plus a verdict:
>
> 1. Correctness — meets the spec & acceptance criteria
> 2. LLM-system design — right technique for the job (schema vs. RAG vs. tool vs. agent)
> 3. Reliability — validation, retries, "I don't know," guardrails, edge cases
> 4. Evaluation mindset — how do you know it works?
> 5. Code quality — typed, readable, extensible
>
>  FDE verdict: Would this survive a customer demo, and what breaks first?

Challenge accepted! During the next 10 days, I'll follow the plan and share my learnings here, one post at a time.

## Learnings

- **Day 1:** LLM calls + structured output
- **Day 2:** Prompt reliability
- **Day 3:** Embeddings + retrieval
- **Day 4:** RAG end-to-end
- **Day 5:** Evals for RAG
- **Day 6:** Tool / function calling
- **Day 7:** Agentic workflow
- **Day 8:** Observability + reliability
- **Day 9:** Productionize
- **Day 10:** Capstone + demo 
