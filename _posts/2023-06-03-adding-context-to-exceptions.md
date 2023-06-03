---
title: "Adding More Context To Exceptions In C#"
excerpt: "In this post, I will show you how to use the Data property of the Exception class to add more context to exceptions."
date: 2023-06-03
tags: [.NET, C#]
---

There are times when you want to add more context to an exception besides the exception message. Or in case of validation, you might not want to throw an exception for every validation error. Instead, you want to collect all the errors and throw a single exception. One way to solve this is to add extra properties to your exception class. How about we do it without needing to add custom properties? It can be done, and I will show you how :smile:. In this very short post, we'll look at how to use the `Data` property on the `Exception` class to add more context to exceptions. The `Data` property is a dictionary that can be used to store additional information about the exception. Let's look at a validation example.

We are going to create a `PersonValidationException` class that inherits from `Exception`. This class will have a `Data` property that we can use to store validation errors. We'll also add a `ThrowIfErrors` method that will throw the exception if there are any errors:

```csharp
public sealed class PersonValidationException : Exception
{
    public PersonValidationException()
        : base("Person validation failed.")
    {
    }

    public void AddError(string key, string message)
    {
        Data.Add(key, message);
    }

    public void ThrowIfErrors()
    {
        if (Data.Count > 0)
        {
            throw this;
        }
    }
}
```

Next, we'll create a `Person` class that will have two properties: `FirstName` and `LastName`. We'll add a constructor that will validate the properties and throw a `PersonValidationException` if there are any errors:

```csharp
public sealed class Person
{
    public string FirstName { get; }
    public string LastName { get; }

    public Person(string firstName, string lastName)
    {
        EnsureValidDetails(firstName, lastName);

        FirstName = firstName;
        LastName = lastName;
    }

    private static void EnsureValidDetails(string firstName, string lastName)
    {
        var exception = new PersonValidationException();

        if (string.IsNullOrWhiteSpace(firstName))
        {
            exception.AddError(nameof(FirstName), "First name is required.");
        }

        if (string.IsNullOrWhiteSpace(lastName))
        {
            exception.AddError(nameof(LastName), "Last name is required.");
        }

        exception.ThrowIfErrors();
    }
}
```

We can now catch one exception and get all the validation errors.

```csharp
try
{
    var person = new Person(null, null);
}
catch (PersonValidationException exception)
{
    foreach (DictionaryEntry entry in exception.Data)
    {
        Console.WriteLine($"{entry.Key}: {entry.Value}");
    }
}
```

This is one of the ways you can use the `Data` property to add more context to exceptions. You can use it in other scenarios as well. For example, adding more context to an exception before re-throwing it. Please note that the `Data` property is not secure so you shouldn't add sensitive information to it.

## Conclusion

In this post, we looked at how to use the `Data` property of the `Exception` class to add more context to exceptions. We also looked at a validation example where we used the `Data` property to collect all the validation errors and throw a single exception. That's it for now. Do not hesitate to leave a comment, question or suggestion below. Till next time, happy coding!
