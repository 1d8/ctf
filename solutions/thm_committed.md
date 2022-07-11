# TryHackMe's Committed - https://tryhackme.com/room/committed

For this challenge, we're given an Ubuntu machine and told that sensitive data, aka our flag, has been committed to a Github repository, but it is unknown where the exact file location that the data was committed to. Our task is to find this sensitive data using `git`!

With `git`, every commit is logged and gets a commit ID which is a long hash-like string. We can view a summary of the commit history by running `git log`:

![](https://i.imgur.com/Ei2piSM.png)

And to view the full commit history, we add in the flag `--all`:

![](https://i.imgur.com/3BSegOK.png)

Notice that all the commits seem to have notes regarding progress made towards the project, then there's the "Oops" commit. We can assume this is where the mistake occurred!

To view the data that was committed, all we need to do is run `git show c56c470a2a9dfb5cfbd54cd614a9fdb1644412b5`:

![](https://i.imgur.com/ZsxXlUn.png)

The green text is what was added while the red text is what was removed. And as you can see, there is a plaintext password that was removed, this is our flag!
