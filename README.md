# rocketchat-zabbix
 Sends Zabbix notifications to Rocket.Chat, an Open Source Slack Alternative (Tested on Zabbix 2.4.5. For higher versions, change similar parameters.)

# Definitions in examples
Zabbix URL: zabbix.example.com
Rocket.Chat URL: rocketchat.example.com

## Rocket.chat
In __Rocket.Chat__, click in *Options > Administration > Integrations > New integration > Incoming WebHook*

* Enabled: True
* Name: Incoming-zabbix
* Post to Channel: #zabbix-channel

Save changes and get __Webhook URL__. Example: http://rocketchat.example.com/hooks/7mbi7xr3akMfKZdna/2wPNTHja7xaREdZY39744eehskYTkw7yx6mwrBpD5Wjphfqg

## Zabbix

### Create a script
In the directory of __AlertScript__, create a file _rocketchat.py_.
```python
#!/usr/bin/python

import sys
import requests
import json

url     = sys.argv[1]
subject = sys.argv[2]
body    = sys.argv[3]

```


### Create a media
In __Administration > Media types > Create media type__:

* Name: rocketchat-script
* Type: script
* Script name: rocketchat.py

Click in add.


### Create a user 
in __Administration > Users > Create user__

Tab User: Define Alias, Name and Groups.

Tab Media:
* Type: rocketchat-script
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
  "text": ":negative_squared_cross_mark: *{TRIGGER.NAME} ({ITEM.VALUE1})*",
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
Click in Enabled, and Add.

#### Conditions tab

* Type of calculation: And/Or (A and B and (C or D)
* Conditions:

Label | Name
------|-----
A | Maintenance status not in _maintenance_
B | Trigger value = _PROBLEM_
C | Trigger severity = _Disaster_
D | Trigger severity = _High

In this configuration, the triggers will only be sent if an event occurs in high severity or disaster. Minor events will not be reported. To add, just include them.

#### Operations tab


