---
title: How to get a free VPS in Oracle Cloud
published: 2022-08-31
description: How to get a free VPS in Oracle Cloud with 4 cores, 24 GB of memory and 200 GB of capacity.
tags: [Server, Hosting]
category: Server
draft: false
---
I'll describe how to set up a free virtual server using Oracle Cloud in this post; you may choose to use these machines to test web applications that require a lot more power than shared hosting, or even to mount your own server, to deploy services using docker or Kubernetes. 

First, you will need to create an account at [https://www.oracle.com/cloud/sign-in.html](https://www.oracle.com/cloud/sign-in.html) with an email address and bank card number (don't worry, it's just to check if your account is real, this is still free, they will charge you around 1$ and then they will give it back to you).

You must enter the administration console and select the option "Create a VM instance":

![](https://blog.jonthan.xyz/media/posts/13/oracle.PNG)

Pick that one.

Set the instance name next:

![](https://blog.jonthan.xyz/media/posts/13/instance-2.PNG)

Now, placement:

![](https://blog.jonthan.xyz/media/posts/13/instance-3.PNG)

Leave it as it is

The following part is important:

![](https://blog.jonthan.xyz/media/posts/13/1.PNG)

Click in "Change image"

![](https://blog.jonthan.xyz/media/posts/13/1-2.PNG)

Select "Canonical Ubuntu"

![](https://blog.jonthan.xyz/media/posts/13/1-3.PNG)

Click on "Select image" at the bottom of the screen.

![](https://blog.jonthan.xyz/media/posts/13/1-5.PNG)

Click on "Change shape"

![](https://blog.jonthan.xyz/media/posts/13/1-4.PNG)

Click on "Ampere" option

![](https://blog.jonthan.xyz/media/posts/13/1-6.PNG)

Set it to the maximum "4 cores, 24 GB memory"

![](https://blog.jonthan.xyz/media/posts/13/1-7.PNG)

Then, once again, click on "Select shape"

Now, generate a SSH key pair. If you do not know how to do it, go to [my post about SSH](https://blog.jonthan.xyz/how-to-configure-ssh-to-be-secure/) and learn how to do it.

Once you have your SSH key pair fresh, regardless of the operating system you are using, check this option and upload your public key:

![](https://blog.jonthan.xyz/media/posts/13/1-8.PNG)

Click on "Upload public key files (.pub)"

Location of your SSH keys in different OS:

-   **Windows:** Should be in _%userprofile%\\.ssh_
-   **Linux:** _~/.ssh_ 
-   **MacOS:** _~/.ssh_ 

Upload the file named **id\_rsa.pub** (this is the default name, if you haven't changed it).

![](https://blog.jonthan.xyz/media/posts/13/1-9.PNG)

Click on "Specify a custom boot volume size" and then set that to 200 GB.

Once the instance configuration is finished, you should see something like this:

![](https://blog.jonthan.xyz/media/posts/13/1-10.PNG)

Now that your instance is up and running, it is time to connect via SSH.

![](https://blog.jonthan.xyz/media/posts/13/1-12.PNG)

Use your public IP to connect to your server.

If you don't know how to use SSH:

**On Linux and MacOS**, you should use open-ssh (it is pre-installed in most cases).

```
ssh ubuntu@<your-ip>
```

On Windows, use Powershell (Powershell has open-ssh pre-installed, or at least that's the case on my machine. Use something like [Putty](https://www.siteground.com/tutorials/ssh/putty/) if you get lost following this step).

```
ssh ubuntu@<your-ip> # same as Linux and MacOS
```

And that's it, folks! Now you have a free vps with 4 cores, 24 GB of memory, 200 GB of capacity in like 5 minutes more or less. Ease peasy.