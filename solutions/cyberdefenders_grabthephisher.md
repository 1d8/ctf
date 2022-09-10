# GrabThePhisher - https://cyberdefenders.org/blueteam-ctf-challenges/95i

After unzipping the file, we go into the `pankewk/` directory and discover various directories which are typical of a web server. But there's also a `metamask/` directory which appears interesting!

Within this directory, we have 2 files & 1 directory:

* `fonts/`
* `index.html`
* `metamask.php`

Opening up the `metamask.php` file:

![](https://i.imgur.com/TirSrlm.png)

We see that it contains **.php** code that captures certain data, then sends it off to a Telegram server, and there's also an interesting comment that's signed with the alias *j1j1b1s@m3r0* 

And if we look at lines 21-26, we see a `$message` variable being created & populated with: a wallet address, a seed phrase, an IP address, and a user. The wallet address within the variable is labeled as **Metamask** so we can safely assume this is the wallet type being targeted in the phishing attack.

Looking at lines 3-7 within the `metamask.php` file, we see that there is a request being made to: `http://api.sypexgeo.net/json/` & the data that's of interest to the attacker includes the country gelocation, the city, & the date.  **Sypex Geo** is the service of choice for the attacker to acquire data about their victims.

Examining line 37 of the `metamask.php` file, we can see that the secret phrase from the wallet is being appended into a file named `log.txt` which is located within the base `pankewk` directory, and if we output the content of this file:

![](https://i.imgur.com/JQQvW6e.png)

We can see that there's **3 seed phrases** that have been captured, with the most recent being: **father also recycle embody balance concert mechanic believe owner pair muffin**

The attacker performs data exfiltration on line 34 of the `metamask.php` file and uses **Telegram**'s API to submit the captured data that was contained within the `$message` variable. The **token** for the Telegram channel is located within the `$token` variable on line 33 and the chat ID for the Telegram chat is **5442785564** & is found on line 32.

We can use both the Telegram token & the chat ID to retrieve data from the Telegram API to dig deeper.

Since the attacker is using the Telegram API and we have the token and ID for their bot, we can leverage the API to gain more information about the threat actor.

We can use the `getChat` API function and perform a GET request to this URL:

* `https://api.telegram.org/bot5457463144:AAG8t4k7e2ew3tTi0IBShcWbSia0Irvxm10/getChat?chat_id=5442785564`

Which will return this data:

```json
{"ok":true,"result":{"id":5442785564,"first_name":"Marcus","last_name":"Aurelius","username":"pumpkinboii","type":"private"}}
```

So the first & last name of the threat actor is **Marcus Aurelius** & his username is **pumpkinboii**
