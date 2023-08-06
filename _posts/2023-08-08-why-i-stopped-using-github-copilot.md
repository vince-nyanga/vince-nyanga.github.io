---
title: "I stopped using GitHub Copilot after 4 months. Here's why"
excerpt: "My experience with GitHub Copilot"
date: 2023-08-06
tags: [General, GitHub Copilot]
---

GitHub Copilot is an AI tool that assists with code suggestions while you write code. It was trained using thousands upon thousands of lines of code from GitHub’s public repositories. Earlier this year I signed up for GitHub Copilot and used it for four months. In this article I will talk about experience and why I eventually stopped using it.

## My setup

I installed Copilot on my Jetbrains Rider and Visual Studio Code IDEs. I write mostly C# code so my feedback is based on C#. If you're using other languages like Python (I heard it's very good with Python), you may have a different experience.

## What I enjoyed

When I started using Copilot, I was amazed at how well it performed. I was especially impressed when it comes to writing unit tests. All I needed to do was write the first test to give it an idea of how I structure my tests. After that, it would generate tests for the other scenarios in the system under test without straying too far off the context.

While writing logic, I'd prompt Copilot to give me suggestions by adding comments to my code. It would provide a couple of suggestions on how to solve the problem. That was impressive!

There were few occasions when I provided Copilot with a code block and asked it to explain what the code was doing. It also performed very well.

## Why I stopped using it

As time went by, my excitement started to fade. I started to realise that I wasn't quite enjoying working with Copilot notwithstanding the nice features it has. Here's why I eventually decided to stop using it:

### Reduced productivity

I know Copilot is supposed to increase developer productivity but that wasn't entirely the case with me. While it really helped me when it comes to writing tests, I found myself spending more time trying to debug its suggestions in my head to ensure they made sense before I could accept them.

### The subtle bugs

The reason why I eventually started to scrutinise Copilot’s suggestions was that there were instances when I blindly accepted the suggestions that looked impressive yet they contained subtle bugs. If I didn't have experience with C# I would have just rolled the code without realising it contained bugs. If you want to use Copilot, I think it's best if you have some experience with the programming language. Otherwise you might introduce some bugs into your codebase.

### It gets in your face sometimes

I don't know how to explain this. I found myself having to switch off Copilot at times because it was getting annoying 😂. I'd be in the zone writing some code and it would add suggestions that are way off. I found it extremely distracting, especially when I really wanted to focus.

## Conclusion

In this article I spoke about my experience with GitHub Copilot and why I eventually stopped using it. Don't get me wrong, GitHub Copilot is a great tool and it can be very useful. However, it just didn't click with me. I'd suggest you give it a go if you haven't already, and see how it fairs. Thanks for reading.
