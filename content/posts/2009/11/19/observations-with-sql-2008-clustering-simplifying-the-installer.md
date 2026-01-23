+++
date = '2009-11-19T13:23:00+01:00'
draft = false
title = 'Some observations to SQL 2008 clustering installation'
+++

Well these last few days I've been really in the thick of it trying to sort out clustering on two new UAT servers running Windows 2008 and get SQL 2008 setup in an active/ active configuration. It's been a real struggle for several reasons. The first is that I was only given a couple of days to learn, implement and configure due to significant overruns from the Technical Support team and the second is due to the fact of how much has changed in this latest release of clustering.

The most significant difference from a Windows perspective is the simplifying the setup of the cluster. In fact, I almost found it too easy and straightforward to the extent where it started to frustrate. The prime example of this is the way in which the wizard automatically assumes what you want and selects it without even providing pre-install opportunity to change it. This is the case in point with the Quorum drive. Not only did the wizard decide the "most suitable drive" behind the scenes, it didn't give me the chance to change it and also selected the clustering model that I should be using. Post install it is very easy to change and surprisingly seems to work well, but for a relative expert user of clustering, this is not a welcome change. The previous incarnation of clustering auto-selected a Quorum but provided the chance to change it during the setup.

Within the SQL Server setup, the install/s were fairly simple, but one change that is both good and bad in equal measure is the synchronous nature of the installer. For instance in order to setup the fail-over instance, first, you must install on the first node through the "Create Cluster" option and then once setup on the other node/s adds them through the "Add Node" option. I think this actually works quite well, allows for higher availability of the cluster and possibly is less prone to error – in particular, I am thinking of the SQL 2005 cluster install problem where multi-node install failure is caused by being logged onto more than one of the nodes during install!

The Service Pack installation appears to have been made ever so slightly easier to perform using this technique, and I have to admit that on previous cluster versions my heart would skip a few beats during Service Pack-ing the SQL cluster because of the asynchronous install and it being prone to failure.

All in all, the changes are not too bad (in fact so far a slight improvement) -which unfortunately is not what I would say about Windows 2008. It's simply ghastly.