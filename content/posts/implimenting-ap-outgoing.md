+++
title= "Implimenting AP, Outgoing Activities"
date= 2024-06-07
draft= true
description = "walktrhough of my own progress implimenting the protocol and things I've learned"
[extra]
has_preview = false
preview_img = "/img/gaterdile_first_look.png"
+++

I've begun implimenting Activitypub with rust and actix, the majority of this is just me going through [Eugen's first post about this from 2018](https://blog.joinmastodon.org/2018/06/how-to-implement-a-basic-activitypub-server/), however that post is out of date and will not sucessfully send an activity

> TLDR, his post doesn't work as mastodon requires a header named digest which contains `SHA-256=` and the base64 encoded "digest" which is just an sha256 hash of the text contents of the body.
