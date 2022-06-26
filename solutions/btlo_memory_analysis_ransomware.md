# Blue Team Labs Online Memory Analysis Ransomware - https://blueteamlabs.online/home/challenge/1

In this challenge, we're given a memory dump and told that the computer the memory dump was captured from, is the victim of a ransomware attack.

There's 7 tasks in this challenge:

1. Run `vol.py -f infected.vmem --profile=Win7SP1x86 psscan` that will list all processes. What is the name of the suspicious process
2. What is the parent process ID for the suspicious process?
3. What is the initial malicious executable that created this process?
4. If you drill down on the suspicious PID (vol.py -f infected.vmem --profile=Win7SP1x86 psscan | grep (PIDhere)), find the process used to delete files
5. Find the path where the malicious file was first executed
6. Can you identify what ransomware it is? (Do your research!)
7. What is the filename for the file with the ransomware public key that was used to encrypt the private key? (.eky extension)

I would usually say to begin this by running volatility's `imageinfo` plugin to determine the profile we'll be using, but the first task gives that away, `Win7SP1x86`.

So we run:

`vol.py -f infected.vmem --profile=Win7SP1x86 psscan`

which outputs:

![](https://i.imgur.com/QooWwlQ.png)

There's several *normal* looking processes running, but there are also a few that stand out, including:

* `@WanaDecryptor` - PID: 3968
* `or4qtckT.exe` - PID: 2732

The PID of `@WanaDecryptor` is 3968 and its *Parent Process ID (PPID)* is 2732, meaning `or4qtckT.exe` started `@WanaDecryptor`, but the PPID of `or4qtckT.exe` is 1456 which belongs to `explorer.exe`, this is likely because whoever ran the `or4qtckT.exe` executable double-clicked it from Windows File Explorer.

Here is a better visualization of this which was outputted by using the `pstree` plugin:

![](https://i.imgur.com/9dBdLwp.png)

So now we know the names of 2 suspicious processes:

1. `@WanaDecryptor`
2. `or4qtckT.exe`

But the answer that BTLO is looking for is `@WanaDecryptor`!

And the PPID of the suspicious process (`@WanaDecryptor`) is 2732, and the initial malicious executable which created the suspicious process (`@WanaDecryptor`) is `or4qtckT.exe`, which can be seen by using the `pstree` plugin:

![](https://i.imgur.com/uOVVGjh.png)

Now we're asked to look for any other processes that has a PPID that belongs to the `or4qtckT.exe` process. We can do this by running:

`vol.py -f infected.vmem --profile=Win7SP1x86 psscan | grep 2732`

Which outputs:

![](https://i.imgur.com/q5IBLe9.png)

* The first column of 4-digit numbers is the PID while the second column of 4-digit numbers is the PPID

And we have a new suspicious process! `taskdl.exe` which was also started by `or4qtckT.exe`! `taskdl.exe` was the process which was responsible for deleting files on the victim's system!

For the next task, we're asked to find the path where the malicious file was first executed. A hint here is within the question, *first executed*, meaning the file that started this entire ransomware process, so we're looking for the parent process of `@WanaDecryptor`, which is `or4qtckT.exe`.

My first thought was to scan for past executed commands via `cmdline`:

`vol.py -f infected.vmem --profile=Win7SP1x86 cmdline`

![](https://i.imgur.com/8TqmMnZ.png)

And now we have the full path of where the executable was located: `C:\Users\hacker\Desktop\or4qtckT.exe`!

Now we have to identify what ransomware variant this is. Judging by the executable named `@WanaDecryptor@.exe`, I'm going to take a wild guess that this is the WannaCry ransomware variant, but I want to be 100% sure, so we can extract the ransomware executable from memory and upload it to Virus Total to confirm our theory:

`vol.py -f infected.vmem --profile=Win7SP1x86 procdump -p 3968 --dump-dir Dumped-Executables`

![](https://i.imgur.com/WshFWTj.png)

You could've also extracted the `or4qtckT.exe` (PID 2732) executable and uploaded it to VirusTotal to confirm the ransomware variant:

![](https://i.imgur.com/6kXOom6.png)

Now onto the last task!

We have to identify the filename of the public key that was used to encrypt the private key, and we're given the hint that this file will have a `.eky` extension!

This can be solved by using the `memdump` plugin to dump the region of memory of the running malicious process `or4qtckT.exe` (PID 2732):

`vol.py -f infected.vmem --profile=Win7SP1x86 memdump -p 2732 --dump-dir Dumped-MemRegions`

* *NOTE: I believe that the reason we wouldn't be able to use the same method of searching through the strings of the executables we extracted earlier is because any mention of the filename string in the executable is likely to be encrypted. But if we dump the region of memory where the executable was already running, it's likely that the filename string is going to be decrypted in memory which we can then dump & read.*

Then we can use the `strings` utility to look for anything that contains the `.eky` file extension:

`strings 2732.dmp | grep .eky`

![](https://i.imgur.com/jUtZo4l.png)

Our filename is: `00000000.eky`!

* You can also acquire the filename by dumping the `explorer.exe` (PID: 1456) process. I believe this is because the `00000000.eky` file was being interacted with, so its filename was stored in the memory of the `explorer.exe` process for easy future access.

## References

* https://cookiess.medium.com/volatility-cheatsheet-6e08deb85afb
