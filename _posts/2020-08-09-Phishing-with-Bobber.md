---
layout: post
title: Phishing with Bobber
---

One of the challenges associated with running a successful phishing campaign is maintaining Operations Security (OpSec). A common approach to maintaining red team OpSec is to put your team's assets (CobaltStrike server, phishing application server, etc) behind redirectors, which enables traffic from compromised assets or users to communicate with your infrastructure, while simultaneously throwing off automatic defenses and some defenders.

I recently saw a GitHub project from Jesse Nebling (@bashexplode) called [otu-plz](https://github.com/bashexplode/otu-plz) which generates PHP that can be emdedded in a phishing application and keeps track of which users load the phishing application. From here, the application will ensure that the target payload is delivered only to users with the specified ID, and that other downloads will result in a bogus payload. This project inspired me to explore how a Flask application can be used to handle traffic on-the-fly for arbitrary websites while applying the same logic of one-time access.

---

## What is bobber?

As stated above, bobber is a Python Flask application that is meant to be an interface to an arbitrary phishing application, but without having to implement any code. To secure your application, only traffic with a specified GET argument is allowed through to the phishing app, and the specified GET argument must have a value that is whitelisted in a token file. Additionally, once that token has been presented to the Flask app, a timer begins that will remove that token's access after a specified amount of time, which enables the end user to browse through the phishing application, but also likely expiring the token before a response team can re-use the phishing URL.

To summarize - traffic is only forwarded to the phishing application if:
1. There is a specific token provided (via GET) that matches a specified key
2. The value of the token matches a token within the token whitelist
3. The token has not expired after being used for the first time

If any of these conditions fail, the Flask app will load content from a site of your choosing and serve that content to the end user under the current URL. These layers are steps to ensure that only the intended target user logs into the application; should a clicked link be forwarded to an investigator, access to the phishing application entirely becomes denied.

Unfortunately, content-heavy sites (like YouTube and Reddit) are unable to load for the time being, but most other small websites have loaded well. As a rule of thumb, if a standard site cloning utility won't capture the dynamic content efficiently, bobber shouldn't be relied on to load it efficiently either.

Here's an a short GIF demonstration of what using bobber looks like:

![Bobber Demo](/images/bobber.gif)

---

## How should I use bobber?
Bobber is most effectively used as a redirecting server that stands in front of a phishing application. If used properly, there should be little to no changes needed for the underlying phishing application - simply update the bobber.py script and launch the flask application.

Additionally, while preventing unathorized users from accessing the phishing app, bobber's use case of serving up legitimate content may be useful in performing domain categorization activities, as bobber serves up functional content from the redirected domain and performs basic checks to ensure links on the legitimate page are re-routed through bobber.

---

## Final thoughts
This tool is a quick concept script that shows how easy it is to leverage a framework like Flask to perform granular controls on intercepted traffic, taking advantage of the ability to run custom code on traffic redirection activities. Leveraging this functionality may make for interesting C2 redirectors in the future depending on what ideas are implemented.

Feel free to fork this project and make any desired changes - just please make sure to credit me for work I've done here if you use it.

---

## Repo

The bobber repo [can be found here.](https://github.com/jfulling/bobber)
