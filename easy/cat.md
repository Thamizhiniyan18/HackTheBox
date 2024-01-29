# Cat

## Overview

Hey everyone, in this write-up we will be solving an HTB challenge Cat.

Link to the challenge: [https://app.hackthebox.com/challenges/cat](https://app.hackthebox.com/challenges/cat)

Let’s Start!!!!!!

***

## Initial Setup

First download and extract the given file.

<figure><img src="../.gitbook/assets/Untitled (11).png" alt=""><figcaption></figcaption></figure>

After extracting the zip, we can see a file named `cat.ab`. I used `file` command on the `Cat.ab` file.

<figure><img src="../.gitbook/assets/Untitled 1 (11).png" alt=""><figcaption></figcaption></figure>

From the output of the file command, we can devise that the given file is android backup file.

***

## Extracting the Backup File

You can extract file from a android backup using `android-backup-extractor`. Download it.

{% embed url="https://github.com/nelenkov/android-backup-extractor/releases" %}

<figure><img src="../.gitbook/assets/Untitled 2 (11).png" alt=""><figcaption></figcaption></figure>

Using the command: `java -jar abe.jar unpack cat.ab file.tar` , you can convert a android backup file to a tar file.

<figure><img src="../.gitbook/assets/Untitled 3 (11).png" alt=""><figcaption></figcaption></figure>

Now its time to extract the tar file using the command `tar xvf file.tar`.

<figure><img src="../.gitbook/assets/Untitled 4 (11).png" alt=""><figcaption></figcaption></figure>

From the output of the `tar` tool, we can see that there are some images in the `shared/0/Pictures` directory.

<figure><img src="../.gitbook/assets/Untitled 5 (11).png" alt=""><figcaption></figcaption></figure>

On taking a look at those images, one of the image contains the flag in it.

<figure><img src="../.gitbook/assets/Untitled 6 (11).png" alt=""><figcaption></figcaption></figure>

We have successfully obtained the flag.

Thank you…….
