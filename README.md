## Overview

Readers will learn how to enable ad-blocking on an EdgeRouter. This how-to guide is inspired by  raspberry pi to an adblocking access-point. Here are just a few of the advantages to blocking ads on the router:

You can apply adblocking to all your clients without installing any browser addons
You can block ads on phone/tablet apps (savings in battery and apps cache)
Your significant other will be exposed to less advertising without even noticing (resulting to reduced consumerism) Smiley Very Happy
The approach overrides the DNS servers of well known ad servers. In my script below, I am using a 'safe' list that is known not to block normal surfing.

The approach doesn't block all ads since it relies only on blocking ad servers and not ads served by the sites themselves, although it works surprisingly well.

### Steps

The steps below should be executed as an administrator or using sudo.

#### Step 1: 
Create the file /config/user-data/update-adblock-dnsmasq.sh and add the following lines (alternatively you can download the file attached to this post and copy it in /config/user-data directory of your router )

```
#!/bin/bash

ad_list_url="http://pgl.yoyo.org/adservers/serverlist.php?hostformat=dnsmasq&showintro=0&mimetype=plaintext"
#The IP address below should point to the IP of your router or to 0.0.0.0
pixelserv_ip="0.0.0.0"
ad_file="/etc/dnsmasq.d/dnsmasq.adlist.conf"
temp_ad_file="/etc/dnsmasq.d/dnsmasq.adlist.conf.tmp"

curl -s $ad_list_url | sed "s/127\.0\.0\.1/$pixelserv_ip/" > $temp_ad_file

if [ -f "$temp_ad_file" ]
then
        #sed -i -e '/www\.favoritesite\.com/d' $temp_ad_file
        mv $temp_ad_file $ad_file
else
        echo "Error building the ad list, please try again."
        exit
fi

/etc/init.d/dnsmasq force-reload
```

In the above script there is a line starting with "#sed ". You can uncomment that, and modify it to remove your favorite sites from the ad blocking list so you can continue to support them. You can add as many of those lines as you'd like. One example would be:

`sed -i -e '/ads\.stackoverflow\.com/d' $temp_ad_file`

#### Step 2: 
Run `chmod a+x /config/user-data/update-adblock-dnsmasq.sh` to make the script executable

#### Step 3: 
Test it by running it `/config/user-data/update-adblock-dnsmasq.sh`
This will create the file `/etc/dnsmasq.d/dnsmasq.adlist.conf` that will be read every time that dnsmasq starts. A sample chunk of the file is as follows:

address=/mbn.com.ua/0.0.0.0
address=/mbs.megaroticlive.com/0.0.0.0
address=/mbuyu.nl/0.0.0.0
address=/mdotm.com/0.0.0.0
address=/measuremap.com/0.0.0.0
address=/media-adrunner.mycomputer.com/0.0.0.0

#### Step 4: 
Test whether adblocking works by running `dig @localhost measuremap.com`
 Note: To use 'dig' you need to install dnsutils using the command `apt-get install dnsutils`. Alternatively you can use the command `host measuremap.com localhost`

```
# dig @localhost measuremap.com

; <<>> DiG 9.7.3 <<>> @localhost measuremap.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 57475
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;measuremap.com.                        IN      A

;; ANSWER SECTION:
measuremap.com.         0       IN      A       0.0.0.0

;; Query time: 3 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Mon Nov 11 23:57:28 2013
;; MSG SIZE  rcvd: 48
```

If you get the value you specified in your script for the pixelserv_ip variable (i.e. 0.0.0.0 in the above example) this means that your dnsmasq is now blocking ad servers

#### Step 5 (optional): 
To update your ad servers lists on a regular basis you can run 'crontab -e' and add the following for automatic updating.
`56 4 * * 6  /config/user-data/update-adblock-dnsmasq.sh`

The current example updates every Sunday at 4:56am

#### Step 6: 
Test via your tablet/phone and/or browser. Although we didn't set a pixelserv server, web pages still render correctly without any problem.

Note: I tried and managed to run a pixelserv server on the router but I had to fiddle with the standard settings of lighthttpd so that it doesn't listen on port 80 (it does only for redirecting browsers to the configured https port). Feel free to try it and remember to change the script so that it uses your router's IP address. I also recommend increasing the Listen option to more than the standard 30, otherwise it chokes and silently dies on the EdgeMAX.

 

Note: After an upgrade, the crontab will not persist and must be re-added.
