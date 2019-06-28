# rocketchat-zabbix
Sends Zabbix notifications to Rocket.Chat, an Open Source Slack Alternative (Tested on Zabbix 2.4.5. For higher versions, change similar parameters.)

# Definitions in examples

Variable | URL
---------|----
Zabbix URL | zabbix.example.com
Rocket.Chat URL | rocketchat.example.com

## Rocket.chat
In __Rocket.Chat__, click in *Options > Administration > Integrations > New integration > Incoming WebHook*

* Enabled: True
* Name: Incoming-zabbix
* Post to Channel: #zabbix-channel

Save changes and get __Webhook URL__. Example: http://rocketchat.example.com/hooks/7mbi7xr3akMfKZdna/2wPNTHja7xaREdZY39744eehskYTkw7yx6mwrBpD5Wjphfqg

## Zabbix

### Create a script
In the directory of __AlertScript__ (Use `grep ^AlertScript /etc/zabbix/zabbix_server.conf`), create a file _rocketchat.py_.
```python
#!/usr/bin/python

import sys
import requests
import json

url     = sys.argv[1]
body    = sys.argv[2]

r = requests.post(url, body)


```


### Create a media
In __Administration > Media types > Create media type__:

* Name: rocketchat alert
* Type: script
* Script name: rocketchat.py

Script parameters:

* {ALERT.SENDTO}
* {ALERT.MESSAGE}

{ALERT.SENDTO} will be changed by webhook url and {ALERT.MESSAGE} will be changed by alert message, formated in JSON.

![zbx_alert](https://github.com/rauhmaru/rocketchat-zabbix/blob/master/img/zbx_alertconf.PNG)


Click in add.


### Create a user 

__IMPORTANT__

> I recommend that you create a group with read-only permission (zabbix-ro) of the items you want to receive alerts, and put the user rocketchat on it.


in __Administration > Users > Create user__

#### User Tab

Define
* Alias: rocketchat
* Name: rocketchat
* Groups: zabbix-ro

#### Media Tab:
* Type: rocketchat alert
* Send to: http://rocketchat.example.com/hooks/7mbi7xr3akMfKZdna/2wPNTHja7xaREdZY39744eehskYTkw7yx6mwrBpD5Wjphfqg
* Status Enabled.

Save changes.

### Create Actions
In __Configuration > Actions__, select "event source: Triggers", and click in Create action.

#### Action Tab
* Name: Rocket.Chat Notifications
* Default subject: Problem {TRIGGER.NAME}
* Default message:
```json
{
  "text": ":negative_squared_cross_mark: *{TRIGGER.NAME} ({ITEM.VALUE1})*",
  "attachments": [
    {
      "color": "#FF0000",
      "title": "[INCIDENT] {HOST.NAME}  ({HOST.CONN})",
      "title_link": "https://zabbix.example.com/tr_events.php?triggerid={TRIGGER.ID}&eventid={EVENT.ID}",
      "text": "Verified in {TIME}, at {EVENT.DATE}",
      "image_url": "https://zabbix.example.com/chart3.php?&width=900&height=200&period=3600&name={HOST.NAME}: {TRIGGER.NAME}&legend=1&items[0][itemid]={ITEM.ID}&items[0][drawtype]=5&items[0][color]=ff0000"
    }
  ]
}
```

Click in Recovery message

* Recovery subject: OK: {TRIGGER.NAME}
* Recovery message:
```json
{
  "text": ":white_check_mark: *{TRIGGER.NAME} ({ITEM.VALUE1})*",
  "attachments": [
    {
      "color": "#00FF00",
      "title": "[OK] {HOST.NAME}  ({HOST.CONN})",
      "title_link": "https://zabbix.example.com/tr_events.php?triggerid={TRIGGER.ID}&eventid={EVENT.ID}",
      "text": "Verified in {TIME}, at {EVENT.DATE}",
      "image_url": "https://zabbix.example.com/chart3.php?&width=900&height=200&period=3600&name={HOST.NAME}: {TRIGGER.NAME}&legend=1&items[0][itemid]={ITEM.ID}&items[0][drawtype]=5&items[0][color]=00ff00"
    }
  ]
}
```
To understand more about Zabbix Macros, check the link:
 https://www.zabbix.com/documentation/3.4/manual/appendix/macros/supported_by_location

Click in Enabled, and Add.

#### Conditions tab

* Type of calculation: And/Or (A and B and (C or D))
* Conditions:

Label | Name
------|-----
A | Maintenance status not in _maintenance_
B | Trigger value = _PROBLEM_
C | Trigger severity = _Disaster_
D | Trigger severity = _High

In this configuration, the triggers will only be sent if an event occurs in high severity or disaster. Minor events will not be reported. To add, just include them.

#### Operations tab

Operation details:
* Operation type: Send message
* Send to Users: rocketchat
* Send only to: rocketchat-script

Click in Add to Operation details and Add in screen.

![Image of Operations tab](https://paste.opensuse.org/images/48396276.png)


So when a trigger is triggered, this will be the message sent to Rocket.Chat:

![Rocket.Chat Trigger](https://paste.opensuse.org/images/58705750.png)

__Note:__
To view the graphs, you need to be logged in to Zabbix.



## Plus: Zabbix Template
__you need to set the user macro ROOT_URL on the host where the template will be applied.__

`{$ROOT_URL}: Your Rocket.Chat URL (ex.: https://rocket.example.com )`

This template monitors:

### Itens
* MongoDB
  * Memory usage
  * MOngoDB service status
* Node
  * Memory usage node main.js
  * Node on port 3000
  * Number of process node main.js
* Web
  * Check status of webpage
  * Check status of api

### Triggers
* Rocket.Chat is down
* Node main.js is NOT RUNNING
* NodeJS port is closed
* MongoDB port is closed

### Graphics
* Memory usage node main.js
* Memory usage mongoDB
 


