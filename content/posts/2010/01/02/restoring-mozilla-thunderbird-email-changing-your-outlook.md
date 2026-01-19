+++
date = '2010-01-02T01:29:32+01:00'
draft = false
title = 'Restoring Mozilla Thunderbird email'
+++

Thunderbird is quiet a nice little app, just about all you want from a mail client and no more. The no more part is good because it keeps it fast. MS Outlook, Microsoft's "killer app" is a very decent client too and I for one am currently tied into it due to years of reliance, but that I sense is changing due to my more flexible attitude of late. The main problem with Outlook as with any of the MS products is the sense that you are firstly "tied in" and secondly that over time they start to look and behave like bloat-ware. This is not necessarily Microsoft's fault, but probably down to the age of the products and endless drive to add new commercial features.

Anyway whilst getting to grips with Thunderbird Inbox migrations I realised that I had missed an old Inbox file which I would like merged into the migrated Inbox. The first migration was a doddle and simply involved copying the entire `~/mozilla-thunderbird` directory (fantastically easy) and launching Thunderbird. In order to "merge" the other missed inbox was a little less obvious due to the fact that there is no Import functionality for Thunderbird -> Thunderbird Mail. The reason there isn't one is because it really isn't necessary.

The first step of the merge was to create a new folder from within the Thunderbird local folders. This now creates two files within the Thunderbird mail folder. The first file will be called exactly the same as the name you gave the local folder and the second file will be the same but have a `.msf` postfix. All you need to do now is close Thunderbird, replace the first file with a copy of the inbox file you wish to merge into the current inbox (retaining the aforementioned local folder name) and simply load Thunderbird back up.

Now you will notice all the old emails in this new folder, so all that is left is to select all these emails and move them into the current inbox. Once done you can then remove the new folder you created earlier and thats it!

…btw….Happy New Year!