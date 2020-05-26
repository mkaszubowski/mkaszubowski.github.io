---
layout: post
title:  "Enabling constraints in software development"
date:   2020-05-11 09:00:00 +0100
tags: systems-thinking complexity cynefin
canonical_url: https://appunite.com/blog/enabling-constraints-in-software-development
---

In my work, I draw a lot of inspiration from Systems Thinking and Complexity Theory. In this post, I'd like to describe a concept of **enabling constraints** and how to use them in software development.

## **Governing and enabling constraints**

A constraint is something that limits what you can do. We can differentiate between two types of constraints: **governing constraints and enabling constraints.**

Governing constraints are strict and don’t allow for much flexibility. Here are some examples:

- “Every pull request has to be accepted by at least two team members”
- Dev team and Operations teams
- “We use Java”
- Mandatory 100% test coverage
- Separate Engineering/Sales/Marketing departments

Governing constraints lead everyone to follow the same, well-defined path.

Enabling constraints are different. They don't show a single path, but enable you to explore new ways of solving problems. To understand why, let's take a look at some examples.

## **No git branches**

Pushing all the code directly to master, without branches, pull requests and blocking code review might seem weird.

I'm sure you can list many problems with this approach. What are they?

- Someone can push bad code to master
- Someone might push a work-in-progress code to master and release it
- How do you do code reviews?
- How do you control what is deployed and when?

Branches and pull requests provide a safety net for our codebase. What if the safety net was off? Well, we'd have to try different ideas. Here are a few:

- instead of blocking code review process, you can try pair/mob programming
- to prevent releasing the code that's in progress, you can use feature toggles or configuration
- instead of pushing big chunks of code, we can push smaller and safer chunks of code more often and test them quickly
- to prevent someone from deploying our code by mistake (when it shouldn't have been deployed), we can automatically deploy each master build

## **One-week iterations**

Here's a great enabling constraint: every Friday, you get the team and stakeholders together and do a demo of what you were able to accomplish, ideally on a live product.

What's the problem here? The most obvious one is that it's not always possible to ship an entire feature during one week. To solve this problem, we have to change the way we work. Again, some ideas:

- instead of shipping the entire feature, we can think about the smallest possible changes that increase the value of the product
- we can use A/B testing, canary deployment or feature toggles to test the new features or enable that only for certain users
- we have to limit "maintenance sprints" and focus on constant, day-to-day improvements (see: [The Boy Scout Rule](https://www.oreilly.com/library/view/97-things-every/9780596809515/ch08.html))

In addition, we have a point in time when the entire team can reflect on the progress, learn, make sure that the work contributes to the long-term plan and goals. This creates **alignment** and provides the crucial **feedback** we need when dealing with complex problems.

A similar example of an enabling constraint is having daily status updates.

## **Other examples:**

Code:

- Static type system
- Immutable data
- No comments allowed
- Relational database
- Asynchronous message queue
- TDD
- Pair programming

Deployment / operations:

- CI pipelines
- Serverless
- Stateless components
- Immutable infrastructure

Work organisation / teamwork:

- No office
- No Project Manager / No QA
- No overtime
- No synchronous communication / no direct messages
- Transparent salaries
- Autonomous and multidisciplinary teams

Can you think what these constraints enable? How do they affect the people who do the work? What behavior emerges when these constraints are there?

## **The role of constraints**

### Governing constraints

You can notice a pattern here. The goal of governing constraints is to ensure a **single proven (and clear) way of doing something**. Governing constraints are like processes - they can give you a lot of **safety and efficiency** when applied correctly.

Governing constraints correspond to top-down decision making. They work well for **complicated** **problems** - when the rules are known, the goal is clear and things change slowly. If you are dealing with a well-defined problem and there’s an established, repeatable solution, setting governing constraints might be a good option.

### Enabling constraints

The goal of enabling constraints is to encourage novelty and innovation. To **generate new ideas**, create many possible solutions, and **experiment** with them. The goal of this is to constantly **learn and improve**. That’s why it’s crucial for enabling constraint to provide constant and fast **feedback**.

Enabling constraints to support **bottom-up decision making**. They are more preferable for **complex problems** - when the rules are vague, uncertainty is high, the environment is rapidly changing, goals cannot be clearly defined and measured, things are unpredictable, and there are no best practices.

## **Summary**

Enabling constraints influence how elements of the system behave and collaborate. [They force alignment and provide feedback](https://theitriskmanager.com/2018/12/09/constraints-that-enable/), while leaving a lot of freedom.

**Constraints can lead to innovation**. Preventing you from doing something doesn't necessarily mean that this thing is bad - it can be a great way to encourage trying new ideas in an uncertain and unstable environment.

Also - I’d encourage you to read more about [Cynefin](https://en.wikipedia.org/wiki/Cynefin_framework) and differences between complex and complicated domains. Software development is full of both, so it is crucial to be able to recognize which kind of problem you're dealing with and act accordingly. **Understanding what kind of problem you’re facing, will help you set the proper constraints.**

If your environment is stable and you want to maximize safety and efficiency, think about governing constraints. If the environment is rapidly changing and unpredictable, you might want to optimize for novelty and innovation instead. In this case, setting the right enabling constraints is probably a good thing to do.

## **Call to action**

What is one practice that you think is absolutely crucial for the work you do? Is it code review? Is it TDD? Is it the relational database? Is it going to the office every day?

Can you imagine setting an enabling constraint that forbids that practice? How would your work look if you did that?

Are you willing to experiment a bit, try that for some time and see what happens? If you do, I'd love to hear about this!

## References

- [Cynefin framework introduction](https://cognitive-edge.com/videos/cynefin-framework-in)
- [Cynefin for Everyone!](https://medium.com/@lunivore/cynefin-for-everyone-d5f47d9bd102)
- [Dave Snowden - Complex Adaptive Systems (DDD Europe 2018)](http://www.youtube.com/watch?v=l4-vpegxYPg)
- [Constraints that enable](https://theitriskmanager.com/2018/12/09/constraints-that-enable/)
- [Constraints that Enable Innovation - Alicia Juarrero](https://vimeo.com/128934608)
