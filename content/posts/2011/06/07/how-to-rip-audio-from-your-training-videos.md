+++
date = '2011-06-07T20:04:00+01:00'
draft = false
title = 'How to rip audio from your training videos'
categories = ['Technology']
tags = ['Audio','Linux','ffmpeg','Utilities']
+++

_**Disclaimer:** Although the following post is Linux related my dear SQL Server geek, you might just find what I am about to say useful in your pursuit of SQL Server greatness. If this possibility doesn’t interest you then I advise to stop reading right... now!_

I don’t normally publish short blog posts and as a result, when I am really busy, it means nothing gets posted due to the large amount of time the bigger articles take. However from time to time I see something that I really want to shout about and this is one of those times.

Regular readers to these pages will be aware that I am a HUGE Linux fan. This might seem a little odd me being a SQL Server freak and all, but I appreciate beauty and greatness and I make no apologies for my schizophrenic software leanings.

Well to prevent this *short* post becoming a long one, I’ll cut to the chase. Most recently I have started working at a new customer site which has meant that my method of travel is probably going to be via the car more often than not. My biggest problem is that my journey time is therefore very unproductive. My immediate thought is to somehow extract the audio track from all my SQL Server training videos and play them on my music player during the drive.

![Boring can be fun!](/images/2011/boringthings.jpg)<br/>
*Boring can be fun!*

After a quick search on the Internet, I believe the answer demonstrates how incredibly powerful and simple your life can be with this Operating System. The answer is to type on the command line `ffmpeg -i <outputfile>` making sure that you add an audio extension to the output filename that ffmpeg can convert to such as `.wmv` or `.mp3`. And if it needs saying, if you don’t already have ffmpeg installed then you simply need to obtain it through your distribution’s package manager. For instance on Ubuntu derived distributions you could install it as easily as typing `apt-get install ffmpeg`.

So from now on dear reader, do not pity me spending an eternity travelling to a customer site whilst twiddling my thumbs learning absolutely zero (if reading books or the Kindle is not possible), just envy me whilst I learn lots of fantastic SQL facts from the audio ripped from my extensive Video training library. So if you would like to do the same as me and you don’t wish to install Linux, all you would need to do is boot your PC into a [Linux Live CD](https://www.linuxmint.com/download.php) then run the install for `ffmpeg` as per my instruction above (please note that this will only install it to memory) and run the extraction process for any of your Videos.

```bash
ffmpeg -i DBA377P.wmv DBA377P.mp3
```

It’s pretty darn easy aint it?

Tomorrow is going to be a good journey!