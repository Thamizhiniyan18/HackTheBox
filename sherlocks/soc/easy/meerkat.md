---
description: Meerkat HackTheBox SOC Sherlocks Writeup by Thamizhiniyan C S
---

# Meerkat

## Sherlock Scenario

As a fast growing startup, Forela have been utilising a business management platform. Unfortunately our documentation is scarce and our administrators aren't the most security aware. As our new security provider we'd like you to take a look at some PCAP and log data we have exported to confirm if we have (or have not) been compromised.

{% embed url="https://app.hackthebox.com/sherlocks/Meerkat" %}

***

## Setting up the Environment

First download the given file and extract it.

<figure><img src="../../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

We got two files extracted. One is a PCAP file and the other one is a JSON file.

Let's start by analysing the PCAP file.

***

## Task 1

### Question

We believe our Business Management Platform server has been compromised. Please can you confirm the name of the application running?

### Solution

While analysing the PCAP file, looking out for HTTP requests using the filter `http`, found multiple requests to the IP `172.31.6.44`.

<figure><img src="../../../.gitbook/assets/Untitled 3.png" alt=""><figcaption></figcaption></figure>

All the above requests has the word "bonita" in its URL path. So I searched google about it and found that Bonitasoft is a open source BPM software.

<figure><img src="../../../.gitbook/assets/Untitled 4.png" alt=""><figcaption></figcaption></figure>

From this we can say that the application running is Bonitasoft BPM.

**Answer:** `BonitaSoft`

***

## Task 2

### Question

We believe the attacker may have used a subset of the brute forcing attack category - what is the name of the attack carried out?

### Solution

In the last task, we saw that a lot of `HTTP` requests were made to the IP `172.31.6.44`. Majority of  those `HTTP` requests were made to the URL path `/bonita/loginservice`.  If we check the `POST` requests made to the `loginservice`, we can see the credentials that were used to login in the HTTP section as shown below.

<figure><img src="../../../.gitbook/assets/Untitled 5 (1).png" alt=""><figcaption></figcaption></figure>

The credentials differ in each request. To view the credentials from each request as a column, `Right Click` on the `Value` field from the `Form Item` section and select `Apply as Column [ ctrl + shift + I ]`.

<figure><img src="../../../.gitbook/assets/Untitled 6 (1).png" alt=""><figcaption></figcaption></figure>

Now we can see all the different credentials used during the login attempts as shown in the figure below.

<figure><img src="../../../.gitbook/assets/Untitled 7 (1).png" alt=""><figcaption></figcaption></figure>

This is behaviour of attempting different login credentials is known as Credential Stuffing.

{% embed url="https://owasp.org/www-community/attacks/Credential_stuffing" %}

**Answer:** `Credential Stuffing`

***

## Task 3

### Question

Does the vulnerability exploited have a CVE assigned - and if so, which one?

### Solution

On analysing the `meerkat-alerts.json` file, found a lot of `alerts` with a `severity` value of `1`.

So I used the python script given below to extract all the alert signatures with a severity value of 1.

```python
#! /usr/bin/python3
import json

data = json.load(open('meerkat-alerts.json'))

for each in data:
    try:
        if each['alert']['severity'] == 1:
            print(each['alert']['signature'])
    except KeyError:
        pass
```

From the results of the above script, we can see a lot of alert signatures with the CVE `CVE-2022-25237`. This is the CVE of the vulnerability exploited.

<figure><img src="../../../.gitbook/assets/Untitled 8.png" alt=""><figcaption></figcaption></figure>

**Answer:** `CVE-2022-25237`

***

## Task 4

### Question

Which string was appended to the API URL path to bypass the authorization filter by the attacker's exploit?

### Solution

Let's take a look at the PoC of the CVE that we found in the last task. You can find the PoC that I referred to in the following repository:

{% embed url="https://github.com/RhinoSecurityLabs/CVEs/tree/master/CVE-2022-25237" %}

In the description section of the `README.md` file in the above mentioned repository, its mentioned that  '_By appending ";i18ntranslation" or "/i18ntranslation/../" to certain API URLs it is possible to bypass authorization for unprivilged users and access privileged APIs. This allows an API extension to be deployed and execute code remotely._'

<figure><img src="../../../.gitbook/assets/Untitled 9.png" alt=""><figcaption></figcaption></figure>

Also in one of the POST requests to the URL path `bonita/API/pageUpload` in the given PCAP file, we can see the string `;i18ntranslation`.&#x20;

<figure><img src="../../../.gitbook/assets/Untitled 10.png" alt=""><figcaption></figcaption></figure>

This shows that the string which was appended to the API URL path to bypass the authorization filter by the attacker's exploit is `i18ntranslation`.

**Answer:** `i18ntranslation`

***

## Task 5

### Question

How many combinations of usernames and passwords were used in the credential stuffing attack?

### Solution

As we saw in Task 2, a lot of `HTTP` requests were made to the IP `172.31.6.44`. Majority of  those `HTTP` requests were made to the URL path `/bonita/loginservice`.  If we check the `POST` requests made to the `loginservice`, we can see the credentials that were used to login in the HTTP section as shown below.

<figure><img src="../../../.gitbook/assets/Untitled 5.png" alt=""><figcaption></figcaption></figure>

The credentials differ in each request. To view the credentials from each request as a column, `Right Click` on the `Value` field from the `Form Item` section and select `Apply as Column [ ctrl + shift + I ]`.

<figure><img src="../../../.gitbook/assets/Untitled 6.png" alt=""><figcaption></figcaption></figure>

