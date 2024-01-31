# Find the Easy Pass

## Overview

Hello everyone, In this writeup we are going to solve Find the Easy Pass from HackTheBox.

Link for the machine : [https://app.hackthebox.com/challenges/5](https://app.hackthebox.com/challenges/5)

Lets Start 🙌

***

## Initial Setup

First download the given file.

The give file is a zip file. Extract the zip file using the following command and the given password:

Command: `unzip <zip_file>`

password: `hackthebox`

We have got an exe file in the zip name `EasyPass.exe`.

***

## Application Interaction

I ran the executable and it asked for password:

<figure><img src="../.gitbook/assets/Untitled (12).png" alt=""><figcaption></figcaption></figure>

I tried some random password, but it throwed me an error stating `Wrong Password!`.

<figure><img src="../.gitbook/assets/Untitled 1 (12).png" alt=""><figcaption></figcaption></figure>

***

## Enumeration

I used the `strings` tool to check out for useful strings/passwords.

<figure><img src="../.gitbook/assets/Untitled 2 (12).png" alt=""><figcaption></figcaption></figure>

From the response of the `strings` tool, I didn’t find anything interesting.

Next I tried to use a debugger to simulate and check what is happening when I enter a password. In my case, I used the `Immunity Debugger`, use can use a debugger of your choice.

***

## Exploitation

I opened the `EasyPass.exe` executable in `Immunity Debugger` and scrolling up the first tab, looking out for the string `Wrong Password!`. I successfully found the string. I added a breakpoint to that line by pressing the `F2` key.

<figure><img src="../.gitbook/assets/Untitled 3 (12).png" alt=""><figcaption></figcaption></figure>

Next press `F9` key to run the simulate the executable to debug. Once you hit the `F9` key, you can see the program screen asking for password:

<figure><img src="../.gitbook/assets/Untitled 4 (12).png" alt=""><figcaption></figcaption></figure>

Enter some random password and click check password in the program and check the immunity debugger’s 4th tab \[ right side bottom ], you can the the password that you entered. On the next few lines, there is another string `fortran!`, to which the password that we enter might be compared to.

<figure><img src="../.gitbook/assets/Untitled 5 (12).png" alt=""><figcaption></figcaption></figure>

Let’s try `fortran!` in the `Enter Password` field.

<figure><img src="../.gitbook/assets/Untitled 6 (12).png" alt=""><figcaption></figcaption></figure>

We have successfully found the password.

Thank you!!!!!!
