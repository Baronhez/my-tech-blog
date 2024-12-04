---
title: Useful CMD commands
published: 2022-09-18
description: From time to time, it is useful to know some CMD commands. Here are the ones I consider most useful. 
tags: [CMD, Windows]
category: Windows
draft: false
--- 
**slmgr /upk** Deactivate w10  
**tracert** Path followed by a package  
**tracert -d** Same, but does not resolve domain names  
**cipher /W:C:** Overwrites the disk, also can encrypt files.  
**fc** Compare files  
**sfc /scannow** Checks the Windows 10 system files.  
**DISM /Online /Cleanup /CheckHealth** Checks the system.  
**DISM /Online /Cleanup /ScanHealth** Scans the system  
**DISM /Online /Cleanup-Image /RestoreHealth** Performs a system restore  
**chkdsk** Checks the disk status  
**chkdsk /r /t** Fixes disk errors  
**tasklist** Lists tasks  
**taskkill /f /t /**Kill a task  
**powercfg /energy** Displays power information  
**powercfg /batteryreport** Show battery information  
**netsh wlan show wlanreport** Displays wifi status  
**netsh interface show** interface Displays the network interface  
**netsh interface ip show address | findstr "IP Address"** Displays the IP of each interface  
**netsh wlan show profile** Show the list of wifi networks  
**netsh wlan show profile wifinetwork key=clear |  
findstr “Key Content** Show the passwords of your wifi networks  
**netsh advfirewall set allprofiles state off** Turns off the firewall  
**netsh advfirewall set allprofiles** state on Turn on the firewall  
**netsh interface ip show dnsservers** Displays the different DNS servers  
**nslookup** Allows us to check DNS addresses  
**ipconfig /all** Show network info  
**ipconfig /release** Releases the DHCP IP  
**ipconfig /renew** Renews the DHCP IP  
**ipconfig /displaydns** Show DNS information  
**findstr** Searches for a line of text in a file or in an output (like grep)  
**clip** Sends the output of a command to a clipboard  
**ipconfig /flushdns** Clears DNS information  
**cls** Clears the information screen  
**getmac /v** Displays the MAC address  
**assoc** Displays the file association  
**ping -t** Ping continuously without stopping  
**netstat -af** Displays open ports listening  
**netstat -o** Show active connections  
**netstat -e -t 5** Displays network statistics every 5 seconds  
**route print** Displays the routing table  
**route add****mask**Adds a route to the routing table  
**route delete** Deletes a route  
**shutdown /r /fw /f /f /t 0** Reboots the PC and enters the BIOS  
**wmic path SoftwareLicensingService get OA3xOriginalProductKey** Shows Product Key (Exec with Admin Privileges)  
**runas /use****r:Administrator cmd** This allows you to run any program as Administrator ("cmd" used as example)  
**copy /b image.extension+folder.zip image.extension** Hide zip or rar files inside an image  
**cipher /E** Encrypt files in a folder  
**attrib +h +s +r foldername** Hide a folder from everyone (**attrib -h -s -r foldername** To unhide it)  
**systeminfo** Display detailed system operating and configuration  
info  
**subst q: c://filelocation** Map a regular folder as a mounted drive  
**subst /d q:** Remove the mounted drive  
**curl wttr.in/location** Show the weather  
**curl checkip.amazonaws.com** Check Your Public IP Address  
**curl qrenco.de/https://jonthan.xyz** Generate a QR code of any webpage  
**start https://jonthan.xyz** Open any website  
**del /q /f /s %temp%\\\\\*** Delete temporary files to clear space  
**del /s /q C:\\\\Windows\\\\temp\\** Same as before, but specifying the disk.  

**_Special mention:_** If you search for "_Sysdm.cpl_" in the Search Bar, and then enter the _Advanced_ tab, you can change your PC's performance-related settings based on system animations. This is useful for when you have a low-level PC. This way, you can disable useless animations to get better performance.