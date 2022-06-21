# CyberDefenders L'espion - https://cyberdefenders.org/blueteam-ctf-challenges/73

This challenge centers around OSINT.

# File -> Github.txt

For task #1, we're asked:

*What is the API key the insider added to his GitHub repositories?*

And the only thing inside the `Github.txt` file is a Github profile: https://github.com/EMarseille99

Scrolling through the profile, there's only one repository which isn't a fork of another Github repository: https://github.com/EMarseille99/Project-Build---Custom-Login-Page 

This is the first place I decided to search since it's more likely to contain original content from the insider than a forked repository is.

* It's likely that the insider forked the repository and didn't add any original content to it, so there's no point in searching through those.

We can see that there's only [four commits](https://github.com/EMarseille99/Project-Build---Custom-Login-Page/commits/master)

The second commit named *Update Login Page.js* contains an API key:

![](https://i.imgur.com/2sUGTVS.png)

And if we continue searching through the commits, we'll find *Create Login Page* which contains hardcoded credentials:

![](https://i.imgur.com/uyarqi8.png)

Base64 Decoding the password returns: **PicassoBaguette99** which is the answer to task #2!



Task #3 asks:

*What cryptocurrency mining tool did the insider use?*

If we search through the insider's forked repositories, we'll find **xmrig** which is a cryptocurrency miner for the coin Monero (XMR):

![](https://i.imgur.com/uDJL1O2.png)

For task #4, we're asked:

*What university did the insider go to?*

Since we already have the insider's job title, the company that they work for, and their full name, which was gathered from their Instagram profile:

![](https://i.imgur.com/5kdvm78.png)

We can take this data to LinkedIn and see if the insider has a profile which details their educational background. If we simply search "Émilie Marseille LinkedIn", we get many results, but only one result is for a backend developer:

![](https://i.imgur.com/uHiX34S.png)

And as you can see from the text preview, it states the insider went to **Sorbonne Universite**!

For task #5, we're asked:

*What gaming website the insider had an account on?*

This can be found by using a username checking tool that will check multiple websites to see if the user has an account on that particular site, [Sherlock](https://github.com/sherlock-project/sherlock) is a great example! For this particular task though, we're specifically interested in *gaming websites*, and the task states that it ends in an *m* and we can safely assume that the name of the website is 5 letters long:

![](https://i.imgur.com/1iv4DAa.png)

There's one site that comes to mind with these clues: **Steam**. We can confirm this by searching for the username *emarseille99* on the **Steam** platform. The URL for searching for particular usernames is:

*https://steamcommunity.com/id/<username>*

![](https://i.imgur.com/p6ZYRpC.png)

And as you can see, this profile has the same profile image as the Instagram account from earlier, so we can confirm this is indeed our insider's **Steam** profile!

## Misc Questions

For task #6, we're asked:

*What is the link to the insider Instagram profile?*

This can be found by Google Dorking the same username as the Github profile from earlier:

![](https://i.imgur.com/tpJLobU.png)

So our insider's Instagram profile is located at: **https://www.instagram.com/emarseille99/**!

For task #7, we're asked:

*Where did the insider go on the holiday? (Country only)*

If we search through the insider's Instagram profile, we'll come across this image with the caption: *Once in a lifetime holiday here, love me some slings x*:

![](https://i.imgur.com/gzUJAdJ.png)

This is where the insider went on vacation. And if we reverse image search the image, the results tell us that it is a nature park, named **Gardens by the Bay**, located in the Central Region of **Singapore**:

![](https://i.imgur.com/3An48l5.png)

For tasks #8, we're asked;

*Where is the insider's family live? (City only)*

If we scroll further down the insider's Instagram page, we'll find that they posted 2 photos:

![](https://i.imgur.com/liwZ4oB.png)

![](https://i.imgur.com/IaM8PIF.png)

With the accompanying caption: *Nice to meet friends & family*

Now we have 2 photos which we can assume were taken while the insider was visiting their family. Just by looking at these photos, I assume that they were taken somewhere in the Middle East due to the environment looking quite sandy and due to the buildings' architectures. The flag in the 2nd photo can help us greatly now that we've narrowed down the region:

![](https://i.imgur.com/fygaQZj.png)

If we search online for a list of Middle Eastern flags, we'll come across this:

![](https://st4.depositphotos.com/9212956/27184/v/950/depositphotos_271840980-stock-illustration-middle-east-vector-flag-set.jpg)

The flag in the image that the insider posted appears to be the flag of the UAE if we compare the color blocking as well as the colors.

Now the first photo has a cityscape view which we can utilize. We can search online for landmark sky scrapers located in the UAE and compare them to this one which we see on the insider's Instagram:

![](https://i.imgur.com/LOhVzKb.png)

I came across this image of the Burj Khalifa:

![](https://imagevars.gulfnews.com/2019/01/28/Burj_Khalifa_10_resources1_16a31079302_original-ratio.jpg)

And the structure of the two buildings looks extremely similar (Notice the building's edges), so we can safely assume that this building on the insider's profile is indeed the Burj Khalifa, located in **Dubai**!

## File -> Office.jpg

For task #9, we're given a photo of what appears to be quite a busy area and the task states:

*You have been provided with a picture of the building in which the company has an office. Which city is the company located in?*

![](https://i.imgur.com/91iSJ2B.jpg)

The first thing that sticks out to me is the large uniquely designed building with the large sign that states *GRAND CENTRAL*. What comes to mind is Grand Central Station which is a well known train terminal located in New York City. But the buidling's architecture looks nothing like the train terminal in NYC, for reference, here are two photos of NYC Grand Central:

![](https://www.grandcentralterminal.com/wp-content/uploads/2017/09/Grand-By-Design-Square-508x508.jpg)

![](https://external-content.duckduckgo.com/iu/?u=http%3A%2F%2Fthesoulofseoul.net%2Fwp-content%2Fuploads%2F2013%2F12%2Fimg_8653.jpg&f=1&nofb=1)

Another thing that stood out to me is the brick building named *ODEON*, so I decided to combine these two landmarks and search online for **Odeon Grand Central**, and I came across this image:

![](https://i2-prod.business-live.co.uk/incoming/article8342463.ece/ALTERNATES/s1227b/odeonquinton.jpg)

It is made of the same brick and has the same color as the Odeon building in the task photo we were given, but we should further confirm this!

This particular photo is from Odeon in Birmingham, England. In the `Office.jpg` photo, there's also a few street signs which give us the name of two theaters:

* Hippodrome Theatre
* Alexandra Theatre

And if we search for the locations of these theaters, specifically if there's any locations in Birmingham, England...

![](https://i.imgur.com/B1XbG5a.png)

![](https://i.imgur.com/AXxWSbB.png)

It is confirmed! These two theaters have locations in Birmingham, England. In fact, The Hippodrome Theatre is even located in the Chinese Quarter of Birmingham, which is another location that appears on the street signs of the `Office.jpg` photo!

* Another way we could've confirmed that these buildings are in Europe is the spelling of *theatre*. *Theatre* is the preferred spelling in Europe while *Theater* is the preferred spelling in the US.'

And if we look online for photos of Grand Central in Birmingham, England, we can quickly confirm that these two buildings are the same:

![](http://1.bp.blogspot.com/-PSzzOAEr0Hg/VhlhP3BoIQI/AAAAAAAANyg/01_xwEVHzSg/s1600/Grand%2BCentral%2BBirmingham%2BV.jpg)

versus the task photo we were given:

![](https://i.imgur.com/91iSJ2B.jpg)

We can compare the architecture from the inside and the outside of the two buildings, and it is clear that the two buildings are indeed the same!

Now we know that the city that the company is located in is Birmingham!

# File -> WebCam.png

For our final task, we're asked:

*With the intel, you have provided, our ground surveillance unit is now overlooking the person of interest's suspected address. They saw them leaving their apartment and followed them to the airport. Their plane took off and has landed in another country. Our intelligence team spotted the target with this IP camera. Which state is this camera in?*

And the webcam image that is being referenced is:

![](https://i.imgur.com/qXy2BaO.jpg)

At first glance, this appears to be a university campus. Notice that there's also the caption: *A View from the Dome*, *Dome* is capitalized as if it is some sort of landmark.

My first thought was to search online for the phrase *A View from the Dome*, and I came across the exact live webcam this image was captured from:

![](https://i.imgur.com/q0M1a7K.png)

And University of Notre Dome is located in Indiana, United States!

Now let's say that the University of Notre Dome took down this live camera view for some reason and we couldn't find it by searching for the phrase: *A View from the Dome* online. We could've resorted to reverse image searching and this would've also gave us the answer:

![](https://i.imgur.com/TPvXgdl.jpg)

One particular image that we find as a result which stood out to me is the Twitter image posted by `@NDadmissions` 4 years ago from the same angle as the Live View camera:

![](https://th.bing.com/th/id/OIP.E4QgaB_KFpGjIrb0h6iHVgHaD2?pid=ImgDet&rs=1)

To add one last thing, if we would've searched online for: "The Dome University", we would've came across the university's website too, which may have led us to the answer:

![](https://i.imgur.com/cTo1bxy.png)


## References

* https://www.grammar.com/theater_vs._theatre
* https://www.atgtickets.com/venues/the-alexandra-theatre-birmingham/info/
* https://www.business-live.co.uk/economic-development/gallery/birmingham-cinemas-through-the-ages-8342538
