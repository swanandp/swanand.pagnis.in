---
layout: post
title: "I am speaking at RubyConf India"
date: 2014-02-09 23:45:07 +0530
comments: true
categories: 
---

[India's premier Ruby conference][1] is happening at Goa this year, in the third week of March.  This year, my talk proposal got accepted and I am up on [day 1 at 2.30pm][2].

I've been digging into real time web applications for the last few months and SSE (Server Sent Events) and WebRTC caught my attention.  [Pusher][3], a fantastic service that allows you easily implement a pub/sub model in your application, uses SSE as a transport.  And [WebRTC][4] is a recent addition to the fore. Well, not so recent, as Google open sourced it in 2011.  A standard API is currently being drafted by the W3C.

These technologies allow two-way communication between server and client and you no longer have to depend on polling to update the clients.  There are arguments that this breaks the hypermedia agreement, and is harder to scale than traditional stateless Request/Response style of serving web resources.  I haven't yet formed an opinion on this, because I am clearly blinded by the awesomeness and ease of using these technologies.

In my talk I aim to introduce the concepts to the audience and then follow it up with code examples and best practises.  As most of the audience would be interested hear about Rails integration, I will cover all examples in the Rails context.  If you are going to be there, do read the Wikipedia page of the involved concepts and then come to the talk, I am certain you will gain a lot more this way.

Oh and I forgot to mention, RubyConfIndia offers these awesome badge:

<a href="http://rubyconfindia.org/"><img src="http://rubyconfindia.org/2014/images/badges/180/speaker.png"></a>


[1]: http://rubyconfindia.org/2014
[2]: http://rubyconfindia2014.busyconf.com/schedule#activity_52cce8d8852da40010000095
[3]: http://pusher.com/
[4]: http://www.webrtc.org/
