+++
title= "creating an open source messaging client for fun and profit"
date= 2024-02-15
draft= false
description = "I've started making a persistent messaging platform with rust"
[extra]
has_preview = true
preview_img = "/img/gaterdile_first_look.png"
+++

I've recently taken to hanging out with a group of silly goobers and we've come to rely on discord for planning, sharing info, and just hanging out. Over time I've had some annoyances and we discussed switching to self hosting another platform like matrix but there wasn't anything that fit our needs.

Last month I was poking around with the rocket framework for rust and started putting a chat application together. I've kept at it for the past while and its starting to take shape. This project is focusing on feature completeness first and a stabilised protocol second, this will allow me to peruse the stuff I felt was missing from matrix without getting bogged down. I do plan on implementing xmpp down the road for federation, however I won't be doing that until after the core features are complete and stable. You can find the repo [here](https://github.com/uberfig/gaterdile-old)

The project currently only supports firefox and there's a roadmap on the repo's readme that's roughly the order I plan to tackle things. The list is very subject to change and I don't plan on having the 0.1 release until after hyper's vulnerabilities get fixed

:3

![final result](/img/gaterdile_first_look.png "screenshot of gaterdile as of now. it currently supports message replies, twimoji and multiline comments with many more features to come")

> update: this project was a nice little learning experience but I've moved on to a new project I've named bayou and it may have messaging at some point 