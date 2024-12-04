---
title: How to activate Windows 10, Windows 11 and Office for free
published: 2022-08-25
description: Because paying is for suckers.
tags: [Windows, Pirate, CMD]
category: Windows
draft: false
---

Hello! If you want to configure a computer to have Windows 10 activated for free (all of this works also for Windows 11 because Microsoft devs repeated the licensing system and like 90% of the internal functionality) then you will have to follow some simple steps.

**Step 1:** You must get an ISO of w10 or w11 from anywhere.  
You can get it from [official sites](https://www.microsoft.com/en-us/software-download) or from [mirrors](https://linuxct.space/Windows/):

All the ISOs, include the Office installer, can be found in this page: [https://massgrave.dev/genuine-installation-media.html](https://massgrave.dev/genuine-installation-media.html)

**W11 ISO**  
Microsoft official:  
[https://www.microsoft.com/en-us/software-download/windows11  
](https://www.microsoft.com/en-us/software-download/windows11)Mirror:  
[https://linuxct.space/Windows/](https://linuxct.space/Windows/)

**W10 ISO:**  
Microsoft official:  
[https://www.microsoft.com/en-us/software-download/windows10  
](https://www.microsoft.com/en-us/software-download/windows10)Pirated Windows, in case you want to choose a specific version:  
[https://tb.rg-adguard.net/public.php](https://tb.rg-adguard.net/public.php)

_**Attention:**_  
As such, if you feel like it, you can download an ISO previously debloated by others and then you can go straight to the heart of the matter.  
[https://ameliorated.info/documentation.html](https://ameliorated.info/documentation.html)

Ok, and now?

**Step 2:** Flash the ISO on a pendrive with at least 8 gigabytes of space (it's a lot, I know, but it's a big ISO).  
You have several options to do it, but if you already have another PC running Windows, I recommend you Rufus:

Here is a quick tutorial:

[https://www.youtube.com/watch?v=Wt0Q-DBejIw](https://www.youtube.com/watch?v=Wt0Q-DBejIw)

Here you can download the program:

[https://github.com/pbatard/rufus/releases/download/v3.17/rufus-3.17.exe](https://github.com/pbatard/rufus/releases/download/v3.17/rufus-3.17.exe)

If you have a Linux PC, I recommend you Ventoy:

[https://linuxkamarada.com/en/2020/07/29/ventoy-create-a-multiboot-usb-drive-by-simply-copying-iso-images-to-it/](https://linuxkamarada.com/en/2020/07/29/ventoy-create-a-multiboot-usb-drive-by-simply-copying-iso-images-to-it/)

Or SUSE imagewriter: [https://software.opensuse.org/package/imagewriter](https://software.opensuse.org/package/imagewriter)

**Step 3:** Insert the flashed usb into the machine where you want to install windows and once the installation environment starts, we can freely hit next until we get to the part that asks for an activation code.  
We have two options:  
We put one of these:  
[https://gist.github.com/Azhe403/d261f2aadccfc2fb20e00414342a3093  
](https://gist.github.com/Azhe403/d261f2aadccfc2fb20e00414342a3093)Or we click the "I do not have an activation code" to continue with the installation.

Well, assuming you haven't put in one of the activation codes I've put above, let's do a little trick to activate Windows.

**Step 4:** You should be, by the end of the installation, in front of a empty windows desktop. Now we have to activate windows.

## Online activation

Execute this code in the powershell command line with admin privileges:

```
irm https://massgrave.dev/get | iex
```

That's it, skip to the next step, the one with the **comand Line** **Interface**.

## Traditional activation with .zip file

If you want documentation for this step, go to https://massgrave.dev/  

Let's enter this webpage:  
[https://github.com/massgravel/Microsoft-Activation-Scripts/releases](https://github.com/massgravel/Microsoft-Activation-Scripts/releases)

![Pick the latest release of the program, in my case was the 1.6 version.](https://blog.jonthan.xyz/media/posts/5/1-2.PNG)

Pick the latest release of the program, in my case was the 1.6 version. Download the .7z file.

Now, we have to unzip this file. But we do not have any zip program yet. Let's jump there to another webpage in order to download useful programs. This webpage is [Ninite](https://ninite.com/). This webpage allows us to install multiple program just by executing one single exe file. Once you have marked the programs that you want to install, click the button "Get your ninite". Then, run the exe file you just downloaded.

Ok, now that you have installed 7zip (I recommend that one, it's the best for me), unzip the file we previously downloaded. For me was _MAS\_1.6\_Password\_1234.7z._ The password is 1234 (obviously).

![Now, enter the ](https://blog.jonthan.xyz/media/posts/5/1-3.PNG)

Now, enter the "All in one" version.

![Execute this file with admin privileges. Right click and then ](https://blog.jonthan.xyz/media/posts/5/1-4.PNG)

Execute this file with admin privileges. Right click and then "Run as administrator".

## Command Line Interface

![If you want to activate only Windows, enter ](https://blog.jonthan.xyz/media/posts/5/1-5.PNG)

If you want to activate only Windows, enter "1" using the keyboard. If you want to use the KMS38 method, enter "2". Windows will be activated until 2038. If you want to activate Office (you have it already installed in your computer), enter "3". Take into account that this activation method only last for 180 days, but it can renovate itself.

![If you chose HWID, you’ll see a second screen which asks if you’d like to continue with activation. Type 1 to continue. The process from here will be automatic.](https://blog.jonthan.xyz/media/posts/5/1.webp)

If you chose HWID, you’ll see a second screen which asks if you’d like to continue with activation. Type 1 to continue. The process from here will be automatic.

![Once finished, you can press any key to go back and then press 8 to close the window.](https://blog.jonthan.xyz/media/posts/5/1-2.webp)

Once finished, you can press any key to go back and then press 8 to close the window.

Let the script run until all it's done. Then, check if windows have been activated.

To check activation status in Windows 10, select the Start button, and then select Settings > Update & Security and then select Activation . Your activation status will be listed next to Activation.

![That’s it! You’re activated.](https://blog.jonthan.xyz/media/posts/5/1-3.webp)

That’s it! You’re activated.

Attention: An alternative to all of this is to use  
[https://github.com/kkkgo/KMS\_VL\_ALL](https://github.com/kkkgo/KMS_VL_ALL)

Ok, by now you should have a windows 10 or 11 installed and activated in your computer :)

**Extra optional steps:** 

You should update Windows right now, even if the install is fresh.  
Debloat Windows deleting telemetry and other stuff: Run this on Powershell:

```
iwr -useb https://git.io/debloat|iex
```

Or this: 

```
iex ((New-Object System.Net.WebClient).DownloadString('https://git.io/JJ8R4'))
```

Disable startup programs. Enter Windows Key+R, write **shell:startup**, hit enter and delete all the programs which start on startup.  
Enter Windows Key+R, write **appwiz.cpl**, hit enter and uninstall all the unnecessary programs in your machine.