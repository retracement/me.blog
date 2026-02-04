+++
date = '2010-05-12T20:53:00+01:00'
draft = false
title = 'Installing the Citrix client on Linux'
categories = ['Technology']
tags = ['Linux']
+++

The first step of installing Citrix client on your Linux machine is to first obtain the package. From Citrix's website download the tarball of the Citrix Receiver.

![Citrix clients](/images/2010/citrix1.jpg)

Extract the tarball...

![Extract](/images/2010/citrix2.jpg)

...and run the setupwfc executable. Select the first menu item to start the installer, and keep defaults. Once the installer has completed, exit the installer.

![Run setup](/images/2010/citrix3.jpg)

Once complete you should try launching a published Citrix Application. At this stage you will probably encounter a client error message *"You have not chosen to trust "", the issuer of the server's security certificate (SSL error 61)."*

![Client error](/images/2010/citrix4.jpg)

Now what is particularly confusing if you look at the installed certificates in Firefox you will no doubt see that it is already present (the certificate in the error message could vary between sites). This is all actually a bit misleading, because actually the Citrix client is looking within its own keystore for the certificate. If Firefox does contain the certificate that the Citrix client requires, the procedure to export it is very simple. Select *Edit/ Preferences/ Advanced/ View Certificates/ Authorities/ Export...*

![Export](/images/2010/citrix5.jpg)

After you have exported the certificate in X500 format to the cacerts directory of the Citrix Receiver, the error message should hopefully be no more. Should you struggle to locate the required certificate within Firefox, then your Citrix administrator should be able to provide you with the required one.

![post installation](/images/2010/citrix6.jpg)

Hope this helps.