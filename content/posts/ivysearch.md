+++
title= "Making A Search Engine From Scratch"
date= 2026-03-10
draft= false
description = "my silly toy search engine ivysearch"
[extra]
preview_img = "/preview.png"
has_preview = true
+++

I was sitting in my graph theory class one day and we started talking about pagerank so I got the urge to create a little toy search engine that can crawl sites, do basic keyword analysis, and perform the pagerank algorithm 

Its very basic and since I didn't use a database as I felt that was cheating I opted to go for the approach of writing everything to a giant json dump (bonus that its readable) which means that it can get big fast and the whole thing needs to be stored in memory which kinda sucks but it was only meant to be a toy anyway

At some point I may consider indexing the abstracts and titles of wikipedia using some downloads but that will have to wait for me to get another random burst of motivation or someone cares enough to poke me and ask me to do it 

if you wanna try it out the engine is up [here](https://search.ivytime.gay/) and the repo can be found [here](https://github.com/uberfig/ivysearch)