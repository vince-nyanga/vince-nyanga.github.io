---
title: "Week 1: Aug 1 2020"
date: 2020-08-01
---

This week I learnt a lot of things but I'm just going to talk about the few that stood out for me. Here we go:

## 1. CSS Flexbox

Ever since my days in school CSS has been my worst nightmare and as a result, I haven't been a big fan of it. In my career so far I have been focusing much on backend development and a bit of mobile development because there's no CSS involved in these two :new_moon_with_face:. When I changed jobs a couple of weeks ago I quickly realised that in addition to backend and mobile development, I needed to also know a bit of CSS so I decided to find the quickest path possible to being productive even though I'm not a fan of it.

After a quick search I stumbled upon CSS Flexbox -- a powerful way of creating flexible layouts with CSS. You'd be very surprised to find out that I didn't know of the existence of flexbox till about seven days ago. It shows how far I tried to stay away from CSS. Since most of my struggles with CSS have been positioning things on the screen, the knowledge of flexbox was a game changer for me and I was able to make a meaningful contribution to the project we're working on. Now whenever I see a CSS problem my first thought is 'how can I use flexbox to do this' because that's as far as I know in the CSS world. That's ok though, at least I can now do certain things that I dreaded a week ago. Here are the two courses that I took in the process:

1. **CSS Flexbox Fundamentals** by Gary Simon on [Pluralsight](https://app.pluralsight.com/library/courses/css-flexbox-fundamentals-2319/table-of-contents)
2. **Creating Responsive Pages with CSS Flexbox** by Jeff Batt on [Pluralsight](https://app.pluralsight.com/library/courses/css-flexbox-creating-responsive-pages/table-of-contents)

## 2. HTTP URL Merging

This is a little bit embarrassing for me since I've worked with the `HttpClient` for a while now. Earlier this week I wanted to make a call to a web service so I did what I always do:

```csharp
var client = new HttpClient
{
    BaseAddress = new Uri("https://my-api.com/sub-path")
};

var response = await client.GetAsync("/v1/blabla");
```

When I tried to run this I got a `503 Service unavailable` error :ok_man:. What just happened? After digging through the logs my colleague and I realised that instead of calling `GET https://my-api.com/sub-path/v1/blabla` I was calling `GET https://my-api.com/v1/blabla` so we figured the `/sub-path` section was getting dropped for some 'mysterious' reason. I then made a quick fix which worked:

```csharp
var client = new HttpClient
{
    BaseAddress = new Uri("https://my-api.com")
};

var response = await client.GetAsync("/sub-path/v1/blabla");
```

But wait, there are instances where I need to make the same call but without the `sub-path` section so the only way to do it is to add it in the base address of my client. I quickly realised that I needed to go back to my first implementation and somehow make sure that `sub-path` wasn't gonna get dropped when I make the call.

I then investigated why my `sub-path` was getting dropped and my investigation led me to the [Uniform Resource Identifier (URI): Generic Syntax](https://tools.ietf.org/html/rfc3986) RFC page and this [section](https://tools.ietf.org/html/rfc3986#section-5.2.3) in particular. That section explains how merging paths work. To summarise, if your base address has a segment in it like the one in the example, it **must** end with a `/` otherwise everything after the right-most `/` will be excluded when paths are merged. Eureka!! there was my solution. All I needed to do was add a trailing `/` to my base address and remove the leading `/` from my `GET` call like this:

```csharp
var client = new HttpClient
{
    BaseAddress = new Uri("https://my-api.com/sub-path/")
};

var response = await client.GetAsync("v1/blabla");
```

This is something I did not know because in every instance prior to this week all base `URL`s didn't have an extra segment in them.
