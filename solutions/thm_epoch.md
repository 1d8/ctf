# Epoch - https://tryhackme.com/room/epoch

Navigating to the webserver, we find an input box where we can enter an Epoch value that's converted to a UTC value:

![](https://i.imgur.com/GenwZqq.png)

We know from the THM room page that there is some form of command injection involved in this room, so my first thought is to use a semicolon in order to terminate the first command being executed, then add on our own command.

So our payload would look like this: `12013;whoami`:

![](https://i.imgur.com/qAJxAXv.png)

And we've successfully executed our own command!

Though there aren't any obvious signs of the `flag.txt` file:

![](https://i.imgur.com/dD3dpug.png)

But if we check the hint, we see that it states the developer likes storing information in environment variables. And we can list all environment variables by running the `printenv -0` command:

![](https://i.imgur.com/vDm7xiB.png)

And we've successfully outputted the flag!

![](https://i.imgur.com/aND7ubB.png)
