---
layout: post
hero_title: Why you Should Avoid Using WordPress
description: Find the reasons why WordPress is a security risk for your website.
date: 2020-02-29
permalink: /:title/
hero_image: /img/avoid-wordpress-hero.jpg
hero_height: is-large
image: /img/avoid-wordpress-hero.jpg
tags: web security wordpress CMS SSG jekyll
---

WordPress is a web-based Content Management System -aka CMS- that allow users to create, manage, and modify content on a website, without needing to know anything about code. 

What makes WordPress the most preferable CMS for the vast majority of website owners is for the considerable community that behind it; WP's empowered by millions of people with plugins & themes of all kind and shapes that available for free to download, and for all tips & tricks that needed for better blogging experience. It's nearly impossible that there's a problem with WP and it has no solution exist on the Internet. Heck, even there are companies who especially dedicated for WP as you'll find who offer premium plugins, themes, or manage services.

All these advantages are true, but it's unfortunately the half of the truth.

Here are the other true facts.


## Fact No.1 .. WordPress is Slow

Since WordPress is build on dynamic-pages, the type that built with a server-side program language, which for WP case is PHP. The main disadvantage for a dynamic website is the fact that it will rely on server's resources to process the operations inside the pages. And imagine that each page will be processed *each time* it requested by a visitor to the page. That will eventually lead to slowness issue, and sometimes to crash the website entirely.

![WordPress crash with 504 gateway time-out error](/img/wordpress-crash-504-gateway-time-out-error.png)

You may not face that at the early stages of your website, but later when you got more recognized and start having a descent traffic, you will start suffering slowness or even crash problematic. At this stage, you'll start looking for a more expensive hosting provider, add CDN, premium lightweight themes, premium boost plugins, and maybe pay for special service to an engineer or a company to solve your website's problem. 

Now imagine how much time and money you'll waste for a website that supposed to bring profit instead of drains your pocket!


## Fact No.2 .. WordPress is attractive to Hackers! 

If there is a software that mentioned the most inside hackers community it should be WordPress. That because the huge list of vulnerabilities that discovered already and still increasing. Which make it a perfect choice to be used as a lab rat by hackers during their tutorials!


**WordPress is a Main Target**

If you search in the vulnerability databases like CVE or WPScan, you may find that WP has a scary number of vulnerabilities that are disclosed for the public.

**Why WordPress?**

For two reasons.

### 1. Tremendous Number of Plugins

To date, The last count from [WordPress plugins page](https://wordpress.org/plugins/) says 55,791 of free plugins available to download. That's an enough number to motivate bug hunters to start install and analyze plugins until find a security bug. As a result of the large-scale bugs hunting around the world against WP Plugins, there are thousands of discovered vulnerabilities disclosed to the Internet in several online vulnerability databases; such as [CVE](https://www.cvedetails.com/) and [wpvulndb.com](https://wpvulndb.com/plugins), the last one alone recorded so far 3858 of unique vulnerabilities in their database; that includes findings in WP core, plugins, and themes. Now this is a big figure, and it makes WP the top CMS with disclosed vulnerabilities.

![WordPress Vulnerabilities](/img/wordpress-vulnerabilities.png)


### 2. It's the Most Popular CMS

Hackers don't like wasting their valuable time finding vulnerabilities on some random applications on the Internet, specially when they work for big teams with big plan; A plan like infecting as many devices on the internet as they can; including, Personal Computers, Smart Phones, Internet Servers, IoT...and so on.

That said, since WP is the most popular CMS with 35% of all the websites on the Internet using it --according to W3techs- and for the huge number of disclosed vulnerabilities that already disclosed, it's sensible that they choose this particular CMS to focus on.

![CMS Infections 2018-2019](/img/cms-infections.png)

The concept used by hackers to infect devices is simple. They create automated scripts -can be available for free also!- which scan millions of websites for known vulnerabilities throughout the web. When a scan identifies a target, the exploit delivers its payload to obtain access to the environment and deploy other malicious tools, later, this infected machine will be used as a botnet to scan other devices worldwide; and ball keeps rolling.


## What are the Alternative Solutions?

As a pentester who used to hunt bugs, I can tell you that using WP as an engine for your website will bring a pain in the head and a time/money waster, as long the solution to over come slowness or security threats will require paying a lot of money on expensive hosting, CDN services with WAF enabled. Not to mention the need to watch out daily and regularly the updates of your CMS and its plugins, and monitor attack attempts that comes to your website. That all will be enough to move away from your own business core value.

Fortunately, there are modern platforms that came up lately which become over time the best alternatives for old fashion platforms, those platforms called **Static Site Generator "SSG"**

While in dynamic-sites fashion, website's pages are delivered on-demand to the visitors, which unnecessary consume server's resources. SSG-based website generates all the pages of the website *once* when there's actually changes to the site. This means there's no moving parts in the deployed website. Caching gets much easier, performance goes up and static sites are far more secure. 

![Static Sites vs Dynamic Sites](/img/dynamic-x-static.png)


If you want to know more about SSGs and how to build one, check out my other post [How to Launch a Secure Website in 2020]({% post_url 2020-02-14-how-to-launch-a-secure-website-in-2020 %})

Should you have questions or suggestions about this topic, please reach me out on [twitter](https://www.twitter.com/tariqhawis), or leave me a comment below.