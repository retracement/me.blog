+++
date = '2009-03-31T02:10:10+01:00'
draft = false
title = 'Login failure user is not associated with a trusted SQL Server connection'
+++
Have you ever come across the following login failure within the SQL Server error logs? "Login failed for user ". (Note the empty user name). This is all a bit unusual if you think about it. Since the user attempting the authentication should theoretically have already been authenticated to the windows domain, you would have expected that this user-name would be displayed here in the authentication failure.

![Login failed for user](/images/2009/untrusted.jpg)

One reason I have seen this happen is when a user has already authenticated to Windows and then changes their password – which is a perfectly reasonable thing to do. It seems that logging off and back on again resolves this issue, but you would think Windows would have updated its SID both remotely and locally! It is unclear to me why the warning does not display the user account in this situation, but most probably has something to do with the system attempting to reverse map the SID to a user name. Since the SID is probably out of date it is possible that windows cannot find it in its database.

In conclusion though, I believe these authentication errors can most probably be ignored since they are not necessarily someone trying to gain unauthorized access to the SQL Server, just someone who has not updated their SID.