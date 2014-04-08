---
layout: post
title: "i-buddy with build scripts"
date: 2013-11-02 06:54:00 +0000
comments: true
categories: bash,hacking,i-buddy,linux
---
I had been thinking for quite some time that I wanted to build something that I could control from the computer.

Last year it flared up again, when I wanted to get a more obvious indication of how our builds were doing at the office. I then remembered that I'd seen something a few years back online called the i-buddy.

The [i-buddy](http://www.i-buddy.com) cool computer gadget out there that for some reason has gone unnoticed by most people.

Since it has many different actions (RGB head, flashing heart, flapping wings, twisting) it sounded like just the thing for this purpose.

The i-buddy was originally made solely for use with Live Messenger, but technically inclined people were kind enough to reverse engineer the protocol ([at least this guy](http://imakethin.gs/blog/hacking-the-usb-i-buddy)). We now have a few linux drivers and programming libraries out there that can control it. I had an Ubuntu box at work, and since I'm a bash fan I went with the linux driver route.

Using [ibuddy-lkm](https://github.com/tietomaakari/ibuddy-lkm), I built and installed a kernel driver. The great thing with this driver is that it gives you a number of virtual files with which you control the i-buddy:

    /proc/driver/ibuddy/0/heart
    /proc/driver/ibuddy/0/twist
    /proc/driver/ibuddy/0/wings
    /proc/driver/ibuddy/0/red
    /proc/driver/ibuddy/0/green
    /proc/driver/ibuddy/0/blue

Just write any value to one of the files, and that feature will be toggled!

So all I had to do in my build script was to check the response code, and perform a set of actions depending on that response code:

{% codeblock lang:bash ibuddy.sh %}
#!/bin/bash
 
function send_commands() {
    for command in $*; do
        echo 1 > /proc/driver/ibuddy/0/$command
    done
}
 
function send_multiple_times() {
    for f in $(seq 1 $2); do
        for commands in "$1" "$1"; do
            send_commands $commands
            sleep 0.15
        done
    done
}
 
"$@"
 
ERROR="$?"
 
if [ ! -e "/proc/driver/ibuddy/0" ]; then
    echo "IBUDDY NOT FOUND" >&2
else
 
( \
 
send_commands reset
 
if [ "$ERROR" = "0" ]; then
    send_commands green
    send_multiple_times "green heart" 7
    send_commands reset
else
    send_multiple_times "red wings twist" 15
    send_commands red
fi
 
) &
 
fi
 
exit $ERROR
{% endcodeblock %}

Now I just had to create an alias by adding the following to my .bashrc file:

{% codeblock lang:bash .bashrc %}
alias mci='$HOME/scripts/ibuddy.sh mvn clean install'
{% endcodeblock %}

I have yet to build something myself that is triggered on builds :)