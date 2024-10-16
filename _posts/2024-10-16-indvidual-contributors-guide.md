---
title: Ryan's Individual Contributor Guide
image: /assets/images/james-ohlerking-Hx-91MSmDto-unsplash.jpg
---
This is what I look out for when reviewing Pull Requests. You can use this guide to develop your own opinion on code reviews, or as a reference for your colleagues to explain why you’re requesting a change. This guide is written for the individual contributor making a contribution to a codebase.

### Before Contributing

**Stop Tactical Programming**

Tactical programming is the type of programming where you look to complete the code as quick as possible. You may find the first solution you have to a problem and go with it, or use a previous solution you know may not be the best instead of looking for new solutions. This is great when you have very limited time to complete a task such as when the deadline is close, or when there is a critical bug in production. However most of programming should be done strategically.

Strategic programming is when you produce code that is very well designed. Your primary focus while programming in this way should be designing your codebase to reduce complexity. Take a step back before writing some code and think about the context of what you will program. Is there a class that already exists which gets the job done? Are there multiple ways to design this? If so, which one is better? In the future, will a developer know what this class does just by looking at the interface? If a developer needs to make a change to my code, will it be easy to do?

> *Am I introducing unnecessary complexity by creating this change?*

**Be Pragmatic**

Understand the context in which you are working. For example, if you are working at a large company where there is less of a rush, you may be able to follow all code standards perfectly. You can take another day or two to mock that library you’ve been meaning to mock out to enable the test suite to have better tests. If you are working in a smaller company where you need to maximize the value you provide in a short time, it becomes more and more important to balance your contribution quality with the time it takes to make your contribution and not to strive for perfection. You may have to settle for one great test, a test that relies on implementation details, or no tests at all.

> *Is what I’m contributing right now more important and urgent than any other contributions I could be making?*

### Commits

**Logically Grouped Changes**

Your commits should contain logically grouped changes. For example, if you are going to refactor code such that it’s easier to implement a new feature, you should have two commits: one for refactoring the code, and one for implementing the new feature. How exactly you logically group your commits should be up to you and your team. I’ve seen some teams group their tests and the new feature they are testing in one single commit.

This is important in the event of a breaking change. If your commit contains changes to your build script and a new feature, and your commit is named “add new feature”, it can be hard to understand at a glance why this new feature broke the build. Now, plenty of tools are out there to look at the git history of only the build script, however it may be difficult for new developers who don’t understand the codebase to find this.

> *If a future developer viewed this commit, would there be any surprise code changes?*

**Messages Should Explain Why**

Put simply, if I wanted to view what changes a commit made, I would look at the file differences for that commit. If I want to understand why a commit was made, I should be able to tell by the commit message.

An important and under-utilized feature in git is multiline commits. If you enter a title in your commit (”fix: force client to load latest changes”), then separate your message by one empty line, you can fill in the rest of the commit message with the body to add additional context (”index.js was being cached by the client, and so the latest changes would not load. closes: #543”).

> *Will a future developer understand why I made this commit?*

### Pull Requests

**Always Review the Diff**

This may seem obvious, but I’m always surprised when I see parts of a pull request that could have been caught if the author had briefly reviewed their code. API keys, personal comments and debug logs are a few things I’ve seen in pull requests.

> *Is there anything immediately obvious that shouldn’t be in my pull request?*

### Testing

**Do not Test Implementation Details**

To the greatest extent possible, your tests should not rely on implementation details. Implementation details are the parts of your code that implement the feature, user story or interaction. Tests that rely on implementation details need to change each time the implementation changes which is much more frequent than any feature, user story or interaction. Writing your tests using implementation details are a major cause for a brittle test suite as tests will break each time time you write new code.

Let’s say you are writing a React-Native application with some deeply nested and complicated navigation. It may be tempting to mock out navigation calls as a whole and simply assert they are called. You could imagine if a screen turned into a popup modal which uses a show/hide mechanism instead of a call to the navigation router, your test would fail because the navigation call is no longer called. The implementation has changed, but the feature has not. Now imagine if all your tests were based on implementation details. How many times would you break the test suite when you change an implementation?

> *Am I testing an interaction with my application, or am I testing implementation details?*

**Mock Inputs and Capture Outputs**

When writing tests, be concerned about user interaction. Anywhere the user may be interfacing with your software (e.g., through an API call, or through a UI) should be where you start your tests. Writing tests beyond user interaction, such as in unit tests, and the value you get for the time you put in greatly diminishes. In general, I follow this framework for writing test cases:

1. Mock all inputs (e.g., GET requests, database reads)
2. Trigger user interaction (e.g., Click on a button, call API function)
3. Capture all outputs (e.g., POST requests, database writes)

At the end of your tests, you should assert that the outputs are what you expected.

> *Am I only mocking external inputs, and not implementation details?*

### Linting

It’s important that you follow some standard of code style guidelines in the project. A codebase with multiple styles is like reading a novel where the author switches tones randomly. Codebases with a single style, like books, increase comprehension by allowing for faster reading.

If your codebase does not have an automatic linter yet, this might be a great feature to bring up with your manager. In the meantime it’s best to follow the style of whichever programmer came before you. Don’t start implementing your own style because you think it’s better, unless you can change the entire codebase to your style. Otherwise your change will most-likely get left for the next developer who introduces their own style until you have style spaghetti.

Ideally, your codebase should have some sort of automatic continuous integration that runs on your pull requests to make sure your code follows the style guidelines of the project. If it doesn’t, this is another great feature to bring up with your manager to improve code quality.

> *Does my contribution follow the style of the project?*

### Logging

**Logging Levels**

| **Level** | **Meaning** |
| --- | --- |
| Debug | Very important message for local development. |
| Information | Useful for understanding the flow of control. No attention is required. |
| Warning | There was an issue although the task was able to continue. You should probably do something about this. |
| Error | There was an issue and the task has reached an unrecoverable state. You need to do something about this. |

**Debug Logs**

Personally written debug logs can be useful, but usually aren’t. Do not commit your personal debug logs because they don’t have any meaning to anyone other than to yourself. If you are going to commit debug logs, they should be useful to the whole team and to future developers. Debug logs like this are not useful “failure”. It should instead be formatted like this: “POST /api/orders/${id} Unsuccessful. Reason: ${reason}”. Notice how the debug log now gives context at what point the flow of control is. The log is easily searchable in the codebase if there were issues. At this point, this log could be considered an information level log.

Too many debug logs are bad, as it can become hard to find new logs you put for your own development. Each time you commit a debug log to the code base, it should be immediately obvious to everyone why it needs to be kept. Additionally, too many well-formatted debug logs should not be tolerated by your team.

> *Will future developers understand why this log is needed?*

### Data Models

**Be more concerned with your data models than your functions**

There is one thing that is guaranteed to live longer than your code, and that’s your data models. Data models are the fundamental abstraction that defines your codebase’s interactions. Those interactions change all the time but your data models will rarely change. Data models are very important for future developers to understand your application. Typing a Java response as a JsonNode, or labelling a typescript object with type “any” gives future developers no insight on what interactions they can make in your application. Even worse, they may add to your data model misconstruing the original abstraction meant to be made by your data model. Think hard about your data models, as they can make or break the future of your application.

> *Can I make my data model more simple?*

### References

[Philosophy of Software Design](https://a.co/d/3626gLq) by John Ousterhout

[The Pragmatic Programmer](https://a.co/d/9empAUk) by David Thomas, Andrew Hunt