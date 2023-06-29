---
layout: post
title: "The Agile Myth"
date: 2021-03-06 15:10:00 -0500
description: |
  Who actually follows the Agile Manifesto? Is it even reasonable to expect that anyone would?
tags:
- software development
- agile
aliases:
- /posts/2021-03-06-the-agile-myth
---

Writing a criticism of Agile after [the word "Agile" has been pronounced dead by one of its authors](https://pragdave.me/blog/2014/03/04/time-to-kill-agile.html#fn:2) may seem like overkill, but nothing has put a stop to the software industry's glut of [Agile certifications](https://www.bmc.com/blogs/agile-certifications/), [occupations of "doing Agile"](https://www.projectmanager.com/blog/what-is-a-scrum-master-everything-you-need), and every engineering manager I've ever spoken to saying "Oh, yes, we do Agile here." I could write an entire blog post haranguing the profiteering and narcissistic industry (see Dave Thomas' post linked above), but today I want to dive deeper, take a hard look at the original document, and ask "Who actually does this?"

![The Agile conference at The Lodge at Snowbird](/images/snowbird.jpeg)

The [Agile Manifesto](https://agilemanifesto.org) consists of four sets of values and twelve principles. The four values are:

- **Individuals and interactions** over processes and tools
- **Working software** over comprehensive documentation
- **Customer collaboration** over contract negotiation
- **Responding to change** over following a plan

In particular, I want to take a close look at the third and fourth points.

## Customer Collaboration

If you are a software developer, I want you to ask yourself the following question:

> When was the last time you spoke to a customer?

I'm willing to bet that if your company's industry doesn't contain the word "software," your answer is "Never." If you work in software consulting or for a B2C software company, your answer is hopefully "recently," and you can feel free to stop reading at this point. But for those of us who work on development teams in industries such as finance, healthcare, or telecom the very idea of "customer collaboration" seems alien.

**Why is that?** I would suggest that the answer lies in looking to the right: "contract negotiation." The only contract that I've negotiated as a software developer is that of my own employment—contracts between my company and customers have always been handled by someone "above" me, whether an engineering manager or even a dedicated sales team. The first I hear of a customer need is when I'm assigned to an [epic](https://www.atlassian.com/agile/project-management/epics) that outlines all of the aspects of a feature that my company is contractually obligated to build within a set timeframe.

To a large degree, this is an artifact of the industry. For instance, the lawyers for a pharmacy chain aren't going to be pleased if your company's approach to building software is "The software will be finished when it's finished." This sentiment is an insult to the very way that company does business! In most industries, when two companies decide to work together they begin with a written contract. In order to protect their investment, the company purchasing services has an interest in outlining concrete units of work that must be completed within set periods of time. If they feel that the vendor they hired is underperforming, they can prove this by referencing the contract and will be legally protected when seeking to terminate the contract or refund their purchase.

By the time the project begins, and the third Agile value has already been eshewed.

## Responding to Change

"Customer collaboration" and "Responding to change" are tightly related concepts. After all, don't most of the changes you make to your software come from your customers? Despite not being involved in contract negotiation, perhaps a software developer could deliver some working software quickly, receive some feedback, and respond. However, that's rarely the case.

See if this sounds familiar to you: I typically spend about a week developing the first draft of a feature. However, I don't have direct contact with the customer to ask if it meets their expectations, so I'll notify my manager that the first draft is ready for the customer to review. My manager will then tell the customer representative, the customer representative will enlist a potential user to try the new feature, that person will provide feedback back to the customer representative, the customer representative will provide that feedback to my manager, and my manager will provide the feedback to me. This process takes _weeks_. I can't just sit on my hands while all of this takes place, I've got to keep working! So by the time that I receive feedback on my work, the feature is already finished or nearly so. If the feedback I ultimately receive is "The customer has determined that this feature isn't actually useful after all," or "The customer thinks that the feature would be more useful if it was completely restructured," weeks worth of my effort have been wasted.

![Typical feature feedback loop](/images/feedback_loop.png)

**In order for developers to truly utilize the Agile approach, they must have direct contact with their customers.**

At this point, let's take a step back and ask: _Why don't developers have direct contact with customers?_

One theory is simply that non-software industries—automotive, manufacturing, etc—have been doing business long before software came on the scene, and are not prepared to let seventeen unelected programmers tell them how to run their businesses. Long before software, these companies were selling products using dedicated sales teams and managing clients with account managers. Agile methodologies may have revolutionized the software industry, but they didn't revolutionize banking.

Another theory is that companies worry about their developer's communication skills—and for good reason! In order for developers to engage in "customer collaboration," developers need to be able to communicate clearly with their non-technical counterparts. The trope of a developer blurting a string of barely coherent jargon in response to questions like "Could we add a 'Save' button to this page?" is founded in reality: I've seen it happen. Out of fear of scaring off customers by insisting that they collaborate with engineers, companies install an engineering manager as a buffer between the engineers and the customer.

Whatever the reason, the truth is that customer collaboration and receiving timely feedback are simply not realities of software development at many, perhaps even most, companies. While managers may say "We do Agile here," this is at least half myth.


