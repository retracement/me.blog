+++
date = '2017-04-18T10:32:00+01:00'
draft = false
title = 'Setting up Samba on Linux Mint (the easy way)'
categories = ['Technology']
tags = ['Linux','Samba','Windows','Installation']
+++

![Sharing](/images/2017/hamster_share.jpg)

The Server Message Block (SMB) Protocol is a network file sharing protocol introduced by Microsoft and can be incredibly useful when moving files across multiplatform machines (particularly if your primary machine is a Windows desktop). Samba is a file and print sharing suite of utilities in Linux which uses and provides integration with other machines using the SMB transport.

# What this quick guide covers

If you only want to provide basic folder sharing capabilities from your Linux distribution of choice, configuration and setup of Samba is (in my humble opinion) over complicated at best and a little bit messy at worst.

This quick guide is specifically targetted to the Linux Mint distribution (although will be applicable to many others) and only describes how to share your Linux filesystem folders and does not go into any detail regarding the advanced Samba functionality.

Even though Linux Mint attempts to make folder sharing more user-friendly, I have never had any success using the GUI based procedure, and have even struggled with the following method described in this [article](https://community.linuxmint.com/tutorial/view/1937). Furthermore, I prefer to understand what is being configured behind the scenes, so I shall keep to the point and keep it simple.

---

# Instructions

The following procedure was tested on the latest release of Linux Mint at the time of writing [18.1 "Serena"](https://www.linuxmint.com/rel_serena_cinnamon.php) but I have also used this successfully against [17.1 "Rebecca"](https://www.linuxmint.com/rel_rebecca_cinnamon.php).

**Configure the share**

The first thing you need to do is configure your share in the samba configuration file.
Edit `/etc/samba/smb.conf` and scroll to the Share Definitions section inserting the following section (replacing the relevant names as required).

```text
[Dropbox]
path = /media/mylinuxuser/Dropbox
valid users = mylinuxuser
read only = No
#create mask = 0777
#directory mask = 0777
```

The name in square braces is your desired share name, the path is obviously the real path to the folder you are sharing and the create mask and directory mask parameters define what permissions are assigned to files and directories created through the share. In the section above, the masks are commented out and Samba defaults should be sufficient, but you can override and provide less restrictive permissions if necessary (from a security perspective, please understand what you are doing first!). Ensure you provide at least one valid user to access the share.

To check share setup run:

```bash
testparm
```

**Install Samba**

If Samba is not installed (you can check this by `sudo service –status-all|grep smbd` OR `sudo service –status-all|grep samba`)

```bash
sudo apt-get install samba
```

**Add SMB password**

Ensure you add an SMB password for every valid user that you wish to access the share:

```bash
smbpasswd -a mylinuxuser
```

**Restart SMBD daemon**

Finally, for the new share to be visible to your remote device you will need to restart samba (you will also need to do this every time you add a new share or reconfigure an existing one):

```bash
sudo service smbd restart
```

---

That is all there is to it. Once you have followed these steps, your share will be available to your remote SMB client from your desktop.