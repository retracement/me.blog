+++
date = '2009-09-16T12:20:15+01:00'
draft = false
title = 'Auto generate database GRANTs'
+++
Just a very quick post here, but sometimes when you wish to grant execute right for multiple stored procedures to a particular database user it is easier to simply run the following script in the respective database in order to generate this assignment script. This is far easier than using the 2005 and 2008 GUI interface (and even 2000/ 7) especially if there are a lot of procedures.

```sql
USE
SELECT 'GRANT EXEC ON .' + name + ' TO [user]' 
FROM sysobjects WHERE xtype = 'P' AND
```

_**Update (18th January 2026):** Upon migrating posts to my new site I notice that this script appears incomplete. Todo: Will seek to update it with a working script when I get the chance (despite its age)._