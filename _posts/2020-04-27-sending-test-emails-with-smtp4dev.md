---
title: "Sending Test Emails Using smtp4dev"
date: 2020-04-27
tags: [Other]
---

Many applications nowadays send emails to their users for a wide range of reasons -- email confirmation, password reset, etc. It is inevitable that as a developer you will have to test that the emails are being sent. Sending emails to real users in development mode may have many implications, from accidentally annoying your clients to incurring huge costs from email API. That it where `smtp4dev` comes in.

`smtp4dev` is a dummy SMTP server that runs on Windows, Linux and Mac OS-X. It allows you to send test emails in your application without sending them to your real customers and eliminates the need to set up a real email server.

## How To Run It

There are various ways to run `smtp4dev` but I am going to talk about two of them.

### Running As A Dotnet Global Tool

If you have the .NET Core SDK 3.1 or greater installed on your computer, you can install `smtp4dev` as a global tool using the following command:

```
dotnet tool install -g Rnwood.Smtp4dev --version "3.1.0-*"
```

And then start running it using this command in your terminal:

```
smtp4dev
```

### Running In Docker

This is my favorite way of running `smtp4dev`. Just run the following command:

```
docker run -p 3000:80 -p 25:25 rnwood/smtp4dev:v3
```

The web interface where you can view the emails will run on `localhost:3000` and your SMTP server on port 25. You can change these ports if you so wish.

## Sending Email In C

Now that you have `smtp4dev` up and running you can easily send emails using the `System.Net.Mail` API:

```csharp
public class LocalSmtpEmailService : IEmailService
{
    public async Task SendEmailAsync(string emailAddress, string subject, string message)
    {
        using(var client = new SmtpClient("localhost"))
        {
            var mailMessage = new MailMessage("no-reply@myapp.com",emailAddress,subject, message);

            await client.SendMailAsync(mailMessage);
        }
    }
}
```

That's it! You can now be rest assured you are not going to spam your real users while testing your applications.

There are other local SMPT services out there and Papercut is one of the popular ones. You may want to check them out to see which one works best for you.

### Further reading

- `smtp4dev` [GitHub repository](https://github.com/rnwood/smtp4dev)
- Papercut [GitHub repository](https://github.com/ChangemakerStudios/Papercut-SMTP)
