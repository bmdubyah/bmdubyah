---
layout: post
title:  "11 - Customer Complaint Analysis"
date:   2022-01-01 01:34:39 -0800
categories: kringlecon2021 objective11 main
---

This is challenge number 11 in the 2021 SANS Holiday Hack Challenge (https://2021.kringlecon.com/). Objective:

>A human has accessed the Jack Frost Tower network with a non-compliant host. Which three trolls complained about the human? Enter the troll names in alphabetical order separated by spaces. Talk to Tinsel Upatree in the kitchen for hints.

The first step in this challenge was to download a packet capture file and open it in Wireshark. There is a mix of TCP and HTTP packets. And you can see after scrolling through there are both GET and POST requests. Given that we are wanting to read complaints that are being submitted, we can create a Wireshark filter to only look at POST requests

{% highlight shell %}
http.request.method == "POST"
{% endhighlight %}


![Objective11 post](/assets/kringlecon2021/objective11/objective11_post.jpg)


Now that we have filtered down to just the POST requests, we can easily step through each request and use the Wireshark display pane to quickly see the complaint messages and the troll id. As we read through the hilarious complaint messages from the trolls we find one thing in common. They are not very well spoken! And they all have names like "Hagg". So when we come across a complaint from "Muffy VonDuchess Sebastian" with the complaint which reads "I have never, in my life, been in a facility with such a horrible staff. They are rude and insulting. What kind of place is this? You can be sure that I (or my lawyer) will be speaking directly with Mr. Frost!" we know this is NOT a troll. This must be the human we are after :)

![Objective11 human](/assets/kringlecon2021/objective11/objective11_human.jpg)

Muffy submitted her room number (1024) with the request, so now we need to go back and look at which trolls complained about the guest in room 1024. An easy way to do this is to update the Wireshark filter to look for all values of "1024" in the form data

{% highlight shell %}
http.request.method == "POST" && urlencoded-form.value contains "1024"
{% endhighlight %}

![Objective11 filtered](/assets/kringlecon2021/objective11/objective11_filtered.jpg)

Now we just need to cycle through and review the names of the trolls that made the post. We discover their names are Flud, Hagg and Yaqh! We have our answer to the challenge!

![Objective11 complete](/assets/kringlecon2021/objective11/objective11_complete.jpg)
