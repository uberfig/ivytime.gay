+++
title= "Implimenting The Activitypub Protocol"
date= 2024-08-23
draft= true
description = "walktrhough of my own progress implimenting the protocol and things I've learned"
[extra]
has_preview = false
preview_img = "/img/gaterdile_first_look.png"
+++

I've begun implimenting Activitypub with rust and actix, working off [Eugen's first posts this from 2018](https://blog.joinmastodon.org/2018/06/how-to-implement-a-basic-activitypub-server/), however I found that post is out of date and will not sucessfully send an activity.

TLDR, his post doesn't work as mastodon requires a header named Digest which contains `SHA-256=` and the base64 encoded "digest" which is just an sha256 hash of the text contents of the body. This allows the signature to verify the contents of the post request itself further preventing spoofing 

to properly federate an activity you will need
- a [Webfinger](#webfinger) to locate the `Actor`
- an [Actor](#actor) to sign and send the `Activity`
- an [Activity](#activity) that you want to send

> note the actor must have a public key and you will need to store the actor's private key somewhere in your implimentation

## Webfinger

the webfinger is basically a handy endpoint that we agree to put in the same place, `/.well-known/webfinger` and the webfinger accepts get requests with a query in the format `/.well-known/webfinger?resource=acct:username@example.com`. For simplicity sake you can just hardcode your endpoint to always return the same file for testing or if you have only a single actor on your implimentation.

if you are interested in implimenting it with persistent storage you simply need to strip off the `acct:` and split at the `@` and you'll get the username and domain and if the domain is your own, query your database for the user. something to note, some implementations will also accept queries that are missing `acct:` and the `@domain` so you may wish to keep this in mind.

a simple example webfinger would look like so: 

```json
example taken from Eugen's post

{
	"subject": "acct:alice@my-example.com",

	"links": [
		{
			"rel": "self",
			"type": "application/activity+json",
			"href": "https://my-example.com/actor"
		}
	]
}
```



## Actor

## Activity