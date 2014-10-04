---
layout: post
title: Running with spotify
comments: true
---

<img src="/public/images/running.jpg" />

I run occasionally. I started running a few years back. When I started, I could not run for more than half a kilometer. Seriously! Excessive panting, chest pain, people staring at me.The works.  Anyhu, I am much better at it now, and music has played a very important role in improving my running. Initially it was just random playlists. After a while i realized that i was running better when a few specific songs where playing on the phone. Joker and the thief, I Don't need no doctor by John Mayer, Rockafeller skank to name a few. 

After a few searches back home, i realized song tempos(BPM) and running was a big thing. There was [jog.fm](jog.fm) . There was a [lifehacker article] (http://lifehacker.com/5905666/music-can-boost-your-running-performance-by-15) about it. I took some time to go through my music collection, used jog.fm to find out the songs with ideal tempos and created new playlists. My distance and pace improved . 

Once i started using spotify, I always wished i could get the songs of a particular tempo directly to spotify ( without manually searching and adding them, once i got a good song from jog.fm ). I found out this [sample website](http://static.echonest.com/playlist-demo/) of echonest which created better playlists based on artists, songs, danceability, energy and tempo. I had to find a way to get the playlist from the site and save it in spotify.
I poked around with spotify desktop apps, and the echonest website, and put up a spotify app called pacemaker. 

The major chunk of work is done by the echonest website. I pulled the code from the website and put it into the app. My only addition was to implement a "Save playlist to spotify" feature. 

Heres how i use this app for running. 

1. Think of a few artists i like to listen to while i run.
2. Fire up spotify on my desktop, type in my artists on the pacemaker app. Add the tempo constraint and click "Go".
3. I like to walk(warm up) at a BPM of 110-130. So i create a playlist for that tempo range.
4. My slow running pace is around 140-160 bpm. I create a playlist for that.
5. My fast running pace is around 160-180. I create a playlist for that too.
6. With this app, i have actually created many playlists, based on many artists, for each tempo range.
7. Before a run, i just go through the playlists, and drag a few songs from each, into a new playlist so that i have a single playlist with each tempo
8. And now that i have spotify on my phone, the playlists can be accessed on the phone.
9. I go running :)

The code is up on github at [Pacemaker](https://github.com/woodstok/pacemaker) . You can find more details on installation and usage inside the repo. Have fun :) 

Here is my [spotify profile](http://open.spotify.com/user/121856107).
<iframe src="https://embed.spotify.com/follow/1/?uri=spotify:user:121856107&size=detail&theme=light" width="300" height="56" scrolling="no" frameborder="0" style="border:none; overflow:hidden;" allowtransparency="true"></iframe>







