---
title: AppSec 101 - Where Do I Begin?
date: 2019-04-04
draft: false
---

I thought I'd write a series of blog posts covering the core aspects of Application Security, and I'm not talking about vulnerabilities. I'm going to start with well.. the start!

## Setting the Stage

Generally, when organisations are first starting to think about or begin to implement an AppSec strategy, the following scenario happens in one way or another.

![CISOgraph](../../img/posts/2019-apr6-whysast.jpg)

Mr CIO walks into work, sits down with his double shot espresso from his local barista and writes up the following email to the organisations CISO.

---

![CIO Yeah](../../img/posts/2019-apr6-cioYEAH.jpg)

> Janice!

> I just came back from **VendorCon&trade;**, and I saw a credible, evidence-based graph from a smart hackerman! He was explaining that security gets more expensive as a project progresses. I don't think we can do projects in shorter timeframes hahahahaha so option one is out the window. So we need to focus on security earlier instead! We have security architecture already in design, but what about BUILD? How do we get developers to do security work too? Think of the cost savings in incidents and pen-testing we can put to other initiatives.

> Let me know your thoughts Jan, I'm excited about this idea!

> -- <cite>Steve McSteveson, CIO CoolGuysInc</cite>

---

Janice sees this as a huge opportunity to secure funding for other important security services and replies with the following.

![CIO Yeah](../../img/posts/2019-apr6-cisowhy.jpg)


> Steve,

> Security is something that our developers already care about trying to get right. They don't have the time or training to become experts in everything.. We'd be trying to hire a bunch of rockstar unicorns! Not sure that an AppSec program is the best way to improve our organisations security posture. Why don't we look into NAC, MFA, or expanding our other security options first?

> Regards, 

> -- <cite>Janice, CISO CoolGuysInc CISSC CISPRI CRISPR CSLLPS CISC2P </cite>

---

... and you know what? She could very well be right! There are a lot of situations where an AppSec program is not worth the investment in time or resourcing in comparison to other infosec initiatives. But let's assume that this organisation would benefit highly from it.

---

Steve explains to Janice that the data does not lie, and our diligent CISO goes to her most trusted resource, goggle to research her options.

![CISOgraph](../../img/posts/2019-apr6-goggle.PNG)

Through her thorough five minutes of research she comes across the world of Application Security and DevSecOps! After hiring a few smart DevSecOps professionals, she tasks them with solving the AppSec item as quickly as possible. But where do they begin?

Now that we have set the stage, we need to talk about where to begin. AppSec is NOT about picking random tools and sticking them into DevOps pipelines and **DING DING DING**! Security has levelled up! There are a lot of considerations before we even begin implementing tools.

## The Stage is Set

Generally, when we talk about introducing an application security capability into an organisation, the most important thing is to spend time getting to know the organisation really well. If you cannot snap answer any of these questions below, then you should be doing some more research! I'll go through each of the below questions and explain WHY they are important to get right early. This post is fairly lengthy, so feel free to read all the **TLDR** instead if you're a CISO short on time. ;)

### What kind of SDLC and operating model does my organisation follow?

If you are unsure whether your organisation is following a Waterfall or Agile or Continuous Delivery development methodology, then it becomes really difficult to select tools that are effective for them.

Some of the Application Security tools function in very different ways, I'll cover in a later blogpost the differences between various SAST tools, but here is an example

Let's take a tool such as [TruffleHog](https://github.com/dxa4481/truffleHog). Trufflehog finds secrets accidentally committed to a git repository. 

Trufflehog relies on computationally fast algorithms (Regex Matches and calculating string entropy across git diffs) but has very limited scope in what it achieves. Because of this, it would work great in an environment leaning much closer to CD or Agile. 

If you pick a commercial security-focussed static analysis tool, it would probably work in a fundamentally different way. Generally, commercial SAST tools compile source code into Intermediate Representations. Software that performs data-flow, control-flow, semantic, structural, or other forms of complicated analysis on these representations is then run, and the results put into some form of defect management system. A quick runthrough is below.

![How SAST operates](../../img/posts/2019-apr6-howsast.jpg)

The issue with these? They go deep! They are precise and find a wide-range of security defects. They are also horrendously slow. This might be fine if you're used to nightly builds or a secure code review per release, but if you're paying for compute and releasing several times a day, then these tools might make your developers balk at the prospect.

This question applies to what AppSec tools you are able to run on a production instance too! If you're a twelve factor organisation, go ahead and instrument your applications or run a bunch of WAFs. But if you're in healthcare or finance, well, it may be outside of your risk appetite to have protective controls running in your production systems in case you block legitimate traffic.

TLDR: **Know your operational model, then you can make optimal tool selections.**

### What existing tools are in place?

Hey, developers are smart cookies. They care a hell of a lot about their craft. Nobody wants to be _that guy_ who deployed a password in a git commit, or wrote a SQL Injectable login form. A lot of developers are already taking active steps to make sure that their applications are secure!

So, don't be afraid, go and talk to them. Yes, I know, you're probably thinking...

> "I got into InfoSec to avoid talking to people and you're asking me to go and talk to people? Don't do this to me please."
>
> -- <cite>Standard InfoSec Rhetoric</cite>

