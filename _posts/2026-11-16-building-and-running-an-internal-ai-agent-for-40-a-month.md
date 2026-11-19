---
title: Building and Running a Company Wide Internal AI Agent for $40 a Month
image: /assets/images/marita-kavelashvili-ugnrXk1129g-unsplash.jpg
---

I proposed, designed and started building an AI agent that lives in our company's Slack and GitHub, and I've led its direction since. Anyone can @mention it to ask about our code and docs. It opens pull requests, reviews code, files issues, runs scheduled jobs and remembers past conversations. An enterprise AI license was out of reach for a company our size, so I built it to be hosted ourselves. Its core runs on one small server for about $40 a month in hosting before model spend.

Since it launched in July, **12 of our 25 employees** have used it. One field manager gets **about 7 hours a week back** because the field team now builds its own deployment plans through the agent. It has opened 59 pull requests, and **48 have been merged, 32 of them without a person changing the code**. I also hot-swapped its main model in production, which **cut model costs by 97% while still passing 95% of our evals**.

The rest of this post walks through how the agent works, from the gateway every request passes through to the evals that made the model swap safe, then covers what went wrong and what I'd do differently.

### The Gateway

Every model call the agent makes, from Slack or GitHub, goes through [LiteLLM](https://www.litellm.ai/), an open source gateway. It tags each request with the user who sent it, enforces a monthly budget per user, and logs every prompt, reply, token count and cost. Since launch it has logged 5,575 model calls, 4,101 tool calls and 199M tokens across 293 threads in 21 channels. The memory service's own model calls go through it too, and failure counts are exported for alerting.

We use the request logs to improve the agent's responses, mostly by finding conversations that went wrong. Two patterns usually stand out: a **rage response** is when a frustrated user keeps asking versions of the same question because the agent isn't doing what they want. An **early drop-off** is when a user stops mid-conversation, like leaving the agent's question unanswered. When we find one, we read the whole conversation and the model's traces to see where it went wrong.

### Tools

The agent can search and read our code, commits, pull requests and issues, search Notion pages and databases, read Slack channels and threads, query our field operations and data APIs, build CSV files, manage scheduled jobs, and propose GitHub issues or coding runs for the user to confirm. Most tools are plain functions in the agent's code. The field operations tools come from a separate service over [MCP](https://modelcontextprotocol.io/specification/2026-07-28), a standard protocol for exposing tools to models, and the gateway passes that service its credentials without the model seeing them. I built that service over our existing Python APIs, and it now handles about a third of the agent's tool calls.

Together, the tools let the agent pull context from places no one person checks. Our field manager used to spend about an hour a day building the next day's deployment plan by hand from Slack reports and three Notion ticket databases. Now the field team asks the agent for it and gets a plan listing each farm, ticket and source link, **which gives the manager about 7 hours a week back**.

### The Agent Loop

Every question goes through up to four steps before a reply is posted. The steps are modelled as a graph using [LangGraph](https://www.langchain.com/langgraph).

#### Classify

Early versions of the agent had two problems. They went into technical detail nobody asked for, and they wrote so much that people stopped reading.

So the first step classifies the user's message by length (succinct, standard or detailed) and audience (plain or technical). Every message starts as succinct and plain. If the user shows some frustration or keeps asking about the same topic, the length moves up to standard or detailed to support a user who wants to dig deeper. If they ask for more technical detail, the audience switches to technical.

Picking from a handful of fixed options doesn't need a model that writes text, so this step uses [Jev](https://typesafe.ai/), a model from TypeSafe that returns a structured classification with a confidence score.

#### Draft

The question then goes to the main model (currently GLM 5.3 Flash), with the chosen length and audience from the classification step added to its instructions.

Most questions take more than one tool call. For example, if someone asks how a feature in our code works, the agent searches the code, reads the file it finds, then reads the file that one imports. So the model and tools run in a loop that can be simplified to:

```ts
for (let step = 0; step < maxSteps; step += 1) {
  const message = await callModel(conversation, tools, onEvent);
  if (!message.tool_calls) return message.content;

  for (const call of message.tool_calls) onEvent({ type: 'tool', name: call.name });
  const results = await Promise.all(message.tool_calls.map(runTool));
  conversation.push(message, ...results);
}
```

The `onEvent` callback is how users see progress. While the loop runs, it streams the model's reasoning and each tool call into a Slack message, **so users can watch the agent work instead of wondering whether it's stuck**. When the answer is ready, the agent deletes that message and posts the reply as a brand new one. Slack doesn't send a notification when a message is edited, so posting fresh is what tells the user the agent has finished.

The loop stops after 50 rounds and posts the last message from the model. A tool that fails returns its error to the model instead of ending the run, and a new message from the user restarts the run.

#### Review and Rewrite

Jev then checks the draft against the recent conversation, the tool results and the chosen length and audience. A draft fails if it misses the audience, repeats itself, contradicts the tool results, skips part of the question or ignores an explicit instruction.

A passing draft goes to the user. A failing one goes to a rewrite step, which sees everything the draft step saw plus the draft and the reviewer's violations. The rewrite goes back to the reviewer, up to three times. If it still fails after the third rewrite, we post it anyway and send an alert.

### Memory

Without memory, you would have to reteach the agent each time you spoke to it. Instead, the agent remembers past conversations using [Hindsight](https://github.com/vectorize-io/hindsight), an open source memory service that we self-host.

**Saving memory isn't the model's job**. People don't stop mid-conversation to decide what to remember. They take in the whole exchange and keep what matters, and the agent works the same way. Once a thread goes quiet, the whole thread is sent to Hindsight, which uses Claude Haiku to extract facts, entities and dates. Each item is turned into an embedding which the model can search later.

Reading memory is the model's choice. Fetching memory for every request would fill the context with irrelevant information and skew the response. It has a search tool and is told to use it when a question needs history, and to prefer first-party sources like Notion and GitHub when memory comes up empty. 

### Scheduled Jobs

Anyone can schedule a job by asking in plain words. The model turns the request into a cron expression in the user's Slack timezone, and the job is saved once they confirm it. When it fires, the saved prompt runs through the same agent loop as described above. Users can list, edit and delete jobs by asking the agent.

Some examples include:

- Each day at 6pm, send me a DM listing the farms visited in this channel over the last 24 hours.
- Each day at 4am, generate a list of open field support tickets and group them by region and severity. Include a link to the ticket in each line.
- Every Monday at 12pm, fetch a list of customer facing features and bug fixes developers have shipped over the last week.
- Every day at 12am, send me a list of fertilizer applications completed today. Order by fields with the least amount of fertilizer applied and include how much fertilizer was applied today.

### Coding Sandboxes

The agent can write code from a Slack thread or a GitHub issue. Say someone on the field team asks in a thread to "add a button on the fleet list that opens Google Maps to the exact latitude and longitude of the base station." The agent replies with the full task it plans to hand off to an agent in the coding sandbox. When the run finishes, the agent posts a new reply in the thread with a link to a pull request that already builds, passes lint, and has a description written for a reviewer who never saw the thread.

Each repository has a small pool of [Fly Sprites](https://fly.io/sprites/), persistent sandboxes that keep installed dependencies between runs. Inside, DeepSeek V4 Pro runs through an [OpenCode](https://opencode.ai/) harness, builds the project and runs the linter before it opens the pull request. Jobs survive restarts without opening duplicate pull requests.

On GitHub, anyone with write access can mention the agent on an issue. We use [Jev](https://typesafe.ai/) to decide whether a mention is a question or a request to implement the issue. Anything uncertain is treated as a question, so it gets a comment instead of queuing a job.

Pull requests are also reviewed by a separate reviewer agent running DeepSeek V4 Pro in its own Sprite. Since August it has reviewed 55 pull requests and **flagged 110 issues before merge**, 8 of them high-severity. 20 of those issues have a confirmed fix.

Since launching in July, the agent has opened 59 pull requests across three of our repositories, and 48 of them have been merged. 32 of those merged without a person changing the code.

It also changes who can work on the product. **Anyone who can describe a change in Slack can turn it into a reviewable pull request** or a feature request linked to an issue. A small request no longer waits for a developer to pick it up and hand it to their own coding agent, so developers spend that time reviewing the change instead of making it.

### Hosting

The agent runs on one t3.medium EC2 instance (2 vCPUs, 4GiB memory), defined in [Pulumi](https://www.pulumi.com/) and started with Docker Compose. Its databases, embedding model and scheduled jobs share one 30GB disk.

### Evals

Evals are regression tests for an agent, whose behaviour can change whenever a system prompt, a model or a tool description changes. They're built like an integration test, with the same fixtures, mocks and assertions, except the calls to the model API are real. Every other API call is mocked. We run them using a framework called [Promptfoo](https://www.promptfoo.dev/).

There are six suites, covering tool selection, response style, the full draft and review path, clarifying questions, proposing work, and the voice of pull request reviews. When we fix or add model behaviour that's not covered by an eval, we add a new one.

Our pull request workflow runs entirely on the developer's machine, evals included. It runs every eval related to the change, sometimes several times to reduce flakiness, then posts a table of results to the PR. Running evals locally uses compute our machines already have instead of paying for it in the cloud.

Evals let us switch from Sonnet 5 to GLM 5.3 Flash with confidence. GLM 5.3 Flash **passed 95% of the existing evals at [97% cost savings](https://x.com/sheppsryan/status/2102389460801368440)**.

### Failures and Limitations

#### Memory Stopped Saving, Twice

Memory is built to fail open. If Hindsight can't save a conversation, the agent still answers. The downside is that a memory failure is silent and the experience quietly degrades. At one point, 324 saves failed over 10 days before I noticed, because nothing alerted on them. In another week, 128 more failed when the OpenRouter credit for saving memory ran out. Answers kept going out, but the agent never remembered those conversations.

#### Wasted Reviewer and Rewrite Loops

The review loop doesn't always converge. Three replies have used all three rewrites and still failed review. One was 72 words against a 60-word limit, and the rewrite model couldn't get it under in three tries. Each of those loops wastes time and tokens.

#### Corrections Cancel the Run

A new message cancels the run in progress, which is meant for corrections. But someone who adds a detail before the answer arrives stops the entire agent run and the context does not carry over from what the agent was doing before the user messaged the agent again. It happened twice in one thread to a colleague, who found it frustrating and stopped using the agent soon after.

### What Went Well

Before building anything, I spent time with the people who would use the agent to learn what their problems were and where AI could help with them. That work is why the agent had support early on.

Observability was the other thing that paid off. Request logs, model traces and spend tracking are how I diagnose errors and find conversations where the agent misbehaved. It isn't complete, as the memory failures show, but adding it later while trying to fix a misbehaving agent would have been much harder.

### What I'd Do Differently

I'd add alerts to every path that fails open from the first day. Failing open kept the agent answering when memory broke, but it also meant 324 saves failed over 10 days before I noticed. I'd also start from a dedicated agent observability tool instead of piecing my own together. Several good ones now exist, including open-source options, that I didn't know about when I started.

### What's Next

First, I want the coding and reviewer agents to help non-technical users make better technical decisions. Their changes should meet the same bar the engineering team holds itself to. Anyone can turn a request into a pull request, but someone outside engineering can't always tell whether the change is a good idea. They also can't always see when a broader change to the system would stop the problem from happening at all. The coder and reviewer should catch those cases and either guide the user or hand the decision to someone who can make it.

Second, I want to support long-running queries over large amounts of data. Today they're hard, partly because of the agent's tooling and partly because our data layer wasn't built for them.

**If you're building something similar, let's chat :). Shoot me a DM on [LinkedIn](https://www.linkedin.com/in/ryansheppardd/)**.
