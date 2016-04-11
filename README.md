# Postfix queue monitor

If you host several Wordpress websites, there's a great deal of possibility
that one of them might be compromised and become a SPAM sender. SPAM is usually
rejected by the recipients so queue gets longer and longer. The script
checks the length of Postfix queue and performs appropriate actions depending
on the queue length.

It'll help you protecting your server from becoming SPAM sender
and getting blacklisted in consequence.

There are two levels defined:

* **warning** - if mail queue is longer than warning value, the script prints
warning to *stdout* since the script should be called from *cron*, the output
is sent to *root* . It also will send a message to the SMS or Telegram gateway

* **shutdown** - if queue is longer than specified `shutdown` parameter,
Postfix is shutdown and SMS or Telegram message will be sent

## Installation

### Install required packages:

```bash
apt-get install python3-plumbum
```

### Clone repository and edit config file:

```bash
git clone https://github.com/1connect/mailq-monitor.git
cd mailq-monitor
cp example.config.ini config.ini
```

### SMS or TELEGRAM API

The script can use either a sms gateway or a Telegram Bot.
If you want to use telegram as alert gateway the first step is to create a [Telegram Bot](https://core.telegram.org/bots#create-a-new-bot)

Then configure the `config.ini` file with the telegramapi **TOKEN** value of your bot

Finally, obtain a chat-id by creating a group where you add your bot and yourself or just send a `/start` command to the bot to start a chat with it.

Eitherway, obtain the **chat-id** using this command:

```bash
curl --silent "https://api.telegram.org/bot${TOKEN}/getUpdates" | jq
```

Obtain the chat-id from the response like this:

```json
{
  "ok": true,
  "result": [
    {
      "update_id": 999999999,
      "message": {
        "message_id": 1,
        "from": {
          "id": 424242,
          "first_name": "Bob",
          "last_name": "Marley",
          "username": "bobmarley"
        },
        "chat": {
          "id": 424242,
          "first_name": "Bob",
          "last_name": "Marley",
          "username": "bobmarley",
          "type": "private"
        },
        "date": 1460403309,
        "text": "/start",
        "entities": [
          {
            "type": "bot_command",
            "offset": 0,
            "length": 6
          }
        ]
      }
    }
  ]
}

```

In this case the **chat-id** will be **424242**

```
[telegramapi]
token=XXXXX:XXXXXXX
chat_id=XXXXX
```


### Add the following line to *crontab* (using `crontab -e`):

```
* * * * * /root/mailq-monitor/mailq_check.py
```

## Testing

To check SMS configuration, you can run:

```bash
./mailq_check.py --test-sms
```
