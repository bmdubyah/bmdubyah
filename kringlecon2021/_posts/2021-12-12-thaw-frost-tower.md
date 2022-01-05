---
layout: post
title:  "3 - Thaw Frost Tower's Entrance"
date:   2021-12-12 01:34:39 -0800
categories: kringlecon2021 objective3
---

This is challenge number 3 in the 2021 SANS Holiday Hack Challenge (https://2021.kringlecon.com/). Objective:

>Turn up the heat to defrost the entrance to Frost Tower. Click on the Items tab in your badge to find a link to the Wifi Dongle's CLI interface. Talk to Greasy Gopherguts outside the tower for tips.

The first step to this challenge is to find the correct location within the game, and use the wireless dongle to scan for Wi-Fi networks. The correct location happens to be right outside the front of Frost Tower. Issuing the scanning command helps us discover the Thermostat network


{% highlight shell %}
elf@a850e3c95635:~$ iwlist wlan0 scanning
wlan0     Scan completed :
          Cell 01 - Address: 02:4A:46:68:69:21
          Frequency:5.2 GHz (Channel 40)
          Quality=48/70  Signal level=-62 dBm  
          Encryption key:off
          Bit Rates:400 Mb/s
          ESSID:"FROST-Nidus-Setup"
{% endhighlight %}

![Objective3 frostentrance](/assets/kringlecon2021/objective3/objective3_wifisignal.jpg)

Now that we have found it, we issue a command to connect to it

{% highlight shell %}
elf@a850e3c95635:~$ iwconfig wlan0 essid FROST-Nidus-Setup
** New network connection to Nidus Thermostat detected! Visit http://nidus-setup:8080/ to complete setup
(The setup is compatible with the 'curl' utility)
{% endhighlight %}

We are told that we need to visit a setup page to complete setup. For this, we can issue a curl command

{% highlight shell %}
elf@a850e3c95635:~$ curl http://nidus-setup:8080/
◈──────────────────────────────────────────────────────────────────────────────◈

Nidus Thermostat Setup

◈──────────────────────────────────────────────────────────────────────────────◈

WARNING Your Nidus Thermostat is not currently configured! Access to this
device is restricted until you register your thermostat » /register. Once you
have completed registration, the device will be fully activated.

In the meantime, Due to North Pole Health and Safety regulations
42 N.P.H.S 2600(h)(0) - frostbite protection, you may adjust the temperature.

API

The API for your Nidus Thermostat is located at http://nidus-setup:8080/apidoc
{% endhighlight %}

Now we are getting somewhere! It appears that access is restricted until we register, but there is a link to review the API documentation. Let's check that


{% highlight shell %}
elf@abd34e49ca4b:~$ curl http://nidus-setup:8080/apidoc
◈──────────────────────────────────────────────────────────────────────────────◈

Nidus Thermostat API

◈──────────────────────────────────────────────────────────────────────────────◈

The API endpoints are accessed via:

http://nidus-setup:8080/api/<endpoint>

Utilize a GET request to query information; for example, you can check the
temperatures set on your cooler with:

curl -XGET http://nidus-setup:8080/api/cooler

Utilize a POST request with a JSON payload to configuration information; for
example, you can change the temperature on your cooler using:

curl -XPOST -H 'Content-Type: application/json' \
  --data-binary '{"temperature": -40}' \
  http://nidus-setup:8080/api/cooler


● WARNING: DO NOT SET THE TEMPERATURE ABOVE 0! That might melt important furniture

Available endpoints

┌─────────────────────────────┬────────────────────────────────┐
│ Path                        │ Available without registering? │
├─────────────────────────────┼────────────────────────────────┤
│ /api/cooler                 │ Yes                            │
├─────────────────────────────┼────────────────────────────────┤
│ /api/hot-ice-tank           │ No                             │
├─────────────────────────────┼────────────────────────────────┤
│ /api/snow-shower            │ No                             │
├─────────────────────────────┼────────────────────────────────┤
│ /api/melted-ice-maker       │ No                             │
├─────────────────────────────┼────────────────────────────────┤
│ /api/frozen-cocoa-dispenser │ No                             │
├─────────────────────────────┼────────────────────────────────┤
│ /api/toilet-seat-cooler     │ No                             │
├─────────────────────────────┼────────────────────────────────┤
│ /api/server-room-warmer     │ No                             │
└─────────────────────────────┴────────────────────────────────┘
{% endhighlight %}

Interesting, the /api/cooler API does not require registration. And we are told how that we can call this API like so:

{% highlight shell %}
curl -XPOST -H 'Content-Type: application/json' \
  --data-binary '{"temperature": -40}' \
  http://nidus-setup:8080/api/cooler
{% endhighlight %}

So let's make that call and jack up the temp!

{% highlight shell %}
elf@a850e3c95635:~$ curl -XPOST -H 'Content-Type: application/json' \
--data-binary '{"temperature": 120}' \
http://nidus-setup:8080/api/cooler


{
  "temperature": 120.49,
  "humidity": 96.32,
  "wind": 0.0,
  "windchill": 88.0,
  "WARNING": "ICE MELT DETECTED!"
}
{% endhighlight %}

Ice melt detected! We've done our job :)


![Objective3 complete](/assets/kringlecon2021/objective3/objective3_complete.jpg)
