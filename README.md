Refer to README_main.md for original README file

## Connect Kamailio SIP server and 2 baresip clients

### Configure Kamailio SIP server
* Install Kamailio from official tutorial: https://kamailio.org/docs/tutorials/5.8.x/kamailio-install-guide-git/
* Under "Ready To Rock" section of tutorial, add user agents to database using following commands:
```
kamctl add bob password
kamctl add alice password
```
* Check if user agents are added to database correctly:
```
mysql

mysql> show databases;
mysql> use kamailio;
mysql> show tables;
mysql> select * from subscriber;
```
* Configure Kamailio config file, replace xxx.xxx.xxx.xxx with your IP address:
```
vim /usr/local/etc/kamailio/kamailio.cfg
```
```
/* add local domain aliases - it can be set many times */
alias="mysipserver.com"

/* listen sockets - if none set, Kamailio binds to all local IP addresses
 * - basic prototype (full prototype can be found in Wiki - Core Cookbook):
 *      listen=[proto]:[localip]:[lport] advertise [publicip]:[pport]
 * - it can be set many times to add more sockets to listen to */
listen=udp:xxx.xxx.xxx.xxx:5060
listen=tcp:xxx.xxx.xxx.xxx:5060
```
```
systemctl restart kamailio.service
```
### Configure baresip clients
* Build and install baresip: https://github.com/baresip/baresip
* It creates single configuration folder in ~/.baresip (https://github.com/baresip/baresip/wiki/Configuration)
* Change it with following command, baresip-ua1 is the folder where you built baresip:
```
baresip -f path-to-baresip/baresip-ua1/settings/
```
* Duplicate baresip-ua1, and rename it with baresip-ua2. Now, there are 2 baresip clients. We need to configure each as follows:
#### ua1:
```
vim path-to-baresip/baresip-ua1/settings/config
```
Change line of sip_listen
```
sip_listen              0.0.0.0:5070
```
```
vim path-to-baresip/baresip-ua1/settings/accounts
```
Add following line
```
<sip:bob@mysipserver.com>;auth_pass=password;outbound="sip:xxx.xxx.xxx.xxx:5060;transport=tcp"
```

#### ua2:
```
vim path-to-baresip/baresip-ua2/settings/config
```
Change line of sip_listen
```
sip_listen              0.0.0.0:5050
```
```
vim path-to-baresip/baresip-ua2/settings/accounts
```
Add following line
```
<sip:alice@mysipserver.com>;auth_pass=password;outbound="sip:xxx.xxx.xxx.xxx:5060;transport=tcp"
```

#### Start SIP session
* Start ua1
```
baresip -f path-to-baresip/baresip-ua1/settings/
```
* Start ua2
```
baresip -f path-to-baresip/baresip-ua2/settings/
```
* Create call with basic commands: https://github.com/baresip/baresip/wiki/Using-Baresip:-Basic-Commands
* Check if user agents are registered to db:
```
mysql> select * from location;
```
