---
title: "Creating UML Diagrams With Ease Using PlantText"
date: 2020-07-25
tags: [General]
---

Unified Modeling Language (UML) has been around the software development circles for a fairly long time now. It helps provide a standardised way to visualise the architecture, design, and implementation of complex software systems. I get mixed reactions from fellow engineers when I ask whether or not they like (or use) UML. Personally, even though I don't like to use it as frequently, I have found it very handy in certain instances. I use it mostly in personal projects to keep a snapshot of my design decisions especially since it might be months before I revisit the project.

This short post is not about whether or not one should use UML. I want to talk about a tool I bumped into that makes creation of UML diagrams a breeze -- [PlantText](https://www.planttext.com/).

## What Is PlantText?

PlantText is a tool that helps you create UML diagrams of various types using code. I find it quite handy since I can quickly create a class diagram using something that I understand so well -- code. When you're done creating your diagram you can generate a `PNG` or `SVG` file, or you can download the code as text.

What makes me like this a lot is that I can version control my designs together with my software project and quickly update them when the need arises.

## Example

Let's create a simple UML class diagram using this amazing tool. Visit the PlantText [website](https://www.planttext.com/) and paste the following code in the text editor:

```csharp
@startuml

title Example Class Diagram (v1)

interface SuperHero{
 Name: string;
 DoHeroStuff(): void
}

class SuperMan implements SuperHero{
}

class SpiderMan implements SuperHero{
}

class Vinarah implements SuperHero{
}

class VinarahSon extends Vinarah{
 DoKidStuff(): void
}

@enduml
```

As you can see, we have a simple `SuperHero` interface that has three classes implementing it. We also have the `VinarahSon` class that inherits from `Vinarah`. This is something a software engineer can easily understand. You can now take this code and push it to your version control repository so that you can update it as you go. The bonus for me is the visual image that gets generated as shown below.

<figure>
<img src="{{ site.baseurl }}/images/sample-uml.png" alt="UML Diagram">
<figcaption>UML class diagram created using PlantText</figcaption>
</figure>

There you have it -- a UML class diagram generated from code.

## Conclusion

In this post I briefly spoke about [PlantText](https://www.planttext.com/), a web tool that allows you to generate UML diagrams of various types -- class diagrams, sequence diagrams etcetera, using code. I've been using this tool for quite some time and decided to share it with someone who might not have encountered it and is looking for something similar. Like always, thanks so much for taking time to read and, if you're reading this during the 2020 Covid-19 pandemic, please stay safe.
