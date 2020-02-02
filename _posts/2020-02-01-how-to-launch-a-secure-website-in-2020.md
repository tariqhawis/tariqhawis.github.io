---
layout: post
title: How To Launch A Secure Website in 2020!
description: How can you create a secure website without having security knowledge 
date: 2020-02-01
hero_image: /img/secure-website-hero1.jpg
hero_height: is-large
image: /img/secure-website-hero1.jpg
tags: web security hacking cybersecurity wordpress jekyll 
categories: Web Security
---

If you have searched in the Internet how to secure a website before you end-up here, then you might overwhelmed with the complicated security guidelines which are meant to be for the web developers and Cybersecurity experts, but who said that you have to be like one of those nerds to have a protected website!

Let me reveal to you some secretes that no technical guy will tell you about, because these secretes will let you build your own website even if have zero knowledge with websites.

So let's get started.


## 1. Make it simple!

Simple website means fast, and secure..

The way to do that is to build a site that focus only on your content, and the better choice for this goal is to build a static site.

### 1.1. What is a static site?

In the world of web there are two kind of websites, static and dynamic. The first one means your site is build on HTML pages - with other objects like Javascript, CSS, image files - in other words, it has nothing to process from the server-end, an ready bread for your browser to eat.. I mean to present.

While on the other hand, dynamic site has all sorts of programming variables, functions, classes, database calls...etc that needs from the server to compile and process all these files to bake at the end the HTML files that your browser can only understand.

Just by realize this comparison you can conclude that the fastest and secure way is the static sites. Fast because no dynamic pages or database calls need to process. Secure because user input that will be inserted to the application and its database which leads to injection or file traversal attacks...etc

Static sites are becoming more and more common thing now a days, there are even some bloggers presume that static sites is the future of the web, personally I couldn't agree more. Why? because what is the real matters to any website owner is his content only, everything else comes next. So why bother make OS-size website with plugins, database, and programming framework (like java, .NET...) only to serve few pages of yours and end up with vulnerable site to the cyber attacks and malware infections.


### 1.2. How to Build a static site?

OK then, How are you going to build a static site? Well, first of all you need to lean how to code with HTML and Javascript... wait WHAT?! 
... just kidding! :)

What you actually need to do is to pick one of static site generators which will take care of the coding stuff and let you focus on your content only. The most popular SSGs these days are Jekyll, Middleman, Roots, and Hugo. To me Jekyll is my favorite, like this site is build on it!

So now how it works? first you need to follow the SSGs instructions, as for [jekyll](https://jekyllrb.com/docs/installation), you need to have ruby on your machine as described in above line, then you install bundler/jekyll gems. Next everything becomes very easy, with three magic words on your terminal you can build your entire site project:

```shell
jekyll new myblog
```
Wait for few seconds... Et Voila! you just built a complete project. No code or database needed. 

Now you may thing that I'm going to tell you how to setup your apache, but no! Here the things is much easier, what you only need to run your site locally by running another tiny command so jekyll will build and compile all pages:

```shell
bundle exec jekyll serve
```

after few seconds you will see on terminal the last two lines:
```shell
Server address: http://127.0.0.1:4000/
Server running... press ctrl-c to stop.
```
It means your site is ready. Now open your favorite browser and put this link: [http://127.0.0.1:4000](http://127.0.0.1:4000/) you will see a default page from jekyll welcoming you to jekyll like this one: 

![New Jekyll Site](/assets/images/new-jekyll-site.png) 

Afterward to learn more on how to create a wonderful website, go ahead and visit jekyll's [Step by Step Tutorial](https://jekyllrb.com/docs/step-by-step/01-setup/), you will learn it in no time.


## 3. Avoid Popular CMS Platforms!

In the world of Cyber security, the more popular a piece of software it becomes, the more it becomes a target for the unfriendly sort of hacker, so live with this fact!

That said, platforms like Wordpress, Drupal, and Joomla are by far the top 3 platforms on the Internet, which make them as well the the top 3 hackers' targets. If you don't want your site to be under the radar of hundreds of hackers around the world then you might need to consider staying away from these platforms. 

I'm seeing that all the time while watching everyday security events, random sources sending HTTP requests with arbitrary crafted exploits that meant to target one of those popular platforms, even though I don't have any of them in my environment. 

This popularity is what attracting both security analysis experts and hackers to dedicate their time and efforts finding out weakness spots on these platforms. 

For instance, Wordpress is one of the most attractive target for these folks, considering that it powers 35% of all the websites on the Internet, which equal one-third of the web! According to W3Techs. It's no wonder that it has a huge number of vulnerabilities listed in [CVE website](https://cve.mitre.org/cgi-bin/cvekey.cgi?keyword=wordpress)

And yes, it's too easy for anyone to find out the exploit that need to execute on your Wordpress site to take it down! And it's available on Internet and for free of charge! included with demo of how to do it! Free websites that offer that database of exploits is like [exploit-db](https://www.exploit-db.com/), just search for any name of software and choose any exploit you like to execute! it's that simple.


## 4. Be Aware of the Common Security Mistakes

So you stayed away from Dynamic/database site. What now, does my website become a hack-proof? No, but you almost there! 

There are still ever-present security threats that can always be there like someone getting your credentials for your server. So you need to pay attention to some security principles, things like making your passwords complex, save it away from unauthorized parties, and keep you recovery email address unknown for anyone on the Internet (like don't use it in social sites or share it with anybody). 

## 5. Pick The Right Hosting Company

Choosing the best hosting company and plan for your website is not an easy task. But it also a must step in your plan for web marketing, because the right choice will save you a lot of money that you needed most to improve your own business. That what you can learn more about in my article [How To Choose a Web Hosting]({% post_url 2020-01-31-how-to-choose-a-web-hosting-in-2020 %}).



So That's all I can tell you about creating a secure and fast site for non-technical owners. I hope you enjoyed reading. If you have any opinion or you think there are other options that I need to add, please leave me a comment.
