---
layout: post
title:  "Don't tell me how to test my system"
date:   2019-08-29 21:47:57 +0100
tags: development tests
---

We argue a lot about how to test code. Some are in favour of typical testing pyramid. Some claim that unit tests don’t give you enough confidence, so you should mainly write integration tests. Neither answer is correct. The problem is that we shouldn’t even look for one answer in the first place.

We strive for simple answers and clear rules. We don’t want to deal with risk and uncertainty. “Always follow the testing pyramid” is easy to follow. But it’s not really helpful in the long run.

You see, when someone is saying “Testing pyramid is the best way”, what they’re really saying is: “ **Based on my experience,** types of projects I built, teams I worked in, I **currently** **believe** that testing pyramid is the best way of testing software”. That’s why people cannot come to terms with one answer — because they have different — often hidden — assumptions and constraints.

Instead of trying to find the “correct” answer, we should try to understand the process of making those decisions.

# **Identifying assumptions and constraints**

Why do you write tests? We do that for various, sometimes conflicting, reasons. That reasons are important factors when choosing the testing strategy.

Do you write tests to:

- be confident that your system works?
- guide and improve design of your system?
- have the tests serve as documentation?
- find regressions?
- make refactoring easier?

Apart from motivation, you should consider the people that build the software. How experienced are the team members? Does the team change a lot? What’s the size of the team? Are there distinct responsibilities or does everyone work on everything?

You should consider the type of software you’re writing. Is this a user-facing system, or a library? Is the system public, or used internally? What are the consequences of bugs or downtime? Is this a new project, or a legacy system? Are bugs frequent? Does the system change a lot? Are the tests good right now? Is this a well-designed, modular system, or a big ball of mud?

All of these assumptions affect the right testing strategy for you. A team of 3 experienced engineers working on a greenfield internal project will test differently than a team of 10 engineers, mostly beginners, working on a legacy, buggy, user-facing software.

# **Questioning assumptions**

To make good decisions, we have to understand our assumptions. We have to question them. We have to try to find out what’s true, and use logic, instead of relying on opinions and intuition.

# **Be precise**

Some people say that integration tests give you more confidence. It’s not precise enough. “Confidence” is not precise enough.

Integration tests can verify that the happy path works well. They can verify that the system does not crash. On the other hand, testing all the corner cases can be really expensive with integration tests, especially if the logic is complicated. So, what does confidence mean to you? Is it more important for the system to be up and running (even with some bugs), or to be 100% correct?

Integration tests can find bugs in… well, integration. They are a great way to keep the system up and running after deploying. But integration bugs are usually easy to find manually. When components are not integrated well, they often fail fast and they fail loud. Sometimes it takes minutes after deploying to notice and fix them or roll back the changes. For some systems, few minutes of downtime is a disaster. But for some (for example: internal, or low-traffic systems) it may be perfectly fine. Sometimes the consequences of a system running with a bug are far more dangerous than consequences of some downtime.

So — be precise with your assumptions and arguments. Instead of saying “Integration tests give you more confidence”, say: “Integration tests decrease the chance of system crashing shortly after deployment”. Then, you can decide if not crashing after deployment is your highest priority.

# **See the bigger picture**

Code design and tests influence each other. The tests can improve the design of the code. On the other hand, the current design affects the tests.

How do you define what a “unit” is? We often consider a *unit* to be a class, module, function, depending on the language. If the system is poorly designed, with lots of development coupling, testing each class in isolation won’t give you much confidence. In this case, writing integration tests may be the best option to quickly address some problems.

For a well-designed modular system, on the other hand, finding a good unit to test is much easier. With good encapsulation and well-defined interfaces, unit tests can be much more reliable. You can treat the test as just another client of module’s interface, ignore implementation details and only care about the behaviour. This will result in tests that give you a lot of confidence and allow for easy refactoring.

To see the bigger picture, we have to notice and understand all the relationships. If we write tests to improve X, what other factors affect X? Is the relationship cyclic? Does X affect the way we write tests?

That’s where the test-code relationship matters. If you notice that unit tests don’t give you enough confidence, you can start writing more integration tests. On the other hand, you can invest in good unit testing and refactoring to bring the system to a state where the software is more modular and unit testing gives just enough confidence. Always try to find and understand the root causes of your problems, instead of just fixing the symptoms.

# **Conclusions**

If you’re wondering what testing strategy to use, don’t blindly follow advice from others. They can only tell you what worked for them (or even worse — what works only in theory).

Instead:

- Understand and be precise about your assumptions (Why do I test? If I test to optimise X, what other factors affect X? Is X really the most important thing for me right now?)
- Try to see the broader picture. If you’re not confident with your tests, maybe the problem is the poor design of the code?
- Experiment. A lot. Use your assumptions and logic to make a hypothesis. Then, experiment and see if you get what you expected. Adjust and iterate.

I hope you’ll realise that this article is not about testing. The point is not to argue for one testing strategy or another. The point is that software development is full of complex problems. Testing is just one of them. What’s important is to learn how to deal with such problems. We have to learn how to understand our assumptions, constraints and tradeoffs. We have to learn to make decisions based on facts and logic. We have to learn to analyse those decisions later. This is the only way we can grow.

Understanding the process of making the decision is far more important than the decision itself. Improving your critical thinking and decision making skills is the best way if you want to excel at what you do.
