# Let's Defend Ransomware Attack - https://app.letsdefend.io/challenge/ransomware-attack/

For this challenge, we're given a Redline memory analysis file named `AnalysisSession1.mans`. This challenge does require [FireEye's Redline software](https://www.fireeye.com/services/freeware/redline.html) to solve.

The 6 tasks we're given include:

1. Finding the full path of the dropped DLL
2. Finding the MD5 hash of the dropped DLL
3. Finding the ransomware note file name
4. Finding the URL that the initial payload was downloaded from
5. The ransomware dropped a copy of the legitimate application into the Temp folder, so we must find the filename of the legitimate application the ransomware also dropped
6. Finding the name of the ransomware

After opening up the analysis file in Redline and going to the *Processes* tab, I noticed that there is a `notepad.exe` process:

![](https://i.imgur.com/viwcQ6v.png)

This caught my attention as I was curious what file was opened with the notepad application. We can find this out by looking at the *Arguments* column:

![](https://i.imgur.com/sGKfQHG.png)

So there is a file at `C:\Users\charles\Desktop\2s6lc-readme.txt` that's being read.

If we search for this filename in the *File System* category, it appears in multiple directories throughout the filesystem:

![](https://i.imgur.com/a00cnTs.png)

This appears to be our ransom note! The task was to find the filename of the ransom note, so it'd be `2s6lc-readme`!

Finding the URL that the inital payload was downloaded from is quite simple with Redline, we can use the *File Download History* section, and then select the *Manual Downloads* filter, and we only have 1 option which looks quite suspicious:

![](https://i.imgur.com/t45eqgp.png)

![](https://i.imgur.com/m2Vk7Dh.png)

`lsass.exe` is a system file, so it's quite odd that the a user would manually be downloading it from a local IP address on port 8111. 

We also see that the file was downloaded to `C:\Users\charles\Downloads`, so we can go to this location in the file system to get more information about this file, such as the MD5 hash!

The MD5 hash of this suspicious file is `bd3c693ecd17dcd9e60b08ab963121de`:

![](https://i.imgur.com/V6kwbrs.png)

Throwing the hash at VirusTotal gives us this result:

![](https://i.imgur.com/q5GDfS4.png)

![](https://i.imgur.com/6jTnA5U.png)

![](https://i.imgur.com/7AMvLet.png)

Now we have confirmation that the downloaded `lsass.exe` is indeed malicious, and it appears to be the Sodinokibi ransomware variant! 

Recall that one of the questions mentions that the ransomware (`C:\Users\charles\Downloads\lsass.exe`) drops a legitimate application into the Temp folder. 

Well if we check the *Processes* section and search for the keyword *Temp*, we find two processes:

* `cmd.exe` which executes `C:\Users\charles\AppData\Local\Temp\MsMpEng.exe`:

![](https://i.imgur.com/ZBcPd0v.png)

* The `MsMpEng.exe` process itself running

I'm not very knowledgeable in Windows system processes, so I decided to check whether or not `MsMpEng.exe` is a legitimate Microsoft process, which it is, and it's apparently a vital part of Microsoft's Defender, it scans downloaded files for malware and will quarantine or remove them if the file is deemed malicious.

To be 100% sure, we should check the MD5 hash of the executable, which is: `8cc83221870dd07144e63df594c391d9`:

![](https://i.imgur.com/kOSe6Va.png)

This is indeed a legitimate executable which the ransomware drops for some reason.

Now for the last task, we must find the DLL which the ransomware dropped. 

I decided to check the *Imports* section of the *File System* to check to see if there's any odd DLL files which the `lssas.exe` file was importing functions from, but they were standard system DLL files:

* My thought process here is that if the malware drops a DLL file, it's likely that it'll also import functions from the DLL file.

![](https://i.imgur.com/l9GMgDM.png)

Let's step back for a second and think about this. A user downloads an executable named `lsass.exe` from a random IP address, then this executable drops a legitimate Microsoft executable, and a DLL file (which we've yet to find). `lsass.exe` then runs the legitimate Microsoft executable. It sounds like the legitimate Microsoft executable may be vulnerable to [DLL Hijacking](https://resources.infosecinstitute.com/topic/dll-hijacking/).

The legitimate Microsoft executable would then execute the malicious DLL that the ransomware dropped. If this is the case, then we should be able to check the DLL imports for the legitimate dropped executable (`MsMpEng.exe`) and see if there's any DLL modules which are placed in a suspicious/uncommon location:

![](https://i.imgur.com/8g7iSkS.png)

So the DLL modules used by this executable are:

* `kernel32.dll`
* `mpsvc.dll`

The location of `kernel32.dll` should be `C:\Windows\System32` and the location of `mpsvc.dll` should also be `C:\Windows\System32`. 

* *NOTE: If you search online for the function ServiceCrtMain which is what is being imported from the mpsvc.dll module, you'll find that this function isn't standard in the Windows mpsvc.dll which should also give away that this is the malicious DLL being sideloaded*

Now we can search the filesystem for both `kernel32.dll` & `mpsvc.dll` and attempt to find these DLL modules in any non-standard locations.

Searching for `kernel32.dll` reveals that it's located in standard locations, including `C:\Windows\System32`.

But searching for `mpsvc.dll` reveals that it's located in 2 locations:

![](https://i.imgur.com/qStJVpr.png)

![](https://i.imgur.com/JVNQXCQ.png)

The DLL is located in both a location which likely requires elevated privileges to write files to (`C:\Program Files\Windows Defender\MpSvc.dll`) and a standard user's temporary directory (`C:\Users\charles\AppData\Local\Temp\MpsVc.dll`).

I don't believe that it would be normal for a system DLL to be placed in a user's Temp directory.

If we browse to the user's temporary directory, we'll find both the legitimate Microsoft Defender executable, `MsMpEng.exe`, & the suspicious DLL, `MpsVc.dll`:

![](https://i.imgur.com/KvOEtRz.png)

The hash of the suspicious DLL in charles' Temp directory is `040818b1b3c9b1bf8245f5bcb4eebbbc`, and if we throw it at VirusTotal...

![](https://i.imgur.com/J0HxKbD.png)

It is indeed malicious as we suspected!

Another way we could've found the dropped DLL would've been to search the same directory that the `MsMpEng.exe` was downloaded into since I believe that in order for a DLL hijacking attack to be successful, the targeted DLL and the executable that uses the DLL must be within the same directory.

To summarize:

* `lsass.exe` is downloaded from a local IP address, then ran

* `lsass.exe` drops a `MsMpEng.exe`, a legitimate Windows Defender component, into the Temp directory as well as `MpsVc.dll`, the malicious DLL.

* `lsass.exe` then runs `MsMpEng.exe` which then will execute functions from the malicious DLL due to DLL hijacking.

## References

* https://www.virustotal.com/gui/file/8bd2067d088dad4df24e11244f5b72ce1fd22b686e2ce9ba6ee8711f8f6a836d
* https://www.virustotal.com/gui/file/0496ca57e387b10dfdac809de8a4e039f68e8d66535d5d19ec76d39f7d0a4402
* https://answers.microsoft.com/en-us/windows/forum/all/msmpengexe/7ca3dd64-a85c-465d-bcca-78aabfe05797
* https://www.processlibrary.com/en/directory/files/mpssvc/413466/
* https://www.processlibrary.com/en/directory/files/kernel32/23314/
