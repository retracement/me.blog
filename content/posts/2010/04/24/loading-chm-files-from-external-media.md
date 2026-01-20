+++
date = '2010-04-24T22:43:00+01:00'
draft = false
title = 'How to load CHM files from external media in Windows'
+++

Another very quick but useful tip that I had to do several years back (and most recently a few days ago) was to have to tell Windows to ignore the usual security constraints and allow other data sources. Otherwise when you attempt to load e-books from these data sources, otherwise you will be blocked.

To work around this you simply need to add the following registry file to your registry. Note that in my case the J: drive is a mapped network drive.

```text
REGEDIT4
[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\HTMLHelp]
[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\HTMLHelp\1.x\ItssRestrictions]
"MaxAllowedZone"=dword:00000001
"UrlAllowList"="J:\"
"NestedProtocolList"="http:;ftp:"
```