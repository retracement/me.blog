+++
date = '2009-06-09T02:09:41+01:00'
draft = false
title = 'Preserve backup chain with COPY ONLY backups'
+++

A welcome addition to SQL 2005 and above is the inclusion of the `COPY_ONLY` clause to the BACKUP statement.

This overcomes the long time problem of breaking the backup chain for your automated backup jobs when taking adhoc backups. For instance, imagine that another dba has taken a full backup in preparation for a change request and then later that day an incident occurs (such as drive failure) which requires a point in time recovery of the database and its logs. Unfortunately your colleague has not informed anyone about their full backup and has gone home for the evening. You will be left in a situation where you can successfully restore your scheduled backups, a differential and possibly some transaction logs but only up to the point before the last full was taken. In order to recover the database properly you need to restore that last full and any subsequent logs, but unfortunately you cannot locate this last full backup.

Another situation could occur if another dba takes an adhoc backup of the transaction log whilst trying to shrink back a transaction log that has overgrown.
Again this would break the transaction log backup chain IF this adhoc backup is not known about or available to the other DBAs.

The `COPY_ONLY` clause overcomes these problems by preserving the backup chain and is to be used exactly for this purpose.

Of course the scenarios mentioned above only occur due to bad practice such as a lack of communication or not retaining backups in a common area but in a real world situation do from time to time occur. This switch is therefore a welcome addition to the backup command and I for one will be insisting upon its use for all adhoc backups when the situation demands it.

For more information on COPY ONLY backups see this [documentation](http://msdn.microsoft.com/en-us/library/ms191495(SQL.90).aspx).