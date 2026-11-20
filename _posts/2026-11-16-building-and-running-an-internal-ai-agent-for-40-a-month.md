---
title: Building and Running a Company Wide Internal AI Agent for $40 a Month
image: /assets/images/marita-kavelashvili-ugnrXk1129g-unsplash.jpg
---

I proposed, designed and started building an AI agent that lives in our company's Slack and GitHub, and I've led its direction since. Anyone can @mention it to ask about our code and docs. It opens pull requests, reviews code, files issues, runs scheduled jobs and remembers past conversations. An enterprise AI license was out of reach for a company our size, so I built it to be hosted ourselves. Its core runs on one small server for about $40 a month in hosting before model spend.

Since it launched in July, **12 of our 25 employees** have used it. One field manager gets **about 7 hours a week back** because the field team now builds its own deployment plans through the agent. It has opened 59 pull requests, and **48 have been merged, 32 of them without a person changing the code**. I also hot-swapped its main model in production, which **cut model costs by 97% while still passing 95% of our evals**.

The rest of this post covers how the agent answers a question, how it does work like writing code, and how I run it, then what went wrong and what I'd do differently.

### Answering a Question

Every question goes through up to four steps before a reply is posted. The steps are modelled as a graph using [LangGraph](https://www.langchain.com/langgraph).

#### Classify

Early versions of the agent went into technical detail nobody asked for and wrote so much that people stopped reading. So the first step classifies the user's message by length (**succinct, standard or detailed**) and audience (**plain or technical**) using [Jev](https://typesafe.ai). Every message starts as succinct and plain, then moves up if the user shows frustration, keeps asking about the same topic or asks for more technical detail.

#### Draft

The question then goes to the main model (currently GLM 5.3 Flash), with the chosen length and audience from the classification step added to its instructions. Most questions take more than one tool call, so the model and tool calls run in a loop. While it runs, we stream the model's reasoning and each tool call into a Slack message, **so users can watch the agent work instead of wondering whether it's stuck**.

#### Tools

The agent's tools cover GitHub, Notion, Slack, and our own APIs. Several of our APIs are exposed over [MCP](https://modelcontextprotocol.io/specification/2026-07-28). Together, the tools let the agent pull context from places no one person can check in a reasonable amount of time. Our field deployment manager used to spend about an hour a day building the next day's deployment plan by hand from Slack reports and three Notion ticket databases. Now the field team asks the agent for it, **which gives the manager about 7 hours a week back**.

#### Review and Rewrite

Jev then checks the draft against the recent conversation, the tool results and the chosen length and audience from the classification step. A draft fails if it misses the audience, repeats itself, contradicts the tool results, skips part of the question or ignores an explicit instruction. A failing draft goes to a rewrite step along with the reviewer's violations, then back to the reviewer, up to three times. If it still fails after the third rewrite, we post it anyway and alert on the failure.

#### Memory

The agent remembers past conversations using [Hindsight](https://github.com/vectorize-io/hindsight), an open source memory service that we self-host. We make it so that **saving memory isn't the model's job**. Once a thread goes quiet, the whole thread is sent to Hindsight, which extracts facts, entities and dates from the conversation. Contrary to writing, reading memory is the model's choice, because fetching it for every request would fill the context with irrelevant information. The model has a search tool and is told to use it when a question needs history.

### Doing Work

#### Scheduled Jobs

Anyone can schedule a job by asking in plain words, such as "every day at 12am, send me a list of fertilizer applications completed today. Order by fields with the least amount of fertilizer applied and include how much fertilizer was applied today." The model turns the request into a cron expression in the user's Slack timezone. When it fires, the saved prompt runs and the model performs like it would in any other conversation.

#### Coding Sandboxes

The agent can write code from a Slack thread or a GitHub issue. Say someone on the field team asks in a thread to "add a button on the fleet list that opens Google Maps to the exact latitude and longitude of the base station." The agent replies with the full task it plans to hand off, and a coding agent (DeepSeek V4 Pro through an [OpenCode](https://opencode.ai/) harness) executes the plan in a [Fly Sprite](https://fly.io/sprites/). When the run finishes, the agent posts a link to a pull request that already builds and passes lint.

A separate reviewer agent reviews each pull request in its own Fly Sprite, and has flagged 110 issues across 55 pull requests. Even so, a person approves every merge. Of the 48 merged so far, 32 went in exactly as the agent wrote them and 16 needed a fix, a test or a copy change from one of us first. This changes who can work on the product because **anyone who can describe a change in Slack can turn it into a reviewable pull request**, so a small request no longer waits for a developer to pick it up. That being said, not everyone knows what a good code change looks like (see the [What's Next](#whats-next) section below).

### Running It

#### Hosting

The agent runs on one t3.medium EC2 instance (2 vCPUs, 4GiB memory), defined in [Pulumi](https://www.pulumi.com/) and started with Docker Compose. Its databases, embedding model and scheduled jobs share one 30GB disk.

#### The Gateway

Every model call the agent makes, from Slack or GitHub, goes through [LiteLLM](https://www.litellm.ai/), an open source gateway. It tags each request with the user who sent it, enforces a monthly budget per user, and logs every prompt, reply, token count and cost.

We use the request logs to find conversations that went wrong. Two patterns usually stand out. A **rage response** is when a frustrated user keeps asking versions of the same question because the agent isn't doing what they want. An **early drop-off** is when a user stops mid-conversation, like leaving the agent's question unanswered. When we find one, we read the whole conversation and the model's traces to see where it went wrong.

#### Evals

Evals are regression tests for an agent, whose behaviour can change whenever a system prompt, a model or a tool description changes. They're built like an integration test, except the calls to the model API are real. We use [Promptfoo](https://www.promptfoo.dev/) to run our evals, which has been excellent so far. Our evals cover tool selection, response style, the draft and review steps, clarifying questions, proposing work, and a consistent "voice". Evals let us switch from Sonnet 5 to GLM 5.3 Flash with confidence, because GLM 5.3 Flash **passed 95% of the existing evals at [97% cost savings](https://x.com/sheppsryan/status/2102389460801368440)**.

### What Went Wrong

#### Wasted Rewrite Loops

The review loop doesn't always converge. Three replies have used all three rewrites and still failed review. One was 72 words against a 60-word limit, and the rewrite model couldn't get it under in three tries. Each of those loops wastes time and tokens.

#### Corrections Cancel the Run

A new message in a thread cancels the run in progress, which is meant for corrections. But someone who only adds a detail before the answer arrives cancels it too, and the work the agent had already done doesn't carry over. It happened twice in one thread to a colleague, who found it frustrating and stopped using the agent soon after.

### What I'd Do Differently

Request logs and model traces are how I diagnose errors and find conversations where the agent misbehaved. I pieced that together myself, and I'd start from a dedicated agent observability tool instead. Several great ones now exist, including open source options, that were not easy to reach for when I started.

### What's Next

I want the coding and reviewer agents to help non-technical users make better technical decisions. Anyone can turn a request into a pull request, but someone outside engineering can't always tell whether the change is a good idea, or whether a broader change to the system would stop the problem from happening at all. The coder and reviewer should catch those cases and either guide the user or hand the decision to someone who can make it.

**If you're building something similar, let's chat :). Shoot me a DM on [LinkedIn](https://www.linkedin.com/in/ryansheppardd/)**.
