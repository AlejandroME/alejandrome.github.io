+++
date = '2026-07-22T15:16:39-04:00'
draft = true
title = 'Road to Forward Deployed Engineer: Part 1'
+++

# Intro

This is the first post in the Road to Forward Deployed Engineer series, where I implemented a simple agent that extracts unstructured support tickets to a typed model. Hitting some non-deterministic behaviour and thinking how to mitigate it was so much fun!

Before I move on, this is the description of the enterprise I am _embedded into_:

> Meridian — a B2B SaaS data-integration platform (connectors, syncs, pipelines). I picked dev-infra deliberately: the support tickets are technical (sync failures, connector auth, API rate limits, plan entitlements), so you can actually judge whether the agent’s answers are correct — which you’ll need when you grade your own RAG and evals.
>
> Its “systems,” which your agent will read from and act on:
>
> Tickets (Zendesk-like) — subject, body, customer, status, priority, history
> CRM (Salesforce-like) — account, plan tier, entitlements, renewal date, health
> Knowledge base — help docs, runbooks, policies (your RAG corpus)
> Billing/usage — plan limits, current usage

The goal of the first part is to turn messy, unstructured support tickets into _typed_, _validated_ data that downstream systems can consume. The raw data follows this structure:

```json
[
  {
    "id": "MER-1001",
    "channel": "email",
    "received_at": "2026-07-18T09:12:00Z",
    "raw_text": "Subject: Salesforce connector stopped syncing\n\nHi team,\n\nSince yesterday afternoon our Salesforce -> Snowflake sync has been failing. The run log just says \"authentication failed\". We did rotate our Salesforce admin password on Tuesday, could that be it? This feeds our revenue dashboard so it's fairly urgent for us.\n\nAccount: northwind-analytics (workspace nwa-prod)\nContact: dana@northwind-analytics.com\n\nThanks,\nDana"
  },
  {
    "id": "MER-1002",
    "channel": "chat",
    "received_at": "2026-07-18T14:47:00Z",
    "raw_text": "[14:47] customer: nothing is syncing. NOTHING. this is completely unacceptable, we are paying you people a lot of money\n[14:47] customer: fix it\n[14:49] customer: hello???"
  },
  {...},
]
```

This data is consumed via an interface that resembles an API on Python:

```python
"""Read-only accessor for Meridian's support tickets.

Treat this as a given customer system API. Don't edit it (or the data) to make
your agent pass — handle whatever the tickets throw at you in your own code.
"""

class Ticket(TypedDict):
    id: str
    channel: str  # "email" | "chat" | "web_form" | "api_alert"
    received_at: str  # ISO 8601
    raw_text: str  # the messy, unstructured ticket content


def load_tickets() -> list[Ticket]:
    """Return every raw support ticket, in the order received."""
    return json.loads(_DATA.read_text(encoding="utf-8"))
```

**Goal:** Build **`agent/extract.py`** that turns each raw ticket served by `load_tickets` into a validated `TicketExtraction`, implemented in the following interfaces:

- `extract_ticket(raw_ticket: str) -> TicketExtraction`
- A `main()` / CLI that runs over every ticket, prints a readable table, and writes `agent/out/day1_extractions.json`

`TicketExtraction` has the following contents:

| Field | Type | Meaning |
|---|---|---|
| `summary` | `str` | one neutral sentence, no secrets |
| `category` | `Literal[...]` | `bug \| how_to \| billing \| feature_request \| outage \| account_access \| other` |
| `product_area` | `Literal[...]` | `connectors \| sync \| api \| auth \| billing \| dashboard \| unknown` |
| `severity` | `Literal[...]` | `low \| medium \| high \| critical` — **inferred**, not stated |
| `sentiment` | `Literal[...]` | `calm \| neutral \| frustrated \| angry` |
| `contains_secret_or_pii` | `bool` | did the customer paste a token / key / card / etc.? |
| `affected_entities` | `list[str]` | connectors, systems, job ids, accounts named |
| `is_actionable` | `bool` | could you act on this as-is? |
| `missing_info` | `list[str]` | if not actionable, what you'd need to ask for |

**Passing criteria**:

1. Extract this unstructured data into `TicketExtraction` _accurately_
2. Gracefully handle the model's `stop_reason` output, retrying where applicable or emitting a flagged record so a human can decide whether to manually retry it or process it manually
3. Enforce business rules beyond the schema
4. If there are secrets in the ticket, those must be redacted during the extraction and must not appear anywhere else
5. It must use the model's structure output capability, not _regexing_ over the model's output

