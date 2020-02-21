---
layout: post
hero_title: How to Launch a Secure Website in 2020
description: You will learn about Static Site Generator and how to use it to build a secure and fast website 
date: 2020-02-14
hero_image: /img/secure-website-hero.jpg
hero_height: is-medium
image: /img/secure-website-hero1.jpg
tags: web security data breaches cyber attacks jekyll static dynamic website SSGs
---

Back in the day, people had no concern about having a secure website. It was not until security flaws in the websites become frequent and the largest companies in the world had reported with millions of account breaches. 

Today, almost every website owner is aware of web attacks and the existence of hackers as threat of their websites.

![Biggest data breaches in 21st century](/img/cso_17-account-breaches.jpg)

#### So what caused all this chaos?

If you dig down to the details of these enormous breaches, you will find that it started with an injection vulnerability, one of the most common web vulnerabilities that used is SQL Injection which allow the intruder to reach website's database to extract the stored data, like users' accounts.

This repeated web incidents have made a bad reputation to the dynamic websites, but then it raised the need to alternative techniques, **and here when Static Sites Generators come from.**


### What is Static Sites Generators

>SSGs are a new, hybrid approach to web development that allow you to build a powerful, server-based website locally on your computer but pre-builds the site into static files for deployment.

This modern approach has become the best alternative to dynamic website that has all sorts of programming variables, functions, classes, database calls that increase the potential vulnerabilities in your website.

![Dynamic Vs Static Websites](/img/dynamic-x-static.png)
<small>*(source: gitlab)*</small>

This difference is giving the static websites many advantages over the dynamic ones, in terms of security, speed, cost, and scalability.

**1. Security**

Unlike dynamic websites, there is no data inputs need to be validated, or database calls, so no way for command injections.

**2. Speed**

Static site is fast, the website is already made of HTML, no php file to compile or database queries, so there is nothing to process from the server-side. 

**3. Cost**

