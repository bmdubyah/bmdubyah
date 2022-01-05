---
layout: post
title:  "2 - Where in the World is Caramel Santaigo?"
date:   2021-12-12 01:34:39 -0800
categories: kringlecon2021 objective2
---

This is challenge number 2 in the 2021 SANS Holiday Hack Challenge (https://2021.kringlecon.com/). Objective:

>Welcome! In this game you will analyze clues and track an elf around the world. Put clues about your elf in your InterRink portal. Depart by sleigh once you've figured out your next stop.
>Be sure to get there by Sunday, gumshoe. Good luck!

This challenge seeks to put your Open-source intelligence (OSINT) skills to the test through game play. The goal is to track an elf that is traveling across the world. The idea is to catch this elf and identify them when you do. If you can't then you lose. In order to correctly guess who the elf is once you catch them, you must investigate and collect information as you play. At the same time, you must investigate to learn what location the elf has flown to next. The kicker is that time passes when you investigate and search for clues in the InterRink portal (taking you closer and closer to Sunday). So this game is about balance between investigating too much and too little. Investigate too much, and you run into the deadline. Not enough investigation and you will fail to guess once you finally catch up.

Location clue #1 starts with:

>Newly renovated, the castle is again host to the best holiday hacker conference in the world, KringleCon. Security specialists from around the world travel here annually to enjoy each other's company, practice skills, and learn about the latest advancements in information security.

Not 100% sure where the elf is headed based on this, so we perform our first investigation. The clue is:

>Their next waypoint was something like 51.219, 4.402

This was a long/lat coordinate that I was able to search on Google maps

![Objective2 longlat](/assets/kringlecon2021/objective2/objective2_longlat.jpg)

This tells us our next location is Belgium. However, we still don't know anything about the elf so lets perform another investigation. The second clue is

>They just contacted us from an address in the 81.244.0.0/14 range.

We already know Belgium, but to double check we can run a whois on that IP range

{% highlight shell %}
88665a561824:Downloads bryan$ whois 81.244.0.0
% IANA WHOIS server
% for more information on IANA, visit http://www.iana.org
% This query returned 1 object

refer:        whois.ripe.net

inetnum:      81.0.0.0 - 81.255.255.255
organisation: RIPE NCC
status:       ALLOCATED

whois:        whois.ripe.net

changed:      2001-04
source:       IANA

# whois.ripe.net

inetnum:        81.244.0.0 - 81.244.15.255
netname:        BE-BELGACOM-ADSL1
descr:          ADSL-GO-PLUS
descr:          Belgacom ISP SA/NV
country:        BE
{% endhighlight %}

So now we have double checked that Belgium is the next location, but we still don't have any intel on the elf. Let's perform another investigation

>They were dressed for 2.0°C and clear conditions. Oh, I noticed they had a Star Trek themed phone case.

We now learn that the elf we are trying to catch had a Star Trek themed phone case. We can use InterRink to lookup elves who are fans of Star Trek

![Objective2 fandom](/assets/kringlecon2021/objective2/objective2_fandom.jpg)

![Objective2 possible elves](/assets/kringlecon2021/objective2/objective2_possibleelves.jpg)

Ok, so we now know that the possible elves are

Sparkle Redberry,
Ginger Breddie,
Noel Boetie,
Tinsel Upatree

Let's depart for Belgium!


![Objective2 belgium](/assets/kringlecon2021/objective2/objective2_belgium.jpg)

We arrive and the first investigation clue for tracking our next destination is:

>They said, if asked, they would describe their next location in three words as "frozen, push, and tamed."

So it sounds somewhere cold. Let's look at investigation clue number 2

>They were checking the Ofcom frequency table to see what amateur frequencies they could use while there.

A quick Google search tells us that Ofcom frequencies has something to do with the UK, so we have our next destination. However, we still need to further narrow down the possible elves. Let's look at clue number 3

>They were dressed for 2.0°C and clear conditions. They kept checking their Snapchat app.

This tells us that the elf we are searching for uses Snapchat. We can now add that to our InterRink search

![Objective2 interrink2](/assets/kringlecon2021/objective2/objective2_interlink2.jpg)

![Objective2 upatree](/assets/kringlecon2021/objective2/objective2_upatree.jpg)

Alright, there is only 1 match for elves who preferred social media is Snapchat and who are fans of Star Trek. We have now determined that Tinsel Upatree is our elf! So now we can move forward to our next destination (London,England)

![Objective2 england](/assets/kringlecon2021/objective2/objective2_england.jpg)

Our first investigation clue is

>I think they left to check out the Défilé de Noël.

A quick google search turns up that  Défilé de Noël is associated with Montreal. We know our next destination! And since we already know our elf we don't need to investigate any further. Off to Montreal!


![Objective2 montreal](/assets/kringlecon2021/objective2/objective2_montreal.jpg)

And this is where we catch up to the elf, and make our guess.

![Objective2 guess](/assets/kringlecon2021/objective2/objective2_guess.jpg)

![Objective2 win](/assets/kringlecon2021/objective2/objective2_win.jpg)

And we see that we guessed correctly. Tinsel Upatree was the correct answer!

![Objective2 win](/assets/kringlecon2021/objective2/objective2_complete.jpg)
