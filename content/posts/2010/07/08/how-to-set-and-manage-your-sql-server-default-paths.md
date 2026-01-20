+++
date = '2010-07-08T23:59:00+01:00'
draft = false
title = 'How to set and manage your SQL Server default paths'
+++

Good practice in any SQL installation is to ensure that all builds have commonality. This ensures that administrating more than one server becomes much easier to perform since, as an administrator you should never have any nasty surprises (such as finding the TEMPDB installed on the System partition).

One such default that should always be set for every SQL instance installed is the default file paths for the `DATA` file (`mdf`/`ndf`) drive, the `LOG` file (`ldf`) drive and the `BACKUP` file drive. SQL Server will use and default to values that have been specified on installation, but sometimes it is not always convenient, desirable or possible to set these to your final destinations at this point. In fact in my automated build, I am only really interested in the drive in which the system databases will be located. This removes the dependency upon many multiple drives at the time of installation, and post installation reconfiguration can take place when all drive arrays have been made ready.

Since the values of the default paths are located in the registry of your operating system, you would think that these are not easily accessibly. SQL actually provides some very useful extended stored procedures to access the `registry xp_instance_regread` and `xp_instance_regwrite`. So using these, lets write the script to set the paths now that we have an instance installed.

So now the paths have been set. To ensure that the SQL instance picks these up all you need to do now is to restart the SQL Service.

The biggest problem you will face in standardising your servers is that there will always be the exception to the rule and this can be a real problem to your automation scripts. For instance, if normally you store all your backups on the H drive, you might decide to hard code this drive in all your scripts which performs backup operations. Now best coding practice normally always dictates that hard coding anything is not a good idea (because of the exceptions) and this is where the `xp_instance_regread` can come in. The idea is that you are providing a layer of obfuscation, so that any scripts that need to reference a drive path will obtain its location through the default paths that you have just set. The first step is to create a SQL User Defined Function that calls the default path and simply returns the path to the calling code.

![Function](/images/2010/udf_getbackuppath2.jpg)

Once all your UDFs have been created this allows for all your code to now be re-written to dynamically access a drive or path through them. The other benefit to this is that defaults can be changed (should standards change at a later date) and all dependant code would not need to be updated.