It is also very cheap, you will only pay few bucks in a year if you run it on a service like [Amazon S3](https://aws.amazon.com/s3/)

**4. Scalability**

While with dynamic sites it's common to have unexpected traffic peaks that crashes your site. This is unlikely the case with static sites that has very low load on the server.


### How to Build a Static Site?

There are many modern Generators become available, the most commons are Jekyll, Next, Hugo, Gatsby, Hexo, and Nuxt. So you pick one of those generators which will take care of the coding stuff and let you focus on your content only.

![source: jekyllrb.com](/img/ssgs.png)

I prefer *Jekyll* for many reasons, it has active community, it's the one that established many of the patterns that other SSGs are now using, from metadata in front matter and structured data folders, to support for Markdown, expressive Liquid templating, and support for categories and tags.


**So let's build our first Jekyll blog.**

The steps will be first building your blog on your own machine, and once you customized it as you want it to be, you can deploy the `_site` folder to the hosting space.

To setup your Jekyll environment, you'll need to install Ruby -the language that Jekyll is build by-, then bundler -the thing that will manage your projects decadencies-, and finally Jekyll itself. 

The instructions are varies according to type of your operating system. If you are using Linux -which I love and recommend-, then it's the same ever-boring "open terminal, then run this apt/yum command" thing.

If by any chance your linux distro is Ubuntu, Follow these steps:

**1- Open your terminal, Dah, and run this command:**

```shell
sudo apt install ruby-full build-essential zlib1g-dev
```

Wait a few seconds for apt to install Ruby with all its essential libraries.

**2- After you get the prompt again, edit bashrc with `nano ~/.bashrc`, then add the following couple lines at the bottom of the file:**

```shell
export GEM_HOME="$HOME/gems"
export PATH="$HOME/gems/bin:$PATH"
```

Save the file by pressing the combination keys "CTRL+O" then "CTRL+x"

**3- Back to terminal now, you need to apply the changes on bashrc by running this command**

```shell
source ~/.bashrc
```

**4-Finally, install Jekyll; You still have terminal open, run this command:**

```shell
gem install jekyll bundler
```

That's it.

For other operating systems "Windows or MacOS", refer to the [jekyll installation guide](https://jekyllrb.com/docs/installation) and click on the operating system of your type.

Now that you have Jekyll environment settled, it's time to build your first lovely jekyll blog. First pick a location where your blog files will be placed on your system, I prefer to be as close to the root as possible, such as `/home/$USER/myblog`, or for MacOS: `/Users/$USER/myblog`, now whatever the location you choose, go to that location with `cd /home/$USER`, and run this command:

```shell
jekyll new myblog
```
Wait for few seconds... Et Voila! You've just created a complete jekyll blog.

Your jekyll blog has a structure that looks like this:

```bash
├── _config.yml # Stores configuration data
├── _data   # Well-formatted site data should be placed here, such as the navigation items
|   └── navigation.yml 
├── _drafts # here placed the unpublished posts.
|   ├── draft-post1.md
|   └── draft-post2.md
├── _includes  # These are the partials that can be mixed and matched by your layouts and posts to facilitate reuse
|   ├── footer.html
|   └── header.html
├── _layouts    #These are the HTML templates that wrap posts
|   ├── default.html
|   └── post.html
├── _posts  # Your own posts saved here.
|   ├── yyyy-mm-dd-post-title1.md
|   └── yyyy-mm-dd-post-title2.md
├── _sass   # These are sass partials which will then be processed into a single stylesheet main.css
|   ├── _base.scss
|   └── _layout.scss
├── _site   # This is where your generated site will be placed
└── index.html # You main page, can also be an 'index.md' with valid front matter
```

Now to see how your website looks like for your visitors, run the following command to build your site and start a local server as well:

```shell
bundle exec jekyll serve
```

After few seconds, you will see on terminal the last two lines:

```shell
Server address: http://127.0.0.1:4000/
Server running... press ctrl-c to stop.
```

Now open your favorite browser and put this link: [http://127.0.0.1:4000](http://127.0.0.1:4000/), you will see a default page from jekyll Saying welcome to you:

![New Jekyll Site](/img/new-jekyll-site.png) 

Yes I know, it looks awful. That's the look of the default gem-based theme called Minima, there are many themes available on the Internet for Jekyll here are some links:


- [jamstackthemes.dev](https://jamstackthemes.dev/ssg/jekyll/)
- [jekyllthemes.org](https://jekyllthemes.org)
- [jekyllthemes.io](https://jekyllthemes.io)
- [jekyll-themes.com](https://jekyll-themes.com)

Installing any theme you like is very easy. In fact, most of the theme creators are providing the instructions for the installation which are pretty same like the following:


1. Edit Gemfile "inside your blog directory", and add `gem "THEMENAME-jekyll-theme"`, replace *THEMENAME* with the name of the theme

2. With the terminal opened on the root of your blog directory, run the command `bundle install`, that will install the theme and its dependancies.

3. Next, edit `_config.yml` and add `theme: THEMENAME-jekyll-theme`, replacing *THEMENAME* with your theme name, that will setup the theme to your site.
    
4. Finally, run `bundle exec jekyll serve` to build and serve your site *in case it is already running, close it with `CTRL+C` and re-run the command to rebuild the site*

Now re-check again with [http://127.0.0.1:4000](http://127.0.0.1:4000/) and see your new website's look.

For further changes to the site, you need to read the documentation at theme's website to set things like the navigation, contact form and social sharing buttons...etc


### Deploy your site

Now that you built your website and added your own touch, it's time to make the website available online. 

At the time when you executed `bundle exec jekyll serve`, Jekyll generated your static site to the `_site` directory inside your blog. You can transfer the contents of this directory to almost any hosting provider to get your site live.

Having a generated static site gives you more hosting options than with the traditional LAMP site. As you can either upload `_site` to any hosting provider using any file transfer i.e. FTP, or you choose one of the modern web hosting that provides automatic deployment, global CDN, one click HTTPS in addition to other awesome features for SSGs-based sites. Some of those providers are Aerobatic, Netlify, and AWS Amplify, Find the full list with the features for each one at [jekyll Docs](https://jekyllrb.com/docs/deployment/third-party/)


### Any Free Options?

If you are looking for a free hosting service especially when your website or business service is in its early stages, but you need a clean website without ads garbage by the provider, and have the freedom to point your own domain, then the best option for you is the Github's awesome service called [Github Pages](https://pages.github.com/). 

There are some steps to be followed before you can host your website on Github Pages, which involves creating a repository on your Github account, and initiate git inside your blog so you can push your changes from local site to the the online repository where your blog resides.

Here is a complete guide from Github to setup your [Jekyll blog on Github Pages](https://help.github.com/en/github/working-with-github-pages/creating-a-github-pages-site-with-jekyll) -I followed it myself-


### Be Aware of the Common Security Mistakes

So you made up your mind, and chose to build a static site, **does that makes your website a hack-proof?**

*I would say no, but you almost there!*

Even with a static website, you won't get a 100 percent of protection. In fact, there are still ever-present security threats that can always be there like using a dictionary password or having outdated or missconfigured service on the server. 

![percentage of attack vectors](/img/attack-vectors.jpg)

So what you need here is to pay attention to some security principles, things like making your passwords complex, save it away from unauthorized parties, and keep your recovery email address unknown for anyone on the Internet (like don't use it in social sites or share it with anybody).


That's it. You are now safe from the most common attack vectors that caused so many sites to be breached while paying few bucks for hosting and domain name.

If you have any question or any suggestion to point out, why not leave me a comment down below, or send me a message over [twitter](https://www.twitter.com/tariqhawis)
