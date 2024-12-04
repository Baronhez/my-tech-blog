---
title: How to install and configure SSH to be secure
published: 2022-08-27
description: It should be ready in just two minutes, as simple as that.
tags: [Server, Linux, SSH]
category: Server
draft: false
---
I will show you how to configure SSH and make it more secure within a couple of minutes.

### Connect to your server using keys

First, connect to your server using SSH. Oh, don't you have SSH in your server yet? Ok, then, run these commands in your server:

```
sudo apt-get install openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh
```

Apt-get doesn't work? Ok, you're not using a debian based distro in your server, so you have to search how to install packages in your distro (it is a 1 minuto google search, I promise).

Ok, now that SSH is working in our server, connect to the server from another computer, just for testing. If it is working, now run this command in your server (is going to create the directory for the keys and change the permissions of that directory):

```
mkdir ~/.ssh && chmod 700 ~/.ssh
```

Now, we have to generate the keys in our computer.

```
# For Windows:
ssh-keygen -b 4096
# The keys are stored in "%userprofile%\.ssh" as id_rsa and id_rsa.pub by default.
#Now, transfer the public key to the server.
scp $env:USERPROFILE/.ssh/id_rsa.pub Username@123.445.566.233:~/.ssh/authorized_keys    
# Replace the Username and IP with your own Username and Server IP.
# For Linux:
ssh-keygen -b 4096
# The keys are stored in "~/.ssh" as id_rsa and id_rsa.pub by default.
# Now, transfer the public key to the server.
ssh-copy-id Username@123.445.566.233
# Replace the Username and IP with your own Username and Server IP.
# For Mac:
ssh-keygen -b 4096
# The keys are stored in "~/.ssh" as id_rsa and id_rsa.pub by default.
# Now, transfer the public key to the server.
scp ~/.ssh/id_rsa.pub Username@123.445.566.233:~/.ssh/authorized_keys    
# Replace the Username and IP with your own Username and Server IP.
```

Ok, if you connect now to your server through SSH, you will connect directly, without having to input a password. Now, let's continue with the next step.

### Disable password login

Edit the file _/etc/ssh/sshd\_config_ using some editor like Nano or Vim. First, search for the **KbdInteractiveAuthentication** line and change the value to no. This way is not possible to use a keyboard during authentication. Next thing we have to change is  **PasswordAuthentication**. Again, the default value is yes, but we’ll be setting it to no.

Some people may say something like "Change the ssh port to another one", but a real hacker (most likely a bot) will scan your ports searching for it, so this act is stupid and annoying without any good reason.

You can also disable root login, but since your user will surely have sudo privileges... well, I leave up to you to decide if it is worth it (I have my root login disabled).

### Remove weak prime numbers

The client and the server use these moduli to negotiate a secure key. Remove the small ones by runnings these commands:

```
cp /etc/ssh/moduli ~/copy_moduli 
awk '$5 >= 3071' /etc/ssh/moduli > /etc/ssh/moduli.safe
mv /etc/ssh/moduli.safe /etc/ssh/moduli    
```

The first command makes a security copy before changing the moduli in the home folder of your user. The second command looks for lines in /etc/ssh/moduli, which on the fifth line have a value bigger than 3071 and writes those in the moduli.safe file. Then, the third command replace the moduli using the moduli.safe file.

### Allow strong cyphers only

We are going to allow only the most secure cyphers. Create a new file _/etc/ssh/sshd\_config.d/ssh\_hardening.conf_. Then, add this content exactly as it is displayed:

```
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org,diffie-hellman-group16-sha512,diffie-hellman-group18-sha512,diffie-hellman-group-exchange-sha256
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr
MACs hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com,umac-128-etm@openssh.com
HostKeyAlgorithms ssh-ed25519,ssh-ed25519-cert-v01@openssh.com,sk-ssh-ed25519@openssh.com,sk-ssh-ed25519-cert-v01@openssh.com,rsa-sha2-256,rsa-sha2-512,rsa-sha2-256-cert-v01@openssh.com,rsa-sha2-512-cert-v01@openssh.com  
```

## Bonus

With all I mention above should be more than enough, the following tips are optional... and will take you more than just a couple of minutes.

### Use Multi Factor Authentication

First, update your system:

```
sudo apt update
sudo apt upgrade -y
```

Now, install the _libpam-google-authenticator_ package.

```
sudo apt install libpam-google-authenticator
```

For MFA, if I'm not mistaken, you will have to enable again the **KbdInteractiveAuthentication** line in _/etc/ssh/sshd\_config._

Once the PAM app is installed you will need to use a helper app that comes with the PAM to generate a Time-Based One-Time Password (TOTP) key for the user you want to add a second factor to. This key is generated on a per user basis, and as a result is not system-wide. To do this simply run:

```
google-authenticator
```

Upon running the command, you’ll be asked a few questions. The first one asks if authentication tokens should be time-based. We want to select **y**:

```
Do you want authentication tokens to be time-based (y/n): y
```

Once we select yes, the app will generate a QR code that can be scanned within your authenticator app of choice.

