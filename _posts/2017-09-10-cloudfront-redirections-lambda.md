---
category: posts
layout: single
mainTopic: 'programming'
title: "CloudFront redirections for your SPA using AWS Lambda (A/B testing, maintenance page,...)"
header:
  image: "http://codeandtechno.com/images/lambda-post/cloudfront-map.png"
---

For many people, the stack of choice for deploying a JavaScript application (SPA) or other assets is to save them to Amazon S3 and serve it to the world over Amazon CloudFront CDN. With its cached edges around the world, it makes sure the users’ browsers are able to download your application in the most efficient way possible.

<figure>
    <a href="/images/lambda-post/cloudfront-map.png"><img src="/images/lambda-post/cloudfront-map.png"></a>
    <center>
    <figcaption>At the time of writing, CloudFront offers around 100 edge locations around the globe.</figcaption></center>
</figure>

A possible drawback of this application/server architecture is that your JavaScript application or other content is cached in various cache servers placed globally, and that seems problematic in cases where you want to suddenly place a redirection switch - serving your users something else, depending on some condition or request. You’re not in complete control of your server(s), and cache invalidations and DNS changes could take forever, especially in urgencies.

One of these cases is showing a “maintenance site”. You know - those “We’ll be back shortly.” notices, when engineers are either sweatingly salvaging a botched back-end or just performing regular, scheduled maintenance.

A popular technique in SPAs is to detect these adversities by listening for `5xx` responses from the API server and rendering a “Maintenance” message. It’s a valid approach, but this comes with a couple of assumptions which make the problem detection unreliable.

 Two major assumptions:

- JavaScript application works OK,
- Server is responding with HTTP code `5xx`.

But, what if your build system deploys a broken bundle which doesn’t work in a browser anymore (or maybe it breaks only in specific browsers)? What if your async API requests are timeouting and you don’t get any response?

In these cases it’s best to have a manual switch **on the server level** that puts your website into maintenance mode while you do the fixing.

With CloudFront you’re able to achieve this by using [Lambda@Edge](http://docs.aws.amazon.com/lambda/latest/dg/lambda-edge.html) functions (an extension of AWS Lambda) in the Viewer Request trigger, which triggers BEFORE the CloudFront cache is checked and returned. Within this function you could do many different things, for example:

- Inspect cookies, User-Agent, headers,...
- Do HTTP redirections, personalize and generate responses depending on location and various other criteria,
- Make network calls to external resources (e.g., authorization).

Returning to the maintenance page switch, we could write a Lambda function like this:

```javascript
exports.handler = (event, context, callback) => {
  const response = {
    status: '302',
    statusDescription: 'Found',
    headers: {
      location: [{
        key: 'Location',
        value: 'https://my-site.com/maintenance.html',
        }],
    },
  };
  callback(null, response);
};
```

When we want to put this redirection to work, we simply assign its ARN to our CloudFront distribution in its behaviour settings:

<figure>
    <a href="/images/lambda-post/cloudfront-lambda-redirect.png"><img src="/images/lambda-post/cloudfront-lambda-redirect.png"></a>
    <figcaption>CloudFront distribution behaviour settings.</figcaption>
</figure>

… and remove it once you don’t want the redirection anymore. The change is usually processed within a very short period of time (which is usually not the case when saving adjustments in CloudFront distributions).

This mechanism can be used for a lot of other practical implementations, like A/B testing redirections, logging and handling web client specifics.
