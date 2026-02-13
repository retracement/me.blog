+++
date = '2018-03-14T13:00:00+01:00'
draft = false
title = 'Reading very BIG text files using PowerShell'
categories = ['Technology']
tags = ['PowerShell','General Tips']
+++

![Big](/images/2018/big.jpg)

I frequently have to troubleshoot a variety of data feeds (populated from very very large files) which are regularly ingested into SQL Server for reporting and data validation purposes. From time to time, these feeds break due to malformed or invalid data provided by the third parties, and often due to the size of the files and limitations of third-party text file applications, loading them into a text file viewer is not possible.

I recently ran into this very same problem in which the bcp error message stated:

An error occurred while processing the file `D:\MyBigFeedFile.txt` on data row *123798766555678*.

Given that the text file was far in excess of notepad++ limits (or even easy human interaction usability limits), I decided not to waste my time trying to find an editor that could cope and simply looked for a way to pull out a batch of lines from within the file programmatically to compare the good rows with the bad.

# PowerShell to the rescue

I turned to PowerShell and came across the `[Get-Content](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-content?view=powershell-5.1)` cmdlet to read the file, which looked like it would give me at least part of what I wanted. At this point, there are quite a few ways to look at specific lines in the file, but in order to look at the line in question and its surrounding lines I could only come up with one perfect solution. By passing the pipeline output into the `[Select-Object](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/select-object?view=powershell-5.1)` cmdlet I could use its rich *skip* and *next* functionality.

Please note that the following code examples all use PowerShell's multiple line wrap functionality for readability purposes on this web page only. Normally I'd put all these statements on a single line (obviously minus the ` line wrapping character/s)
Basically, the following will read the file in question, skipping the first *123798766555672* lines and return the next 6 lines (I can therefore deduce that my error row will be the last line in the set with 5 good lines above):

```powershell
Get-Content D:\MyBigFeedFile.txt | `
Select-Object -skip 123798766555672 -first 6
```

Ok. So that's cool and works very quickly given the size of the file (in this case it was approximately 3 minutes to return!). Perhaps the final thing we would want to do is to pipe into another text file for repeat viewing (so we don't have to keep re-reading this huge text file over and over) or perhaps you might want to use the [`Out-GridView`](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/out-gridview)` cmdlet. For either option, all we need to do is direct the pipeline output now into the respective cmdlet.

Therefore the full solution to my specific problem is:

```powershell
Get-Content D:\MyBigFeedFile.txt | `
Select-Object -skip 123798766555672 -first 6 | `
Out-File Output.txt
```

---

# Conclusion

I think this final solution is pretty cool, is incredibly fast and works like a dream. It appears much quicker than anything else I could put together (or use out of the box) and seems to avoid any of the limitations (particularly in 32bit applications) of GUI based editors. However it is worth pointing out that I have not monitored memory consumption whilst executing this piece of code on very large files so I cannot confirm how efficient or safe this would be to do on a mission-critical server, but it is my belief that the [`Get-Content`](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-content) cmdlet is a stream operation and so *should* be safe. Either way you should do your own testing folks!