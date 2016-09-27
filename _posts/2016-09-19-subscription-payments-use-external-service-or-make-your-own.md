---
category: posts
layout: single
title: "Recurring subscription payments: use external service or make your own?"
excerpt: "Billing is far more complex than people tend to believe, and much more so with subscriptions."
---

Billing is far more complex than people tend to believe, and much more so with subscriptions.

<blockquote class="twitter-tweet tw-align-center" data-lang="en"><p lang="en" dir="ltr">Remind me to never make a subscription service again.</p>&mdash; Jeffrey Biles (@JeffreyBiles) <a href="https://twitter.com/JeffreyBiles/status/775371558768390144">September 12, 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>
<center><small>I feel your pain, Jeffrey. It's hard to get it right.</small></center><br>

Subscriptions are a common business model in paid online products, especially in SaaS world. That means the customers are charged in a recurring fashion; most commonly monthly or yearly.

After you convinced your customer to enter their payment vehicle (credit card, PayPal, etc), there are two ways you can trigger acquiring currency when their payment is due:

1. Use an external billing service ([Braintree](https://www.braintreegateway.com/), [Recurly](https://recurly.com/product/), [Stripe](https://stripe.com), [Chedargetter](https://cheddargetter.com/),...) and let it do **all** the billing work. Let it charge your customers on a recurring basis and send you event notifications, which you use to update your system - prolong or expire customers' subscription. Simple, fast, reliable, but severely limited in customization as your business needs grow.
2. Also use one of the billing services*, but only manually - implement your own payment scheduler (usually with a cron job or a run loop) and bill the customers using your own server. Periodically use the payment service's API to bill customers, and handle all the results yourself. It's more complex and bug prone, takes more time to implement, but you’re in complete control and know exactly how your billing system works, especially when something breaks.
<br><br><span>*</span> <small>assuming you’re offloading PCI responsibility to a payment processor by only storing customer tokens on your server instead of raw payment method data. </small>
&nbsp;

So, what does this recurring work actually include? The most basic flow is this: every month, **determine** payment amount depending on customer’s subscription plan, **charge** the amount using their payment method, **update** their subscription in your application's database, **send** an invoice.

## You need to decide what kind of billing mechanics your business needs.

<span style="margin-top: 20px; display: inline-block;">Let’s compare both ways a little bit more in depth.</span>

Using option 1, you populate the external service with your subscription plans, add-ons, coupons and perhaps configure trial periods. When customers subscribe to your product, your application registers them to the billing service. It takes care of deducting your customers' money every month/year and sending it to your wallet. For every payment activity (successful charge, declined card, etc), they send an event notification, usually as HTTP requests to your API - also known as "reverse API". You use these notifications to extend or halt your customers’ subscriptions.

**The main benefit here is that you spend very little time developing a billing system.**

When you are just starting out, or when you’re certain your billing dynamics aren’t going to be very diverse, implementing your own billing system is most probably a waste of time. This way you usually also get some basic billing analytics with it, which is really useful for monitoring customer and revenue churn rates.

## As your business grows, things can quickly get complicated.

<span style="margin-top: 20px; display: inline-block;">Consider diverse subscription plans, coupons and add-ons with variable durations and prices depending on various conditions, multi-currency, multi-tenancy and dunning.</span>

Let's list some examples.

What if...

- a customer decides to upgrade from a monthly to a yearly subscription in the middle of the month, and you have some custom rules for a pro-rate refund?
- a coupon is valid only for an upgrade to a larger subscription?
- there is a second trial period for some customers?
- an add-on is cheaper on a larger subscription plan?
- there is a special discount for people paying with a debit visa card?

See the pattern?

In cases like this, you can’t escape having a custom tailored business logic for determining the amount to charge. As your pricing plans become more convoluted with discounting, optional add-ons and conditionals it becomes hard not just to bill, but also to keep track of activities, ordering, and reporting.

<center><img src="http://i.imgur.com/cLU8h3D.png" alt="RankTrackr Billing" title="RankTrackr billing" style="width: 520px; margin-top: 20px;"></center>
<center><small>At <a href="http://ranktrackr.com">RankTrackr</a> we built a custom billing service. It runs every day at 10AM and shows the results in Slack. </small></center>

<br>
<span style="margin-top: 15px; display: block;">Subscription payments are one of the most delicate parts of your online business, so do your homework, because migrating to a different subscription payment implementation is distressing and expensive.</span>
