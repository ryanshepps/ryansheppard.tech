---
title: Managing Multiple Release Versions
image: /assets/images/ankush-minda-multiple-balloons-released.jpg
---

Recently, I made a mistake while releasing a new version of a component. My team maintains various Spring components whose versions match Spring Boot's release versions. With the release of Spring Boot 3, I made the change to upgrade our component. A few weeks later, a colleague needed to make a change to an older version of the component. I wasn't sure what to do in this situation, which took me down a rabbit hole on how others manage multiple release versions.

### Spring Boot

For Spring Boot, everything that goes on the main branch is preparation for the next release version. When a minor version is released, a new branch is created off of main. Interestingly, each branch includes both the major and the minor version for each release (for example, `2.3.x` or `3.1.x`). This is an obvious strategy to accomodate their [support policy](https://github.com/spring-projects/spring-boot/wiki/Supported-Versions). If a change to an older release is needed, the Spring Boot team will merge the older release branch with the change into the newer release branches until the change is merged into the main branch. 

The Spring Boot team uses very little automation in their release strategy. Each pull request goes through a rigorous build process, but maybe this is all that is required in a repository that is intensely active and frequently changed. Managing multiple minor versions requires more work, but the Spring Boot team has the amount of work down to just the essentials.

In my research, this is by far the most popular release strategy for managing multiple versions. Other projects that use this type of release strategy are the [Phoenix Framework](https://github.com/phoenixframework/phoenix), [Scala](https://github.com/scala/scala), and [Kafka](https://github.com/apache/kafka).

### Vue

Vue uses different repositories to manage their major versions. Since changes won't need to be merged upstream, and the codebases are largely different, this is an appropriate design for managing multiple versions. Where Vue gets interesting, is when it needs to make a change to a previous version. When Vue 3 needs to revert a change, for example, they will release the change under a new incremented version number instead of fixing the old version and merging the changes upstream.

### Alternative Considerations

**Feature Toggle**

It's important to consider whether managing multiple versions is really necessary. A common alternative is enabling [feature toggles](neede://martinfowler.com/articles/feature-toggles.html) for new changes. For example, you can first release a change into the wild and gradually enable it for a subset of your users. Once it's been well tested by more than half of users, it's safe to release the feature and remove the toggle from your code. This type of release strategy reduces the need for managing multiple versions since changes should be well tested, and there should be no need to "go back" and make a change.

**Non-decreasing Versioning**

Code that is not depended upon by other applications most likely doesn't need multiple release versions. Web applications are a good example of code that doesn't need multiple release versions. If a previous version needs a change, it can be added into the next release. Since services that may depend on your application are interfaced via some API, not a dependency version, releasing the new change in an incremented version works just fine and solves many of the headaches that come with managing multiple release versions.

## Conclusions

Every company will have their own strategies for maintaining multiple releases of the same component. Establishing a support policy will help decide the effort required to maintain multiple releases, or if maintenance of multiple releases is really needed at all.

In my case, since our component is matches the release version of Spring Boot, it makes sense to adopt the same release strategy Spring Boot uses for releasing their framework.
