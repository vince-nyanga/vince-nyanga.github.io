---
title: "Domain Driven Design : My April Read"
date: 2020-04-25
categories: [Design, Architecture]
---

At the beginning of this month I challenged myself to learn more about Domain Driven Design. I have come across this topic multiple times but had never focused much on it. Instead of reading blog posts and watching courses on Pluralsight (which I eventually did), I decided to first read Eric Evans' [book](https://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215) in which he introduces the concept. In this post I am going to talk about what I have learnt about Domain Driven Design and how it can help solve complex software problems. I am going to talk about core concepts in Domain Driven Design as I understand them.

## What Is A Domain?

Before we talk about Domain Driven Design we need to understand what a domain is. According to this [definition](https://www.lexico.com/en/definition/domain) a domain is a specified sphere of activity or knowledge. In software terms a domain is a subject area to which a software program is applied, for instance, e-commerce, insurance, entertainment, etcetera. Every software program seeks to solve a problem in a specific domain.

## Domain Driven Design

Now that we have an understanding of what a domain is let us now talk about what Domain Driven Design is. From an English standpoint the phrase domain-driven design simply means a design that is driven by and revolves around a particular domain. But what is it exactly? Domain Driven Design is an approach to software development that places primary focus on the core domain of the software program. The first point of call for developers should be to understand the problem domain of the software they want to build and then create an abstract model of the domain. If one doesn't fully understand the problem domain that the software seeks to solve it is virtually impossible to solve it properly.

As software developers we tend to think about every problem we face in technical terms -- the data structures, algorithms etc. This however, is not supposed to be the case. Eric Evans points out that the most significant complexity of many software applications is not technical but rather in the domain itself -- the activity or business of the user. Without a proper design of the core domain one's technical prowess is of no value at all.

Domain Driven Design is not a once-off event. It is an iterative process in which the model is refactored and refined as more knowledge and insight of the domain is gained.

## Domain Driven Design Concepts

We have already spoken about the domain which, in my opinion, is the core around which everything revolves. I am going to talk about a few terms I picked up in Eric Evans' book that I think are important in Domain Driven Design.

### Model

In Domain Driven Design a model is a set of abstractions that describe the aspects and processes of the domain. It is a simplification of the 'bigger picture' of the problem domain and describes the relationships among the entities within the domain.

### Ubiquitous Language

This is the common language structured around the domain model and is used by everyone involved with the project. In Domain Driven Design everyone working on the project should speak the same language that is free of ambiguities. Both the domain experts and the software developers should understand each other when talking about terms in the domain.

Domain experts have limited understanding of technical terms used in software development and, on the other hand, developers may struggle to understand what the domain experts are saying. Without a common language some meaning will be lost in 'translation' which might lead to problems with the design of the solution.

### Bounded Context

This is a description of a boundary within which a particular model is defined and applicable. In a large system you often find the same name of a model being used in different subsystems. For instance, a `Customer` can be a model used in both the sales subsystem as well as the support subsytem. These two models, though they have the same name, may mean completely diffent things and thus behave differently. If there are no clear boundaries between these subsytems developers may fall into the temptation of using the same `Customer` model in both subsystems which might lead to problems.

There should be a clear seperation between models in different contexts. Even though they may share the same name they usually carry different meanings and thus should be treated as completely different things to allow them to evolve indepent of each other. Points of contact between contexts should then be defined, outlining explicit translation for any communication and highlighting any sharing.

## The Role Of Domain Experts

Domain experts play a significant role in Domain Driven Design. They are the ones who fully understand the problem domain and, in most cases, they are the ones who will use the software. There should be constant communication between developers and the domain experts to iron out all ambiguities and develop a ubiquitous language.

## When To Use Domain Driven Design

Domain Driven Design, in my opinion, can be used in any software application. However, it shines the brightest when you are faced with a problem that has complex domain logic and business rules. Using Domain Driven Design on simple CRUD applications may be an overkill. I'm certainly no expert in DDD so don't take my word for it. This is just the observation I made when I was reading Eric's book.

## Conclusion

In this post I spoke about what I understand about Domain Driven Design after finishing Eric Evans' book on the subject. There are a lot of things I left out such as the use of entities, value objects, services etc. I may talk about them in a later post. The aim of this post was to talk about what DDD is and what it entails. Once again thank you so much for taking your time to read. If you're reading this during the COVID-19 pandemic of 2020 stay safe and observe social distancing.

### Further Reading

- Eric Evans' DDD [book](https://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215)
- Martin Fowler' [blog](https://martinfowler.com/tags/domain%20driven%20design.html) posts
