# Let's Defend - Revil Ransomware

For this challenge, we're presented with 3 files:

1. 993ixjlb-readme.txt
2. bad day.PNG
3. AnalysisSession1.mans

The `.mans` file is a memory dump which can be analyzed with [FireEye's Redline security tool](https://www.fireeye.com/services/freeware/redline.html). 

We're also presented with 10 challenge questions:

1. What is the Operating System which the Redline image is being collected on?
2. What is the Logged in User while the Redline image is being collected?
3. What is the location of the ransomware on the filesystem?
4. What is the MD5 of the ransomware?
5. What is the extension for the encrypted file on the filesystem?
6. What is the onion website for paying the ransom?
7. What is the secondary website for paying the ransom?
8. What is the Child Command Line Process being executed after the ransomware being executed?
9. What is the Mitre ATT&CK Technique ID of this ransomware impact stage?
10. What is the name of the Ransomware? 

Many of these questions can actually be answered quite quickly without even opening Redline to analyze the memory dump yet.

Questions 5, 6, 7, and 10 could be answered by looking through the `.txt` & `.png` file which were packaged in the challenge's `.zip` file:

![](https://i.imgur.com/kmjUkvp.jpg)

The image tells us that `993ixjlb-readme.txt` contains instructions, so it's likely the ransomware note.

If we `cat` this file, we're able to read its contents and determine that:

* The file extension of the encrypted files is `993ixjlb.`

![](https://i.imgur.com/JwC4BEo.png)

* The Tor onion website to pay the ransom is: `hxxp aplebzu47wgazapdqks6vrcv6zcnjppkbxbr6wketf56nf6aq2nmyoyd.onion/4FE49B3286F992CB`
* The secondary website to pay the ransom is: `hxxp decoder.re/4FE49B3286F992CB`

![](https://i.imgur.com/rV2TBGK.png)

Question 10 asks for the name of the ransomware which can be determined by the challenge name, `revil`. This can also be determined by acquiring the hash of the ransomware and inputting it into Virus Total, then looking through the *Community* tab.

Question 2 asks for the username of the user that was logged in while the Redline image was collected. This can be determined by opening up the image in Redline > System Information > User Information > Logged in User:

![](https://i.imgur.com/7lawJc0.png)

* More information on Redline usage [here](https://resources.infosecinstitute.com/topic/memory-analysis-using-redline/)

Another way we could determine the username of the logged in user is using Linux's `strings` command on the memory image!

We could run:

`strings AnalysisSession1.mans | grep Desktop | more` find the instances of the string `Desktop` in the dump. The reason I chose `Desktop` is since it's a common Windows directory and it contains the username of the user in the file path (EX: `C:\Users\<username>\Desktop`):

![](https://i.imgur.com/Fzr2HDS.png)

We were lucky in this output since the answer is on the first line!

To find the OS that the Redline image was collected from, System Information > Operating System Information > Operating System:

![](https://i.imgur.com/K71jzmD.png)

Now let's determine the MD5 hash of the ransomware executable!

I began by looking through the *processes* tab in Redline, but nothing stood out of the ordinary to me so I transitioned into the *File System* section which is where I found this in the *Downloads* folder:

* I realize now that since this memory dump was taken after the ransomware was executed & the ransom note was dropped onto the system, there would likely be no use in looking for evidence in the *processes* tab since it would've been done running.

![](https://i.imgur.com/aWuWdSC.png)

This is quite a suspicious executable to me.

The *Signature Description* states this is not a signed binary.

I assume that the *Accessed* column is referring to when the binary was executed:

![](https://i.imgur.com/ZXzEBw5.png)

We can also determine the time that the ransom note was dropped onto the victim's system in hopes to match that up with the time that the potential ransomware binary was executed. We can do so by searching through the *Event Logs* of Redline. 

Search for the string `readme` within the event logs since we know from the file `bad day.png` that the ransom note contained this string in the filename:

![](https://i.imgur.com/5d52JRE.png)

There's 3 instances of it found!

Showing more details about this event reveals that the ransom note was dropped at `20:06:44 on 2021-07-31`, recall that the ransomware binary was executed at `20:06:31 on 2021-07-31`:

![](https://i.imgur.com/zxzR4bW.png)

The hash of the suspicious file is: `94d087166651c0020a9e6cc2fdacdc0c`

![](https://i.imgur.com/SsJQqLS.png)

And if we input this hash into Virus Total, we can confirm that this is indeed Revil Ransomware:

![](https://i.imgur.com/y7o70no.png)

So the ransomware executable was located at: `C:\Users\SecurityNinja\Downloads\bad day.exe`

And if we visit [Joe's Sandbox's full report](https://www.joesandbox.com/analysis/443860/0/html), we can confirm the Child Command-Line Process that was ran after the ransomware binary was executed is: 

`netsh advfirewall firewall set rule group='Network Discovery' new enable=Yes`

which will make the victim's machine visible to other machines on the same network (it enables Network Discovery).

We can also confirm that the Mitre ATT&CK Technique ID at the impact stage is `T1486` which involves encrypting victim data in an effort to disrupt availability to system & network resources as well as requesting compensation for the decryption ke as well as requesting compensation for the decryption key.

## References

* https://app.letsdefend.io/challenge/revil-ransomware/
* https://www.virustotal.com/gui/file/9b11711efed24b3c6723521a7d7eb4a52e4914db7420e278aa36e727459d59dd/community
* https://www.joesandbox.com/analysis/443860/0/html
* https://attack.mitre.org/techniques/T1486/
