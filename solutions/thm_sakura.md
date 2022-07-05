# Sakura Room - TryHackMe - https://tryhackme.com/room/sakura

Our first task is to find the username of the attacker and we are given a `.svg` image file which tend to contain a trove of data that could be read just by viewing the source of the file!

Line 20 of the source of the .svg file gives us the full filepath of the image, including the attacker's username: `SakuraSnowAngelAiko`:

![](https://i.imgur.com/t0Zuis5.png)

Our next two tasks are:

1. Find the full email address of the attacker
2. Find the full name of the attacker

We can use some Google Dorks using the attacker's username to find other websites which contain the username. The Google Dork would be:

`intext:"SakuraSnowAngelAiko"`

We find the attacker's [github](https://github.com/sakurasnowangelaiko?tab=repositories) by doing so!

One repository which is of interest to me is the PGP repository. *Pretty Good Privacy (PGP)* is an encryption method which implements both symmetric and public key encryption. The reason the PGP repository is of interest is because PGP is commonly used with emails so this repository may help lead us to the attacker's email.

The attacker has stored their public key in the repository:

![](https://i.imgur.com/6p7VyG9.png)

And since it's base64 encoded data, we can decode it and try to make sense of the decoded data:

![](https://i.imgur.com/EBg2x9B.png)

We've found the attacker's email address!

Our next task is finding the attacker's full name. The instructions hints that we may be able to find the attacker's real information on a job hunting platform, so the first thing that comes to my mind is  LinkedIn.

We have two possible usernames that the attacker may use:

* SakuraSnowAngel83
* Sakurasnowangelaiko

A Google Dork we can use to try to find the attacker's LinkedIn would be:

* `intext:"SakuraSnowAngel83" site:linkedin.com`
* `intext:"sakurasnowangelaiko" site:linkedin.com`

And we successfully found the attacker's LinkedIn:

![](https://i.imgur.com/bQ2BFrX.png)

![](https://i.imgur.com/yOAEB0A.png)

Our next set of tasks involve finding the attacker's cryptocurrency wallet address & analyzing it.

One thing I noticed on their Github is that they forked lots of repositories related to cryptocurrency mining, including: cpuminer, cpuminer-multi, ethminer, and xmrig. But the only repository that they have which is related to cryptocurrency and isn't forked is the [ETH repository](https://github.com/sakurasnowangelaiko/ETH) which contains a mining script:

![](https://i.imgur.com/ZruFpZj.png)

This repository also has 2 commits, meaning the attacker uploaded, or committed, data to it 2 times.

Viewing these commits:

![](https://i.imgur.com/ypxpyAm.png)

We can see that the attacker first created the miningscript & then updated it on the same day

And if we view the initial commit...

![](https://i.imgur.com/Ryggedt.png)

We got an Ethereum wallet address!

You can use a variety of sites to view this wallet's transaction history, I personally used [Etherchain](https://www.etherchain.org).

One of the questions is finding what mining pool the attacker received payment from on January 23, 2021 (01/23/2021) which can be found by viewing the wallet's transaction history:

![](https://i.imgur.com/mqppsoF.png)

And the last question is what other cryptocurrency did the attacker exchange with their cryptocurrency wallet, this can also be found by viewing their transaction history, specifically the `To` section. All transactions are exclusively hexadecimal wallet addresses, except for 3 which are labeled as `Tether USD`.

For the 5th part of the challenge, we're given a Twitter Direct Message exchange with the attacker's old Twitter handle:

![](https://raw.githubusercontent.com/OsintDojo/public/main/taunt.png)

Finding their new Twitter username using their old one is quite simple, we just have to search their old username online:

`AikoAbe3 site:Twitter.com`

And we find that their new Twitter username is `@SakuraLoverAiko`!

![](https://i.imgur.com/fslPp91.png)

The next questions to answer are:

* What is the URL for the location where the attacker saved their WiFi SSIDs & passwords?
* What is the BSSID of the attacker's home WiFi?

If we scroll the attacker's Twitter, we see that they posted their Wireless Access Point Names & passwords somewhere on the Internet:

![](https://i.imgur.com/HaYAei9.png)

The attacker states that anyone trying to find them will have to do a real `DEEP` search to find where the attacker `PASTE`d them.

The fully capitalized words give quite the hint, plus the green colored text on black background give me real deep web vibes, so we can assume that the attacker posted them somewhere on the deepweb.

Since the words DEEP & PASTE are capitalized, the first thing that comes to my mind is searching for "DeepPaste" to try to find any sites where users can anonymously post text, such as Pastebin:

![](https://i.imgur.com/iMnAIAh.png)

And it appears such a site with that name does exist!

For some reason, when I tried visiting the website, it was down but if you click the `hint` on TryHackMe, it informs you that the website may go down for multiple days at a time, and it gives you a screenshot of the DeepPaste post that the attacker made:

![](https://raw.githubusercontent.com/OsintDojo/public/main/deeppaste.png)

The post has the SSID of the attacker's home WiFi which can be very useful in finding the attacker's home!

The attacker's home SSID: `DK1F-G`

We can cross-reference this with [Wigle](https://wigle.net/) to find the location of the attacker's Home WiFi

You need a Wigle account to search for SSIDs, but creating one is relatively quick and it's a useful tool to keep in your backpocket for future OSINT activities!

To search for an SSID on Wigle:

* view > advanced search > Enter the SSID > Query

And we get our result:

![](https://i.imgur.com/sxneIrB.png)

To view more information regarding the location, we can click *View Map*:

![](https://i.imgur.com/pmAsm2k.png)

Moving onto section 6 of this challenge, we've already found the answer to the city in which the attacker considers "home": Hirosaki where his Home WiFi is located.

But as for the other questions:

* What airport is closest to the location the attacker shared a photo from prior to getting on their flight? 
* What airport did the attacker have their last layover in?
* What lake can be seen in the map shared by the attacker as they were on their final flight home?

We'll need to scroll through the attacker's Twitter.

This is the photo that the attacker shared prior to catching their flight home:

![](https://pbs.twimg.com/media/Esh-uTvUcAc-sXC?format=jpg&name=small)

I tried reverse image searching this image, but that didn't yield any useful results.

But one thing to note is that the caption of this image details that the attacker checked out some *cherry blossoms* before heading home.

If we scroll through the attacker's Twitter, there's another post (specifically a [retweet](https://twitter.com/ArielleLBaker/status/1353483416159858688?cxt=HHwWgMC52d7_xMglAAAA)) regarding cherry blossoms.

The retweet is from an image taken in Bethesda, Maryland!

Also on the attacker's Twitter account is a partial photo of their home country:

![](https://pbs.twimg.com/media/EsiNRuRU0AEH32u?format=jpg&name=small)

So we know that the attacker is not from the United States, but instead was visiting the states, and while visiting, they checked out some cherry blossoms. 

What are the chances that [this](https://pbs.twimg.com/media/Esh-uTvUcAc-sXC?format=jpg&name=small) photo is Bethesda, Maryland?

Recall that the question is asking what airport is closest to the location where the photo was taken.

We know that the attacker is flying internationally back to their home country, so we can search for any major airports near Bethesda, Maryland and we find a few:

![](https://i.imgur.com/XvjMjpM.png)

I first entered my guess of Ronald Reagan Washington National Airport (3-digit Airport code being DCA) and it turned out to be correct!

* I made this guess initially since it was closest to Bethesda, Maryland

Now we must find the airport in which the attacker had their last layover in!

The attacker posted a photo in a Tweet of their final layover [here](https://twitter.com/SakuraLoverAiko/status/1353717763097899010/photo/1):

![](https://i.imgur.com/RXMDBSt.png)

If we search for "JAL First Class Lounge Sakura Lounge" online, we'll find the airline to be Japan Airlines, but the answer is the specific airport that this lounge is located in.

We can find it by reverse image searching.

I personally like using https://www.reverseimagesearch.com/ since it allows you to search multiple platforms at once.

We're presented with multiple images which look quite similar to the image the attacker shared:

![](https://i.imgur.com/JRMAgKQ.png)

But one thing to keep in mind with the image our attacker shared is that the wall is structured in a way that it looks like it curves inward:

![](https://i.imgur.com/y1sUfoj.png)

The one shared here looks most similar in terms of the structure of the wall:

![](https://i.imgur.com/olBFC0U.png)

And this was taken in the JAL first class lounge in Tokyo Haneda International Airport, airline code HND!

Onto our last question!

We are to find the lake that's visible on the map shared by the attacker in this [tweet](https://twitter.com/SakuraLoverAiko/status/1353733617487241217/photo/1)

![](https://pbs.twimg.com/media/EsiNRuRU0AEH32u?format=jpg&name=medium)

I had the most luck with Yandex's reverse image search, and I found this:

![](https://i.imgur.com/yR11IUl.png)

Looks like the same exact region to me!

If we translate the Tweet that the image comes from, we'll find out that this is an image of the region of Tohoku in Japan!

And we can find a labeled map of the Tohoku region on [Wikipedia](https://en.wikipedia.org/wiki/T%C5%8Dhoku_region)

![](https://upload.wikimedia.org/wikipedia/commons/thumb/8/85/Japan_Tohoku_Map.svg/1024px-Japan_Tohoku_Map.svg.png)

Now there's 2 areas which may be considered lakes that I was able to spot on the image that the attacker shared:

![](https://i.imgur.com/OXV3bjb.jpg)

It's likely that it's the one to the right though since it is significantly larger.

Switching to the Wikipedia map:

![](https://i.imgur.com/gSLrx9B.png)

We find that it's Lake Inawashiro-ko!

This lake is also referred to as Lake Inawashiro which is the answer that TryHackMe is looking for.

Overall a very fun challenge room!
