---
description: Blue writeup by Thamizhiniyan C S
---

# Blue

## Overview

Hello everyone, In this blog we are going to solve Blue from HackTheBox.

Link for the machine : [https://app.hackthebox.com/machines/51](https://app.hackthebox.com/machines/51)

Lets Start 🙌

Connect to the HTB server by using the OpenVpn configuration file that’s generated by HTB.

\[ [Click Here](https://help.hackthebox.com/en/articles/5185687-introduction-to-lab-access) to learn more about how to connect to vpn and access the boxes. ]

After connecting to the vpn service, click on Join Machine to access the machine’s ip.

After joining the machine you can see the IP Address of the target machine.

***

## Reconnaissance

First I started by scanning for open ports on the target machine.

### Rustscan

<figure><img src="../../.gitbook/assets/Untitled (6).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/Untitled 1 (7).png" alt=""><figcaption></figcaption></figure>

### Results

From the scan results, I found the following service: Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP). I google about this service and got this:&#x20;

{% embed url="https://www.rapid7.com/db/modules/exploit/windows/smb/ms17_010_eternalblue/" %}

<figure><img src="../../.gitbook/assets/Untitled 2 (6).png" alt=""><figcaption></figcaption></figure>

***

## Exploitation

I opened metasploit and run the above mentioned exploit:

<figure><img src="../../.gitbook/assets/Untitled 3 (7).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/Untitled 4 (7).png" alt=""><figcaption></figcaption></figure>

The attack was successful and I got the meterpreter shell back. Now its time to find the flags.

***

## Getting the User Flag

Found the user flag at `C:\Users\haris\Desktop\user.txt`

<figure><img src="../../.gitbook/assets/Untitled 5 (7).png" alt=""><figcaption></figcaption></figure>

***

## Getting the Root Flag

Found the user flag at `C:\Users\Administrator\Desktop\root.txt`

<figure><img src="../../.gitbook/assets/Untitled 6 (7).png" alt=""><figcaption></figcaption></figure>

We have successfully obtained all the flags.

Thank You!!!!
