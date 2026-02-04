+++
date = '2011-06-09T20:07:00+01:00'
draft = false
title = 'Add metadata to your audio files'
categories = ['Technology']
tags = ['Audio','Windows','Utilities']
+++

In my recent post [How to rip audio from your training videos](/posts/2011/06/07/how-to-rip-audio-from-your-training-videos.md) I gave Linux a very big thumbs up for providing users with a very easy (free and fast) route to rip audio from our SQL Server training videos. Now as much as I like Linux, it is not the answer to every single problem. In fact it is undeniable that Microsoft has established Windows as a platform of choice for many very polished and useful third party applications -free or otherwise. Sometimes finding open source equivalents on Linux can be a real pain in the posterior since the reality is that every now and again the best application is a Windows application (take SQL Server as an example 🙂) and the application I will demonstrate today is exactly that.

So if you remember where we had left off we had ripped out the audio file from our SQL training video presentation as mp3 format. We are left with something very playable for our media player, but oh impetuous one you have still one little thing to do to give polish to your dirty little audio files. If you upload it to your player in their current state you will find out quite quickly that the files have no meta tags set which consequently means that they will be displayed as unknown in most cases. This is not an ideal situation, especially should we have several of these files.

Rescue comes in the form of a quite awesome piece of freeware (it is in my top 10 list of free tools) and is called [MP3Tag](https://www.mp3tag.de/en/) and runs on windows. This application allows you to edit the meta tags of many formats for one or many files and is quite simply awesome. It does exactly what it says on the tin. The important thing to say here is that as far as your player is concerned, you should ensure you set the Title, Artist and Album fields to ensure that your training audio files can be logically grouped together making it easier for you to find and play them on your media player.

![MP3Tag](/images/2011/mp3tag2.jpg)<br/>
*Set your meta tags*

I've now given you the whole solution to ripping audio files from your training videos so you can become much more productive with your time and hopefully technically excellent. So what are you waiting for?... get ripping and tagging!

p.s. I should add before I go that I didn't have to use Windows to run MP3Tag. If you only have a Linux machine then it runs perfectly happily within [WINE (Wine Is Not Emulator)](https://www.winehq.org/) on Linux, however that choice is totally up to you 🙂