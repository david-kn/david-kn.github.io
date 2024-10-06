---
title: "Django static media failure"
date: 2024-10-01
tags: python django
---

This short article should serve as probably one of "the last resort hint" when you're (most probably in a corporate environment (?)) struggling with a static media failure. Or maybe with even more strange and random behaviour that static media were sometimes loaded, sometimes they weren't or that the static media files took long time (many seconds even up to 20 seconds) to load or to end up failing to load at the end. I guess, you would not end up here, if a standard fix would help you (e.g. check the actual presence of files in your static files folder & permissions to it; in case of multiple container (e.g. docker-compose) setup a proper volume mounting; nginx access and valid routing etc.)

I checked and tried all the standard tips both on the django error page itself and those found on the web... The random, very slow time-to-time loading (strange behavior in my words) made me wonder where is the issue and what the hell is the root cause. Well, if nothing helped, try this: if you're uncertain then ask your network engineers a question about having a HTTPS inspection on some of the network element along the way.. and if there is, ask for disabling it / bypassing it so you can check whether it could play its role in this mystery behaviour. It did in my case!

I was surprised to find this out because we had 2 instances of an app (using Django as a underlying backed) running like a charm and without issues for years and with HTTPS ispection enabled! Then another 2 new instances quite recently spinned off where one was just like the others - running smootly withou any signs of troubles. But the other server, that was a troubleboy. Somehow hit very significantly by CheckPoint firewall inspection and only when the https communication inspection was turned off (set to bypass our specific URL) the static media were loaded quickly and without any errors.

Looking back at the crumbles, yes - AVI balancer logs contained messages about timed out requests, client ended up connections etc. But as the other 3 systems runnig in a very similar / identical environment a network segmentation setup, it didn't cross my mind to actually focus on HTTPS inspection aspect on the firewall.

