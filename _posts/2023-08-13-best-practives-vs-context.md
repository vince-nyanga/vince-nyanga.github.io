---
title: "Best Practices, Patterns & Principles vs Context In Software Development"
excerpt: "In this post I will talk about the importance of context when applying best practices in software development."
date: 2023-08-13
tags: [General, Opinion]
---

“You cannot do that! It violates X principle…” I have heard this statement countless times in my career. At first, when I was still young and clueless, I would feel dirty, like I have committed a mortal sin on which the software gods looked away in disgust. As I went on in my career, I began to question such statements. Did I violate a principle? Am I not using the best practice? Does this best practice apply to what I am doing? Some of the time the answer to that question is yes, but in most cases I have found it not to be the case. In this very short post I am going to talk about the importance of context in software engineering.

## The Buzz Words Plague

The software engineering world never runs out of buzz words. Everyday there is something cool. A better way to do something — a best practice. I have witnessed in awe as my fellow professionals (and myself sometimes) flock to the new shiny thing, shunning yesterday’s best practices for the most relevant. All of a sudden, that which was once a best practice has instantaneously morphed into an anti pattern.

Again, in some cases that is true. With the constant improvements in technology some of the things that we held in high regard are no longer relevant. However, in most cases, we fall in the trap of going wherever the wind is blowing, blindly applying solutions where they don’t apply.

## Principles, Patterns And Best Practices

What are software development principles? Software development principles are a set of guidelines that help software engineers write quality, maintainable software. They come about naturally as we encounter similar problems or as we repeatedly make the same mistakes. They provide templates or guides to solve recurring problems. I will using principles and patterns interchangeably in this post though I think they are slightly different.

Take the Don’t Repeat Yourself (DRY) principle for instance. It is a result of people getting caught out when all of a sudden they are required to change the same logic that’s scattered all over their codebase. It’s definitely a good guideline. But is it the law? Certainly not!

Best practices on the other hand, are things that I find mostly misused or dare I say, abused, in the software engineering industry. What is a best practice anyway? To me, a best a practice is something that solves a particular problem better than other options (that the person has managed to come up with). I try to avoid the word best because chances are there is a better way of solving that problem. Are best practices bad? Not if they are applied correctly to the problems that suit them. This is where context comes in.

## The Importance Of Context

Like they say, the best answer in software engineering ‘is it depends’. There is an opportunity cost to every decision we make. Everything has a tradeoff. Choosing which best practice or pattern to use should always be made within the context of the problem space. Not all problems are the same. They may appear similar at face value, which usually leads to incorrectly applying a solution that doesn’t efficiently or effectively solve the problem. The pattern (or best practice) that worked on your previous project won’t necessarily apply to the next.

Should you repeat yourself (violating the DRY principle)? Probably not, but if, in your context, you need to do so, please go ahead. I have witnessed two pieces of logic that appear similar initially, diverge as the project grows and requirements change. Context matters.

I have countless partial projects on GitHub, most of them trying to solve the same problem using whatever the buzz word was at that moment — Clean Architecture, DDD, microservices, you name it. Most of the time I stopped midway because of pure laziness. However, in some cases I just hit a brick wall when I realised that I was over engineering the solution while trying to follow the best practice. Certainly that best practice/pattern didn’t fit my problem space very well.

Conclusion

Am I saying patterns, principles and good practices are a bad thing? No. They are very useful and most of the time help us avoid banging our heads against the wall while try to solve certain problems. However, they should not be applied blindly without taking into consideration the context. Context is king!

I would like to hear what your opinion is on this topic. Please feel free to leave a comment below. Thanks so much for taking your time to read.
