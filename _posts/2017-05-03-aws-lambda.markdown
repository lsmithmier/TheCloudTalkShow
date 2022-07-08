---
layout: post
title:  "Episode 3 – AWS Lambda, itty-bitty shared computing on the cloud for pennies …"
date:   2017-05-03 10:00:00 -0400
categories: aws lambda
---
{% include youtubePlayer.html id="CCkRwyBSRaM" %}
Welcome to another episode of Cloud Talk!  So Larry decides to splurge and dive into his AWS account and show us one of the key things the cloud is for … computing (duh, computers, right?)   But there is a lot here so Larry is nice enough to blog about it in detail which follows here in lieu of our regular credits.

And now … Larry Smithmier!

---

In show #3 I showed how to build, deploy, and run an Amazon Lambda function.  In the video I mentioned that it was cheap to run and promised to break down the costs.  The answer surprised me!

First, let’s break down how Lambda functions are billed.  The amount you are billed for a call is related to how long it takes the call to complete times the amount of memory you reserve for the request.  The memory comes in 64MB chunks, 2 chunk minimum.  The duration is measured from the time your code begins to execute until it terminates, rounded up to the nearest 100ms.

Here is the breakdown for the call on the show:

- The Lambda function was created with a 128MB memory footprint
- It ran for 1856ms which gave a billed duration of 1900ms (it rounds up to the nearest 100ms) or 1.9 seconds.

Looking at the chart of Lambda pricing I find that a 128MB function costs $0.000000208 per 100ms of run time (yes, I counted the 6 zeros several times) for a total run cost of $0.000003952 or 4 thousandths of a cent.  But wait!  There’s more.  There is a charge of $0.0000002 per request after the first million requests.  Everyone using Lambda functions get a Free Tier buffer of one million free requests per month and 400,000 GB-seconds of compute time.  We will need to do a lot more demos to get close to that amount!

---

Thanks, Larry … for you folks at home, please post your favorite links to using your favorite itty bitty compute services (Azure Functions, Google, etc., etc.)  and we will see you next time!