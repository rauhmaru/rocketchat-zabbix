# rocketchat-zabbix
 Sends Zabbix notifications to Rocket.Chat, an Open Source Slack Alternative (Tested on Zabbix 2.4.5. For higher versions, change similar parameters.)

## Rocket.chat
In __Rocket.Chat__, click in *Options > Administration > Integrations > New integration > Incoming WebHook*

* Enabled: True
* Name: Incoming-zabbix
* Post to Channel: #zabbix-channel

Save changes and get __Webhook URL__. Example: http://rocket.chat/hooks/7mbi7xr3akMfKZdna/2wPNTHja7xaREdZY39744eehskYTkw7yx6mwrBpD5Wjphfqg

## Zabbix

### Create a script in 

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
* Send to: http://rocket.chat/hooks/7mbi7xr3akMfKZdna/2wPNTHja7xaREdZY39744eehskYTkw7yx6mwrBpD5Wjphfqg
* Status Enabled.

Save changes.

