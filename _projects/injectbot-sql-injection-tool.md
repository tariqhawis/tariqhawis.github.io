---
title: Introducing InjectBot v1.0
subtitle: The Easiest and Fastest SQL Injection Tool
summary:  |-
    Many penetration testers prefer to use GUI rather than CLI tools specially for professinals who have several security systems to deal. Injectbot has borken this barrier with a friendly web-based sql injection tool.
layout: project
image: /img/injectbot/injectbot-logo.webp
seo:
  name: InjectBot SQLi penetration testing tool
  author: Tariq Hawis
features:
  - label: Scan the target only to see whether it's vulnerable.
    icon: fa-fighter-jet
  - label: Get target's database and server information.
    icon: fa-fighter-jet
  - label: Get tables information from the current database.
    icon: fa-fighter-jet
  - label: Fetch data rows of selected table (select from the table list saved previously).
    icon: fa-fighter-jet
  - label: Switch between classic and blind SQLi in options above.
    icon: fa-fighter-jet
rating: 5
permalink: /:title/
redirect_from:
  - /projects/injectbot-the-easiest-and-fastest-SQLi-pentest-tool
---

The first-ever SQL Injection tool with GUI Interface, high-speed results, and less intense on the target.

SQL Injection is a special vulnerability to me, that's because it's the one that introduced me to the Hacking world back in 2007. It's then when I started my career in hacking an bug hunting focusing on this particular vulnerability which was very popular in web applications.

In 2008, I built a tool called Injr0b0t to automate my work on bug hunting which saved a tremendous time for me; from detecting the vulnerability, to creates and inject the payload and so on. _Injr0b0t_ was never intended to be public at that time, that's because of the fear from being used by the wrong people.

> The years from 2006 until 2014 were considered as glory times for SQL Injection, with millions of websites and web applications vulnerable to it, and it still there to date. reference: [wikipedia.org](https://en.wikipedia.org/wiki/SQL_injection#Examples)

![Injr0b0t SQLi tool from 2009](/img/injectbot/injrobot.webp)
<small>_Injr0b0t SQLi tool from 2009_</small>

Many things changed since then, we have now white hat hackers, pentesters and red teames who need such tools with easy interface, powerful results, and most importantly, less impact on the target. that's why I decided to reproduce Injr0b0t focusing on three targets: 1- friendly interface, 2- speed results, and 3- less impact on the target. This way it will more helpful for the web developers and security teams to do security testing very quicker and more accurate. And that is InjectBot.

# So What is InjectBot?

InjectBot is a web-based penetration testing tool developed to be used by web developers, penetration testers and security analysis teams to check web applications against SQL injection vulnerability.

# How it Works?

InjectBot basically mimics the penetration tester and security analyst by automating the checking process of variables of web application's URL to see whether the application has input validation countermeasures in place to sanitize the user's input. This process involves sending bunch of SQL queries to the target's URL then analyze what comes back from the target by looking on the returned page for things like SQL errors or some significant changes with the theme, content, and so on. while this analysis process may take few minutes or more by the pentester, it takes just a fraction of a second by InjectBot!

# Why InjectBot?

Unlike other SQLi tools which are being used for long time now, InjectBot has re-defined the security tools from terminal-parameter-based to a form-based tool, not only this, but with bullet options (no overwhelming options), faster results, and last but not least, less impact om the target, leveraging some cool techniques cush as:

1- Web features like session management, has helped the tool to not repeat any request to the target, so if a request send to confirm the vulnerability, it will be saved for the entire time of the user's session with the tool, and that alone reduced a lot of connection numbers and hence reduced the impact on the target.

2- Used multi curl functions. This function has proved by benchmarks its unbelievable speed response compared to normal curl and other functions like file_get_content or get_headers.

# DEMO

A complete scan to a web app showing all the above features in only 23 seconds.

![InjectBot SQLi scanning Demo](https://www.tariqhawis.com/img/injectbot/injectbot-demo.gif)

# Who should use InjectBot?

Whether you are a web developer or pen tester, red or blue teamer, this tools would be perfect to be in your tools arsenal to test web applications against SQL Injection vulnerabilities.

# Installation

Choose your best option below then open at your browser: `http://localhost:11111`

- Option #1: Suitable for linux and MacOS users..

Just run the script `run.sh` that comes with the script as follows

```bash
./run.sh
```

> Note: in this option you need to have php and php-curl installed in you machine.

- Option #2: Suitable for Windows (and Linux & MacOS) users..

Your best option here is to use docker, for that you may build your image and run the container:

```bash
docker build -t injectbot .
docker run -d --name injectbot -p 11111:80 injectbot
```

Or call the updated image from our docker hub:

```bash
docker run --name injectbot -d -p 11111:80 tariqhawis/injectbot:latest
```

If you have any issue with the installation, contact me at [github issues](https://github.com/tariqhawis/injectbot/issues), and I will be glad to help:)

# Version History/Changelog

InjectBot v1.0 - rebuild and published release from the old private 0.5 Injectbot 2009.

- [+] Complete transition from procedural to the object-oriented structure.

  - [-] Troubleshooting has become much easier with modularity.
  - [-] Scalability is now possible, adding features such as new SQLi techniques is much easier.
  - [-] The code structure is understandable, any developer who wants to jump in can understand the idea of each step, also comments are added for that purpose.

- [+] Increase the speed of HTTP response by 20 times

  - [-] Use Multi cURL functions instead of file_get_contents for faster response.
  - [-] Ability to send multi requests in parallel.
  - [-] Send header requests for blind SQLi.
  - [-] Use content-length from the header instead of str_len for the whole page!

- [+] Remove iterations by saving scanning state.

  - [-] Use sessions to save the state of the target while the scanning progress.

- [+] Add blind SQLi scanning mode for fetching table schema and data records

- [+] Bugfixes

  - [-] Fixed content-length with returned -1.
  - [-] Fixed exploitable number inside html tags for some webpages.

- [+] New front-end design based on Bootstrap 4.

# Have an idea for InjectBot?

There are plenty of improvements this script could use, If you want to add something and have any cool ideas related to this tool, please contact me using [github issues](https://github.com/tariqhawis/injectbot/issues) and I will update the master version.

If you are a PHP developer yourself, feel free to PR this tool, and I will merge the good ideas.

# Looking for a useful SQL Injection Course?

Contact me if you are looking for a course on web penetration testing, web application security, or a course explosively on SQL Injection, I am preparing for attackers and defenders (100% technical).

# Advisory

This tool should be used for authorized penetration testing and/or educational purposes only.
Any misuse of this software will not be the responsibility of the author or any other collaborator.
Use it at your networks and/or with the network owner's permission.

GPL-3.0 License 2020 InjectBot&trade; - By Tariq Hawis