This PAM allows for time-based or sequential-based tokens. Using sequential-based tokens mean the code starts at a certain point and then increments the code after every use. Using time-based tokens mean the code changes randomly after a certain time elapses (usually 60s). We’ll stick with time-based because that is what apps like Google Authenticator anticipate.

The remaining questions inform the PAM how to function. We’ll go through them one by one.

```
Do you want me to update your "/home/zephr/.google_authenticator" file? (y/n)  
```

This writes the key and options to the **~/.google\_authenticator** file. If you select no, then the program will quit and nothing is saved which in turn results in the authenticator application not working. Therefore we want to select **yes** for this! 

```
By default, tokens are good for 30 seconds and in order to compensate for
possible time-skew between the client and the server, we allow an extra
token before and after the current time. If you experience problems with poor
time synchronization, you can increase the window from its default
size of 1:30min to about 4min. Do you want to do so (y/n) n
```

Selecting yes for this question enables up to 8 valid codes in a moving four minute window. By answering no, you limit it to 3 valid codes in a 90 second rolling window.

Unless you find issues with the 90 second window, answering no is the more secure choice.

```
If the computer that you are logging into isn't hardened against brute-force
login attempts, you can enable rate-limiting for the authentication module.
By default, this limits attackers to no more than 3 login attempts every 30s.
Do you want to enable rate-limiting (y/n) y
```

Rate limiting means a remote attacker can only attempt a certain number of guesses before being blocked. I recommend you to answer **yes**.

Once you finish this setup, if you want to back up your secret key, you can copy the **~/.google-authenticator** file to a trusted location. 

Once we've been through the steps to configure google authenticator, the next step is to setup our SSH config to allow authenticator to function:

```
sudo nano /etc/pam.d/sshd
```

Add the following line to the bottom of the file:

```
# Standard Un*x password updating.
@include common-password
auth required pam_google_authenticator.so nullok  
```

The **nullok** word at the end of the last line tells the PAM that this authentication method is optional. This allows users without a OATH-TOTP token to still log in using their SSH key. If you remove **nullok** from the line, this MFA would be mandatory.

Next, we’ll configure SSH to support this kind of authentication. Open the SSH configuration file for editing.

```
sudo nano /etc/ssh/sshd_config
```

 We are going to make SSH aware of MFA by adding AuthenticationMethods and UsePAM:

```
# To disable tunneled clear text passwords, change to no here!
PasswordAuthentication no
#PermitEmptyPasswords no

# Change to yes to enable challenge-response passwords (beware issues with
# some PAM modules and threads)


#ChallengeResponseAuthentication yes
KbdInteractiveAuthentication yes
UsePAM yes
AuthenticationMethods publickey,password publickey,keyboard-interactive
```

Save the config and restart the SSH service.

```
sudo systemctl reload sshd.service
```

Now, you have MFA for your SSH.

### Enable Firewall and make use of Rate-Limiting

First, install Uncomplicated Firewall(UFW).

```
sudo apt install ufw
```

Now, set a rate limit:

```
## ufw limit ssh various usage ##
ufw limit ssh comment 'Rate limit for openssh server'
```

When a limit rule is used, ufw will normally allow the connection but will deny connections if an IP address attempts to initiate six or more connections within thirty seconds.

You can also allow or disallow connections from specific IPs:

```
   sudo ufw allow from 1.2.3.4 to any port 22
```

The above example only allows connections from 1.2.3.4 to port 22.

### Set up Fail2Ban

We can go a little further by setting up Fail2Ban. Fail2Ban essentially actively looks out for signs of potential password authentication abuses to filter out IP addresses and regularly update the system firewall to suspend these IP addresses for a certain period.

To install and setup Fail2Ban, run the following commands:

```
sudo apt install fail2ban
sudo cp /etc/fail2ban/jail.{conf,local}
sudo nano /etc/fail2ban/jail.local
```

To configure Fail2Ban, modify the following lines:

```
bantime = 1d
```

If you assign a negative value, the ban will be permanent.

```
findtime = 10m
```

**findtime** defines the time-duration allowed between consecutive login attempts. If the multiple login attempts were made within the time defined by findtime, a ban would be set on the IP.

```
maxretry = 5
```

**maxretry** defines  the exact number of failed login attempts allowed within the findtime. If the number of failed-authorization attempts within the findtime exceeds the maxretry value, the IP would be banned from logging back in.

Fail2ban also allows you to grant immunity to IP addresses and IP ranges of your choice.

```
ignoreip = 127.0.0.1/8 ::1 222.222.222.222 192.168.55.0/24
```

Upon setting all the options you want, simply start the service then check the status:

```
sudo systemctl start fail2ban
sudo systemctl status fail2ban
```

## Credits

I found all this tweaks and tricks in random sources from internet, but to sum it all, all of them can be found in two webpages, so the credit goes to the authors of [https://disknotifier.com/blog/simple-ssh-security/](https://disknotifier.com/blog/simple-ssh-security/) and [https://blog.zsec.uk/locking-down-ssh-the-right-way](https://blog.zsec.uk/locking-down-ssh-the-right-way/)/.