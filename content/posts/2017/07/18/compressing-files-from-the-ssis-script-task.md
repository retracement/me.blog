+++
date = '2017-07-18T15:10:00+01:00'
draft = false
title = 'Bitesize SSIS - Compressing files from the SSIS Script Task'
categories = ['Technology']
tags = ['Dotnet','SQL','SSIS']
+++

I hate SSIS. It seems to me that it is full of certain nuances and unless you are regularly developing SSIS packages, they are easy to forget or it is easy to miss specific important steps. I first started using SSIS back in 2005 when it was directly introduced to replace DTS, but even today I am constantly going around in circles whenever I have to return to write certain functionality.Therefore I have decided to put together a "Bitesize" series of posts that encapsulate simple operations in order to help not just you, but more importantly, remind me! Hopefully, this will save me time in the long run...

---

Ok, so before I get started I will caveat this quick post by saying that there is an easier (or preferred) way to perform compression on files in SSIS using the ZipFile Class in the recent version of the .NET framework, this is sadly not available if you are using any legacy deployments.

While I have not spent ages adapting the following code to perform what should be a simple thing to do, I have spent enough time to justify sharing what I have done, so hopefully you will also find some benefit from the code below too.

Before you get started...
- Add in the namespaces `System.IO` and `System.IO.Compression` into the Namespaces region of your script task script.
- Define the *static Compress* method which takes a `FileInfo` type parameter (the FileInfo Class is essentially a wrapper class created from a file path).
- Utilize the Compress method by instantiating a new FileInfo object from a source file name.

```csharp
#region Namespaces

using System.IO;
using System.IO.Compression;
#endregion
 
public static void Compress(FileInfo file)
{
    //Compress file
    using (FileStream originalFileStream = file.OpenRead())
    {
        if (File.GetAttributes(file.FullName) != FileAttributes.Hidden & file.Extension != ".gz")
        {
            using (FileStream compressedFileStream = File.Create(file.FullName + ".gz"))
            {
                using (GZipStream compressionStream = new GZipStream(compressedFileStream,
                   CompressionMode.Compress))
                {
                    originalFileStream.CopyTo(compressionStream);
                }
            }
        }
    }
}
 
public void Main()
{
    // get the source file 
    string inputfile = (string)Dts.Variables["$Project::TemporaryFile"].Value;
 
    FileInfo file = new FileInfo(inputfile);
    Compress(file);
 
    Dts.TaskResult = (int)ScriptResults.Success;
}
```

In the code above I simply use a SSIS Project variable to pass in the required file path to create the compressed archive. Obviously you should modify the code to meet your needs, but his is enough for you to get started.

If you are interested in the reference article that I have adapted (and simplified) to produce this code, then visit [How to: Compress Files](https://msdn.microsoft.com/en-us/library/ms404280(v=vs.100).aspx).

---

Want more Bitesize SSIS tips? Then keep an eye open for the other posts in the series!