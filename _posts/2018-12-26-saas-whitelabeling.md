---
category: posts
layout: single
mainTopic: 'programming'
title: "What is SaaS white labeling and how to do it?"
header:
  image: "http://codeandtechno.com/images/whitelabel-post/whitelabeled-bottles.jpg"
---


Depending on your online business, white labeling can turn out to be a fantastic business model. This applies especially when your target customers are actually <b>customers who have their own customers</b>, for example agencies, consultancies and client-oriented businesses.

In this case there is a real chance that adding support for white labeling in your product will improve your chances of attracting advanced users and position your product higher in the eyes of corporate customers (i.e. those who can pay top dollar).

### But what exactly is white labeling?


White labeling means taking a blank product and slapping a label for another company on it.

<figure class="half">
    <a href="/images/whitelabel-post/whitelabeled-bottles.jpg"><img src="/images/whitelabel-post/whitelabeled-bottles.jpg"></a>
    <center>
    <figcaption></figcaption></center>
</figure>

It means other companies can take your (online, in this case) product and brand it as their own.

### White labeled SaaS example

Let's imagine you built a website performance tracker that makes reports of websites' speed. On one hand, you have individual customers that use your service to track their own websites. On the other hand, there are many web performance consultancies and agencies who make a living tuning *other* people’s (i.e. their customers') websites. To deliver reports, they have 3 options:

1. Build their own website tracking solution. This is expensive to build, and expensive to maintain. Not all businesses are in a position to afford and maintain it.
2. Use a 3rd party solution (like yours). Their clients will notice they are relying on an external service, which may impair their trust.
3. Use a 3rd party solution (like yours), but brand it (white label) as their own. Their clients will think the reporting solution (which is actually yours, but branded differently) is an integral part of their service, which may make them look better in their clients' eyes.

### White labeling settings in SaaS

White labeling should be approached the same way as <b>internationalization</b>. The principle is the same. Configurations can be saved in asset folders or in a database, and conditionally shown depending on the user and their settings.

Here is an example of how we did these settings at <a href="https://nightwatch.io" target="_blank">Nightwatch.io</a>:

<figure>
    <a href="/images/whitelabel-post/whitelabel-settings-example.png"><img src="/images/whitelabel-post/whitelabel-settings-example.png"></a>
    <center>
    <figcaption></figcaption></center>
</figure>


### White label (sub)domain

This is the entry point to a white labeled SaaS product.

Your customers who wish to offer your product branded as their own will want to customize its URL.

The DNS part is easy, usually solved with a *CNAME* record. Returning to our previous example, let's suppose that your SaaS product URL is *WebPerformanceTracking.com*. Let's suppose that one of your customers, who runs their consultancy at WebPerfCompany.com wants to white label your service and make it accessible at *tracker.WebPerfCompany.com*.

What you can do is set up a *whitelabel.WebPerformanceTracking.com*, point it to your server and instruct your customer to set up a CNAME record on their *tracker.webperfcompany.com* subdomain. This will route the requests to their white labeled service back to your server.

### There’s a problem though

<figure>
    <a href="/images/whitelabel-post/page-not-secure.png"><img src="/images/whitelabel-post/page-not-secure.png"></a>
    <center>
    <figcaption></figcaption></center>
</figure>

The described connection to your customer’s white labeled service is unsecure (non-https). Since browsers are complaining about it more and more, and you don’t want to break clients’s trust, it only makes sense to secure it.

To make secure connections from your customers' white labeled domains to your server, you have two options.

1. Convince your customer to send you a cetificate for the white labeled domain and use it on your server. This is not really convenient nor desirable, because it’s a hard to maintain, hard to scale, needs a lot of support and it makes you a keeper of foreign certificate, which brings extra burderns.
2. Use or build a proxying service with a middleware or CDN that creates Let’s Encrypt certificates on the fly.

If you want hassle free whitelabeling, then you want to go with option 2. Depending on your scale, you probably want to use a 3rd party solution for that.

How this usually works is that your customer still has to create a CNAME record, but there is an additional step you have to make. You have to report the white labeled domain to your selected proxying service via their API so they could setup the certificate in the background and do the necessary routing.

CloudFlare is probably the simplest service you can use to secure white labeled connections, but at the time of writing, it’s available only as a part of an [expensive corporate plan](https://www.cloudflare.com/saas/). Alternatively, you can use CDN services, like [Fly.io](https://fly.io/) or [Zeit](https://zeit.co) to deploy a proxying app which responds to secure whitelabel connections and serves data from your origin server.

### Final step

When you reached the stage where connections from white labeled domains are succesfully hitting your server, it’s time to write some application logic that matches a hostname against a white labeled account and decorates the requests with the info on which white labeled account it belongs to.

When you have this information for every request, you can easily use it inside your application interface to show white labeled branding, e.g. custom design, logos, emails, texts and other customly branded pieces of the app.