I am using Claude Opus 4.8 with the [Python SDK](https://platform.claude.com/docs/en/cli-sdks-libraries/sdks/python).

# Prompting

With the interfaces defined, and the restrictions, I took a stab at a one-shot prompt:


> Your task is to validate the support ticket's text and parse it according to the TicketExtraction pydantic model provided. > Your evaluation must follow these business rules:
>
>    - It must summarize the ticket and set it in the `summary` field. If there's any PII, secrets, API keys, card numbers or any sensitive information, it must NOT be included in the summary
>    - It must assign a ticket category based on the `category` values list
>    - It must assign a product area based on the `product_area` values list
>    - It must assign a severity area based on the `severity` values list
>    - It must assign a sentiment area based on the `sentiment` values list
>    - It must check the message and determine if there is any PII or sensitive information, like API keys, passwords, credentials, card numbers or anything that is sensitive and can be exploited. Please note that the user can mention things like "we changed our password", but that doesn't mean they shared sensitive information. 
>    - It must extract the names of the affected entitites and set it in the `affected_entities` list. Affected entities can be any connectors, systems, job ids, accounts, organizations, environments
>    - It must determine if the ticket is actionable for a support agent, based on the information shared by the customer and set it in the `is_actionable` field. If it's not actionable, it must set a list of actions or data needed from the customer to make it actionable. Set these in the `missing_info` list

It's the first prompt that came to mind, and the results I got were mixed: while the model was capable of extracting most of the data accurately, the classification on _constrained_ fields wasn't accurate. This is one example:

**Input:**

```json
{
    "id": "MER-1001",
    "channel": "email",
    "received_at": "2026-07-18T09:12:00Z",
    "raw_text": "Subject: Salesforce connector stopped syncing\n\nHi team,\n\nSince yesterday afternoon our Salesforce -> Snowflake sync has been failing. The run log just says \"authentication failed\". We did rotate our Salesforce admin password on Tuesday, could that be it? This feeds our revenue dashboard so it's fairly urgent for us.\n\nAccount: northwind-analytics (workspace nwa-prod)\nContact: dana@northwind-analytics.com\n\nThanks,\nDana"
  },
```

**Output:**

```
summary="Salesforce to Snowflake sync failing since yesterday afternoon with 'authentication failed' error in run log. Customer rotated their Salesforce admin password on Tuesday, which may be the cause. This feeds their revenue dashboard and is urgent." 
category='bug' 
product_area='connectors' 
severity='high' 
sentiment='neutral' 
contains_secret_or_pii=True affected_entities=['Salesforce connector', 'Snowflake', 'northwind-analytics', 'nwa-prod'] is_actionable='true' 
missing_info=['Confirmation of whether Salesforce credentials in the connector were updated after the password rotation', 'Specific failing sync/job ID', 'Timestamp of the last successful sync']
```

`is_actionable` and `missing_info` goes hand in hand, and the rule I wanted to enforce here is: if the ticket doesn't contain enough information to investigate and trobleshoot it, it should be marked as `is_actionable=false` AND list what's missing. 
 

Naive prompt:

```python
def extract_ticket(raw_ticket: str) -> TicketExtraction:

    client = Anthropic()

    resp = client.messages.parse(
        model="claude-opus-4-8",
        max_tokens=1024,
        messages=[{"role": "user", "content": f"""
            Your task is to validate the support ticket's text and parse it according to the TicketExtraction pydantic model provided. Your evaluation must follow these business rules:

                - It must summarize the ticket and set it in the `summary` field. If there's any PII, secrets, API keys, card numbers or any sensitive information, it must NOT be included in the summary
                - It must assign a ticket category based on the `category` values list
                - It must assign a product area based on the `product_area` values list
                - It must assign a severity area based on the `severity` values list
                - It must assign a sentiment area based on the `sentiment` values list
                - It must check the message and determine if there is any PII or sensitive information, like API keys, passwords, credentials, card numbers or anything that is sensitive and can be exploited. Please note that the user can mention things like "we changed our password", but that doesn't mean they shared sensitive information. 
                - It must extract the names of the affected entitites and set it in the `affected_entities` list. Affected entities can be any connectors, systems, job ids, accounts, organizations, environments
                - It must determine if the ticket is actionable for a support agent, based on the information shared by the customer and set it in the `is_actionable` field. If it's not actionable, it must set a list of actions or data needed from the customer to make it actionable. Set these in the `missing_info` list
            
            The ticket content is: {raw_ticket}
            """}],
        output_format=TicketExtraction,
    )

    print(f"LLM output is: {resp.parsed_output}")
    print(f"Stop reason is: {resp.parsed_output}")
```

Output:

```sh
$ uv run python -m main

LLM output is: summary="Salesforce to Snowflake sync failing since yesterday afternoon with 'authentication failed' error in run log. Customer rotated their Salesforce admin password on Tuesday, which may be the cause. This feeds their revenue dashboard and is urgent." category='bug' product_area='connectors' severity='high' sentiment='neutral' contains_secret_or_pii=True affected_entities=['Salesforce connector', 'Snowflake', 'northwind-analytics', 'nwa-prod'] is_actionable='true' missing_info=['Confirmation of whether Salesforce credentials in the connector were updated after the password rotation', 'Specific failing sync/job ID', 'Timestamp of the last successful sync'] confidence=0.86 needs_human=False
``
