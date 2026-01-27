+++
date = '2011-04-01T14:51:00+01:00'
draft = false
title = 'How to run SQL Server on Linux'
categories = ['Technology']
tags =['SQL','Linux','Funny']
+++

*UPDATE 2011-04-10: PLEASE NOTE THIS WAS AN APRILS FOOL JOKE 🙂*

Well I have finally cracked it, after years of trying (and a little help from the R2 release) to get SQL Server to install and run happily on the Linux Platform. I could hardly believe it when it all came together, but I'm very pleased that it has. What is even more exciting is that the performance of the SQL Engine is going through the roof.

I am not entirely sure why this is, but I am assuming it is partly related to the capabilities of the EXT4 filesystem and the latest Linux Kernel improvements.

![Installing in WINE](/images/2011/wine.png)

So here's how I did it:

1. Install WINE
1. Install the .NET 3.5 framework into WINE. This is very important, otherwise the SQL Server 2008R2 installer will fail.
1. Change a couple of WINE settings to allow for 64bit M.U.G extensions.
1. Install the Application Release 1 for Linux Free Object Orientated Libraries by `sudo apt-get install aP-R1l\ f-0Ol`
1. Ensure that you run `setup.exe` from SQL Server R2 Enterprise Edition – please note that SQL Server 2008 Release 1.5 *will not work*, and I additionally had problems with the Standard Edition of R2 (but not entirely sure if there is a restriction here with SQL Standard Edition Licensing on  Linux).

![SQL installer](/images/2011/sqlonlinux.png)
*SQL running happily in WINE on Linux Mint 10 (x64)*

I think that the EXT4 filesystem is key to the success of your installation, since when I attempted this deployment using the EXT2 and EXT3 filesystems, SQL Server appeared to have issues installing.

I hope to provide more instructions and performance feedback to you all over the coming months. Enjoy!