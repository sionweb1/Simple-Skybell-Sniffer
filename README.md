# Simple-Skybell-Sniffer (No Homebridge Required)
## A Simple Low Latency Sniffer for Skybell HD
This is a fork of [thoukydides's](https://github.com/thoukydides) [gist](https://gist.github.com/thoukydides/27eb6abd1bb84c78f2f9a4f0d9d111a2) containing the sniffer component files for his [Webhooks sniffer](https://github.com/thoukydides/homebridge-skybell/wiki/Webhooks-Sniffer) for
[Skybell](https://www.amazon.com/SkyBell-SH02300BZ-Bronze-Video-Doorbell/dp/B01DLLU1AI/ref=sr_1_1?ie=UTF8&qid=1536003498&sr=8-1&keywords=skybell+hd), a video doorbell.  This sniffer allows you to trigger events (like playing mp3 files or executing shell scripts) when Skybell is pressed. There are methods to detect a Skybell press by logging into Skybell's servers but it is painfully slow. This sniffer allows a response time within a second or two. See Thoukydides' project for a Homebridge Skybell plugin for a detailed [explanation](https://github.com/thoukydides/homebridge-skybell/). 
 The sniffer can't read the encrypted messages going between Skybell and its AWS servers but it can detect the package length which is enough to detect the button presses. 
 
 This project takes only those scripts from the Thoukydides project which need to run the basic sniffer without Homebridge and makes some revisions to get the sniffer working (I found I had to edit the package order to find the trigger for a button push - see the explanation [here](https://github.com/thoukydides/homebridge-skybell/wiki/Protocol-CoAP) from Thoukydides).  For some reason his script did use some other method to detect button presses but it did not work for me (to see my changes do a diff of skybell-sniff.pl ).

# Requirements
- a Linux server
- a router which the server script can ssh into (routers that have ddwrt firmware do this)

# How it Works
The service runs on your raspberry pi or other linux server and opens up an ssh session to your router where it runs a tcpdump command on your router which listens for the skybell button push. Once the Service detects the button push it runs a custom command. 

# Why Use It ? 

-  Make your doorbell (or motion detection) trigger any command you wish.  
-  No existing doorbell or can't hook up your Skybell to your existing doorbell wiring ? No problem, just have your play an mp3 of a doorbell
- Use [Google Home Notifier](https://github.com/harperreed/google-home-notifier-python) to cast to your Google Home Devices or any chromecast an  MP3 when someone presses the doorbell or have the Google Assistant announce "You have a visitor at your door" 
- Send custom telegram notifications to your phone using the [Telegram CLI](https://github.com/vysheng/tg)


For me this solved a problem of an old "Music & Sound" doorbell and intercom system - I hooked up the Skybell to the M&S transformer and then used this script to play the doorbell sound throughout the house with a chromecast hooked up to the aux port of the Music & Sound system. I didn't have to replace or install transformers or do any major rewiring. 

## Installation
This is basically a repeat of the [Homebridge Webhooks Sniffer](https://github.com/thoukydides/homebridge-skybell/wiki/Webhooks-Sniffer)
without the Homebridge stuff. 
1. Configure the router to allow ```ssh``` login without a password, e.g. by adding your public key to its ```authorized_keys``` file.
2. On your raspberry pi (or other linux distribution) save the files to appropriate locations.  On a recent Ubuntu destribution (16.04) using ```systemd``` they are placed as follows:

File | Location
---- | --------
`skybell-sniff-pl` | `/usr/local/bin/skybell-sniff.pl` (or an alternative location of your choice)
`skybell-sniff.service` | `/etc/systemd/system/skybell-sniff.service`
`skybell-sniff` | `/etc/default/skybell-sniff`


### Configure `/etc/systemd/system/skybell-sniff.service`

The following configuration options may need to be tweaked:

Option      | Description
----------- | -----------
`ExecStart` | If the Perl script has been saved to an alternative location then update the path to match.
`User`      | Optionally change this to a suitable non-privileged account. This is recommended but not essential.

### Configure `/etc/default/skybell-sniff`

Modify the following environment variables as appropriate:

Variable                    | Examples                    | Description
--------------------------- | --------------------------- | ----------
`SKYBELL_HOST`              | `skybell` or `192.168.0.42` | The hostname or IP address of the doorbell.
`SKYBELL_NAME`              | `'Doorbell'`                | The name of the doorbell within the SkyBell HD app.
`ROUTER_HOST`               | `192.168.0.1`               | The hostname or IP address of the gateway/router.
`ROUTER_USER`               | `root`                      | The `ssh` username for connecting to the gateway/router to run `tcpdump`.

`SKYBELL_CMD_TCPDUMP` and `SKYBELL_CMD_MOTION` may also be modified to change how `tcpdump` is launched, or to execute a different command when a button press or motion is detected.

## Starting and Stopping

Start the doorbell sniffer process using:
```
sudo systemctl daemon-reload
sudo systemctl start skybell-sniff
sudo systemctl enable skybell-sniff
```

The following commands can be used to control and monitor the process:

Command                                | Description
-------------------------------------- | -----------
`sudo systemctl daemon-reload`         | Re-read `/etc/systemd/system/skybell-sniff.service` after any changes
`sudo systemctl start skybell-sniff`   | Start the process now
`sudo systemctl stop skybell-sniff`    | Stop the process
`sudo systemctl enable skybell-sniff`  | Start process automatically at boot
`sudo systemctl disable skybell-sniff` | Prevent the process from being automatically started
`sudo systemctl status skybell-sniff`  | Check status
`journalctl -u skybell-sniff -f`       | Monitor activity
