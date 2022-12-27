---
title: S22 Work Term Report
image: /assets/images/valueconnectteam.jpg
---

Some things have changed, and some things have remained the same compared to my [last coop work term](https://ryansheppard.tech/documentation/coop_work_reports/valueconnect).

# Information About the Employer

This is my second coop with Value Connect. To get a general overview of what Value Connect is and what they do, check out the second section in my **[S21/F21 Work Term Report](https://ryansheppard.tech/documentation/coop_work_reports/valueconnect)**, or you can check out our website at [**valueconnect.ca**](http://valueconnect.ca).

Compared to my last work term, the company has shifted it’s efforts towards bringing more lenders on to our platform. We’ve got multiple projects on the go, and much of my coop work term goals relate to the company’s efforts. We’re also growing at a notably faster rate than my last work term. The company is larger than it was before, and each position has a person who is remarkably stronger than the last.

# Job Description

At Value Connect a Software Developer is expected to work on something software related that progresses the company in one way or another, raise the development teams knowledge base by sharing and/or implementing their ideas, learnings and findings and be receptive to feedback from the co-workers and business.

The big difference in this work term compared to my last is my promotion from Junior Software Developer to Software Developer 🎉. My promotion meant an increase in responsibilities and a requirement to make important decisions related to the development team and company. It doesn’t feel like much had changed compared to my last position. I always tend to stick my head into important developer related decisions and maybe that’s why they kept me around for a third work-term. It’s definitely not an impeccable coding ability.

# Goals

## Update Spring Boot to Version 2 ✅

Over the course of my work term, upgrading our legacy web application through a new major version was my assigned wildly important goal and was single-handedly the biggest impact I could make on the business over the next four months. Spring Boot 2.0.0.RELEASE is the minimum version required to work with the [Spring Authorization Server](https://github.com/spring-projects/spring-authorization-server), which supports the standard OAuth 2.0 authentication protocol and allows us to implement Single Sign On (SSO) capabilities for our entire suite of tools.

Admittedly, the update to Spring Boot 2.0.0.RELEASE should have happened much sooner. It’s a very daunting task, and that’s perhaps why it’s been put off for so long. To put it in perspective, Spring Boot is an opinionated library with a whole bunch of opinionated books. This library contains books that they want; no older, no newer. The problem is that this rule was lost in translation between all the changes to the codebase. Some teams (even mine, early on) naively inserted their own opinionated books over Spring Boot’s opinionated books. Unfortunately, our opinionated books are not compatible with newer versions of their opinion, and we’re now missing out on all the cool new things that their newer opinionated books provide.

I originally thought that I completed this goal because of luck. When Chris (the CEO), Ben (the Lead Developer) and I went out for dinner to discuss the future of the web application, we originally estimated that the updates would take over 5000 man hours. This was the worst case, but it was the most reasonable. The most important thing that I learned from this work term goal is about planning for unplanned work. All too often business projects, internal IT projects, updates and changes are given the light of day when planning developer’s work. It feels good to give time to unplanned work and accomplish your goals on time.

## On-board new Developer ✅

At the start of my work term, our development team had shrunk to a meager three people, one of whom had put in his two weeks notice. It was important to get a new developer onto our team, and get them up to speeds so that we can continue the velocity that we’ve had previously. After a ton of searching, a software developer named Rafael gives a stellar interview and joins our team two weeks later.

Interestingly, I think that this goal taught me more than I taught Rafael. When he would ask a question, it was easy to realize that what he was asking about was over-complicated, or didn’t have enough documentation surrounding it. Sometimes he would ask questions that I didn’t know the answer to, and we would figure it out together. He took a lot more in depth approach to learning our web application, and he would teach me things that I had no clue about. Moreover during code reviews, Rafael was quick to implement some design patterns that I’ve never seen before.

Overall, Rafael is a great hire for our team. He’s been successfully on-boarded onto our team, and our velocity has improved since his hiring.

## Implement New Authorization Server ❌

As mentioned earlier, a new authorization server allows the company to implement Single Sign On (SSO) capabilities for our entire suite of tools. This means a single login for our web application, mobile application, and any future Value Connect service. This may not seem like a huge feat at first, but behind the scenes it means that we can tie any data to a single Value Connect account. If something goes wrong with a user’s purchase made using an Apple ID or Google Play Account, we can trace it to a single Value Connect user and better serve them that way.

I failed to achieve this goal because of how long the Spring Boot 2 update took. In hindsight, this was an unreasonable goal. I set it naively thinking how great it would be to complete all these things in one work term, and how much I could help the company progress in doing so. If there is anything I can learn from this, I want to keep in mind [SMART](https://www.atlassian.com/blog/productivity/how-to-write-smart-goals) the next time I create goals.

# Conclusions

I had more fun this work term than my last. The best thing about working for a small company is the impact that I’ve been able to make. Want to work with the latest and greatest technology? Want to create a development team social every week to catch up on non-work related topics? Are you interested in working more on development operations than developing the software? With a small company, you can be the change you want to see. Obtaining these things is as simple as talking to your supervisor, and compromising with the company’s needs. All three of those questions were something that I asked myself during my work term, and all three were answered with compromises that I was happy with.

# Acknowledgements

A huge thanks to the success and sales teams for putting up with my team for a third of this year, and anybody else who was hindered by the web application’s updates but also saw the value and importance in them.

Thanks to my team, Ben and Rafael, for continuing to spark my flame. I learned more this work term than I have previously, and that’s because of your willingness to share knowledge, provide constructive feedback and constantly push the limits of our team’s ability. You guys are awesome, it’s been a joy talking to you every day.
