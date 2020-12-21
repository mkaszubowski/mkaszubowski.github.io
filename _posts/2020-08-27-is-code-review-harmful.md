---
layout: post
title: "Is code review harmful?"
date: 2020-08-27 11:30:00 +0100
tags: teams
skip_related: true
canonical_url: https://appunite.com/blog/is-code-review-harmful
description: Use code review for collaboration, not verification.
excerpt: Code review is a great tool. But when it's mandatory, it can hide some problems. In this post, I explore how we can improve the team effectiveness by fixing the underlying problems and using code review for collaboration, not verification.
---

Ok, this might be controversial. **If code review in your team is mandatory, I think it causes more harm than benefits in the long run**.

It's not an argument against code review in general. I think there's no denying that the practice of code review has numerous benefits.

But if you can't imagine your team operating without code review, you should probably reconsider your approach.

## **Trust**

Mandatory code review is perfect for open source. When you have a bunch of strangers contributing changes to an open source project, it makes sense to require all the changes to be approved by the maintainer. The reason is that there is **no trust between the maintainer and most contributors**.

If you use the same process for code review inside your team, what does it say about the relationship between people? With an absence of trust being one of the [biggest issues for teams](https://www.goodreads.com/book/show/20642542-the-five-disfuntions-of-a-team-a-leadership-fable?ac=1&from_search=true&qid=whzGMninaX&rank=1), it's worth thinking about it for a while.

{: .center.with-caption}
![The Five Disfunctions of a Team](https://appunite-blog.s3.eu-central-1.amazonaws.com/images/3acea0a0/21d0/ZHlzZnVuY3Rpb25zLnBuZw==)

{: .image-caption}
Source: [The Five Dysfunctions of a Team](https://www.goodreads.com/book/show/20642542-the-five-disfuntions-of-a-team-a-leadership-fable)

Using the same tool/process for a completely different environment should always be done with extreme care.

## **False confidence**

I won't discuss typical problems with code review. Unnecessary discussions, arguing about style, waiting too long for the review... These are relatively easy to fix. But there's a bigger issue.

Mandatory **code review is something that can hide other problems.**

There's a bunch of issues, whose consequences can be somehow mitigated by code review:

- Not enough mentoring and some team members lacking the skills/knowledge to write correct and good-quality code.
- Poor planning, which doesn't include time for good collaboration, pair/mob programming.
- Not enough design work (not to confuse with Big Design Up Front) or poor domain model.
- No shared ownership for the product.
- Not working in small batches which can be safely released.
- Poor deployment practices. No way to safely and quickly deploy, test, and roll back every change you make.
- Lack of reliable test suite (not to confuse with "100% test coverage").
- Lack of automated tools like linters/formatters etc.
- ...

## **The challenge**

This false sense of confidence is one of the most challenging things about code review. Because **if these problems are there, not going through a code review process might actually uncover them.** If a team notices that things got worse, they will probably quickly go back to requiring approval for each change.

That's why it's a controversial topic to talk about. With many great advantages of code review and many real-life stories of things going poorly when code review is missing, it's really hard to believe that it might be the code review itself which contributes to some of these problems.

## **Work on fundamentals**

If you cannot imagine working without code review, think hard about what is causing that. I believe that we should always work on the underlying issues and good fundamentals, instead of dealing with the symptoms.

Try to build trust, give your team members time to learn, improve your processes. I'm not saying that you should get rid of code review. But **code review should be about collaboration, not verification.**

## **Some ideas**

If you rely on mandatory code review right now, I wouldn't recommend stopping it right now. But there are some things you can do to make your team rely on it less:

- Do more pair-programming. It's a great way to build trust, shared understanding and ownership, to share knowledge, and to receive quick feedback.
- Invest in mentoring. It's one of the most effective ways of building skills and knowledge and a clear win-win for both the mentor and a mentee.
- Include time for learning and research in your work time.
- Do some collaborative design before you start working on something. Workshops like Event Storming or a quick collaboration session in front of a whiteboard can help you find most of the potential problems and save a lot of time. Alternatively, sketch out a solution yourself and then present it to your team before moving to implementation.
- Work in smaller batches. It will shorten the feedback cycle and decrease risks significantly.
- Try to improve your deployment and releasing process. Being able to quickly deploy, test, and roll back every change is a game changer. Learn about feature flags, zero-downtime deployments, blue green deployments and canary deployments.
- Invest in a reliable test suite, bug reporting, monitoring, logging. Being able to quickly detect, debug and rollback/fix bugs is often more important than preventing them (since you cannot prevent them all).

I believe these are low hanging fruit you can start doing with low cost and without much risk. They take some time, but the benefits come quickly (and are really long-lasting).

## **Summary**

Trust is essential for well-performing teams. Processes and tools can be useful, but if we rely on them too much (and can't imagine working without them), it's always a sign of some underlying issues.

I hope you will try some of the things I mentioned and focus on building great fundamentals for your team.

If you don't agree with this article, though, please reach out! I would love to hear your thoughts on that subject. __You can reply to [this tweet](https://twitter.com/mkaszubowski94/status/1300456801452937216) to start a conversation.__

## **Related**

- You might want to look at my article about [Enabling Constraints](https://mkaszubowski.com/2020/05/11/enabling-constraints-in-software-development.html) as an alternative approach. Sometimes setting the rule of "no code review allowed" (even as an experiment) might be a good way of finding and solving problems really fast.

*Originally published on [AppUnite blog](https://appunite.com/blog/is-code-review-harmful)*
