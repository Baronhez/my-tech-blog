---
title: How to get notifications each time someone enters your server
published: 2022-08-27
description: Just in case you want to be notified anytime someone enters your server.
tags: [Server, Linux]
category: Server
draft: false
---
## Get a notification on Telegram or Slack

## Telegram notification 

Firstly, create a telegram bot. To create one, you need to contact the BotFather, which is essentially a bot used to create other bots.

The command you need is /newbot which leads to the following steps to create your bot:

![](https://blog.jonthan.xyz/media/posts/12/telegr.png)

Setting a bot is as simple as this.

The next step is to setup a group where you want your alerts to go and add yourself, the bot you just created and [IDbot](https://t.me/myidbot).

IDBot will give you the CHATID. You can do this by sending "_/getgroupid"_ into the group. It'll return the group ID for the channel, simply prepend a hyphen to the number and that's the chatID: _\-123456789_ as an example. Grab this, plus the HTTP API key of your own bot, and add them to the following script:

```
# Login Notifications
CHATID=CHANGEME
BOTKEY=CHANGEME

# get hostname
HOSTNM=$( hostname )

# get external IP address
IP=$( curl -s http://whatismyip.akamai.com/ )

# find IP address of person last logged in
LOGININFO=$( last -1 -i | head -n 1)

# parse into nice format
LOGININFO1=$( python3 -c "login='$LOGININFO'.split('   '); del login[1]; del login[1]; print(''.join([x.strip(' ') + '   \n' for x in login]));" )

# send information to telegram notification bot
curl -X POST -H 'Content-Type: application/json' -d "{\"chat_id\": \"$CHATID\", \"text\": \"Log in to: $HOSTNM\n$IP\nfrom: $LOGININFO1\", \"disable_notification\": false}" https://api.telegram.org/bot$BOTKEY/sendMessage --silent > /dev/null
```

This can then either be bundled into a bash script and called from your profile or written straight into the profile:

-   If your shell is ZSH: 

```
nano /etc/zsh/zprofile
```

-   If your shell is Bash:

```
  nano /etc/profile
```

## Slack notification

You will need a Slack webhook. [Here](https://api.slack.com/messaging/webhooks) is the documentation about creating one.

After getting a webhook, you should have a URL like this one:

https://hooks.slack.com/services/VALUE1/VALUE2

To get a Slack notification I will use a python script. For this reason, you need to: install python, install the Slack SDK and write the script.

```
sudo apt update
sudo apt install python3
pip install slack_sdk    
nano notification.py
```

The content of **notification.py** must be the following:

```
from slack_sdk.webhook import WebhookClient
import os

url = " https://hooks.slack.com/services/VALUE1/VALUE2"
webhook = WebhookClient(url)
HOSTNM = os.popen('hostname').read()
IP = os.popen('curl -s http://whatismyip.akamai.com/').read()
LOGININFO=os.popen('last -1 -i | head -n 1 | cut -d " " -f 1').read()

response = webhook.send(text='Log in to: {}\n from {}\n by\n User: {} \n'.format(HOSTNM,IP,LOGININFO))
assert response.status_code == 200
assert response.body == "ok"
```

Save the changes, then run the script just for testing purposes:

```
python3 /path/to/the/file/notification.py
```

Then, after checking that the script runs flawlessly, add the command at the end of your profile, the same way you would add the telegram bash script to your profile, depending of the shell you're using.

And that's it, now you should be receiving notifications on Telegram or Slack everytime someone enters your server.

### Credits

Credits to [ZeroSec](https://blog.zsec.uk/) for this info.