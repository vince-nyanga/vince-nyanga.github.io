---
title: "How I Achieved 99% Cost Saving After Migrating My Application To Azure"
excerpt: "In this post I am going to talk about how I managed to make huge savings by migrating by applications to Azure."
date: 2022-10-15
tags: [Azure]
---

Almost every year I try to come up with a side project to work on in order to keep my skills as sharp as possible as well as explore new technologies. Most of these projects never see the light of day but I'm extremely proud of my Regulatory Exam (RE) Made Easy applications ([RE 5 Made Easy](https://re5.remadeeasy.co.za/) and [RE 1 Made Easy](https://re1-made-easy.web.app/)) that I developed in 2019 and 2020 respectively. At the time of writing, these apps have combined downloads of over 35,000. I am a super proud dev :smile:. In this post I am going to talk about how I migrated one of the components used by both applications to Azure and saved 99% in costs.

## Admin Portal

I built a separate web application that serves as a back office admin portal. It also has a small user-facing component to it that helps users carry out certain tasks that I didn't expose in the apps. When I started back in 2019, I built the portal using [Flask](https://flask.palletsprojects.com/en/2.2.x/), a Python framework for building web apps and APIs. One of the main reasons for using Flask was to learn something new. Both the apps and the admin portal were built using technologies that I had never worked with before -- [Flutter](https://flutter.dev/) for the mobile apps and Flask for the admin portal. Talk about throwing oneself in the deep-end.

### Initial Architecture

The old admin portal was a simple Flask web application that used PostgreSQL for persistence. The UI component was built with [Jinja](https://jinja.palletsprojects.com/en/3.1.x/) templates that were styled using [Tailwind CSS](https://tailwindcss.com/).

It also used a custom-built authentication/authorization layer (very brave I know) for the back office admin section. I also integrated with [Firebase](https://firebase.google.com/) -- where I handle all my mobile app backend services from auth, image storage, etcetera. Integration with [SendGrid](https://sendgrid.com/) was also added so I could send transactional emails to the users. A simplified diagram of the architecture is shown in **Figure 1** below:

<figure>
<img src="{{ site.baseurl }}/images/old-architecture.svg" alt="Old architecture">
<figcaption>Figure 1: Old architecture</figcaption>
</figure>

This was a simple architecture that enabled me to have a relatively short time to market -- something that was very paramount at the moment. However, it had a few drawbacks:

- **Security**: I'm not a big fan of custom-built authentication/authorization services. I could have opted for an external provider but I didn't want to bite more that I could chew.
- **No segregation**: The same Flask web app served as both the back office admin portal as well as a user-facing portal. The back office portal was protected by the custom auth layer I wrote while the user facing endpoints allowed anonymous access. This did not sit very well with me.
- **Cost**: I hosted the app on Heroku for a total cost of $16.00 per month. That's around R294.00 in South African Rands at the time of writing. This is not an huge amount at all but considering the sporadic usage of both the back office portal and user facing portal, it was not something I was happy with.

### Out With The Old, In With The New

Around April this year (2022) I made up my mind that I was going to migrate the portal to Azure. Initially it wasn't much about the cost at all. I just wanted to get more practical exposure to some Azure services I had learned about during my Azure Developer Associate [certification](https://www.credly.com/earner/earned/badge/68c8fc98-1478-44f7-b940-8e8348d7282b).

Around June/July I set out to come up with an architecture that I was going to use. One of the most important aspect of the architecture was the segregation between the user-facing web app and the back office portal. All of a sudden I was so obsessed with finding an architecture that worked for my needs but for as little cost as possible. Things are tough these days, every cent counts :wink:. **Figure 2** below is what I eventually came up with after a few iterations.

<figure>
<img src="{{ site.baseurl }}/images/architecture.svg" alt="New architecture">
<figcaption>Figure 2: New architecture</figcaption>
</figure>

It looks more complex than the old architecture but wait till you hear how much it costs me now :smile:. The new architecture comprises of the following components:

- Two [Angular](https://angular.io/) web apps -- one user facing and one for back office operations. The websites are hosted in Azure [storage accounts](https://learn.microsoft.com/en-us/azure/storage/common/storage-account-overview). However, instead of hitting the storage accounts directly to serve the websites, I added [Azure CDN](https://learn.microsoft.com/en-us/azure/cdn/cdn-overview) endpoints that cache the static files and efficiently deliver the websites to the users. This helps to reduce read costs on my storage accounts. To even save more money, I could change the access tiers of the storage accounts from Hot to Cold since the blobs are not going to be accessed frequently. Keep your eyes open for some posts on how to do this as well as how to automate the deployment and CDN purging.
- The backend logic is spread across multiple Azure [functions](https://learn.microsoft.com/en-us/azure/azure-functions/functions-overview). I used Azure [API management](https://learn.microsoft.com/en-us/azure/api-management/api-management-key-concepts) as a gateway API to consolidate all my functions and manage authentication and role based access control (RBAC) using Azure [Active Directory](https://learn.microsoft.com/en-us/azure/active-directory/fundamentals/active-directory-whatis) for back office endpoints, and subscription keys for Firebase functions that are used to synchronize the data I need in some of my Azure functions. I chose the Consumption plans for both the Azure functions and the API management due to the sporadic usage of my services. I have to pay a penalty of cold starts for my functions but it's something I can live with. Posts on how to set up RBAC with Azure AD in the API management service are in the pipeline.
- Azure [Table storage](https://learn.microsoft.com/en-us/azure/storage/tables/table-storage-overview) is used for data storage. I chose it because it is very affordable and my data models as well as the queries I run on them are pretty straight forward.
- Some of the Azure functions publish events to Azure [Event Grid](https://learn.microsoft.com/en-us/azure/event-grid/overview) topics to which an Azure function responsible for sending out emails subscribes.

So how much does the new architecture cost you may ask. After working on the migration for a couple of weekends in July/August, I eventually switched off the Flask app on Heroku at the beginning of September and the new applications were up and running in Azure. At the end of the month the invoice came. The entire system cost me a whopping **$0.16** or **R2.94** in South African Rands. That's exactly 1% of what the old architecture cost me per month. I'm pretty chuffed about the 99% savings I achieved. I'm still yet to decide what to do with all that money but chances are I'm going to use it to fund another side project that I have in mind.

## Conclusion

In this post I spoke about how I managed to save 99% after migrating one of my applications to Azure. Like always, thank you so much for taking your time to read. Don't hesitate to leave a comment, question or suggestion in the comments section below.
