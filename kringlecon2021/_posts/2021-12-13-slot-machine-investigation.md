---
layout: post
title:  "4 - Slot Machine Investigation"
date:   2021-12-13 01:34:39 -0800
categories: kringlecon2021 objective4 main
---

This is challenge number 4 in the 2021 SANS Holiday Hack Challenge (https://2021.kringlecon.com/). Objective:

>Test the security of Jack Frost's slot machines. What does the Jack Frost Tower casino security team threaten to do when your coin total exceeds 1000? Submit the string in the server data.response element. Talk to Noel Boetie outside Santa's Castle for help.

We begin this challenge by visiting Jack Frost's slot machines at https://slots.jackfrosttower.com/.

![Objective4 slots](/assets/kringlecon2021/objective4/objective4_slots.jpg)

The first thing I did here was configure the browser to use Burp web proxy, and generated traffic by playing the slot machine.

![Objective4 burp](/assets/kringlecon2021/objective4/objective4_burp.jpg)

Next I sent the spin traffic to the repeater so I could further analyze and modify the requests


![Objective4 burp](/assets/kringlecon2021/objective4/objective4_repeater.jpg)

Inspecting this traffic leads to discovering 3 input parameters


{% highlight shell %}
betamount=1&numline=20&cpl=0.1
{% endhighlight %}

So now we can start tampering with the parameters to see what happens. After trying different values for each of the parameters, we achieve success when we set the cpl to a negative value. When cpl is set to a negative number it always returns with a spin success. After I discovered this, I was able to set the bet amount to a high value and earn more than 1000 credits.

![Objective4 cpl](/assets/kringlecon2021/objective4/objective4_cpl.jpg)

We can also observe the security team response which contains the answer to the challenge!

>I'm going to have some bouncer trolls bounce you right out of this casino!

![Objective4 complete](/assets/kringlecon2021/objective4/objective4_complete.jpg)
