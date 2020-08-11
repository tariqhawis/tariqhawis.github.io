---
layout: post
hero_title: HTB Under Construction Web Challenge
description: Walk-through hack the box's web challenge CTF called Under Construction
date: 2020-08-10
permalink: /:title/
redirect_from: /2020/08/10/htb-under-construction-web-challange.html
hero_image: /img/Hack-The-Box-logo.png
hero_height: is-large
image: https://www.tariqhawis.com/img/Hack-The-Box-logo.png
tags: HackTheBox HTB CTF jwt node.js sqli burp
---

HackTheBox is one of the greatest place to sharpen your skills when it comes to practising real life based penetration testing. 


Before you start you must be a registered member of HTB, and for that you need to prove that you deserve it by hacking through their registration portal!


Moving further,

This particular web challenge was an interesting one for me since it focused on new technologies, having node.js as the web back-end/front-end, SQLite for the database. That's way I decided to write this article to be a walk-through for this challenge.

Let's get started...


**Under Construction** is one of The HackTheBox's web challenges by makelarisjr & makelaris.

![HTB Under Construction Web Challenge](/img/htb- under-construction.png)

This challenge has 30 points for successfully completing it.

Before you start the challenge the need is to connect to the HTB servers via VPN. You will find the connection file under access directory. Once you get it downloaded all you need is to run the below command in your terminal.

```shell
openvpn YourFile.ovpn
```

Once connected you will move onto the web challenges section and click on the drop-down to start the instance. Once the instance has been initiated you will get the IP/Domain:Port to access the web challenge.


On opening the Under Construction Ip/Domain:Port we got a login portal.

![HTB Under Construction Login Page](/img/htb-under-construction-login-page.png)

I have a habit of trying bypass login pages during challenges, so I tried here some SQL injection syntax such as:

`' or ''='`  // for both user and pass fields, useful if SQL comment doesn't work

`' or 1=1-- -`

In this challenge bypass techniques seems not working, so let's move on.

This login page containing a register button which could lead to something, such as a portal with an upload page to upload our shell.

Then I found myself at this page:

![HTB Under Construction Portal Page](/img/htb-under-construction-portal.png)

So that's not the case here. That's why it called "Under Construction" challenge, obviously!

At this stage I have to use burp suite to see what's going on while registering an account and login with it then maybe I can find something suspicious there.

I launched burp suite, enabled proxy and made sure my browser configured with it.

I then repeat the same steps above but this time though burp suite:


First I registered with a new username:

![](/img/register-with-burp1.png)

While proxy is on, I need to click Forward every time a response received, so after clicked on forward I got "registered successfully".

![](/img/register-with-burp2.png)

So far nothing exceptional has happened yet.

Let's login with that user account and click on "send":

![](/img/login-with-burp.png)

**Finally some curious stuff**. 

![](/img/login-with-burp1.png)

As you can see there is another request generated from my browser with a cookie parameter and a long line of an encryption value. Let's save this value in my editor for now, and copy this request to the repeater for further testing.

From repeater, I tried to manipulate the cookie's value a little bit, then I got "internal server error"

![](/img/login-repeater1.png)


To understand what is going on behind the scene during the authentication process I need to see the code. Luckily this challenge provided website's source code along with it. The first file I would like to see is AuthMiddleware.js because this one -as the name suggested- might contained the authentication code.

![](/img/auth-jwt-code.png)

I can see by calling the file "JWTHelper" that the website is using  JSON Web Token, that explains the cookie's value.

What is JWT? 

>It stands for JSON Web Token, an Internet standard for creating data with optional signature and/or optional encryption whose payload holds JSON that asserts some number of claims. The tokens are signed either using a private secret or a public/private key. For example, a server could generate a token that has the claim "logged in as admin" and provide that to a client. The client could then use that token to prove that it is logged in as admin. The tokens can be signed by one party's private key (usually the server's) so that party can subsequently verify the token is legitimate. If the other party, by some suitable and trustworthy means, is in possession of the corresponding public key, they too are able to verify the token's legitimacy.

Now that we understand the nature of the cookie's value used in the application which is nothing but a jwt generated token. 

