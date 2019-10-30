---
title: SameSite Cookies! Coming to break your website Feb2020.
date: 2019-10-30
---
Hey friends!

Today i wanted to talk about the SameSite cookie attribute, and how the browser flip from Off to On by default is well. .. kind of going to **screw up the internet** internet in Australia in Feb 2020.

## What is SameSite

_SameSite_ is a cookie security attribute that was introduced to address cross-site request forgery. A cookie is a piece of data sent from a website and stored in the web browser. Cookies were designed to maintain state when the web was becoming more interactive. 

Most people know of cookies in terms of authentication (Session Cookies), but they are commonly used for tracking, advertising, personalisation, and other functionality that require keeping state.

A bunch of cookie attributes have been introduced over the years to try to address security flaws with cookies.

* Secure: Cookie can be transmitted over a HTTPS connection **only**. This was introduced to prevent theft of sensitive data over un-encrypted connections.
* HttpOnly: Cookie can not be accessed via client-side APIs (JavaScript). This prevents cookie-theft with XSS, but there are still a number of vulnerabilities (notably CSRF) that can be exploited here.
* __Host or __Secure prefixes: Prevents attackers from overriding data in MITM contexts such as HTTP sites or subdomains.
* Domain/Path: Defines the scope of the cookie is. This reduces the attack surface of where the cookie is submitted to.
* Expires/Max-Age: Deletes cookies after a period of time. Expires is an absolute reference (eg Saturday), Max-Age is a relative reference (eg 36000 seconds from now).  

These were a step in the right direction for security, but...

Based on [this paper](https://tools.ietf.org/html/draft-west-http-state-tokens-00) we are seeing only 9.9% of cookies set with HttpOnly, 8.8% Secure, and a whopping **84.2%** with none of the above security attributes.

Some of these controls have existed since 1997, so, that's pretty damn sobering. So what do we need to do to increase adoption?

## Pointing and Calling

I'm heading to Japan for a holiday in a week. One of the things I was reading about was the railway system, and what they do to make it safe and efficient. Australia has adopted parts of their "Shisa Kanko" system. Idea being that [making gestures and speaking](https://www.youtube.com/watch?v=9LmdUz3rOQU) out the status helps keep focus and attention on a problem. If you've ever gone to Central station you'll see CityRail staff perform the following:

* Blow the whistle indicating no more boarding. (Kinda Speaking)
* Raise the White Flag if no problems, other colours if there are. (Large Gesture)
* Driver/Guard look down the platform and check all flags are White. (Speak and Count White Flags)
* Train is safe to leave. (Safe!)

This system dramatically reduces the likelihood that the train will leave with something being wrong or unsafe on the platform. Now, I think aussies are pretty lazy with their adaptation compared to our Japanese friends, but eh, she'll be right.

Why does this matter? We are relying on people to remember Secure/HttpOnly/SameSite/Domain/Max-Age/Prefixes for each cookie. Why are we not providing a system that makes it hard to NOT do the right thing.

Yes, it might be interesting to point and call "This is a Cookie, It is HTTPOnly, It is Secure, etc" as part of a Peer Review, but Google has opted for a different approach. There's currently an [IETF draft for HTTP State Tokens](https://tools.ietf.org/html/draft-west-http-state-tokens-00) which is one method to address state as a challenge. But in the meantime, Chrome has taken some of those ideas and implemented them for cookies.

## Chrome 80

The Chrome team has taken a bold step to introduce two features in May 2019, earlier this year. 

* [Cookies default to SameSite=Lax](https://www.chromestatus.com/feature/5088147346030592)
* [Reject insecure SameSite=None cookies](https://www.chromestatus.com/feature/5633521622188032)

These features will go live to the world Feb 2020, and most other browsers manufacturers have already implemented this change.

The idea is that we will move from an opt-in to opt-out model for a security feature. It's generally in google's interest to ensure customers are protected from adversaries, and by being the distributor of the most used web browser in the world, that gives them a great opportunity to help improve the security of potentially millions of people. Most other browser manufacturers don't want security to be a market differentiator, so they'll go ahead with it too.

These features **CAN** and **WILL** break many websites that rely on cross-origin functionality from cookies. Either that, or the company will try to do a sneaky and set SameSite=None and get wrecked by not having their HTTPS issues sorted out.

## Australian Sites

Chrome has provided us with some flags to test the impact of these changes to websites before the February 2020 DDay. I thought it would be a good idea to test a bunch of Australian Websites and see what they are using cookies for and what potential impact the changes might have. 

![The Experimental flags in Google Chrome](../../img/posts/2019-10/Flags.jpg)

I visited around 200 Australian Websites with my developer tool tab open to monitor what they were doing. The companies were across a bunch of industries, but generally picked from what i remembered exist in Westfield or are known aussie brands.

# Here's the fun stuff!

### Almost all Australian companies use some form of user behaviour tracking 

Advertising and Tracking cookies are being served from a heck of a lot of different companies. Some are A/B testing companies, some track mouseEvents and keyboardEvents, some are storing your navigation through a site, etc. Universally, these invade privacy of users but are beneficial for companies looking for insight into what does and does not work for their customers. These are the most likely to break as the technology is almost always hosted on a third party domain AND the secure attribute has rarely (once over 200 domains) been set to true. Check this example from TheIconic.com.au, a clothing distributor in Australia.

![The Iconic has 33 advertising cookies at a minimum and a PHPSESSID wtf](../../img/posts/2019-10/Iconic.jpg)

### Some companies rely on cookies that cross a lot of origins to function correctly.

I don't really want to speculate why this is the case (software architecture is hard) but I can say that this architectural decision will likely cause disruption to functionality in some way. Example here is Telstra.com.au. Like, not bashing Telstra, but the fact that they have at least 8 Origins that cookies are spread across for their authentication page is almost certainly going to cause some weird service disruption February 2020. Good luck friends!

![Telstra has lots of origins which will cause problems](../../img/posts/2019-10/Telstra.JPG)

### SelfXSS is a vector some companies are aware of

Department of Human Services and Facebook display this warning when trying to use the developer console on their webpages. This is a good step and hopefully will prevent some people from being tricked into executing XSS in their browser via the developer console. Who would go through these hoops? Desperate developers copying from StackOverflow or non-technical people to be frank.

![Human Services is a good place](../../img/posts/2019-10/DHS.jpg)
![As is Facebook](../../img/posts/2019-10/FB.jpg)

### Cookies leak server information

TheIconic, Vegemite, Nintendo, HelloFresh, and more use PHPSESSID. Coles uses ASP.NET_SessionId. Officeworks uses JSessionID. Look, it's not a huge deal, but these mistakes add up to give attackers more information to attack your domains. I'd look at what seek.com.au does with JobseekerSessionId as an example of what to actually do.

# What next

So, from that sample above maybe it's time to revisit a few things for your company. 

1. How do your analytics, tracking, and A/B testing work? Do you need to make sure their origins are HTTPS?
2. Does your website use multiple origins for functionality? What option is more achievable SameSite:None+HTTPS or Re-Architecting?
3. Which HTTP methods are your requests using? GET only? Lax may not cause impact then!
4. Do you *need* cookies? 
5. Are you supporting and contributing to the development of HTTP State Tokens ?

Thanks for reading.

Cole