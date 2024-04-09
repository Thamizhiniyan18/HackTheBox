---
description: Templated HackTheBox Web Challenge Writeup by Thamizhiniyan C S
---

# Templated

## Overview

Greetings everyone,

In this write-up, we will tackle Templated from HackTheBox.

Challenge link: [Templated](https://app.hackthebox.com/challenges/templated)

Difficulty Level: Easy

Let's Begin 🙌

First start the instance and navigate to the given IP address.

***

## Information Gathering - Website

<figure><img src="../../../.gitbook/assets/Untitled (1).png" alt=""><figcaption></figcaption></figure>

We got the response back as site under construction, with a message `Proudly powered by Flask/Jinja2`. From this we can devise that the server is made up of Flask and it uses `Jinja2` template engine.

If we try to access a route which is not available, the server responds with a page not found error.

<figure><img src="../../../.gitbook/assets/Untitled 1 (2).png" alt=""><figcaption></figcaption></figure>

If we take a look at the error, we can see that the error has reflected the route which we tried to access. This might be vulnerable to template injection.

***

## Testing Template Injection

So I looked out for Jinja2 payloads and found the following website:&#x20;

{% embed url="https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/README.md" %}

I tried the Detection payload from the above website to check whether it is vulnerable to template injection.

<figure><img src="../../../.gitbook/assets/Untitled 2 (1).png" alt=""><figcaption></figcaption></figure>

I used Postman to send requests because it will be easy to modify and send the request each time.

<figure><img src="../../../.gitbook/assets/Untitled 3 (1).png" alt=""><figcaption></figcaption></figure>

As mentioned in the Detection section, the server thrown a Error. So this site is vulnerable to template injection.

***

## Generating the Payload and Exploiting the Vulnerability

Next I tried some random payloads from the site and found the following which worked:

<figure><img src="../../../.gitbook/assets/Untitled 4 (1).png" alt=""><figcaption></figcaption></figure>

When I tried one of the above mentioned payloads, It worked:

<figure><img src="../../../.gitbook/assets/Untitled 5.png" alt=""><figcaption></figcaption></figure>

Next again I tried some of the payloads in the above section and found the following payload to be working:

<figure><img src="../../../.gitbook/assets/Untitled 6.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/Untitled 7.png" alt=""><figcaption></figcaption></figure>

Okay, now we are able to execute commands and get the output for those commands. So I modified the request to list the contents of the directory:

**Modified Payload:** `{{ self.**init**.**globals**.**builtins**.**import**('os').popen('ls').read() }}`

<figure><img src="../../../.gitbook/assets/Untitled 8.png" alt=""><figcaption></figcaption></figure>

If we take a look at the response we can see the flag file `flag.txt`. This time I modified the payload to view the contents of the `flag.txt` file:

**Modified Payload:** `{{ self.**init**.**globals**.**builtins**.**import**('os').popen('cat flag.txt').read() }}`

<figure><img src="../../../.gitbook/assets/Untitled 9.png" alt=""><figcaption></figcaption></figure>

And we got the flag…….