Well, drink a cup of concrete and harden up. If you want to do right by yourself and your clients, then go and talk to your developers. They have probably researched and added a few tools into their workflows like [Brakeman](https://brakemanscanner.org/) or [Snyk](https://snyk.io/). Why not continue to use the things they are already comfortable with and actively maintaining?

TLDR: **Talk to your developers. Don't make assumptions.**

### Do we use cloud services? Which ones?

If your organisation is using cloud services, then the types of controls available to you can change dramatically. Do not think in terms of "_**What DevSecOps tools do I need to deploy**_". Instead, try "_**What cloud services do I need to integrate with my apps**_".

Amazon, Azure, Alibaba, even the goggles have security services available to use, and it's far better to look into using those services than it is to manage your own.

Amazon provides [Inspector](https://aws.amazon.com/inspector/), [GuardDuty](https://aws.amazon.com/guardduty/), and [AWS Web Application Firewall](https://aws.amazon.com/waf/). No need to be investing in a bunch of Rapid7, Nessus, or nmap scans or managing a WAF when you can take advantage of what Amazon has available already to plug in. Chinese competitor Alibaba offers similar services with [Website Threat Inspector](https://www.alibabacloud.com/product/avds) and their [Web App Firewall](https://www.alibabacloud.com/product/waf) as well. 

If you're not in the cloud yet, or are progressing towards it, then this knowledge will help drive design discussions for cloud-first applications in the future or alternatively to select offerings with airgap capability, with the latter being more relevant if you're one of [those spooky people](https://asd.gov.au/careers/currentvacancies.htm).

TLDR:

* **If you're a cloud first org, take advantage of cloud security services, best place to be!** 
* **If you're starting a cloud journey, use your foresight and knowledge of The Cloud&trade; to steer discussion into adopting security early in the process.**
* **If you're super not into cloud, great! Use tools that work in airgapped environments instead!**

### How big is our organisation?

This matters a lot. If you are a tiny start up, then things such as governance, compliance, regulatory affairs, defect management, federation of services, homogenous or hetergenous environments, etc are not as important to you. All these things become drastically more important as your organisation increases in size. Eventually you end up with a weird Venn diagram with conflicting requirements and no real way to read properly.

![Dumb](../../img/posts/2019-apr6-dumb.jpg)

When you work for a larger organisation, there are a lot more aspects to consider and because of this you need to work out what the highest priorities are and remember that you can't boil the ocean.

Using data from your educational initiatives, your vulnerabilitiy programs, or even your security incidents is a great place to  begin working out where to start. Otherwise you'll be stuck in mud being unable to accomplish anything.

TLDR:

* **If you are a small organisation, you have the freedom to experiment and implement as you see fit. Enjoy!**

* **If you are a huge organisation, paper-based assessments will fail due to the complexity of the engagement. Work out one weakness and focus on getting that up to scratch first. Crawl before Run hey? Most huge organisations are crawling, be patient and accept that.**

### What programming languages are we using?

This matters for tool selection. Some tools have great coverage across a number of languages, but lack depth and need triaging before use like [GRAudit](https://github.com/wireghoul/graudit/). Others are specifically tied to a language like [npmcheck](https://github.com/dylang/npm-check). Knowing what languages you need to support should drive your tooling decisions.

This is especially relevant if you work in a faster-paced organisation where things change a lot. If you need to update tools a bunch and remain bleeding edge, there is no point investing in security tools that become useless after 6 months of use.

Similarly, there are tools on the market that do one job and do it well, and if you're a **.NET4Life** kind of organisation, then they'll do splendidly!

TLDR: **Know what you're developing in, choose tools that suit.**

### What are the most important assets our business?

If you don't know what matters to your business, then you're not going to be an effective security service. Get an authoritative person from Business or Risk to clarify exactly what software is needed to keep the business running, and which things have the highest amount of risk associated with them. Then focus your AppSec on them.

Who cares about SQLI on a stand alone marketing website if your trading platform gets taken down by a Regex DOS at 9am Monday?

TLDR: **Risk is Everything. Focus AppSec on servicing critical applications first**

### ... Do I have money ?

Last but not least, and this is a simple one. Money.

![Dumb](../../img/posts/2019-apr6-shutup.jpg)

Funding for infrastructure, resourcing, education and training, or just to purchase commercial variants of tools is a huge part of starting an AppSec program. Because of this, your operational model needs to keep into account whether you provide the service for free and gets absorbed by your organisations cost centre, or whether individual projects or applications need to pay for access to these services.

Cost influences adoption, if things are too expensive, developers may look elsewhere or ignore your services altogether. But too far in the other direction and you'll have anaemic services that developers will pay limited attention to.

Achieving a balance is difficult here, and every organisation will need a different approach to ensuring that you keep your budget afloat, your services in use, and your services valuable.

Also, negotiate, always negotiate. Here's a few to look at:

* Vendor Pricing
  * Tool access
  * Support costs
  * Implementation assistance
  * Training in tool usage
* Developers not wanting to pay for access
* Developers not wanting to fix issues
* Manager-speak blah blah won't do this
* Operational Management Costs
* **Ice Cream on fridays**

[Here's a good resource on negotiation](https://www.jordanharbinger.com/alex-kouts-the-secrets-you-dont-know-about-negotiation-part-one/).

So keep cost and your operational costs at the forefront of your mind before you embark on your appsec journey. If you don't, it'll bite you further down the track. Whether this is in having to completely drop services, let go of resources, or to accept working with outdated technology, it is not a good realisation to have three years into your program.

TLDR: **Budget allocation and recovery methods are important to keep in mind and define early in your appsec engagement. Negotiate where possible.**

## Exit Stage Left

Hey thank you so much for reading and I hope you've learned a few pieces of useful information for kickstarting your appsec program! Next few blog posts will focus on specific tooling and the strategies you can put in place to get it right.

If you've got any questions, leave me a comment below or message me by email or on linkedin friends!

Cole Cornford