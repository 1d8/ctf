# REvil Corp - https://tryhackme.com/room/revilcorp

**Q: What is the compromised employee's full name?**

**Q: What is the operating system of the compromised host?**

By searching in the *System Information* section of Analysis data, we can find the answer to these 2 questions:

![](https://i.imgur.com/8ByCBBN.png)

**Q: What is the name of the malicious executable that the user opened?**

**Q: What is the full URL that the user visited to download the malicious binary? (include the binary as well)**

If we find the URL of the malicious binary that the user downloaded, we'll also find the malicious executable that the user opened! And we can find the full URL that the user downloaded the ransomware from by looking at the *File Download History* section which will show us two files:

![](https://i.imgur.com/QJjsK3K.png)

* The malicious executable
* The Tor browser

**Q: What is the MD5 hash of the binary?**

**Q: What is the size of the binary in kilobytes?**

Since we've found the full path of the malicious executable, all that's needed to find the MD5 hash & size of the binary is to go to the *File System* & then locate the malicious exectable, then click *Show Details* in the lower right corner of Redline which will give us many details of the executable including the MD5 hash and file size:

![](https://i.imgur.com/UEVa398.png)

**Q: What is the extension to which the user's files got renamed?**

The file extension which was added to the files after they were encrypted is easy to find, we simply need to search any directory in the *File System* & we're bound to find it, I chose to search the Desktop:

![](https://i.imgur.com/y3bk3Bu.png)

**Q: What is the number of files that got renamed and changed to that extension?**

For this question, we must go to Redline's timeline which will show us different events which have occurred, including files being modified which occurs when ransomware encrypts files!

All we must do is select only *Modified, Changed* as our filters under the *Files* category, then search for the file extension that's being added to encrypted files:

![](https://i.imgur.com/OKc32uV.png)

Then we'll get a count of how many files were renamed and had their extensions changed!

**Q: What is the full path to the wallpaper that got changed by an attacker, including the image name?**

Continuing in the timeline section, this can be found by searching for files that were *Accessed* & filtering for the extension `.bmp`:

![](https://i.imgur.com/j8TkWR2.png)

And the first result is our answer!

**Q: The attacker left a note for the user on the Desktop; provide the name of the note with the extension.** 

All we must do for this challenge is go to the *File System* and search the Desktop for a file which appears to be a ransomware note! *Hint: It contains the words readme*

**Q: The attacker created a folder "Links for United States" under C:\Users\John Coleman\Favorites\ and left a file there. Provide the name of the file.**

We must look through the *File System* section to solve this question, we can also search only through the *Favorites* directory & filter for the name *Links for United States*:

![](https://i.imgur.com/SDMps2a.png)

And we find 5 files inside of that directory! But 2 of those files are files which we've discovered already and 1 other file is a standard file found in many Windows directories, so those cannot be the answers!

That leaves us with 2 other files! TryHackMe's hint tells us that the filename includes a Spanish term, and of those 2 files, only 1 of them has a Spanish term in the filename!

**Q: There is a hidden file that was created on the user's Desktop that has 0 bytes. Provide the name of the hidden file.**

To solve this question, we continue to search through the file system, specifically the Desktop, and we can also search for the *hidden* attribute:

![](https://i.imgur.com/eG04qm9.png)

And we also know that the size of the file must be 0 bytes and there's only file which matches all these conditions!

**Q: The user downloaded a decryptor hoping to decrypt all the files, but he failed. Provide the MD5 hash of the decryptor file.**

Searching through the Desktop, we'll find an aptly named file which is our decryptor, all we must do is select it in order to get the MD5 hash:

![](https://i.imgur.com/O4OrM9i.png)

**Q: In the ransomware note, the attacker provided a URL that is accessible through the normal browser in order to decrypt one of the encrypted files for free. The user attempted to visit it. Provide the full URL path.**

I assume this is the same URL that the user downloaded the decryptor file from the previous question, and we can find the full URL in the *Browser URL History* section!

After skimming through the URLs for some time, we'll find an aptly named URL that the user visits in order to download a sample decryptor:

![](https://i.imgur.com/ernxOXq.png)

**Q: What are some three names associated with the malware which infected this host? (enter the names in alphabetical order)**

Recall that MD5 hash that we acquired earlier for the malicious binary that the user ran? We can use that MD5 hash in order to search VirusTotal & online (Google dork: "<insert hash here>") to find the names that the ransomware is known as! And of course, the name of the TryHackMe Room is also a major hint!
