+++
title= "Custom Physics in Bevy"
date= 2023-12-31
draft= false
description = "I had a little fun implimenting simple physics in bevy"
[extra]
has_preview = true
preview_img = "/img/bevyfun.png"
+++

{{ varlet() }}

I picked up bevy and I was having a lot of fun with it so I figured I would impliment a basic physics solver in it for fun 

> please note, it might not run very nicely on mobile and weaker devices as the collision detection is just assuming a collided object just collided in its substep causing the simulation to be less stable the larger the gap between substeps 

you can grab the source code [here](https://github.com/uberfig/simplevarlet)

I made this project following along this fantastic video:

{{ youtube(id="lS_qeBy3aQI", autoplay=false) }}