---
layout: post
title: "i-buddy with raspberry pi"
date: 2014-04-08 10:03:00 +0000
comments: true
categories: [hacking,i-buddy,linux,python,raspberry pi]
---
I had the i-buddy plugged in at work to the Ubuntu box for a long time, but one day I switched to a laptop, which makes it a little bit less convenient to plug in the i-buddy. The poor thing was collecting dust in a drawer for many months, until this summer I decided to plug it in to my Raspberry Pi as a stand-alone physical event notifier.

Since I had had luck with the linux drivers last time I decided to try it again on the Pi. No luck. It just wouldn't find it! I was probably doing something stupid. Anyways, I decided to try out a python library instead called [pybuddy](https://github.com/ewall/pybuddy). This was fine by me as most Pi projects are written in Python anyway.

##pybuddy

pybuddy can be used in two ways. You can either use pybuddy.lib in your own program, or you can use pybuddy-daemon.py to start a standalone server which takes commands via port 8888 on 127.0.0.1.

Whether you use pybuddy as library or a server, you might have to configure it. It assumes that your i-buddy's product id is 0x0001. In my case it was 0x0002.

When plugging the i-buddy in, call dmesg:

{% codeblock lang:bash %}
pi@raspberrypi ~/hack $ dmesg
 ...
 
[   23.603277] usb 1-1.2: new low-speed USB device number 5 using dwc_otg
[   23.710086] usb 1-1.2: New USB device found, idVendor=1130, idProduct=0002
[   23.710116] usb 1-1.2: New USB device strings: Mfr=0, Product=2, SerialNumber=0
 
...
{% endcodeblock %}

In this case the product id is 0002, so pybuddy has to be updated [here](https://github.com/ewall/pybuddy/blob/master/pybuddylib.py#L77):

{% codeblock lang:python pybuddylib.py %}
### iBuddy Device Class
class iBuddyDevice:
  USB_VENDOR  = 0x1130
  USB_PRODUCT = int(0x0001) # <-- change this
  BATTERY     = 0
  SETUP       = (0x22, 0x09, 0x00, 0x02, 0x01, 0x00, 0x00, 0x00)
  MESS        = (0x55, 0x53, 0x42, 0x43, 0x00, 0x40, 0x02)
{% endcodeblock %}

Or equivalent in pybuddy-daemon.py.

To make sure it works, you can run pybuddy-daemon.py (it is self-contained, so remember to update the product id in this file as well!):

{% codeblock lang:bash %}
$ su pybuddy-daemon.py
{% endcodeblock %}

Then in a separate terminal, type:

{% codeblock lang:bash %}
$ echo "DEMO" | nc -4u -w1 127.0.0.1 8888
{% endcodeblock %}

and you should see the i-buddy flapping, twisting and flashing. Please note that if your USB power plug isn't powerful enough your Pi might actually reboot instead! I got it working with my 2A power plug (though I think the Raspberry PI is not supposed to use more than 1A anyway). I use a powered USB hub just to make sure.
