+++
date = '2010-05-10T08:47:00+01:00'
draft = false
title = 'Webcam troubleshooting on Linux'
+++

The other day, due to a long distance friendship, I really thought it was time to try and get Skype up and running on one of my Linux machines (currently running the Helena release of Mint).

Upon plugging in the USB cam (a nice little Logitech Quickcam Communicate STX), all initially looked promising with a little blue light briefly coming on.

Downloading and installation of Skype occurred without incident, and I then proceeded to set-up my account. Unfortunately when I can to set-up my video device, even though Skype looked like it could see one, clicking the test button just had no effect and the screen was black.

After a little bit of delving I came across the following article.

https://help.ubuntu.com/community/Webcam
![Testing your webcam](/images/2010/videocam.jpg)

Now the article mentions using the application Ekiga to test that your cam is working, unfortunately this wasn't pre-installed with Helena (although the article does say it is with Ubuntu). Not a problem, this is resolved by a simple `apt-get`. Once installed, after firing up the application I could definitely confirm that the cam was indeed working.

The article then goes on to talk about driver problems caused by changes post Ubuntu 8.10. Since Helena is derived from 9.04 I suspected that this might be the problem. Therefore I changed the application properties as the article described and to my surprise the video started working perfectly in Skype!

Lovely.