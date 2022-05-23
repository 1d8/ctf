# Memory Forensics - https://tryhackme.com/room/memoryforensics

In this challenge, we're given 3 memory dump files totaling 3.2 GB in size:

* Snapshot6.vmem
* Snapshot14.vmem
* Snapshot19.vmem

I will be using `remnux`'s virtual machine to solve these tasks along with the volatility memory analysis tool!

## Task 2

In our second task, we're given `Snapshot6.vmem` and asked to find John's password. 

The first step from all these tasks that I took is identifying the correct memory profile to use with volatility which can be done via:

`vol.py -f <memory dump file path> imageinfo`

which outputs:

![](https://i.imgur.com/01gYbQJ.png)

Usually the first suggested profile is the profile of choice, so in this case it'd be `Win7SP1x64`

Now that we have the correct profile, we must determine the volatility plugin to use. The volatilty plugin used to output the password hashes from a memory dump is `hashdump`

So we run:

* `vol.py -f <memory dump file path> --profile=Win7SP1x64 hashdump`

which outputs:

![](https://i.imgur.com/p0bK9Ql.png)

We're only interested in the John user's password, so now we can use [crackstation](https://crackstation.net/) to crack the password!

![](https://i.imgur.com/kbdw5gK.png)

Success!

## Task 3

For task 3, we're given the `Snapshot19.vmem` file and asked two questions:

1. When was the last shutdown time?
2. What did John type into the command line?

Let's determine the profile of this memory dump:

* `vol.py -f <memory dump file path> imageinfo`

![](https://i.imgur.com/ISOTTPd.png)

We'll be using the Win7SP1x64 profile!

In October 2015, [volatility added a new plugin](https://www.volatilityfoundation.org/25) which will tell us the last shutdown time of a machine, it's called `shutdowntime`

So we would run:

* `vol.py -f <memory dump file path> --profile=Win7SP1x64 shutdowntime`

which outputs

![](https://i.imgur.com/83z6Nmo.png)

And for the second question, there's various plugins we can use to view past commands ran by a computer, including:

* `cmdline`
* `consoles`
* `cmdscan`

Let's first try the `cmdline` plugin:

* `vol.py -f <memory dump file path> --profile=Win7SP1x64 cmdline`

The `cmdline` plugin will output what was/what would've been entered into the command-line in order to run a process with certain flags:

![](https://i.imgur.com/2EoPj4y.png)

![](https://i.imgur.com/fT6jrJd.png)

This doesn't help us determine the exact thing that was typed by John unfortunately, but the other two plugins will!

Let's now try `consoles`:

* `vol.py -f <memory dump file path> --profile=Win7SP1x64 consoles`

![](https://i.imgur.com/mqvpn6v.png)

![](https://i.imgur.com/pivKIXl.png)

The `consoles` plugin outputs exactly what we need!

Lastly, let's try the `cmdscan` plugin:

* `vol.py -f <memory dump file path> --profile=Win7SP1x64 cmdscan`

![](https://i.imgur.com/Y0ukD3V.png)

The `cmdscan` plugin also outputs what we need!

# Task 4

For the last task, we're given `Snapshot14.vmem` and told that TrueCrypt was found on the suspect's machine and that the suspect also had an encrypted partition, our job is try to find the passphrase that'll decrypt that partition from the memory dump!

Let's first determine the profile we'll be using:

* `vol.py -f <memory dump file path> imageinfo`

![](https://i.imgur.com/G27GBkZ.png)

It seems that this is an optional feature of TrueCrypt and is called *passphrase caching*, it's disabled by default according to this forum post:

* https://www.forensicfocus.com/forums/forensic-software/find-data-from-truecrypt-with-volatility/

There's also some plugins in that post which are meant to uncover the passphrase from TrueCrypt!

The user in the forum post stated that they had found the process TrueCrypt.exe running, so we should first confirm that this condition is also met from our memory dump by listing the running processes. This can be done via the `pstree` plugin:

* `vol.py -f <memory dump file path> --profiile=Win7SP1x64 pstree`

which outputs:

![](https://i.imgur.com/Tc1YGQQ.png)

![](https://i.imgur.com/gXr5fh9.png)

We find that `TrueCrypt.exe` was running at the time the memory dump was captured, and it has a PID of 1904!

Now let's try using the plugins listed in the forum post:

* `vol.py -f <memory dump file location> --profile=Win7SP1x64 truecryptmaster`

outputs:

![](https://i.imgur.com/ftqkgaS.png)

There doesn't appear to be any useful information to us here, let's move onto the `truecryptsummary` plugin:

* `vol.py -f <memory dump file location> --profile=WinSP1x64 truecryptsummary`

outputs:

![](https://i.imgur.com/u2YzmfZ.png)

And we got the passphrase! It was located at the memory offset `0xfffff8800512bee4`!

And we could've also retrieved the TrueCrypt passphrase by using the `truecryptpassphrase` plugin mentioned in the forum post:

![](https://i.imgur.com/tfajAOZ.png)

I also believe that the only reason that this method worked to retrieve the passphrase from memory is because *Passphrase caching* was enabled.

### References

* https://www.forensicfocus.com/forums/forensic-software/find-data-from-truecrypt-with-volatility/
* https://andreafortuna.org/2017/11/15/how-to-retrieve-users-passwords-from-a-windows-memory-dump-using-volatility/
* https://cookiess.medium.com/volatility-cheatsheet-6e08deb85afb
