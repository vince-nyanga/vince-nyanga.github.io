---
title: "Facial Recognition In .NET With Kairos"
date: 2020-05-22
tags: [C#, AI, Library, Open Source]
---

A little over 2 years ago I worked on a project that involved facial recognition. Even though I eventually used Python for the project, I initially wanted to use the .NET stack using Amazon [Rekognition](https://aws.amazon.com/rekognition/) and [Kairos](https://www.kairos.com/) interchangeably. Amazon Rekognition had a nice .NET API that I could use, but not so much for Kairos (at least at that time) so I decided to create a library that wraps the Kairos API. After 2 years of procrastination I eventually did. In this very short post I will talk about how to use [Hones.Kairos.Net](https://www.nuget.org/packages/Hones.Kairos.Net/) -- an open source .NET library for Kairos.

## About Kairos

Kairos is a company that offers facial recognition services that allow developers to integrate facial recognition into their applications through a simple API. Their API offers many features including face detection, face enrolment, face recognition, gender detection, among others. To find out more about the features they offer check out their [website](https://www.kairos.com/features). They have a REST API that you can try [here](https://www.kairos.com/docs/api/).

## Hones.Kairos.Net

This is a simple .NET wrapper for the Kairos API that I wrote. I created this library as a way to learn some things such as Azure Pipelines and Continous Delivery (CD). Talk of killing two birds with one stone. The libary has very simple API, some of which I will talk about. You can install the library from NuGet by running this command if you are using the .NET CLI:

```
dotnet add package Hones.Kairos.Net
```

Before you use the library you need to get create a Kairos Developer account and obtain the credentials you will need to use. Follow the steps outlined [here](https://www.kairos.com/docs/getting-started-with-kairos-face-recognition) to do that. When you have everything ready you can initialize the `KairosClient` class:

```csharp
var client = new KairosClient(APP_ID,APP_KEY);
```

Now let's talk about what you can do with the API.

### Enrolling Faces

When enrolling a face to Kairos, the API takes a photo, finds all faces within it and stores them into a gallery you create. You can do this by calling the `EnrollFaceAsync` method:

```csharp
var response = await client.EnrollFaceAsync(
                            new Uri("[PUBLICLY_ACCESSIBLE_URL]"),
                            "Name of person or any unique ID",
                            "Gallery name");

Console.WriteLine($"FaceId: {response.FaceId}");

foreach (var image in response.Images)
{
    Console.WriteLine($"Age: {image.Attributes.Age}");
    Console.WriteLine($"Gender: {image.Attributes.Gender.Type}");
    Console.WriteLine($"Lips: {image.Attributes.Lips}");
    Console.WriteLine($"Confidence: {image.Transaction.Confidence}");
}

// Or use the Base64 overload
var response = await client.EnrollFaceAsync(
                            (Base64Image)"[BASE64_ENCODED_PHOTO]",
                            "Name of person or any unique ID",
                            "Gallery name");
```

### Verifying Faces

This takes a photo, finds the face within it, and tries to compare it against a face you have already enrolled into a gallery. Call `VerifyFaceAsync` method to do this:

```csharp
var response = await client.VerifyFaceAsync(
                            (Base64Image)"[BASE64_ENCODED_PHOTO]",
                            "Name of person or any unique ID",
                            "Gallery name");

```

The method, like all other methods that upload images, has an overload that takes a publicly accessible url of a photo.

### Recognizing Faces

This takes a photo, finds faces within it, and tries to match them against the faces you have enrolled into a gallery:

```csharp
var response = await client.RecognizeFaceAsync(
                new Uri("[PUBLICLY_ACCESSIBLE_URL]"),
                "Gallery name");
foreach (var image in response.Images)
{
    foreach (var candidate in image.Candidates)
    {
        Console.WriteLine($"{candidate.SubjectId}\tConfidence: {candidate.Confidence}");
    }
}
```

### Detect Faces

Kairos can also be used to detect faces in a photo. This method takes a photo and returns the facial features it finds:

```csharp
var response = await client.DetectFacesAsync(
                            (Base64Image)"[BASE64_ENCODED_PHOTO]");

foreach (var image in response.Images)
{
    foreach (var face in image.Faces)
    {
        Console.WriteLine($"Age: {face.Attributes.Age}");
        Console.WriteLine($"Gender: {face.Attributes.Gender.Type}");
        Console.WriteLine($"Lips: {face.Attributes.Lips}");
    }
}
```

These are the four methods you're going to work with the most, in my opinion, when adding facial recognition to your .NET project.

## Contribution

This library is open source and you can contribute to it on [GitHub](https://github.com/vince-nyanga/Kairos.Net) in whatever way you can. There is still a lot of things to be done before we get to `v1.0.0` so you'll definitely find something to add (or subtract).

## Conclusion

In this post I spoke about [Hones.Kairos.Net](https://www.nuget.org/packages/Hones.Kairos.Net/), an open source library that is a wrapper around the [Kairos](https://www.kairos.com/) API. You're most welcome to contribute to the project if you have a contribution to make. Once again, thank you so much for taking your time to read and stay safe if you're reading this during the 2020 Covid-19 pandemic.
