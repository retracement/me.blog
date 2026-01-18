+++
date = '2009-10-13T21:32:15+01:00'
draft = false
title = 'How to connect to SMB targets in Linux'
+++

After much scrabbling around for days I have finally found it.

Let me explain. After a recent rebuild of my Linux Mint laptop I could not for the life of me remember how to connect back to a UNC. Although I obviously had the command syntax:

```bash
sudo mount -t cifs //hostname/share /media/unc -o user=username
```

It just would not work and kept throwing up the following error:

```
mount: cannot mount block device
```

All I remembered doing from previously getting it to work was that (I thought) I needed to apt-get a package. Thankfully after much trawling around and another poster, I came across the package and hey presto all is working again:

```bash
sudo apt-get install smbfs
```

Well now at least my problem is blogged and hopefully there for all eternity for me to refer back to.

Its small things like this that really put off Win-doze users from Linux (at one time myself included) and should imo be included by default in a distro, but I guess you cannot have everything otherwise it (Linux) would not be as fast and customizable for purpose as it is.

Anyway back to copying those large files across my wireless connection…


_**Update (7th January 2013):** As of Linux Mint 14 (Nadia) I no longer see this as a installable package, there is smbnetfs but I couldnt seem to get that to work. I installed cifs-utils -which does seem to work._