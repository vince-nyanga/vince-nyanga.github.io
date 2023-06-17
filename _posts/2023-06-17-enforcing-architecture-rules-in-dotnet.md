---
title: "Enforcing Architecture Rules In .NET"
excerpt: "In this post, I will show you how you can write tests to enforce architecture rules in .NET."
date: 2023-06-17
tags: [.NET, C#, Testing]
---

Every green field project starts out with the best intentions. The architecture is well thought out, and the code is clean. But as the project grows, the architecture starts to degrade. Inconsistencies start to creep in. The code starts to rot. The architecture starts to look like a big ball of mud. This is where architecture enforcement comes in. In this post, I will show you how you can write tests to enforce architecture rules in .NET.

I am going to talk about two libraries that you can use to enforce architecture rules in .NET: [ArchUnitNET](https://archunitnet.readthedocs.io/en/latest/) and [NetArchTest](https://github.com/BenMorris/NetArchTest). Both libraries are inspired by the [ArchUnit](https://www.archunit.org/) library for Java. They both allow you to write tests to enforce architecture rules in .NET. Let's look at how to use them.

## Example

I am going to write tests using both libraries to enforce a few rules, including custom rules. I have both libraries installed in my project. First we are going to load the assemblies that we want to test. It is recommended that you do it once for performance reasons.

```csharp
 // Loading the architecture from the test for ArchUnitNET
private static readonly Architecture Architecture =
    new ArchLoader().LoadAssemblies(typeof(Program).Assembly).Build();

// Loading the types from the current test for NetArchTest
private static readonly Types Types = Types.InAssembly(typeof(Program).Assembly);
```

Now we can write tests to enforce some rules:

1. All classes in the `Brokers` namespace should be internal and sealed:

```csharp
// using ArchUnitNET
[Fact]
public void BrokersShouldBeSealedAndInternalV1()
{
    // given
    var brokerClasses =  Classes().That().ResideInNamespace("ArchitectureTestSample.Brokers")
        .As("Brokers");

    // when
    IArchRule rule = Classes().
        That().Are(brokerClasses)
        .Should().BeInternal()
        .AndShould().BeSealed();

    // then
    rule.Check(Architecture);
}

// using NetArchTest
[Fact]
public void BrokersShouldBeSealedAndInternalV2()
{
    // given
    var conditions = Types.That()
        .AreClasses()
        .And().ResideInNamespace("ArchitectureTestSample.Brokers")
        .Should().BeSealed()
        .And().NotBePublic();

    // when
    var result = conditions.GetResult();

    // then
    AssertConditionsAreMet(result, "brokers should be sealed and internal");
}
```

2. Classes in the `Services` namespace should have dependencies only on classes in the `Brokers` namespace:

```csharp
// using ArchUnitNET
[Fact]
public void ServicesShouldDependOnBrokersV1()
{
    // given
    var serviceClasses = Classes().That().ResideInNamespace("ArchitectureTestSample.Services")
        .As("Services");

    // when
    IArchRule rule = Classes().
        That().Are(serviceClasses)
        .Should().DependOnAnyTypesThat().ResideInNamespace("ArchitectureTestSample.Brokers");

    // then
    rule.Check(Architecture);
}

// using NetArchTest
[Fact]
public void ServicesShouldDependOnBrokersV2()
{
    // given
    var conditions = Types.That()
        .AreClasses()
        .And().ResideInNamespace("ArchitectureTestSample.Services")
        .Should().HaveDependencyOn("ArchitectureTestSample.Brokers");

    // when
    var result = conditions.GetResult();

    // then
    AssertConditionsAreMet(result, "services should depend on brokers");
}
```

3. We are going to write custom rules for both libraries to enforce the maximum number of constructor parameters. For ArchUnitNET, we need to create a class that implements the `ICondition<Class>` interface:

```csharp
internal sealed class MaximumConstructorParametersCondition : ICondition<Class>
{
    private readonly int _maximumParameters;

    public string Description => $"should have no more than {_maximumParameters} constructor parameters";

    public MaximumConstructorParametersCondition(int maximumParameters)
    {
        _maximumParameters = maximumParameters;
    }

    public IEnumerable<ConditionResult> Check(IEnumerable<Class> objects, Architecture architecture)
    {
        foreach (var @class in objects)
        {
            var constructors = @class.GetConstructors().ToList();

            if (constructors.Count == 0)
            {
                yield return new ConditionResult(@class, pass: true);
            }

            if (constructors.Any(x => x.Parameters.Count() <= _maximumParameters))
            {
                yield return new ConditionResult(@class, pass: true);
            }

            yield return new ConditionResult(@class, pass: false, failDescription: $"has a constructor with more than {_maximumParameters} parameters");
        }
    }

    public bool CheckEmpty() =>
        true;
}
```

Now we can write a test to enforce the rule:

```csharp
[Fact]
public void AllClassesShouldHaveMaximumOfThreeConstructorParametersV1()
{
    // given
    var maximumConstructorParametersCondition = new MaximumConstructorParametersCondition(3);

    // when
    IArchRule rule = Classes().Should()
        .FollowCustomCondition(maximumConstructorParametersCondition);

    // then
    rule.Check(Architecture);
}
```

Next, we create a custom rule for NetArchTest. We need to create a class that implements the `ICustomRule` interface:

```csharp
internal sealed class MaximumConstructorParametersRule : ICustomRule
{
    private readonly int _maximumParameters;

    public MaximumConstructorParametersRule(int maximumParameters)
    {
        _maximumParameters = maximumParameters;
    }

    public bool MeetsRule(TypeDefinition type)
    {
        var constructors = type.Methods.Where(x => x.IsConstructor).ToList();

        return constructors.Count == 0 || constructors.All(x => x.Parameters.Count <= _maximumParameters);
    }
}
```

And the test to enforce the rule:

```csharp
[Fact]
public void AllClassesShouldHaveMaximumOfThreeConstructorParametersV2()
{
    // given
    var maximumConstructorParametersRule = new MaximumConstructorParametersRule(maximumParameters: 3);
    var conditions = Types.That().AreClasses()
        .Should().MeetCustomRule(maximumConstructorParametersRule);

    // when
    var result = conditions.GetResult();

    // then
    AssertConditionsAreMet(result, "all classes should have maximum of three constructor parameters");
}
```

## Comparing The Two Libraries

These two libraries are virtually identical. Picking one over the other is a matter of preference so I would suggest that you play around with both of them and see which one you like the most. I personally, I don't mind using either of them.

## Conclusion

That's it. In this post, I showed you how you can write tests to enforce architecture rules in .NET. using two very powerful libraries. You can find the code for this post on [GitHub](https://github.com/vince-nyanga/architecture-test-sample). If you have any questions or comments, please leave them in the comments section below. Until next time, happy coding :smile:.
