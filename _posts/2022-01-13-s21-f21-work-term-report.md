---
title: S21/F21 Work Term Report
published: false
---

“We should remember that good fortune often happens when opportunity meets with preparation.” - [Thomas Edison](https://www.thomasedison.org/edison-quotes). I’m very lucky to be in the position I’m in. But I think there is one consistent action I’ve done to aid my luck: seizing opportunities. Spotting an opportunity, bringing it to fruition and acting on it is something that I’ve gotten really good at. In fact, none of my goals this work term are related to my job tasks. Many of them are the result of seeing an opportunity and acting on it.

![Value Connect Logo](/assets/images/valueconnectlogo.png)

As part of my first coop work term, I’ve been working full-time with Value Connect.

Value Connect is a property appraisal marketplace that’s changing the way everyone feels about appraisals. Today’s property appraisals are mostly offline. They include frustrating back-and-forth and mundane data re-entry. We’re saving underwriters [85%](https://www.valueconnect.ca/) of the average time spent on each appraisal by making them ridiculously simple and efficient.

I chose Value Connect because I see opportunity. There is opportunity in getting to talk to my CEO every day, being an integral part of creating my team’s culture and making decisions for the foundation of the company. There is ample opportunity here, and I’m thrilled to be a part of it.

## Job Description

I work on a lot of projects. At the beginning of my work term, my responsibility is to work alongside our designer to develop a replacement front-end for our appraisal order system. I work hard to integrate myself with the processes and workflows that my teammates use.

In the following months, I work on the mobile inspection tool. An app that solves common issues and concerns of property appraisers. 

For the last two months, I work full-time on our web application. I couldn’t be happier with the refreshing shift in focus. 

While not my main focus, much of my job also includes enhancing development operations. My team and I have streamlined the process of pushing new features to production.

I’m proud to say that I’m a part of fostering a culture that does not need any tools, frameworks or languages to get started. Many of the skills required to start at Value Connect include an appetite for learning, an eye for opportunity and an optimistic attitude. Given that this is my first work term, it’s awesome being able to start with no industry knowledge, but be able to work on as many projects as I have.

# Summer 2021 Goals

## Create a Pull Request for an Issue Related to the Back-End ✅

**Technology**: Java, [Spring](https://spring.io/why-spring), [Spring Security](https://spring.io/projects/spring-security), Cross Site Request Forgery.

**Skills**: Initiative, Problem Solving.

While I work on the new front-end, the rest of the team tackles the back-end. The work that I am doing may make an impact months from the time that I write it. I see back-end bug fixes move into production every other day. I want to make an impact on the company as soon as possible, and the fastest way to do that is by working on an issue related to the back-end.

I see my first opportunity when working on the mobile inspection tool. There is an issue when making a request to register a mobile appraiser. I spend a couple hours figuring out the best way to solve the problem by studying how the system handles web requests. I learn a lot about professional Java code and the [Spring](https://spring.io/why-spring) framework, but it’s not enough to solve the issue. I employ the help of a co-worker who points me in the direction of a Spring project called [Spring Security](https://spring.io/projects/spring-security).

It’s here that I learn about Cross Site Request Forgery(XSRF) and how attackers can leverage data from a different browser window to make a web request on the user’s behalf. Spring Security enables XSRF protections by default for each web request. Fortunately, XSRF is only really possible on a web browser. I make a pull request that disables XSRF protections for our mobile requests. My first goal is complete.

## Add a Linter to the Front-End Build Process ✅

**Technology**: [ESLint](https://eslint.org/docs/user-guide/getting-started), BASH, Pipelines.

**Skills**: Creativity.

[Git](https://www.atlassian.com/git/tutorials/what-is-git) has a cool feature that shows who wrote each line of code and how long ago it was changed. In Value Connect’s code base, some lines have not been changed in five years. It’s awesome that developers can visibly see who implemented what infrastructure. How can I create infrastructure that other developers will see?

`console.log(';)'); You, 2 months ago - commit message here`

Thinking about where I can create some infrastructure, I come across code formatting. [ESLint](https://eslint.org/docs/user-guide/getting-started) is a tool that automatically formats code and warns of language anti-patterns. Enforcing the same format makes the code base easier to read. Imagine a textbook that has different writing style throughout. Additionally, the less time developers spend reading, the more time they can spend writing. Implementing a linter on my current project would be a great idea!

I complete this goal by creating a linting configuration file that tells ESLint how I want the code to be formatted. I carefully configure rules and include why rules are enforced. I want future Value Connect developers to make informed decisions on code enforcement. I write a BASH script that checks for linting errors before committing code to the code base. Lastly, I include a step in the pipeline that checks whether code is linted or not. The pull request takes 6 iterations before it’s finally committed to the code base and is there forever.

## Have a Cleaner Commit History ✅

**Technology**: [Git](https://www.atlassian.com/git/tutorials/what-is-git).

**Skills**: Communication, Writing.

20% of my time reading code is glancing over the lines that my predecessor’s wrote. The other 80% is looking at the commit history wondering what on earth they were thinking. Commit history *should* accompany code changes with why a change is being made

It’s clear to me that nobody always gets their commit history right the first time. In fact, getting commit history wrong is so common that [Git](https://www.atlassian.com/git/tutorials/what-is-git) provides a suite of tools to edit commit history. Over the next couple months, I spend extra time ensuring that I leave a good commit history. Here is a list of tools that I learned to complete this goal:

`reset`

- Undoes commits. Particularly useful when I miss code that needs to be in my last commit, or when I include code that should not have been in the last commit.

`alias`

- I use Git aliases to visually show code differences on the screen that I make commit messages. Now I can see exactly what I’m committing as I’m writing the commit message.

```
update content styles

inline and block code now look more similar
# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# On branch documentation-page
# Your branch is up to date with 'origin/documentation-page'.
#
# Changes to be committed:
#   modified:   assets/css/content.css
#
# Changes not staged for commit:
#   modified:   content/coop_work_reports/valueconnect.md
#   modified:   pages/documentation/_section/_slug.vue
#
# ------------------------ >8 ------------------------
# Do not modify or remove the line above.
# Everything below it will be ignored.
diff --git a/assets/css/content.css b/assets/css/content.css
index c0f8fe6..67ad0ed 100644
--- a/assets/css/content.css
+++ b/assets/css/content.css
@@ -36,10 +36,13 @@
     font-size: 1.05em;
 }
 
-.nuxt-content code {
+.nuxt-content code, .nuxt-content pre code span {
     font-size: 1rem;
-    background-color: var(--codeBackground);
+    background-color: #f5f2f0;
+    font-family: 'JetBrains';
+}
+
+.nuxt-content p code {
     padding: 3px;
-    border-radius: 5px;
 }
```

`rebase`

- Allows you to manually edit commit history. It includes a lot of features like changing commit messages and adding or removing code from commits.

`patch`

- An interactive menu for selecting which code to add to the next commit. As opposed to conventionally committing entire files.

# Fall 2021 Goals

## Add Linting to the Mobile App Pipeline ⚠️ 

**Technology**: [ESLint](https://eslint.org/docs/user-guide/getting-started), BASH, Pipelines.

**Skills**: Planning, Problem Solving.

In April, I begin working on the mobile app. Linting the front-end was very successful, and I can see that our mobile app code base could really use the same love. However, there were a couple issues with linting the front-end code base that I hope to remedy here.

While linting the front-end code base, I continually played catch up with new code or code that had been out before my linting pull request got merged. I had to keep linting new pull requests that did not contain my linting configuration until the entire code base was linted.

During this work term, I’m taking Systems Programming (CIS\*3050) which has a whole unit on BASH scripting. I take this opportunity to write a simple bash script that checks if changed files are linted. Instead of keeping track of whether every one’s pull requests are linted, the pipeline runs the script and checks this for me. It will take longer for the entire code base to be linted, but there is a lot less manual work.

This goal is currently in progress. There is a pull request out for it that is waiting to be reviewed.

## Successfully set up Logging in the Mobile App ❌

**Technology**: [Sentry](https://docs.sentry.io/platforms/javascript/).

**Skills**: Creativity, Organization.

My team hosts a book club where we read [Clean Code](https://www.amazon.ca/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882/ref=sr_1_1?crid=2X1EZUV4VNU3&keywords=clean+code&qid=1642081873&sprefix=clean+code%2Caps%2C544&sr=8-1) by Robert Martin to help develop our skills at work. In this book, I learn about wrapping third party code with your own code. The most fascinating reason why you should do this, is because if you ever need to replace third-party code (sometimes libraries get deprecated, or the library is not performing as well as one wants), you only need to change the file that wraps the third-party code. Better yet, you know exactly what the new library you’re implementing needs to do because the functions that wrapped the last library give you context!

```javascript
const cleanPreviousTransactions = () => {
  if (Platform.OS === 'android') {
    RNIap.flushFailedPurchasesCachedAsPendingAndroid();
  } else if (Platform.OS === 'ios') {
    RNIap.clearTransactionIOS();
  }
};
```

I really want to try this out with with the mobile app. We’re currently using a library called Sentry. It’s a powerful logging solution, but it’s not properly set up and our implementation is all over the place. My plan for completing this goal goes like this: See where we can improve on our current use of Sentry. Write some documentation to standardize the use of logging. Create a wrapper that wraps the Sentry library calls using the documentation I wrote.

Start ups are very fast paced. Before I knew it, I get moved to a new project and am no longer working on the mobile app. I am unsuccessful in completing this goal.

## Add Integration Tests to the Mobile App ❌

**Technology**: [Detox](https://github.com/wix/Detox), Pipelines, [React Native](https://reactnative.dev/docs/getting-started).

**Skills**: Initiative.

There are already a fair amount of unit tests in the mobile app, but there are no integration tests. Time and time again, we’re about to release a new version of the app, but someone finds an issue in an obscure area. The culprit? A code change on a completely different side of the app. Unit tests are great for testing isolated parts of the code base. Integration tests ensure isolated parts are working in cohesion with each other.

The next step to catching bugs in the mobile app is to add integrations tests. Once I was browsing some open source code to check out how they do things in [React Native](https://reactnative.dev/docs/getting-started). I came across this cool integration test library called [Detox](https://github.com/wix/Detox). It looks awesome, has pipeline capabilities, and works with React Native.

As with the last goal, I am unable to complete this goal. I see it as a side-effect of my opportunity to work in this field. I’m not going to waste time dwelling on my failure, rather I’ve already developed goals for the new project that I’m working on.

# Conclusion

This work term exceeds my expectations. I expect to be a junior developer completing tasks for those more experienced than me. Instead, I learn that it’s not about how long you’ve been in the industry, but about seizing opportunities. Developers can go their entire career without touching a linting configuration. But I see an opportunity, and create one. Now, I’m the main guy on the team when it comes to linting questions. I see many instances where someone acts on an opportunity and is now the most knowledgeable on the team when it comes to that area.

Before this work term, I would have said that I’m in coop to get a head start on industry experience before I graduate. After completing my first work term, **I’m in coop because of the information that I learn about the industry before I graduate**. The type of information that you can only obtain by working in the industry and keeping an open eye. I would not have learned how crucial it is to seize opportunities or how to provide value to your team without experience. That’s the most important part of my work term, and of coop in general.

I’m excited to take what I’ve learned during my work term to my academics. I already realize how many opportunities I have to grow. I can’t wait to improve more in my next work term.

# Acknowledgements

Throughout my work term, I received a great deal of support.

I cannot begin to express my thanks to my supervisor, Ben Pearo, who gave me my first opportunity by reaching out during his hiring process. You allow me to turn my spark into a flame. It’s amazing to see how much you have grown in a managerial role.

I want to extend my gratitude to my colleagues at Value Connect for their patience and practical suggestions. In particular, I would like to thank my CEO, Chris Bisson, for his astonishing amount of patience and charitability. You all work so hard, and you create an inordinate amount of opportunities for the development team.

Special thanks to my girlfriend, who has been instrumental to my success at work. You’ve supported me in multiple ways. Your drives and unconditional patience will forever be appreciated.

I’d like to recognize the assistance of you, the reader. If you’re reading this, you most-likely improve my life one way or another. Whether you have a hand in the coop program, or a friend who contributes to my well-being, thank you.
