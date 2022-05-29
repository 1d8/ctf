# Log Analysis Compromised Wordpress - https://blueteamlabs.online/home/challenge/9

In this challenge, we're given a log file and told that it belongs to a Wordpress site which was compromised. The way in which the site was compromised is unknown, but it's suspected that an installed plugin was vulnerable to Remote Code Execution (RCE) and an attacker exploited this to gain access.

The task questions are:

1. Identify the URI of the admin login panel that the attacker gained access to (include the token).
2. Can you find the two tools the attacker used?
3. The attacker tried to exploit a vulnerability in *Contact Form 7*. What CVE was the plugin vulnerable to? (Do some research!)
4. What plugin was exploited to gain access?
5. What is the name of the PHP web shell file?
6. What was the HTTP response code provided when the web shell was accessed the final time?

For the first task which asks us to find the URI of the admin login panel that the attacker gained access to, we can find this by simply searching for the keyword *login* which would bring us to `/wp-login.php`:

![](https://i.imgur.com/ZSjuIrW.png)

but the question asks that we include the token so this wouldn't be the answer, but we're on the right path!

If we instead search for `/wp-login.php`, we cut down the number of results to 278 from 630 so this will make it quicker to find the URI which includes the token.

After some scrolling, I came across some URIs which included the `itsec-hb-token` value that was set to `adminlogin` and also had a 200 response code:

![](https://i.imgur.com/hBQ89K2.png)

This looks like it could be the full URI, token included. And if you search the rest of the log file to find any other values that this token may be set to, it is almost always set to `adminlogin`, confirming that this is our token value, since it's the only value that the token is set to!

* *NOTE: There is one other value that I noticed the `itsec-hb-token` to be set to which is: p@y<\"'p@y, but I believe that either this is simply a malformed value*

Q: Identify the URI of the admin login panel that the attacker gained access to (include the token)
A: `/wp-login.php?itsec-hb-token=adminlogin`

Onto the next task of identifying the two tools that the attacker employed!

The user-agent field of a request is what will allow a server to identify what application or OS (along with the application's version & OS's version) is making the request!

For example, I noticed in this particular log file, there was a request made using Python's `requests` library. This can be identified by the user-agent being set to `python-requests/2.24.0`:

![](https://i.imgur.com/WU7O5ey.png)

So when software makes web requests, it isn't unusual for it to embed the name of the software along with the version into the user-agent field, this is also true of penetration testing tools, so we should be able to tell what tools the attacker used by viewing the User-Agent field!

* *NOTE: It's also common for the user-agent field to be spoofed and many tools include this functionality (EX: [nmap](https://www.infosecmatter.com/nmap-nse-library/?nse=http-useragent-tester)*

But there's a little over 2k lines of text in this file and combing through each user-agent would consume too much time, so I chose to write this quick python script to parse the log file, scrape the user-agent, then output only the unique user-agents:

```python3
f = open("access.log", "r")
data = f.readlines()
f.close()


uniqueUserAgents = []
for line in data:
    userAgent = line.split("-")[-1]
    if userAgent not in uniqueUserAgents:
        uniqueUserAgents.append(userAgent)


for i in uniqueUserAgents:
    print(i)
```

It doesn't work perfectly, in fact, it strips out parts which it should be including, such as the Python user agent:

![](https://i.imgur.com/G7ZDGGe.png)

But the script cuts down the amount of lines we must read through to 600 from 2k so that's beneficial, and it also doesn't cut out the name of the tools!

If we scroll through the list of user agents, at line 267, we'll see a user-agent for *WPScan* which is a tool used to enumerate the plugins installed on a Wordpress site:

![](https://i.imgur.com/FRdIBXH.png)

And if we scroll a bit further to line 483, we find a user-agent for *SQLmap* which is a tool used to find & exploit SQL injection vulnerabilities:

![](https://i.imgur.com/BA4VOV4.png)

Q: Can you find two tools the attacker used?
A: WPScan sqlmap

Onto the next question which is asking for the CVE of a Contact Form 7 vulnerability which the attacker tried to exploit!

I wasn't quite sure what a Contact Form 7 was, so after doing some research, I discovered that it's a Wordpress plugin used to create contact forms so a user could contact an organization! The contact forms could also include a file upload functionality.

When I searched online for vulnerabilities relating to Contact Form 7, the results mainly involved CVE-2020-35489 which is a vulnerability that allowed an attacker to upload any file they'd like, leading to remote code execution.

What seemed to lead to this vulnerability is the fact that special characters aren't removed from the filename, so an attacker could add double extensions into the filename (EX: `.php.png`) and it would accept the upload, and remove the last extension! Then the attacker simply needed to visit the uploaded file to execute the code embedded in it.

Now that I know CVE-2020-35489 was quite a widespread vulnerability, I decided to take a shot in the dark and try that as an answer, and it was correct!

Now onto the last 3 questions:

4. What plugin was exploited to get access?
5. What is the name of the PHP web shell file?
6. What was the HTTP response code provided when the web shell was accessed for the final time?

Question 5 gives away the partial solution to question 4, we now know that there was a PHP web shell that was uploaded, and this is likely how the attacker gained access.

Since it's a PHP file, it'll likely end in or include `.php` and since it involves a web shell being uploaded, it's going to be a POST request, so we can `grep` for these two characteristics:

`cat access.log | grep -i .php | grep -i POST`

which results in 389 lines!

![](https://i.imgur.com/CqRpNl6.png)

But the first thing I noticed is that many of these POST requests are being sent to URIs which do not appear to be related to uploading files (EX: `/wp-cron.php`, `/wp-admin/admin-ajax.php`), so we can add the keyword *upload* into our `cat` command to further cut this down:

`cat access.log | grep -i .php | grep -i POST | grep -i upload`

which now results in 16 lines:

![](https://i.imgur.com/ggCqDGh.png)

* *NOTE: You could've also just opened up the file in a text editor and searched for the word 'upload' and you would only have ~103 results to look through*

One thing that I noticed were POST requests made to `/wp-content/uploads/simple-file-list/`, all of which resulted in 200 responses, meaning they were successful. One of the files which was uploaded was `fr34k.php` and another was `ee-upload-engine.php`. 

To me, `ee-upload-engine.php` seems like a normal file, but `fr34k.php` does not, and the fact that `fr34k.php` was uploaded multiple times is also quite suspicious, it seems as if the attacker may have been modifying their code, and re-uploading their web shell.

Q: What is the name of the PHP web shell file?
A: `fr34k.php`

Now we know that the attacker's web shell is named `fr34k.php`, and we know that they uploaded it to the `/wp-content/uploads/simple-file-list/` directory, we must determine what plugin was exploited to upload this file. We can determine from the upload URI that the plugin is [Simple File List](https://wordpress.org/plugins/simple-file-list/) which allows users to upload, download, and edit files on Wordpress. The question also requests the version of the plugin, so we must search online for vulnerabilities in Simple File List.

[Wpscan](https://wpscan.com/vulnerability/10192) has an exploit and quick explanation regarding a Simple File List vulnerability in which an unauthenticated attacker could upload a file which would lead to remote code execution which is what appears to be happening in our log file. It also details that a fix for this vulnerability was published in version 4.2.3 so the vulnerable version would be 4.2.2!

Q: What plugin was exploited to get access?
A: Simple File List 4.2.2

Onto the final task, finding the HTTP response code when the web shell (`fr34k.php`) was accessed for the final time. All we have to do here is look for the last GET request to `fr34k.php` & find the corresponding HTTP response code.

`cat access.log | grep -i fr34k.php | grep -i GET`

![](https://i.imgur.com/WM3mmlz.png)

* We can also just open up the `access.log` file in a text editor & look for the last instance of `fr34k.php`

![](https://i.imgur.com/kXylq73.png)

## References

* https://nvd.nist.gov/vuln/detail/CVE-2020-35489
* https://www.acunetix.com/vulnerabilities/web/wordpress-plugin-simple-file-list-arbitrary-file-upload-4-2-2/
* https://blog.wpsec.com/contact-form-7-vulnerability/
