---
layout: post
title: "Learning Android"
date: 2014-12-13 20:17:58 +0000
comments: true
categories: android, learning, lollipop, development
---

I have always liked Android. The very idea that you can replace almost anything by installing an app.

I have tried to find online material for learning to program before, but I find that when you want to learn something
technical you really need a physical book. I have been able to write very simple programs, but the GUI part has just
been a mystery. You need something that tells you _why_ you do something in a certain way.

On amazon, the highest ratated android book as 'Android Programming - The Big Nerd Ranch Guide' by Bill Phillips and
Brian Hardy. After having read about 12 chapter I have to agree that it is a really good book!

Unfortunately it book is now quite old, and they aim to teach you how to write apps that run on devices all the way
down to android 4.2. They say in the book that Froyo (2.2) and Gingerbread (2.3.x) make up of about 50% of all devices
out there. Well according to the [latest statistics](http://developer.android.com/about/dashboards/index.html), that number is now < 10%.

To be honest, since I don't aim to capitalise I wouldn't even have bothered with 50%, only that it would work on my
phone :)

With the arrival of Lollipop I thought I might as well give it a more serious try. No matter what you think about the
new design, at least they've tried to come up with some relatively clear guidelines!

And this is where I have had a couple of bumps from following the book. It looks like they have broken a few widgets
in Lollipop. One widget (view) that seems to have a quite serious overhaul is the DatePicker. The OnChangedDateListener
will NOT be triggered when you are in calendar mode. Instead you have to set `android:datePickerMode=spinner` to get
the old style. You have the same problem with TimePicker. You can read more about it
[here](http://forums.bignerdranch.com/viewtopic.php?f=409&t=8298).

Even with these issues, I would still recommend this book if you are interested in learning Android. I would say that
it is almost a requirement to know Java though, you will be struggling otherwise!
