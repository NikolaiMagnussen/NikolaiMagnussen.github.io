---
layout:     post
title:      "OverTheWire - Krypton Writeup"
date:       2017-01-23 13:37:00
author:     "Nikolai Magnussen"
header-img: img/WarGames.jpg
---

## Krypton Wargame
[OverTheWire](http://overthewire.org/wargames/) offer a multitude of wargames.
Wargames are a series of security related challenges that vary in difficulty.

This blog post will cover the game that is recommended to play after [Bandit](http://overthewire.org/wargames/bandit), namely [Krypton](http://overthewire.org/wargames/krypton).
Krypton is a game about cryptography, ranging from very simple, to somewhat more advanced.
I highly recommend all to try it!

## Level 0
> Welcome to Krypton! The first level is easy. The following string encodes the password using Base64:
>
> S1JZUFRPTklTR1JFQVQ=
> Use this password to log in to krypton.labs.overthewire.org with username krypton1 using SSH. You can find the files for other levels in /krypton/

To get the password we use for the ssh, we **base64-decode** it:

	❯ echo S1JZUFRPTklTR1JFQVQ= | base64 -d -  
	KRYPTONISGREAT%

And connecting to the server by entering the password `KRYPTONISGREAT`.

## Level 0 ➡ Level 1
> The password for level 2 is in the file ‘krypton2’. It is ‘encrypted’ using a simple rotation. It is also in non-standard ciphertext format. When using alpha characters for cipher text it is normal to group the letters into 5 letter clusters, regardless of word boundaries. This helps obfuscate any patterns. This file has kept the plain text word boundaries and carried them to the cipher text. Enjoy!

If we view the **README** file in **/krypton/krypton1**:

	krypton1@melinda:/krypton/krypton1$ cat /krypton/krypton1/README       
	Welcome to Krypton!
	--- SOME LINES OMITTED FOR BREVITY ---
	The first level is easy.  The password for level 2 is in the file 
	'krypton2'.  It is 'encrypted' using a simple rotation called ROT13.  
	It is also in non-standard ciphertext format.  When using alpha characters for
	cipher text it is normal to group the letters into 5 letter clusters, 
	regardless of word boundaries.  This helps obfuscate any patterns.

	This file has kept the plain text word boundaries and carried them to
	the cipher text.

	Enjoy!

This mean that we can use the great [tr](http://pubs.opengroup.org/onlinepubs/9699919799/utilities/tr.html) utility to perform the [rot13](https://en.wikipedia.org/wiki/ROT13) to decode the string:

	krypton1@melinda:/krypton/krypton1$ tr "a-zA-Z" "n-za-mN-ZA-M" < krypton2
	LEVEL TWO PASSWORD ROTTEN

Yielding the password `ROTTEN` for the next level.

## Level 1 ➡ Level 2
> ROT13 is a simple substitution cipher.
> 
> Substitution ciphers are a simple replacement algorithm. In this example of a substitution cipher, we will explore a ‘monoalphebetic’ cipher. Monoalphebetic means, literally, “one alphabet” and you will see why.
> 
> This level contains an old form of cipher called a ‘Caesar Cipher’. A Caesar cipher shifts the alphabet by a set number. For example:
> 
> plain:  a b c d e f g h i j k ...
> cipher: G H I J K L M N O P Q ...
> In this example, the letter ‘a’ in plaintext is replaced by a ‘G’ in the ciphertext so, for example, the plaintext ‘bad’ becomes ‘HGJ’ in ciphertext.
> 
> The password for level 3 is in the file krypton3. It is in 5 letter group ciphertext. It is encrypted with a Caesar Cipher. Without any further information, this cipher text may be difficult to break. You do not have direct access to the key, however you do have access to a program that will encrypt anything you wish to give it using the key. If you think logically, this is completely easy.
> 
> One shot can solve it!
> 
> Have fun.

Let's first create a working directory, make the keyfile accessible and our working directory accessible for **krypton3**, which is the owner of the encrypt binary.

	krypton2@melinda:~$ cd `mktemp -d` 
	krypton2@melinda:/tmp/tmp.1Bvm8qkbrF$ ln -s /krypton/krypton2/keyfile.dat 
	krypton2@melinda:/tmp/tmp.1Bvm8qkbrF$ chmod 777 .

Then, let's create a known plaintext file that we can use to crack the key:

	rypton2@melinda:/tmp/tmp.1Bvm8qkbrF$ echo "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ" > plain
	krypton2@melinda:/tmp/tmp.1Bvm8qkbrF$ /krypton/krypton2/encrypt plain
	krypton2@melinda:/tmp/tmp.1Bvm8qkbrF$ cat plain ciphertext 
	abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ
	MNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKL

And again, we can use [tr](http://pubs.opengroup.org/onlinepubs/9699919799/utilities/tr.html), first for verification, and then for decryption:

	krypton2@melinda:/tmp/tmp.1Bvm8qkbrF$ tr "M-ZA-L" "A-NO-Z" < ciphertext 
	ABCDEFGHIJKLMNOPQRSTUVWXYZABCDEFGHIJKLMNOPQRSTUVWXYZ
	krypton2@melinda:/tmp/tmp.1Bvm8qkbrF$ tr "M-ZA-L" "A-NO-Z" < /krypton/krypton2/krypton3 
	CAESARISEASY

And we can acces **krypton3** by using `CAESARISEASY` as the password.

## Level 2 ➡ Level 3
Coming soon..
