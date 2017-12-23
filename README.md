Raspbian Power Monitor
======================

Raspberry Pi UPS Monitor with Email Notifacations Setup Tutorial
----------------------------------------------------------------
Written: 15/07/17

Last updated: 22/12/17

In this tutorial we will use a Raspberry Pi with **"APCUPSD"** installed to monitor the UPS's activity and send us an Email with the power goes out _(Assuming that your Router and Modem are connected to the UPS so they stay on durning the Black-Out)_

First, we will need to install the software that allows our Raspberry Pi to communicate with the UPS. So open up terminal (or SSH into the Pi) and run the following command:

```shell
sudo apt-get update
sudo apt-get install apcupsd
```

Next we need to modify the configuration scripts, so APCUPSD can find the UPS. So use nano and open the following file

```shell
sudo nano /etc/apcupsd/apcupsd.conf
```

And then change the config file so it matches the following

```shell
# vim /etc/apcupsd/apcupsd.conf

#UPSCABLE smart
UPSCABLE usb

#UPSTYPE apcsmart
UPSTYPE usb

#DEVICE /dev/ttyS0
DEVICE
```
Ok, Now that that's done we want to test out connection to the UPS, So make sure your UPS is connect to the pi and run this command:

```shell
sudo apctest
```

if you see an out put similar to this, everything is working so far.

```shell
2017-07-14 21:32:10 apctest 3.14.10 (13 September 2011) debian
Checking configuration ...
Attached to driver: usb
sharenet.type = Network & ShareUPS Disabled
cable.type = USB Cable
mode.type = USB UPS Driver
Setting up the port ...
Doing prep_device() ...

You are using a USB cable type, so I'm entering USB test mode
Hello, this is the apcupsd Cable Test program.
This part of apctest is for testing USB UPSes.

Getting UPS capabilities...SUCCESS
```

Now, we need to tell APCUPSD that we are ready to roll. We do this by editing the "default" file

```shell
sudo nano /etc/default/apcupsd
```
and change the ISCONFIGURED line to =yes

```shell
# Defaults for apcupsd initscript

# Apcupsd-devel internal configuration
APCACCESS=/sbin/apcaccess
ISCONFIGURED=yes
```

Then restart the service

```shell
sudo service apcupsd restart
```

Next, we will set the service to start at boot

```shell
sudo update-rc.d apcupsd defaults
```

then we can check all the info out UPS is sending to the Pi
```shell
sudo apcaccess
```

#### Congrats! APCUPSD is all setup! Now we will configure it to send us an email on power failure.
---------------------------------------------------------------------------------------------------

To do this, we need to edit two scripts, **"On Battery"** and **"Off Battery"**

First we can edit the "On Battery" script (This is the script executed when the power goes out)

```shell
sudo /etc/apcupsd/onbattery
```

Then add the following
changing **GMAIL_ADDRESS** and **GMAIL_PASSWORD** to the Gmail address you want the email to be sent from and the **to_email** to your email where you want to receive the alert.

You can also change the **msg_subjact** and **msg_text** to whatever you would like.

```shell
#!/usr/bin/env python

import smtplib
import email.mime.text
import syslog

syslog.openlog('[UPS]')
def log(msg):
    syslog.syslog(str(msg))

GMAIL_ADDRESS = 'xxx@gmail.com'
GMAIL_PASSWORD = 'xxx'

from_email = GMAIL_ADDRESS
to_emails = ["xxxxxxxxxx@tmomail.net"]  # cell phone address

msg_subject = "ALERT: Main line Power Failure"
msg_text = "Auto Notification"

log(msg_subject)

msg = email.mime.text.MIMEText(msg_text)
msg['Subject'] = msg_subject
msg['From'] = from_email
msg['To'] = ", ".join(to_emails)
s = smtplib.SMTP_SSL('smtp.gmail.com', '465')
s.login(GMAIL_ADDRESS, GMAIL_PASSWORD)
s.sendmail(from_email, to_emails, msg.as_string())
s.quit()
```

Then just edit the Off Battery script to reflect the previous

```shell
sudo /etc/apcupsd/offbattery
```

```shell
#!/usr/bin/env python

import smtplib
import email.mime.text
import syslog

syslog.openlog('[UPS]')
def log(msg):
    syslog.syslog(str(msg))

GMAIL_ADDRESS = 'xxx@gmail.com'
GMAIL_PASSWORD = 'xxx'

from_email = GMAIL_ADDRESS
to_emails = ["xxxxxxxxxx@tmomail.net"]  # cell phone address

msg_subject = "OK: Main line Power Recovered"
msg_text = "Auto Notification"

log(msg_subject)

msg = email.mime.text.MIMEText(msg_text)
msg['Subject'] = msg_subject
msg['From'] = from_email
msg['To'] = ", ".join(to_emails)
s = smtplib.SMTP_SSL('smtp.gmail.com', '465')
s.login(GMAIL_ADDRESS, GMAIL_PASSWORD)
s.sendmail(from_email, to_emails, msg.as_string())
s.quit()
```
-----------------------------------------------------------------
 Now just save everything, reboot the Pi using "sudo reboot" and test the power failure scripts by unpluging the UPS!

 If you have any problems, feel free to contact us by going to our website Pretzelcomputers.com
