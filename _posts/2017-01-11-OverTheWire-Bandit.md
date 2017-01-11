---
layout:     post
title:      "OverTheWire - Bandit Writeup"
date:       2017-01-11 13:37:00
author:     "Nikolai Magnussen"
header-img: img/WarGames.jpg
---

## Bandit Wargame
[OverTheWire](http://overthewire.org/wargames/) offer a multitude of wargames.
Wargames are a series of security related challenges that vary in difficulty.

This blog post will cover the game that is recommended to play first,
namely [Bandit](http://overthewire.org/wargames/bandit/).
I would highly recommend all that feel inclined to try this game.
It is a very gentle introduction that try to expect no previous knowledge, meaning that people familiar with \*NIX and using the terminal will not be challenged to the same degree in the early levels.
But bear with me, because there sure will be some challenges at the later levels!
Regardless of your current level of expertise, I would recommend you complete it, but it is by no means necessary.

## Level 0
> The goal of this level is for you to log into the game using SSH.
> The host to which you need to connect is bandit.labs.overthewire.org.
> The username is bandit0 and the password is bandit0.
> Once logged in, go to the [Level 1](http://overthewire.org/wargames/bandit/bandit1.html) page to find out how to beat Level 1.

We can log into the ssh server by:

	❯ ssh bandit0@bandit.labs.overthewire.org

and entering the password `bandit0`.

## Level 0 ➡ Level 1
> The password for the next level is stored in a file called readme located in the home directory. Use this password to log into bandit1 using SSH. Whenever you find a password for a level, use SSH to log into that level and continue the game.

When logged in, we can find and read the file where the password is stored.

	bandit0@melinda:~$ ls
	readme
	bandit0@melinda:~$ cat readme
	boJ9jbbUNNfktd78OOpsqOltutMc3MY1

This mean that the password to the next level is `boJ9jbbUNNfktd78OOpsqOltutMc3MY1`, and we can connect to the next level by:

	❯ ssh bandit1@bandit.labs.overthewire.org

and entering the password `boJ9jbbUNNfktd78OOpsqOltutMc3MY1`.

## Level 1 ➡ Level 2
> The password for the next level is stored in a file called - located in the home directory

We can find and read the file where the next password is stored, but different from the previous level; we have to escape the file name by providing the path to it so the shell does not interpret it as the start of a switch, or to read from stdin.

	bandit1@melinda:~$ ls
	-
	bandit1@melinda:~$ cat ./-
	CV1DtqXWVFXTvM2F0k09SHz0YwRINYA9

Now, we can easily see that the password to the next level is `CV1DtqXWVFXTvM2F0k09SHz0YwRINYA9` and we connect to it the same way as with the previous, but as the user **bandit2**.

## Level 2 ➡ Level 3
> The password for the next level is stored in a file called spaces in this filename located in the home directory

Again, we want to find and read the file where the password is stored.
In the same spirit as the previous level, we have to escape the spaces in the file name so the spaces are considered a part of the file name, and not a new file. Which can be achieved by prepending spaces with a \\.

	bandit2@melinda:~$ ls
	spaces in this filename
	bandit2@melinda:~$ cat spaces\ in\ this\ filename
	UmHadQclWmgdLOKQ3YNgjWxGoRMb5luK

The password for level 3 is  `UmHadQclWmgdLOKQ3YNgjWxGoRMb5luK`.

## Level 3 ➡ Level 4
> The password for the next level is stored in a hidden file in the inhere directory.

First, we find and enter the directory:

	bandit3@melinda:~$ ls
	inhere
	bandit3@melinda:~$ cd inhere

And then we try to find the file, but `ls` does not display hidden files, but adding a few switches display them. Then we can read the file and the password:

	bandit3@melinda:~/inhere$ ls
	bandit3@melinda:~/inhere$ ls -lah
	total 12K
	drwxr-xr-x 2 root    root    4.0K Nov 14  2014 .
	drwxr-xr-x 3 root    root    4.0K Nov 14  2014 ..
	-rw-r----- 1 bandit4 bandit3   33 Nov 14  2014 .hidden
	bandit3@melinda:~/inhere$ cat .hidden
	pIwrPrtPN36QITSp3EQaw936yaFoFgAB

The password for level 4 is `pIwrPrtPN36QITSp3EQaw936yaFoFgAB`.

## Level 4 ➡ Level 5
> The password for the next level is stored in the only human-readable file in the inhere directory. Tip: if your terminal is messed up, try the “reset” command.

This mean that all files but the one with the password only contain data, and we can find out which by using the `file` utility:

	bandit4@melinda:~$ file inhere/*
	inhere/-file00: data
	inhere/-file01: data
	inhere/-file02: data
	inhere/-file03: data
	inhere/-file04: data
	inhere/-file05: data
	inhere/-file06: data
	inhere/-file07: ASCII text
	inhere/-file08: data
	inhere/-file09: data

Now, we can easily see that **-file07** is the correct one, so let's read it:

	bandit4@melinda:~$ cat inhere/-file07
	koReBOKuIDDepwhWk7jZC0RTdopnAYKh

Yielding the password for the next level: `koReBOKuIDDepwhWk7jZC0RTdopnAYKh`.

## Level 5 ➡ Level 6
> The password for the next level is stored in a file somewhere under the inhere directory and has all of the following properties: - human-readable - 1033 bytes in size - not executable

Most likely, there are few files that are 1033 bytes large, which is something `find` can search for:

	bandit5@melinda:~$ find inhere -size 1033c
	inhere/maybehere07/.file2

And now we only have to read it:

	bandit5@melinda:~$ cat inhere/maybehere07/.file2
	DXjZPULLxYr17uwoI01bNLQbtFemEgo7

Giving the password for the next level: `DXjZPULLxYr17uwoI01bNLQbtFemEgo7`.

## Level 6 ➡ Level 7
> The password for the next level is stored somewhere on the server and has all of the following properties: - owned by user bandit7 - owned by group bandit6 - 33 bytes in size

This can be done in the same manner as the previous level, because `find` can search for more than just size, and because we will get many errors because of permissions, we redirect the errors to `/dev/null`:

	bandit6@melinda:~$ find / -user bandit7 -group bandit6 -size 33c 2> /dev/null
	/var/lib/dpkg/info/bandit7.password

Again, what is left is for us to read it:

	bandit6@melinda:~$ cat /var/lib/dpkg/info/bandit7.password
	HKBPTKQnIay4Fw76bEy8PVxKEDQRKTzs

And the password for level 7 is `HKBPTKQnIay4Fw76bEy8PVxKEDQRKTzs`.

## Level 7 ➡ Level 8
> The password for the next level is stored in the file data.txt next to the word millionth

This mean that we should be searching for the word **millionth**, which is something `grep` handles effortlessly:

	bandit7@melinda:~$ grep millionth data.txt
	millionth       cvX2JJa4CFALtqS87jk27qwqGhBM9plV

Meaning `cvX2JJa4CFALtqS87jk27qwqGhBM9plV` is the password for level 8.

## Level 8 ➡ Level 9
> The password for the next level is stored in the file data.txt and is the only line of text that occurs only once

First, we want to `sort` the lines, such that we can determine which lines are `uniq` and only print the unique ones:

	bandit8@melinda:~$ sort data.txt | uniq -u
	UsvVyFSfZZWbi6wgC7dAFyFuR6jQQUhR

Yielding the password `UsvVyFSfZZWbi6wgC7dAFyFuR6jQQUhR` for level 9.

## Level 9 ➡ Level 10
> The password for the next level is stored in the file data.txt in one of the few human-readable strings, beginning with several ‘=’ characters.

The password have to be human-readable, meaning that we can extract the `strings`, and only have the ones that start with **=**:

	bandit9@melinda:~$ strings data.txt | grep ^=
	========== password
	========== ism
	========== truKLdjsbJ5g7yyJ2X2R0o3a5HQJFuLk

By assuming the same pattern as the previous passwords, the password for level 10 is `truKLdjsbJ5g7yyJ2X2R0o3a5HQJFuLk`.

## Level 10 ➡ Level 11
> The password for the next level is stored in the file data.txt, which contains base64 encoded data

Because the file is `base64` encoded, we simply have to decode it:

	bandit10@melinda:~$ base64 -d data.txt 
	The password is IFukwKGsFW8MOq3IRFqrxE1hxTNEbUPR

And the password for level 11 is `IFukwKGsFW8MOq3IRFqrxE1hxTNEbUPR`.

## Level 11 ➡ Level 12
> The password for the next level is stored in the file data.txt, where all lowercase (a-z) and uppercase (A-Z) letters have been rotated by 13 positions

This can be decrypted by simply translating the letters from **a-z** to **n-z** and **a-m** both for upper and lower case, which can be accomplished using `tr`:

	bandit11@melinda:~$ tr 'a-zA-Z' 'n-za-mN-ZA-M' < data.txt
	The password is 5Te8Y4drgCRfCx8ugdwuEX8KFC6k2EUu

And we can easily see that the password for level 12 is `5Te8Y4drgCRfCx8ugdwuEX8KFC6k2EUu`.

## Level 12 ➡ Level 13
> The password for the next level is stored in the file data.txt, which is a hexdump of a file that has been repeatedly compressed. For this level it may be useful to create a directory under /tmp in which you can work using mkdir. For example: mkdir /tmp/myname123. Then copy the datafile using cp, and rename it using mv (read the manpages!)

First, we need to convert the file from hexdump format back to the original file, which can be done by `xxd`. Then, we need to unpack the file by repeatedly unpacking and decompressing, and which method can be found by using the `file` utility. But here is the complete extraction sequence:

	bandit12@melinda:~$ xxd -r data.txt | gunzip | bunzip2 | gunzip | tar xO | tar xO | bunzip2 | tar xO | gunzip
	The password is 8ZjyCRiBWFYkneahHwxCv3wb2a1ORpYL

Meaning that the password for level 13 is `8ZjyCRiBWFYkneahHwxCv3wb2a1ORpYL`.

## Level 13 ➡ Level 14
> The password for the next level is stored in /etc/bandit\_pass/bandit14 and can only be read by user bandit14. For this level, you don’t get the next password, but you get a private SSH key that can be used to log into the next level. Note: localhost is a hostname that refers to the machine you are working on

Now, we simply need to connect to ourself as the **bandit14** user authenticating using the private SSH key:

	bandit13@melinda:~$ ls
	sshkey.private
	bandit13@melinda:~$ ssh -i sshkey.private bandit14@localhost
	Could not create directory '/home/bandit13/.ssh'.
	The authenticity of host 'localhost (127.0.0.1)' can't be established.
	ECDSA key fingerprint is 05:3a:1c:25:35:0a:ed:2f:cd:87:1c:f6:fe:69:e4:f6.
	--- LINES OMITTED ---
	bandit14@melinda:~$ 

Now that we are logged in as **bandit14**, we need to read the password:

	bandit14@melinda:~$ cat /etc/bandit_pass/bandit14
	4wcYUJFw0k0XLShlDzztnTBHiqxU3b3e

Yielding the password `4wcYUJFw0k0XLShlDzztnTBHiqxU3b3e` for level 14.

## Level 14 ➡ Level 15
> The password for the next level can be retrieved by submitting the password of the current level to port 30000 on localhost.

To submit the password to **localhost:30000**, we use netcat, or `nc` for short:

	bandit14@melinda:~$ nc localhost 30000
	4wcYUJFw0k0XLShlDzztnTBHiqxU3b3e
	Correct!
	BfMYroe26WYalil77FoDi9qh59eK5xNr

And the password for the next level is `BfMYroe26WYalil77FoDi9qh59eK5xNr`.

## Level 15 ➡ Level 16
> The password for the next level can be retrieved by submitting the password of the current level to port 30001 on localhost using SSL encryption.

Because this is using encryption, netcat is not the correct tool for the job, but rather `openssl` and it's `s_client` but make sure to ignore end of file:

	bandit15@melinda:~$ openssl s_client -connect localhost:30001 -ign_eof
	CONNECTED(00000003)
	depth=0 CN = li190-250.members.linode.com
	verify error:num=18:self signed certificate
	--- LINES OMITTED ---
	    Start Time: 1484143032
	    Timeout   : 300 (sec)
	    Verify return code: 18 (self signed certificate)
	---
	BfMYroe26WYalil77FoDi9qh59eK5xNr
	Correct!
	cluFn7wTiGryunymYOu4RcffSxQluehd

	read:errno=0

And we can easily determine that the next password is `cluFn7wTiGryunymYOu4RcffSxQluehd`.

## Level 16 ➡ Level 17
> The credentials for the next level can be retrieved by submitting the password of the current level to a port on localhost in the range 31000 to 32000. First find out which of these ports have a server listening on them. Then find out which of those speak SSL and which don’t. There is only 1 server that will give the next credentials, the others will simply send back to you whatever you send to it.

First, we have to scan the specified ports:

	bandit16@melinda:~$ nmap localhost -p 31000-32000

	Starting Nmap 6.40 ( http://nmap.org ) at 2017-01-11 14:08 UTC
	Nmap scan report for localhost (127.0.0.1)
	Host is up (0.00051s latency).
	Not shown: 996 closed ports
	PORT      STATE SERVICE
	31046/tcp open  unknown
	31518/tcp open  unknown
	31691/tcp open  unknown
	31790/tcp open  unknown
	31960/tcp open  unknown

	Nmap done: 1 IP address (1 host up) scanned in 0.08 seconds

Then, we have to find out which port will provide the next credentials:

	bandit16@melinda:~$ nc localhost 31046
	test
	test

	bandit16@melinda:~$ nc localhost 31518
	test
	ERROR
	140737354049184:error:1408F10B:SSL routines:SSL3_GET_RECORD:wrong version number:s3_pkt.c:351:

	bandit16@melinda:~$ nc localhost 31691
	test
	test

	bandit16@melinda:~$ nc localhost 31790
	test
	ERROR
	140737354049184:error:1408F10B:SSL routines:SSL3_GET_RECORD:wrong version number:s3_pkt.c:351:

	bandit16@melinda:~$ nc localhost 31960
	test
	test

Of all the servers, two are only accessible over SSL and the rest are echo servers, returning whatever they receive.
So let's try to access the two SSL-speaking servers:

	bandit16@melinda:~$ openssl s_client -connect localhost:31518
	CONNECTED(00000003)
	depth=0 CN = li190-250.members.linode.com
	verify error:num=18:self signed certificate
	--- LINES OMITTED ---
	    Start Time: 1484144012
	    Timeout   : 300 (sec)
	    Verify return code: 18 (self signed certificate)
	---
	test
	test

	bandit16@melinda:~$ openssl s_client -connect localhost:31790
	CONNECTED(00000003)
	depth=0 CN = li190-250.members.linode.com
	verify error:num=18:self signed certificate
	--- LINES OMITTED ---
	    Start Time: 1484144175
	    Timeout   : 300 (sec)
	    Verify return code: 18 (self signed certificate)
	---
	cluFn7wTiGryunymYOu4RcffSxQluehd
	Correct!
	-----BEGIN RSA PRIVATE KEY-----
	MIIEogIBAAKCAQEAvmOkuifmMg6HL2YPIOjon6iWfbp7c3jx34YkYWqUH57SUdyJ
	imZzeyGC0gtZPGujUSxiJSWI/oTqexh+cAMTSMlOJf7+BrJObArnxd9Y7YT2bRPQ
	Ja6Lzb558YW3FZl87ORiO+rW4LCDCNd2lUvLE/GL2GWyuKN0K5iCd5TbtJzEkQTu
	DSt2mcNn4rhAL+JFr56o4T6z8WWAW18BR6yGrMq7Q/kALHYW3OekePQAzL0VUYbW
	JGTi65CxbCnzc/w4+mqQyvmzpWtMAzJTzAzQxNbkR2MBGySxDLrjg0LWN6sK7wNX
	x0YVztz/zbIkPjfkU1jHS+9EbVNj+D1XFOJuaQIDAQABAoIBABagpxpM1aoLWfvD
	KHcj10nqcoBc4oE11aFYQwik7xfW+24pRNuDE6SFthOar69jp5RlLwD1NhPx3iBl
	J9nOM8OJ0VToum43UOS8YxF8WwhXriYGnc1sskbwpXOUDc9uX4+UESzH22P29ovd
	d8WErY0gPxun8pbJLmxkAtWNhpMvfe0050vk9TL5wqbu9AlbssgTcCXkMQnPw9nC
	YNN6DDP2lbcBrvgT9YCNL6C+ZKufD52yOQ9qOkwFTEQpjtF4uNtJom+asvlpmS8A
	vLY9r60wYSvmZhNqBUrj7lyCtXMIu1kkd4w7F77k+DjHoAXyxcUp1DGL51sOmama
	+TOWWgECgYEA8JtPxP0GRJ+IQkX262jM3dEIkza8ky5moIwUqYdsx0NxHgRRhORT
	8c8hAuRBb2G82so8vUHk/fur85OEfc9TncnCY2crpoqsghifKLxrLgtT+qDpfZnx
	SatLdt8GfQ85yA7hnWWJ2MxF3NaeSDm75Lsm+tBbAiyc9P2jGRNtMSkCgYEAypHd
	HCctNi/FwjulhttFx/rHYKhLidZDFYeiE/v45bN4yFm8x7R/b0iE7KaszX+Exdvt
	SghaTdcG0Knyw1bpJVyusavPzpaJMjdJ6tcFhVAbAjm7enCIvGCSx+X3l5SiWg0A
	R57hJglezIiVjv3aGwHwvlZvtszK6zV6oXFAu0ECgYAbjo46T4hyP5tJi93V5HDi
	Ttiek7xRVxUl+iU7rWkGAXFpMLFteQEsRr7PJ/lemmEY5eTDAFMLy9FL2m9oQWCg
	R8VdwSk8r9FGLS+9aKcV5PI/WEKlwgXinB3OhYimtiG2Cg5JCqIZFHxD6MjEGOiu
	L8ktHMPvodBwNsSBULpG0QKBgBAplTfC1HOnWiMGOU3KPwYWt0O6CdTkmJOmL8Ni
	blh9elyZ9FsGxsgtRBXRsqXuz7wtsQAgLHxbdLq/ZJQ7YfzOKU4ZxEnabvXnvWkU
	YOdjHdSOoKvDQNWu6ucyLRAWFuISeXw9a/9p7ftpxm0TSgyvmfLF2MIAEwyzRqaM
	77pBAoGAMmjmIJdjp+Ez8duyn3ieo36yrttF5NSsJLAbxFpdlc1gvtGCWW+9Cq0b
	dxviW8+TFVEBl1O4f7HVm6EpTscdDxU+bCXWkfjuRb7Dy9GOtt9JPsX8MBTakzh3
	vBgsyi/sN3RqRBcGU40fOoZyfAMT8s1m/uYv52O6IgeuZ/ujbjY=
	-----END RSA PRIVATE KEY-----

It seems the server running at port **31790** is the correct one, but contrary to other levels, we are rewarded a private key, which we probably can use to SSH into the server as **bandit17**.
So first, we have to copy and save the private key. Then we must change permissions such that only we, the owner can read it before using it to connect:

	bandit16@melinda:~$ cat > /tmp/key_kake 
	-----BEGIN RSA PRIVATE KEY-----
	MIIEogIBAAKCAQEAvmOkuifmMg6HL2YPIOjon6iWfbp7c3jx34YkYWqUH57SUdyJ
	imZzeyGC0gtZPGujUSxiJSWI/oTqexh+cAMTSMlOJf7+BrJObArnxd9Y7YT2bRPQ
	Ja6Lzb558YW3FZl87ORiO+rW4LCDCNd2lUvLE/GL2GWyuKN0K5iCd5TbtJzEkQTu
	DSt2mcNn4rhAL+JFr56o4T6z8WWAW18BR6yGrMq7Q/kALHYW3OekePQAzL0VUYbW
	JGTi65CxbCnzc/w4+mqQyvmzpWtMAzJTzAzQxNbkR2MBGySxDLrjg0LWN6sK7wNX
	x0YVztz/zbIkPjfkU1jHS+9EbVNj+D1XFOJuaQIDAQABAoIBABagpxpM1aoLWfvD
	KHcj10nqcoBc4oE11aFYQwik7xfW+24pRNuDE6SFthOar69jp5RlLwD1NhPx3iBl
	J9nOM8OJ0VToum43UOS8YxF8WwhXriYGnc1sskbwpXOUDc9uX4+UESzH22P29ovd
	d8WErY0gPxun8pbJLmxkAtWNhpMvfe0050vk9TL5wqbu9AlbssgTcCXkMQnPw9nC
	YNN6DDP2lbcBrvgT9YCNL6C+ZKufD52yOQ9qOkwFTEQpjtF4uNtJom+asvlpmS8A
	vLY9r60wYSvmZhNqBUrj7lyCtXMIu1kkd4w7F77k+DjHoAXyxcUp1DGL51sOmama
	+TOWWgECgYEA8JtPxP0GRJ+IQkX262jM3dEIkza8ky5moIwUqYdsx0NxHgRRhORT
	8c8hAuRBb2G82so8vUHk/fur85OEfc9TncnCY2crpoqsghifKLxrLgtT+qDpfZnx
	SatLdt8GfQ85yA7hnWWJ2MxF3NaeSDm75Lsm+tBbAiyc9P2jGRNtMSkCgYEAypHd
	HCctNi/FwjulhttFx/rHYKhLidZDFYeiE/v45bN4yFm8x7R/b0iE7KaszX+Exdvt
	SghaTdcG0Knyw1bpJVyusavPzpaJMjdJ6tcFhVAbAjm7enCIvGCSx+X3l5SiWg0A
	R57hJglezIiVjv3aGwHwvlZvtszK6zV6oXFAu0ECgYAbjo46T4hyP5tJi93V5HDi
	Ttiek7xRVxUl+iU7rWkGAXFpMLFteQEsRr7PJ/lemmEY5eTDAFMLy9FL2m9oQWCg
	R8VdwSk8r9FGLS+9aKcV5PI/WEKlwgXinB3OhYimtiG2Cg5JCqIZFHxD6MjEGOiu
	L8ktHMPvodBwNsSBULpG0QKBgBAplTfC1HOnWiMGOU3KPwYWt0O6CdTkmJOmL8Ni
	blh9elyZ9FsGxsgtRBXRsqXuz7wtsQAgLHxbdLq/ZJQ7YfzOKU4ZxEnabvXnvWkU
	YOdjHdSOoKvDQNWu6ucyLRAWFuISeXw9a/9p7ftpxm0TSgyvmfLF2MIAEwyzRqaM
	77pBAoGAMmjmIJdjp+Ez8duyn3ieo36yrttF5NSsJLAbxFpdlc1gvtGCWW+9Cq0b
	dxviW8+TFVEBl1O4f7HVm6EpTscdDxU+bCXWkfjuRb7Dy9GOtt9JPsX8MBTakzh3
	vBgsyi/sN3RqRBcGU40fOoZyfAMT8s1m/uYv52O6IgeuZ/ujbjY=
	-----END RSA PRIVATE KEY-----
	bandit16@melinda:~$ chmod 400 /tmp/key_kake
	bandit16@melinda:~$ ssh -i /tmp/key_kake bandit17@localhost
	Could not create directory '/home/bandit16/.ssh'.
	The authenticity of host 'localhost (127.0.0.1)' can't be established.
	ECDSA key fingerprint is 05:3a:1c:25:35:0a:ed:2f:cd:87:1c:f6:fe:69:e4:f6.
	--- LINES OMITTED ---
	bandit17@melinda:~$

Being authenticated as **bandit17**, we can read the password file:

	bandit17@melinda:~$ cat /etc/bandit_pass/bandit17
	xLYVMN9WE5zQ5vHacb0sZEVqbrp7nBTn

And the password for level 17 is `xLYVMN9WE5zQ5vHacb0sZEVqbrp7nBTn`.

## Level 17 ➡ Level 18
> There are 2 files in the home directory: passwords.old and passwords.new. The password for the next level is in passwords.new and is the only line that has been changed between passwords.old and passwords.new

To find differences between two files, `diff` can be used, and it will output what was changed as well as what is was changed to:

	bandit17@melinda:~$ diff passwords.old passwords.new       
	42c42
	< BS8bqB1kqkinKJjuxL6k072Qq9NRwQpR
	---
	> kfBf3eYk5BPBRzwjqutbbfE887SVc5Yd

Which mean that `BS8bqB1kqkinKJjuxL6k072Qq9NRwQpR` was the original in **passwords.old**, and `kfBf3eYk5BPBRzwjqutbbfE887SVc5Yd` is the new password from **passwords.new**.
And thus, `kfBf3eYk5BPBRzwjqutbbfE887SVc5Yd` is the password for level 18.

## Level 18 ➡ Level 19
> The password for the next level is stored in a file readme in the home directory. Unfortunately, someone has modified .bashrc to log you out when you log in with SSH.

If we try to connect regularly, we automatically get logged off, meaning that we can't get an interactive session, as we would normally get.
But as part of the SSH logon, we can also supply a command that will be executed regardless:

	❯ ssh bandit18@bandit.labs.overthewire.org cat readme

	This is the OverTheWire game server. More information on http://www.overthewire.org/wargames

	Please note that wargame usernames are no longer level<X>, but wargamename<X>
	e.g. vortex4, semtex2, ...

	Note: at this moment, blacksun is not available.

	bandit18@bandit.labs.overthewire.org's password: 
	IueksS7Ubh8G3DCwVzrTd8rAVOwq3M5x

So upon writing the password, we get the password for level 19 back; namely `IueksS7Ubh8G3DCwVzrTd8rAVOwq3M5x`.

## Level 19 ➡ Level 20
> To gain access to the next level, you should use the setuid binary in the home directory. Execute it without arguments to find out how to use it. The password for this level can be found in the usual place (/etc/bandit\_pass), after you have used the setuid binary.

Let's take a look at the setuid binary in our home directory:

	bandit19@melinda:~$ ls -lah
	total 28K
	drwxr-xr-x   2 root     root     4.0K Nov 14  2014 .
	drwxr-xr-x 172 root     root     4.0K Jul 10  2016 ..
	-rw-r--r--   1 root     root      220 Apr  9  2014 .bash_logout
	-rw-r--r--   1 root     root     3.6K Apr  9  2014 .bashrc
	-rw-r--r--   1 root     root      675 Apr  9  2014 .profile
	-rwsr-x---   1 bandit20 bandit19 7.2K Nov 14  2014 bandit20-do

The `s` in the permissions for **bandit20-do** mean that it will be executed as the owner, namely bandit20:

	bandit19@melinda:~$ ./bandit20-do 
	Run a command as another user.
	  Example: ./bandit20-do id
	bandit19@melinda:~$ ./bandit20-do whoami
	bandit20

Which mean that we can use this to read the password from the regular location:

	bandit19@melinda:~$ ./bandit20-do cat /etc/bandit_pass/bandit20 
	GbKksEFF4yrVs6il55v6gwY5aVje5f0j

And the password for level 20 is `GbKksEFF4yrVs6il55v6gwY5aVje5f0j`.

## Level 20 ➡ Level 21
> There is a setuid binary in the home directory that does the following: it makes a connection to localhost on the port you specify as a commandline argument. It then reads a line of text from the connection and compares it to the password in the previous level (bandit20). If the password is correct, it will transmit the password for the next level (bandit21).

Here, we have a binary that will send the next password, given the current one. There is only one catch, we have to connect to it.
This mean that must have two sessions; one that listens for the connection, and one that runs the binary.

First, we listen on a particular port using netcat:

	bandit20@melinda:~$ nc -l localhost 13337

And in the second session, we run the binary and point it to the same port:

	bandit20@melinda:~$ ./suconnect 13337

Then, from the `nc` connection, we can send the current password:

	bandit20@melinda:~$ nc -l localhost 13337
	GbKksEFF4yrVs6il55v6gwY5aVje5f0j
	gE269g2h3mw3pwgrj0Ha9Uoqen1c9DGr

Meaning that `gE269g2h3mw3pwgrj0Ha9Uoqen1c9DGr` is the password for level 21.

## Level 21 ➡ Level 22
> A program is running automatically at regular intervals from cron, the time-based job scheduler. Look in /etc/cron.d/ for the configuration and see what command is being executed.

First, we look at the files in `/etc/cron.d/`:

	bandit21@melinda:~$ ls -la /etc/cron.d
	total 104
	drwxr-xr-x   2 root root 4096 Oct  5 09:14 .
	drwxr-xr-x 108 root root 4096 Jan 11 10:17 ..
	-rw-r--r--   1 root root  102 Feb  9  2013 .placeholder
	-r--r-----   1 root root   46 Nov 14  2014 behemoth4_cleanup
	-rw-r--r--   1 root root  355 May 25  2013 cron-apt
	-rw-r--r--   1 root root   61 Nov 14  2014 cronjob_bandit22
	-rw-r--r--   1 root root   62 Nov 14  2014 cronjob_bandit23
	-rw-r--r--   1 root root   61 May  3  2015 cronjob_bandit24
	-rw-r--r--   1 root root   62 May  3  2015 cronjob_bandit24_root
	-r--r-----   1 root root   47 Nov 14  2014 leviathan5_cleanup
	-rw-------   1 root root  233 Nov 14  2014 manpage3_resetpw_job
	-rw-r--r--   1 root root   51 Nov 14  2014 melinda-stats
	-rw-r--r--   1 root root   54 Jun 25  2016 natas-session-toucher
	-rw-r--r--   1 root root   49 Jun 25  2016 natas-stats
	-r--r-----   1 root root   44 Jun 25  2016 natas25_cleanup
	-r--r-----   1 root root   47 Aug  3  2015 natas25_cleanup~
	-r--r-----   1 root root   47 Jun 25  2016 natas26_cleanup
	-r--r-----   1 root root   43 Jun 25  2016 natas27_cleanup
	-rw-r--r--   1 root root  510 Oct 29  2014 php5
	-rw-r--r--   1 root root   63 Jul  8  2015 semtex0-32
	-rw-r--r--   1 root root   63 Jul  8  2015 semtex0-64
	-rw-r--r--   1 root root   64 Jul  8  2015 semtex0-ppc
	-rw-r--r--   1 root root   35 Nov 14  2014 semtex5
	-rw-r--r--   1 root root  396 Nov 10  2013 sysstat
	-rw-r--r--   1 root root   29 Nov 14  2014 vortex0
	-rw-r--r--   1 root root   30 Nov 14  2014 vortex20

Most likely, **cronjob_bandit22** is the correct one and it is readable, so let's see what it does:

	bandit21@melinda:~$ cat /etc/cron.d/cronjob_bandit22
	* * * * * bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null

Let's follow the bread crumbs further:

	bandit21@melinda:~$ cat /usr/bin/cronjob_bandit22.sh  
	#!/bin/bash
	chmod 644 /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
	cat /etc/bandit_pass/bandit22 > /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv

Meaning that every time it is run, the file `/tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv` is made readable by all users and the password of **bandit22** is written to the file.
Which in turn can be read by us:

	bandit21@melinda:~$  cat /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
	Yk7owGAcWjwMVRwrTesJEwB7WVOiILLI

Meaning that the password for level 22 is `Yk7owGAcWjwMVRwrTesJEwB7WVOiILLI`.

## Level 22 ➡ Level 23
> A program is running automatically at regular intervals from cron, the time-based job scheduler. Look in /etc/cron.d/ for the configuration and see what command is being executed.

First, let's try to find the script being executed:

	bandit22@melinda:~$ ls -la /etc/cron.d/
	total 104
	drwxr-xr-x   2 root root 4096 Oct  5 09:14 .
	drwxr-xr-x 108 root root 4096 Jan 11 10:17 ..
	-rw-r--r--   1 root root  102 Feb  9  2013 .placeholder
	-r--r-----   1 root root   46 Nov 14  2014 behemoth4_cleanup
	-rw-r--r--   1 root root  355 May 25  2013 cron-apt
	-rw-r--r--   1 root root   61 Nov 14  2014 cronjob_bandit22
	-rw-r--r--   1 root root   62 Nov 14  2014 cronjob_bandit23
	-rw-r--r--   1 root root   61 May  3  2015 cronjob_bandit24
	-rw-r--r--   1 root root   62 May  3  2015 cronjob_bandit24_root
	-r--r-----   1 root root   47 Nov 14  2014 leviathan5_cleanup
	-rw-------   1 root root  233 Nov 14  2014 manpage3_resetpw_job
	-rw-r--r--   1 root root   51 Nov 14  2014 melinda-stats
	-rw-r--r--   1 root root   54 Jun 25  2016 natas-session-toucher
	-rw-r--r--   1 root root   49 Jun 25  2016 natas-stats
	-r--r-----   1 root root   44 Jun 25  2016 natas25_cleanup
	-r--r-----   1 root root   47 Aug  3  2015 natas25_cleanup~
	-r--r-----   1 root root   47 Jun 25  2016 natas26_cleanup
	-r--r-----   1 root root   43 Jun 25  2016 natas27_cleanup
	-rw-r--r--   1 root root  510 Oct 29  2014 php5
	-rw-r--r--   1 root root   63 Jul  8  2015 semtex0-32
	-rw-r--r--   1 root root   63 Jul  8  2015 semtex0-64
	-rw-r--r--   1 root root   64 Jul  8  2015 semtex0-ppc
	-rw-r--r--   1 root root   35 Nov 14  2014 semtex5
	-rw-r--r--   1 root root  396 Nov 10  2013 sysstat
	-rw-r--r--   1 root root   29 Nov 14  2014 vortex0
	-rw-r--r--   1 root root   30 Nov 14  2014 vortex20
	bandit22@melinda:~$ cat /etc/cron.d/cronjob_bandit23 
	* * * * * bandit23 /usr/bin/cronjob_bandit23.sh  &> /dev/null

And the contents of that script:

	bandit22@melinda:~$ cat /usr/bin/cronjob_bandit23.sh
	#!/bin/bash

	myname=$(whoami)
	mytarget=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)

	echo "Copying passwordfile /etc/bandit_pass/$myname to /tmp/$mytarget"

	cat /etc/bandit_pass/$myname > /tmp/$mytarget

The target file is generated based on the hashing of a string containing the user's name.
This mean that the file name can be recreated by manually replacing `$myname` with **bandit23**:

	bandit22@melinda:~$ echo I am user bandit23 | md5sum | cut -d ' ' -f 1
	8ca319486bfbbc3663ea0fbe81326349

And we can read the contents of that file:

	bandit22@melinda:~$ cat /tmp/8ca319486bfbbc3663ea0fbe81326349
	jc1udXuA1tiHqjIsL8yaapX5XIAI6i0n

Which mean that `jc1udXuA1tiHqjIsL8yaapX5XIAI6i0n` is the password for the next level.

## Level 23 ➡ Level 24
> A program is running automatically at regular intervals from cron, the time-based job scheduler. Look in /etc/cron.d/ for the configuration and see what command is being executed.

Again, let's see what we have and what the job does:

	bandit23@melinda:~$ ls -la /etc/cron.d
	total 104
	drwxr-xr-x   2 root root 4096 Oct  5 09:14 .
	drwxr-xr-x 108 root root 4096 Jan 11 10:17 ..
	-rw-r--r--   1 root root  102 Feb  9  2013 .placeholder
	-r--r-----   1 root root   46 Nov 14  2014 behemoth4_cleanup
	-rw-r--r--   1 root root  355 May 25  2013 cron-apt
	-rw-r--r--   1 root root   61 Nov 14  2014 cronjob_bandit22
	-rw-r--r--   1 root root   62 Nov 14  2014 cronjob_bandit23
	-rw-r--r--   1 root root   61 May  3  2015 cronjob_bandit24
	-rw-r--r--   1 root root   62 May  3  2015 cronjob_bandit24_root
	-r--r-----   1 root root   47 Nov 14  2014 leviathan5_cleanup
	-rw-------   1 root root  233 Nov 14  2014 manpage3_resetpw_job
	-rw-r--r--   1 root root   51 Nov 14  2014 melinda-stats
	-rw-r--r--   1 root root   54 Jun 25  2016 natas-session-toucher
	-rw-r--r--   1 root root   49 Jun 25  2016 natas-stats
	-r--r-----   1 root root   44 Jun 25  2016 natas25_cleanup
	-r--r-----   1 root root   47 Aug  3  2015 natas25_cleanup~
	-r--r-----   1 root root   47 Jun 25  2016 natas26_cleanup
	-r--r-----   1 root root   43 Jun 25  2016 natas27_cleanup
	-rw-r--r--   1 root root  510 Oct 29  2014 php5
	-rw-r--r--   1 root root   63 Jul  8  2015 semtex0-32
	-rw-r--r--   1 root root   63 Jul  8  2015 semtex0-64
	-rw-r--r--   1 root root   64 Jul  8  2015 semtex0-ppc
	-rw-r--r--   1 root root   35 Nov 14  2014 semtex5
	-rw-r--r--   1 root root  396 Nov 10  2013 sysstat
	-rw-r--r--   1 root root   29 Nov 14  2014 vortex0
	-rw-r--r--   1 root root   30 Nov 14  2014 vortex20
	bandit23@melinda:~$ cat /etc/cron.d/cronjob_bandit24
	* * * * * bandit24 /usr/bin/cronjob_bandit24.sh &> /dev/null

That script does the following:

	bandit23@melinda:~$ cat /usr/bin/cronjob_bandit24.sh
	#!/bin/bash

	myname=$(whoami)

	cd /var/spool/$myname
	echo "Executing and deleting all scripts in /var/spool/$myname:"
	for i in * .*;
	do
	    if [ "$i" != "." -a "$i" != ".." ];
	    then
	        echo "Handling $i"
	        timeout -s 9 60 "./$i"
	        rm -f "./$i"
	    fi
	done

Which mean that all script in the `/var/spool/bandit24` will be executed.
Now, we can create our own script that will get us the password, make it executable and copy it where the cronjob can see and execute it:

	bandit23@melinda:~$ cat > /tmp/password.sh
	#!/bin/bash

	cat /etc/bandit_pass/bandit24 > /tmp/bandit24_password
	bandit23@melinda:~$ chmod +x /tmp/password.sh
	bandit23@melinda:~$ cp /tmp/password.sh /var/spool/bandit24/

Finally, let's checkout the resulting file:

	bandit23@melinda:~$ cat /tmp/bandit24_password
	UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ

Now we have access to **bandit24** using the password `UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ`.

## Level 24 ➡ Level 25
> A daemon is listening on port 30002 and will give you the password for bandit25 if given the password for bandit24 and a secret numeric 4-digit pincode. There is no way to retrieve the pincode except by going through all of the 10000 combinations, called brute-forcing.

Let's first create a file containing all possible combinations by writing a python script:

	bandit24@melinda:~$ python3
	Python 3.4.3 (default, Sep 14 2016, 12:36:27) 
	[GCC 4.8.4] on linux
	Type "help", "copyright", "credits" or "license" for more information.
	>>> with open("/tmp/pins", "w") as f:
	...     for pin in range(10000):
	...         f.write("UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ {:04}\n".format(pin))

If we now inspect the file, we can see that the pattern is as we want it to be:

	bandit24@melinda:~$ cat /tmp/pins
	UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 0000
	UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 0001
	UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 0002
	UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 0003
	UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 0004
	UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 0005
	UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 0006
	--- LINES OMITTED ---
	UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 9994
	UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 9995
	UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 9996
	UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 9997
	UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 9998
	UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 9999

This file can the be used as input to `nc` such that each line of that file is sent through netcat.
But that will generate a very large amount of data to skim through, so we can use `grep` to search for **Correct**:

	bandit24@melinda:~$ nc localhost 30002 < /tmp/pins | grep -C 5 Correct
	Wrong! Please enter the correct pincode. Try again.
	Wrong! Please enter the correct pincode. Try again.
	Wrong! Please enter the correct pincode. Try again.
	Wrong! Please enter the correct pincode. Try again.
	Wrong! Please enter the correct pincode. Try again.
	Correct!
	The password of user bandit25 is uNG9O58gUE7snukf3bvZ0rxhtnjzSGzG

	Exiting.

Now, we have the password for level 25; `uNG9O58gUE7snukf3bvZ0rxhtnjzSGzG`.
	

## Level 25 ➡ Level 26
> Logging in to bandit26 from bandit25 should be fairly easy… The shell for user bandit26 is not /bin/bash, but something else. Find out what it is, how it works and how to break out of it.

The hint here is that once we log into bandit26, we are not encountered by bash, but something else.
Login shells are located in the `/etc/passwd` file, so the login shell for bandit26 is:

	bandit25@melinda:~$ cat /etc/passwd | grep bandit26
	bandit26:x:11026:11026:bandit level 26:/home/bandit26:/usr/bin/showtext

`showtext` is not a commonly known binary, so that file must be checked out:

	bandit25@melinda:~$ file /usr/bin/showtext
	/usr/bin/showtext: POSIX shell script, ASCII text executable
	bandit25@melinda:~$ cat /usr/bin/showtext
	#!/bin/sh

	more ~/text.txt
	exit 0

This mean that once **bandit26** is logged into, more is run.
But it is actually possible to run commands from both `more` and `less` if the text does not fit on one screen full:

	 _                     _ _ _   ___   __  
	| |                   | (_) | |__ \ / /  
	| |__   __ _ _ __   __| |_| |_   ) / /_  
	--More--(50%)

Trying to execute a simple `ls`:

	 _                     _ _ _   ___   __  
	| |                   | (_) | |__ \ / /  
	| |__   __ _ _ __   __| |_| |_   ) / /_  
	!ls
	 _                     _ _ _   ___   __  
	| |                   | (_) | |__ \ / /  
	| |__   __ _ _ __   __| |_| |_   ) / /_  
	--More--(50%)

Which is not how it is supposed to look...
It turns out, all commands executed will be executed by the current shell, which in our case is `/usr/bin/showtext`.
Because `/usr/bin/showtext` does not use any arguments; the command supplied is irrelevant, and we have hit a dead end.

Scouring the `man more` some more; we see that `v` will start an editor at the current line, which in our case is vim:

	help.txt        For Vim version 7.4.  Last change: 2012 Dec 06

	VIM - main help file
	k
	help.txt [Help][RO]                            1,1            Top
	  _                     _ _ _   ___   __
	 | |                   | (_) | |__ \ / /
	 | |__   __ _ _ __   __| |_| |_   ) / /_
	 | '_ \ / _` | '_ \ / _` | | __| / / '_ \
	 | |_) | (_| | | | | (_| | | |_ / /| (_) |
	 |_.__/ \__,_|_| |_|\__,_|_|\__|____\___/
	~
	~
            
And from vim, we are able to read the file containing the password using the command `:e /etc/bandit_pass/bandit26`:

	5czgV9L3Xx8JPOyRbXh6lQbmIOWvPT6Z
	~
	~
	~
	~
	~
	~
	~
	"/etc/bandit_pass/bandit26" [readonly] 1L, 33C 

We can now use the password `5czgV9L3Xx8JPOyRbXh6lQbmIOWvPT6Z` to access level 26.

This is currently the last level, and the challenge is completed.
Congratulations if you followed and managed to solve it yourself!

Thanks to the team at [OverTheWire](http://overthewire.org/wargames/) for providing this great and exciting wargame!
