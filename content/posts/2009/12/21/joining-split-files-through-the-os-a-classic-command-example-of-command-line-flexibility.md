+++
date = '2009-12-21T14:06:00+01:00'
draft = false
title = 'Rejoining split files in Windows and Linux'
+++

If you frequently download large files in many parts then you are probably not a stranger to HJSplit. This application can be used to connect these parts back together and is quite frankly completely unnecessary since your operating system likely already has a way! So how do you do it?

Lets say you have just downloaded the following files:-

big_file_part.iso.001
big_file_part.iso.002
big_file_part.iso.003

To join these into one big file:

**In Windows**

```bash
copy /b big_file_part.iso.00? big_file_part.iso
```

**In Linux**

```bash
cat big_file_part.iso.00? > big_file_part.iso
```

Note the /b switch in the Windows command – this is important because it forces a binary copy. Its so simple why bother with that third party app!