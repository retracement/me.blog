+++
date = '2018-02-06T19:27:00+01:00'
draft = false
title = 'Installing Docker on Linux Mint'
categories = ['Technology']
tags = ['Linux','Installation','Docker']
+++

![Docker Mint](/images/2018/docker-mint.jpg)

Ok, so first things first. This is not a ground shaking post of revelation, and ultimately all the information you need can be found directly from Docker, but like all good posts this is intended to address any confusion or ambiguity you may find when installing Docker on Linux Mint and join all the dots for you.

A web search will almost certainly point you to lots of similar posts, mostly (if not) all of which start instructing you to add unofficial or unrecognized sources, keys etc. Therefore my intention with this post is not to replace official documentation, but to make the process as simple as possible, whilst still pointing to all the official documentation so that you can be confident you are not breaking security or other such things!

You can head over to the following Docker page [Get Docker CE for Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/) for the initial setup and updates, but for simplicity, you can follow along below.

First run the script below in order to update your local files from configured repositories, install required packages, and add the official Docker GPG key.

```bash
# Ensure your repositories are up to date
sudo apt-get update
 
# Install required packages
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
 
#Add Docker's official GPG key:
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
 
#Check the GPG fingerprint successfully added (should see output from this command)
sudo apt-key fingerprint 0EBFCD88
```

Now that the package repository has been added, you can now install Docker Community Edition from apt as follows:

```bash
sudo apt-get update
sudo apt-get install docker-ce
```

Once this has been done, next up is perhaps the most important step (in terms of potential problems) -and that is adding the correct repository for your version of Linux Mint. The issue you face is that Linux Mint uses its own release codenames and so the default script (provided by Docker) picks this up rather than the (required) Ubuntu release -its the `$(lsb_release -cs)` piece of code in their script. Instead, you will need to find out your Mint release name and replace this with the correct Ubuntu package base.

Find out your Linux Mint short codename by running the following:

```bash
lsb_release -cs
```

In my case I find that I am running Linux Mint Serena. Next, you need to find out the short codename of the Ubuntu base build that your edition of Mint is derived from. To do this, visit the [Linux Mint Releases](https://linuxmint.com/download_all.php) page.

From this page, I can see that *Serena* uses the *Xenial* package base (as below):

![Linux Mint Releases](/images/2018/mintrelease.png)

So now all we need to do is add the right repository for the right package base (note I have added *xenial* to the script below). In your case, you may be using a new or older edition of Mint, so simply replace the word *"xenial"* in the script with the correct package base relevant to the version of Mint you are using.

```bash
#Add the Docker repository for the Xenial build
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   xenial \
   stable"
```

Once this is completed you then need to perform the Docker post-installation tasks which you can find [here](https://docs.docker.com/install/linux/linux-postinstall/). These tasks are really there to prevent you having to keep running all Docker commands using the privileged sudo command. For instance, without going any further you *could* already now run the following command to list all current downloaded docker images (there should be none).

```bash
#list docker images (using privelaged mode)
sudo docker image ls
```

But we can avoid having to keep specifying sudo by running the following:

```bash
# Create a new docker group
sudo groupadd docker
 
# Add your user to the docker group.
# This script assumes that your current user
# is the one you want to be a docker admin
sudo usermod -aG docker $USER
```

<mark>It is now important that you log out of your session and back in, in order to pick up the new security context in your session, otherwise you may be greeted with the following text when attempting to run your docker command without sudo:</mark>

```bash
docker images
```

>Got permission denied while trying to connect to the
Docker daemon socket at unix:///var/run/docker.sock:
Get http://%2Fvar%2Frun%2Fdocker.sock/v1.35/images/json:
dial unix /var/run/docker.sock: connect: permission denied

So if you have followed the instructions correctly, you should be able to list docker images (or any other docker command) without requiring sudo as follows:

```bash
#list docker images (non-privelaged mode)
docker image ls
```

And that's it. Docker is now ready for you to run containers on your shiny Linux Mint desktop.