Let use the [jwt.io online tool](https://jwt.io/) to reveal what is inside that value:

![](/img/jwt-token.png)

The decode box shows my username that I registered with along with the public key. let's save this public key value for now in file named public.key.

Let's go back to the source code and see what happening when the authentication is done, I found what I'm looking for inside routes/index.js

It seems that this file contains all of the login stages.

The line that got my attention was this one:

```
        let user = await DBHelper.getUser(req.data.username);
```

This is an important function, whenever I log on to the system successfully, this function is used. So let's see the details of this function inside /helpers/DBHelper.js file:

![](/img/dbhelper.png)

As you can see the username variable is directly used inside the followed SELECT query which could leads to a potential SQL injection. If this is true then a value like this: `blabla' or 1=1-- -` should returns true and the login should occurs successfully. 

But, how can we do that, I mean, we need to sign our value with the same private key used in the website that mentioned in JWTHelper.js which is impossible except if there is a directory traversal vulnerability which is not exist in this challenge. 

So I searched for any vulnerabilities on JWT that can be exploited, and there I found this awesome tool from ticarpi called [jwt_tool](https://github.com/ticarpi/jwt_tool), to use it, I followed these steps below:

First clone the tool and then install the following library:

```shell
git clone https://github.com/ticarpi/jwt_tool
pip3 install pycryptodomex
```
Then I copied the value I saved before and use it with the tool as follows:

![](/img/jwt-tool1.png)

select 1 and hit Enter

![](/img/jwt-tool2.png)

Then select 0 to continue to the next list and hit Enter, then select 1

![](/img/jwt-tool3.png)

Here you need to insert your new value for the username, let's use the injection value I suggested above then hit enter:

![](/img/jwt-tool4.png)

Next, I need to find out what jwt vulnerability can be used here so we can sign the value without knowing the private key, in this case the best option is the "Sign with HS/RSA key confusion vulnerability", so type 4 and hit enter.

![](/img/jwt-tool5.png)

Next it needs to provide the public key file. save the public key that we saved it before in a file named `public.key` and save it at the same folder of jwt tool.

Finally, type in the key name and hit enter, now I got new jwt token:

![](/img/jwt-tool6.png)

What left now is to send this value through burp and see what we get:

![](/img/login-sqli-repeater.png)

**Voila! No errors received means the query is injected successfully.**

Let's move further until we can find our flag.

First, let's find out if we can inject UNION SELECT. To do so I need to find the right column numbers of the table/ So I go back to jwt tool and repeat all the steps, except the username value should be this time 

```sql
test1' order by 4;--`
```

I chose 4 because normally a users table have 3 columns (id, username, password), so to save my time I tested if four columns would be rejected which could means to me that the table has three columns. 

After generate our token with the injection above and send it through burp I got this results:

![](/img/login-sqli-order-repeater.png)

This error is what I am looking for, since it proof that we have less than four columns. Now let's make a new jwt token with UNION SELECT and three columns.

```sql
test1' and 1=0 union select 111,sqlite_version(),333;--
```
Again to save some time I anticipated that the username is at the second column so I tried to use the function sqlite_version() at the username place to see if it's going to work, and actually it is:

![](/img/login-sqli-version-repeater.png)


Now nothing left but to get our flag, let see what tables we have there using this SQLite query:

```sql
test1' AND 1=0 UNION SELECT 1,(SELECT group_concat(sql) FROM sqlite_master),3;--
```

I made a jwt token out of it and sent it with burp, the results was what I am looking for

![](/img/login-sqli-mster-repeater.png)

**Finally got our flag table. Another SELECT query to this flag table gave me my flag:**

![](/img/login-key-extracted.png)



###Good lessons from this challenge:

**1- Do not ever let the user input used in the application without any validation. Another option is to map values in the array to placeholders (question marks ), for example:**

```sql
db.get(`SELECT * FROM users WHERE username = ?`, username)
```

**2- Received JWT should be validated before used by the application. And follow the [current jwt best practices](https://tools.ietf.org/html/draft-ietf-oauth-jwt-bcp-07) from the JWT developers at Auth0**


Thank for reading.