Now we can see all the different credentials used during the login attempts as shown in the figure below.

<figure><img src="../../../.gitbook/assets/Untitled 7.png" alt=""><figcaption></figcaption></figure>

To find the count of the combinations of credentials used in the attack, use the following filter:

`http and (urlencoded-form.value != install)`

Once the filter is applied you can see all the credentials in the `Value` column. Sort the value column by clicking the heading `Value`.

<figure><img src="../../../.gitbook/assets/Untitled 11.png" alt=""><figcaption></figcaption></figure>

Once the Value column is sorted, you can find a total of 59 credentials of which 3 credentials are duplicates. So the total number of credential combinations is 56.

**Answer:** `56`

***

## Task 6

### Question

Which username and password combination was successful?

### Solution

If we check the last login attempt with a response status code of `204`, we can find the successful username and password combination from the given PCAP file.

<figure><img src="../../../.gitbook/assets/Untitled 12.png" alt=""><figcaption></figcaption></figure>

The correct credential combination is `seb.broom@forela.co.uk:g0vernm3nt`.

**Answer:** `seb.broom@forela.co.uk:g0vernm3nt`

***

## Task 7

### Question

If any, which text sharing site did the attacker utilise?

### Solution

On one of the requests made using the bonita application, we can see the usage of the URL [https://pastes.io/raw/bx5gcr0et8](https://pastes.io/raw/bx5gcr0et8), as shown in the below figure.

<figure><img src="../../../.gitbook/assets/Untitled 13.png" alt=""><figcaption></figcaption></figure>

Thus the text sharing site that the attacker used is `pastes.io`.

**Answer:** `pastes.io`

***

## Task 8

### Question

Please provide the filename of the public key used by the attacker to gain persistence on our host.

### Solution

The paste bin URL that we found in the last task is: [https://pastes.io/raw/bx5gcr0et8](https://pastes.io/raw/bx5gcr0et8)

{% hint style="warning" %}
The above mentioned URL didn't work after some time of this writeup since `pastes.io` has changed their domain to `pastebin.ai`. Thus the updated URL is: [https://pastebin.ai/raw/bx5gcr0et8](https://pastebin.ai/raw/bx5gcr0et8)
{% endhint %}

On visiting the above mentioned link, there is a bash script, in which a CURL request made to another pastebin URL.

<figure><img src="../../../.gitbook/assets/Untitled 14.png" alt=""><figcaption></figcaption></figure>

The other pastebin URL found in the above image is: [https://pastes.io/raw/hffgra4unv](https://pastes.io/raw/hffgra4unv)

{% hint style="warning" %}
The above mentioned URL didn't work after some time of this writeup since `pastes.io` has changed their domain to `pastebin.ai`. Thus the updated URL is: [https://pastebin.ai/raw/hffgra4unv](https://pastebin.ai/raw/hffgra4unv)
{% endhint %}

We can see the contents of the above mentioned URL in the following figure:

<figure><img src="../../../.gitbook/assets/Untitled 15.png" alt=""><figcaption></figcaption></figure>

Thus the filename of the public key used by the attacker to gain persistence on our host is `hffgra4unv`.

**Answer:** `hffgra4unv`

***

## Task 9

### Question

Can you confirmed the file modified by the attacker to gain persistence?

### Solution

Pastebin URL : [https://pastes.io/raw/bx5gcr0et8](https://pastes.io/raw/bx5gcr0et8)

{% hint style="warning" %}
The above mentioned URL didn't work after some time of this writeup since `pastes.io` has changed their domain to `pastebin.ai`. Thus the updated URL is: [https://pastebin.ai/raw/bx5gcr0et8](https://pastebin.ai/raw/bx5gcr0et8)
{% endhint %}

On visiting the above mentioned link, there is a bash script, in which a CURL request made to another pastebin URL, which contains a SSH public key as shown in the following image:

{% hint style="warning" %}
The URL shown in the following image didn't work after some time of this writeup since `pastes.io` has changed their domain to `pastebin.ai`. Thus the updated URL is: [https://pastebin.ai/raw/hffgra4unv](https://pastebin.ai/raw/hffgra4unv)
{% endhint %}

<figure><img src="../../../.gitbook/assets/Untitled 15.png" alt=""><figcaption></figcaption></figure>

The SSH public key found above is appended to the `/home/ubuntu/.ssh/authorized_keys` file as shown in the following image.

{% embed url="https://www.ssh.com/academy/ssh/authorized-keys-file" %}

<figure><img src="../../../.gitbook/assets/Untitled 16.png" alt=""><figcaption></figcaption></figure>

Thus the file modified by the attacker to gain persistence is `/home/ubuntu/.ssh/authorized_keys`.

**Answer:** `/home/ubuntu/.ssh/authorized_keys`

***

## Task 10

### Question

Can you confirm the MITRE technique ID of this type of persistence mechanism?

### Solution

You can find the MITRE technique info about gaining persistence using SSH Authorized Keys in the following page:

{% embed url="https://attack.mitre.org/techniques/T1098/004/" %}

<figure><img src="../../../.gitbook/assets/Untitled 17.png" alt=""><figcaption></figcaption></figure>

The MITRE technique ID of this type of persistence mechanism is `T1098.004`.

**Answer:** `T1098.004`

***

## Summary

In this Sherlocks room, I learned about how to analyse PCAP files and JSON log files, specifically:

* Filtering HTTP requests in PCAP files
* Understanding JSON log files
* Using pastebins for downloading scripts
* Persistence mechanism using SSH Authorized Keys
