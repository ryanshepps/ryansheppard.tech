---
title: Organization Wide Agents
image: /assets/images/luc-dobigeon-BdvZQL0iebM-unsplash.jpg
---

I recently joined an organization where AI usage is pretty low, and I’ve made it my mission to free up people’s time so they can spend more time being high performers and less time doing menial work. One colleague is already getting an hour of their day back. This article notes the lessons learned along the way.

### Stack

This is a small(ish) company so purchasing an enterprise license is not in the cards. I’ve opted for manually hosting and gluing everything together which has a higher maintenance cost but a lower financial investment — a path that is more and more viable in today’s AI world. Here is what my stack looked like:

- **LiteLLM**: LLM routing, governance and observability.
- **AWS EC2**: VM for hosting the agent on.

Cost is always going to be a factor when dealing with agents. I reached for something off the shelf that can give us fine-grained insights on users and query cost. LiteLLM is great because it is the only AI gateway I could find with everything I needed and was not locked behind an enterprise license.

### Tools

The agent isn’t super useful unless it has context to the information the business relies on. I needed a model that was great at tool calling and relatively cost-effective. For me, this is Sonnet 5, but I may experiment with something like DeepSeek V4 Pro when I get enough users and evals to determine the quality versus cost hit.

I thought about the apps employees use the most at the company. For me this was Notion for their internal documentation, Slack for communication and GitHub for code. I created tools for all these applications and instructed the agent on when to use them in the system prompt. Early testing with Sonnet has been great as I’m consistently surprised at how good it is at using the tools appropriately.

### Stigma

The first step to tackling stigma is **fishing for dissent**. This is the act of actively seeking out disapproval. This paid off in a few ways:

1. I found problems I hadn’t thought about before.
2. I learned what to focus on to garner people’s support.
3. People felt included in the decision and were invested in helping me rather than putting up barriers.

For me, I found people who I knew had been hesitant to fully adopt AI or had made remarks about using AI before. One surprising outcome from fishing for dissent is that people tend to talk candidly and openly about their concerns while simultaneously hearing me out on why I think this will be good for the company. The conversations I had were very fruitful and renewed my excitement for getting this deployed.

### Finding Champions

I didn’t want to release something like this into the wild for people to use all at once. Given AI’s nondeterminism, a brand new harness with kinks still to work out was likely to have underwhelming results. The last thing I wanted was for people to “remember” that time I first introduced my agent and it completely flopped. This is where I found those few key people who are eager to try new things and are okay with it not working 100% the first time. I thought of this as an internal product, and these were my early adopters.

For me, there was someone that I noticed who was subtly commenting on certain messages in Slack about how they used ChatGPT or Claude for one thing or the other. I knew this person was comfortable using AI and would be a prime candidate for championing this for me. To my delight, she surprised me with how well versed she was with LLMs and I think this will be a great relationship. The second person I sat beside and saw how much time he spent throughout the day on things that an agent could do in two minutes. He was easy to convince.

### Conclusion

This is an ongoing experiment, but the early results have already made it worth it.

Next up I want to spin up sandboxes where the agent can write and test its own code so I can turn bug reports and alerts into PRs. The software team is currently inundated with small requests that LLMs can knock out in five minutes.

I’ll keep writing down what breaks and what sticks as more people come on board.
