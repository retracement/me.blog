+++
date = '2020-02-25T10:28:00+01:00'
draft = false
title = 'Dropbox Appears Stuck Synchronizing Files'
+++

Normally I don’t like off-topic style posts, and [Dropbox](https://www.dropbox.com) is certainly not within my key areas of interest, however, I am an avid user and every now and then I run into a problem that is frustrating and worrying -given a large number of important files I store on that platform. This post is my placeholder to help myself – and hopefully you!

I recently ran into an issue where my Dropbox icon just kept spinning and spinning and believed it was related to some weird conflict resolution problem across different machines that I had run into by accident.

I had somehow made the mistake of renaming a folder name from Pascal casing to lowercase and moved some files around, and ever since that time, my Dropbox would not complete synchronization (of at least that folder casing change) despite other new files seemingly syncronizing fine.

After many days I decided to investigate further.

Looking at the Dropbox icon feedback, all I could see was a large number of files apparently needing downloading, indexing of contents occurring, and a certain number of files uploading. The odd thing was that these stats just did not change.

I decided to try and find out where the process was “stuck” and wanted to detect file access. Since my environment was Linux based I used the following:
ls -l /proc/$(pidof dropbox)/fd | egrep -v 'pipe:|socket:|/dev'

If you are using a Windows environment you can use the procman utility to monitor file access.

What this led me to (aside from access to the dropbox meta-database) was a directory path (coincidently the one and the same I had moved) which contained in its directory structure an empty folder name called .Dropbox.

When I thought about it, these folders had at one time been part of another dropbox installation from many years ago. But surely the fact it was empty meant it was a red herring? Well, I’d recently run into a similar issue with Git, where a .git subfolder had caused all sorts of strange behaviors, so I had very strong suspicions about this, and promptly deleted the folder and restarted Dropbox.

SUCCESS!

Dropbox started up and the icon was reporting movement with indexing, uploading, and downloading.

Conclusion
Just because Dropbox automates the synchronization of files does not mean that it can’t be prone to bugs or strange behaviors. Be careful of what you allow into your Dropbox folder (especially if you are consolidating folders from different machines) and be mindful to work your way backward of things you might have done to break synchronization.