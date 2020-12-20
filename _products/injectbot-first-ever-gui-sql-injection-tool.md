---
title: Introducing InjectBot v1.0
subtitle: First Ever GUI SQL Injection Tool
layout: product
image: /img/injectbot/hero.png
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
redirect_from: 
    - /projects/injectbot-the-easiest-and-fastest-SQLi-pentest-tool
---

The first ever SQL Injection tool with GUI Interface, high speed results, and less intense on the target.

SQL Injection is a special vulnerability to me, that's because it's the one that introduced me to the Hacking world back in 2007. It's then when I started my career in hacking an bug hunting focusing on this particular vulnerability which was very popular in web applications.

InjectBot was previously called Injr0b0t. In 2008, I built Injr0b0t to automate my work on bug hunting which saved a tremendous time for me; from detecting the vulnerability, to creates the exploit query, to inistiate foothold and backdoor. Injr0b0t was never intended to be public at that time, that's because of the fear from being used by the wrong people.

> The years from 2006 until 2014 were considered as glory times for SQL Injection, with millions of websites and web applications vulnerable to it, and it still there to date. reference: [wikipedia.org](https://en.wikipedia.org/wiki/SQL_injection#Examples)



![Injr0b0t SQLi tool from 2009](/img/injectbot/injrobot.png)
<small>*Injr0b0t SQLi tool from 2009*</small>



But now that this reason doesn't make sense anymore with so much hacking tools out there; and the demand I saw from my fellows in security field to have such tool with less impact and high speed results, yet easy to use, I decided to respond to this demanding and start to reproduce my Injr0b0t tool, with focusing on discovers the bug rather than the create any damages on the target which make it more helpful for the web developers and security teams for their security testing process. And that is InjectBot.


# So What is InjectBot?

InjectBot is a web-based penetration testing tool developed to be used by web developers, penetration testers and security analysis teams to check web applications against SQL injection vulnerability.



# How it Works?

InjectBot basically mimics the penetration tester and security analyst by automating the checking process of variables of web application's URL to see whether the application has input validation countermeasures in place to sanitize the user's input. This process involves sending bunch of SQL queries to the target's URL then analyze what comes back from the target by looking on the returned page for things like SQL errors or some significant changes with the theme, content, and so on. while this analysis process may take few minutes or more by the pentester, it takes just a fraction of a second by InjectBot!



# Why InjectBot?

Unlike other SQLi tools which are being used for long time now, InjectBot has re-defined the security tools from terminal-parameter-based to a web-form-based tool, not only this, but also with the faster results, and that's not just an empty claim (see demo below).



# DEMO


A complete scan to a web app showing all the above features in only 23 seconds.



![InjectBot SQLi scanning Demo](https://www.tariqhawis.com/img/injectbot/injectbot.gif)




# Who should use InjectBot?

Whether you are a web developer or pen tester, red or blue teamer, this tools would be perfect to be in your tools arsenal to test web applications against SQL Injection vulnerabilities.



# Requirements

* Option #1: Use Docker engine to pull and run injectbot image as described under installation section below, for more details about how to use docker, refer to [Docker Docs](https://docs.docker.com/get-docker/)

* Option #2: A need your own web server up and running and have PHP 7.4 plus php-curl library.



# Installation

The easiest way is to use the docker image which already has injectbot installed & updated at docker hub, just run this below command:

```bash
docker run --name injectbot -d -p 8080:80 tariqhawis/injectbot:latest
```

If you have not received any error message, go to the URL: `http://localhost:8080/`
Now InjectBot is up and running.

Second option is to clone this repository and point your web server to the cloned path.

If you have any issue with installation, contact me at [github issues](https://github.com/tariqhawis/injectbot/issues), and I will be glad to help:)




# Version History/Changelog

InjectBot v1.0 - rebuild and published release from the old private 0.5 Injectbot 2009.

* [+] Complete transition from procedural to object-oriented structure.
	* [-] Troubleshooting has become much easier with modularity.
	* [-] Scalability is now possible, adding features such as new SQLi techniques is much easier.
	* [-] The code structure is understandable, any developer who wants to jump in can understand the idea of each step, also comments added for that purpose.

* [+] Increase the speed of HTTP response by 20 times
	* [-] Use Multi cURL functions instead of file_get_contents for faster response.
	* [-] Ability to send multi requests in parallel.
	* [-] Send header requests for blind SQLi.
	* [-] Use content-length from the header instead of str_len for the whole page!

* [+] Remove iterations by saving scanning state.
	* [-] Use sessions to save the state of the target while the scanning progress.

* [+] Add blind SQLi scanning mode for fetching table schema and data records

* [+] Bugfixes
	* [-] Fixed content-length with returned -1.
	* [-] Fixed exploitable number inside html tags for some webpages.

* [+] New front-end design based on Bootstrap 4.



# Have an idea for InjectBot?

There are plenty of improvements this script could use, If you want to add something and have any cool idea related to this tool, please contact me using [github issues](https://github.com/tariqhawis/injectbot/issues) and I will update the master version.

If you are a PHP developer yourself, feel free to PR this tool, and I will merge the good ideas.



# Looking for a useful SQL Injection Course?

Contact me if you are looking for a course on web penetration testing, web application security, or a course explosively on SQL Injection, I am preparing for attackers and defenders (100% technical).



# Advisory

This tool should be used for authorized penetration testing and/or educational purposes only. 
Any misuse of this software will not be the responsibility of the author or of any other collaborator. 
Use it at your own networks and/or with the network owner's permission.


GPL-3.0 License 2020 InjectBot&trade; - By Tariq Hawis
