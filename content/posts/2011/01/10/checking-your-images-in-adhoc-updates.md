+++
date = '2011-01-10T13:55:00+01:00'
draft = false
title = 'Review image hex data before storing it in a database'
+++

I really had to laugh (and not in a good way) when a request recently dropped into my lap to update a production database server with the script like the one that you will see shortly below. It is my firm belief that NO DML should ever be applied directly to production (Yes you read that right… *NO DML*). You might be thinking to yourself, *"well that's a bit weird, what's the point of having the database then if you can't even query it?"* and that is a valid point. The key words in my original statement are *applied directly* and by that I mean querying or updating the Database via the native SQL application tools such as `sqlcmd` or more likely SSMS. If you are having to do so then I am betting there are usually one of the two following problems with your production system:

1. A buggy application requiring constant DML "fixes"
1. A lack of functionality in your application – such as a missing image updater

The reasons that I believe performing adhoc `UPDATE`s, `DELETE`s and `INSERT`s on production are especially dangerous are numerous. In short you have to understand that a change outside of an application is *UNTESTED*, is therefore a *RISK* and should therefore be considered as something that should require *CHANGE CONTROL*. I am sorry to say that for this particular organization (lets call them ACME Corp), DML changes are not considered to be a RISK and therefore adhoc DML changes are incredibly frequent. I am betting that it is only a matter of time before a serious incident occurs due to the nature of these promotions but the real question is *will they even be aware of what caused a production issue when it finally does hit?* I think that it will be hard to detect the trigger of an incident due to the frequency, lack of testing and uncontrolled nature of these changes.

```sql
UPDATE [ImageRepository]
SET [Description] = 'A very important picture'
,[Image] = 0x89504E470D0A1A0A0000000D494844520000012C0000
01BF08020000006DED7DCF000000017352474200AECE1CE9000000097
048597300000B1300000B1301009A9C180000000774494D4507DB0109
162432D5D14F92000020004944415478DA34BBC9B36ED971DDB732777
3CEF9BADB --please note this HEX string has been reduced for brevity
WHERE ImageId = 1
```

Anyway, lets move onto this particular issue in question. Above you will see a similar script to the one that was passed to me, I have truncated the image string for obvious reasons. It occurred to me *how do I know this is a valid image?*, and my immediate answer was to convert the HEX into a file and have a look. Well I thought I'd at least try this out to prove the case in point -it is a rule of mine that in IT until you have tried something at least once then it is all just theory.

The first thing I needed to do this was a hexeditor, so I grabbed one from the Internet. I wasn't entirely sure that a copy and paste of the hex string straight into the editor would work without any further effort and it was no surprise that on my first attempt it failed. The problem was that I was starting the string from the wrong point and decided to simply skip the 0x at the start of the string. Once my paste into the editor had succeeded I attempted to save to the file system.

![Hex Editor](/images/2011/hex_snippet.png)

So what did I find when I got there? Well not quite the picture that you see below, *but instead it was the electronic signature of two individuals*. I am sure that should those two people be aware that their signature was being sent around in an unencrypted format and to be stored in a database they would be rather concerned about the security implications of this. I have at very least raised the issue with the security compliance team. Anyway this aside, whilst these adhoc changes are performed I have proven that I have a fairly quick test to check that the image data is actually valid before I run through the `UPDATE`s and `INSERT`s of it.

![It wasn't this image](/images/2011/star_shrek.png)