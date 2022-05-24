# Disclose The Agent - https://app.letsdefend.io/challenge/disclose-the-agent/

For this challenge, we're given a pcap along with a series of questions:

1. What is the email address of Ann's secret boyfriend?

2. What is Ann's email password?

3. What is the name of the file that Ann sent to his secret lover?

4. In what country will Ann meet with her secret lover?

5. What is the MD5 value of the attachment Ann sent?

So we know that our main focus is going to be with SMTP traffic (the name of the `.pcap` file also proves this: `smtpchallenge.pcap`), so we can insert `smtp` into the Wireshark display filter to only look at the SMTP traffic:

![](https://i.imgur.com/mV9xrcP.png)

The first thing that catches my eye is the `AUTH LOGIN` packet (packet 60) which likely has credentials:

![](https://i.imgur.com/j0TlNpy.png)

We can right-click this packet & follow its TCP stream to see the entire conversation:

![](https://i.imgur.com/NhiZiNj.png)

We see the authentication process:

* `334 Username request`
* `sneakyg33k@aol.com`
* `334 Password request`
* `558r00lz`

Judging from the rest of the packet capture, we can determine this is likely Ann's email account information

* If you dive deeper into the packet capture, you can see multiple indicators that indicate Ann is the client in this request:

![](https://i.imgur.com/DnB0k66.png)

* The first packet is `ECHLO annlaptop`
* The *From* header of the email is "Ann Dercover"
* Ann signs off with her name on the email

The email Ann sent states she won't be able to do lunch as she'll be out of town, this email is sent to `sec558@gmail.com`.

If we scroll further through the SMTP traffic, we see data fragment packets which indicates to me that a large file transfer likely occurred:

![](https://i.imgur.com/L4fVZsU.png)

If we right-click & follow the TCP stream of any of these data fragment packets, we see an email conversation exchanged between `sneakyg33k@aol.com` (Ann Dercover) and a new email address we hadn't seen before, `mistersecretx@aol.com`!

We can see that Ann is sending `misstersecretx@aol.com` an email:

![](https://i.imgur.com/ejGCIhU.png)

We can also see that Ann refers to this person as *sweetheart* and asks that they bring their fake passport and a bathing suit, as well as informing them that the address is attached. The subject of the email is also rendezvous and we can see that the attachment is named `secretrendezvous.docx`:

![](https://i.imgur.com/5g1Xy7o.png)

We now know:

* The email address of Ann's secret lover
* The name of the file that Ann sent to her secret lover

The file attachment is base64 encoded, so we simply need to decode it and put it into a `.docx` file!

To do so:

* Click on any of the data fragment packets
* Expand the *Simple Mail Transfer Protocol* section:

![](https://i.imgur.com/NShMP86.png)

* Click the *Reassembled DATA in frame: 557* to be taken to frame 557
* Right-click the *MIME Multipart Media Encapsulation* section > export packet bytes > save the raw data

Now we have a file that looks like this:

![](https://i.imgur.com/uVTrAuq.png)

We simply have to copy the base64 encoded data part of the file, then decode it and put it into a `.docx` file. 

I chose to use this site: https://base64.guru/converter/decode/file to decode it and put it into a file for me.

After it's decoded, simply download the file and then you can use [this site](https://products.groupdocs.app/viewer/total) to view the `.docx` file through your browser:

![](https://i.imgur.com/9kZL6ly.png)

Now we know that the country in which Ann will meet her secret lover is Mexico!

For the last question, the md5 value of the attachment that Ann sent to her secret lover, we can simply run:

* `md5sum <docx file path>`
