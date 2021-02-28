---
layout: blog
title: "OverTheWire: Natas Writeups"
date: 2020-05-03T16:01:13.340Z
tags:
  - ctf_writeup
---

Overthewire Natas Wargame CTF Writeups

In this post I will share my writeup in OverTheWire war-games, this writeup will be updated!

<!--more-->

## natas0

> You can find the password for the next level on this page.
>
> http://natas0.natas.labs.overthewire.org/

**how to solve:**

inspect element or view source code

```html
<!--The password for natas1 is gtVrDuiDfck831PqWsLEZy5gyDz1clto -->
```

> natas1: gtVrDuiDfck831PqWsLEZy5gyDz1clto

---

## natas1

> http://natas1.natas.labs.overthewire.org/\
> You can find the password for the next level on this page, but rightclicking has been blocked!

**how to solve:**

inspect element using shortcut

```html
<!--The password for natas2 is ZluruAthQk7Q2MqmDeTiUij2ZvWy2mBi -->
```

> natas2: ZluruAthQk7Q2MqmDeTiUij2ZvWy2mBi

---

## `natas2`

> There is nothing on this page![](http://natas2.natas.labs.overthewire.org/files/pixel.png)\
> http://natas2.natas.labs.overthewire.org/

**how to solve:**

after you try to view the page source (view-source:http://natas2.natas.labs.overthewire.org/)

```html
<img src="files/pixel.png" />
```

you see there is a path to image location on the server, you can try to see the image at the link `http://natas2.natas.labs.overthewire.org/files/pixel.png` there nothing there, but can we see the list of files? we can go to `http://natas2.natas.labs.overthewire.org/files/`

![list files](/images/uploads/7537fd23-5862-49a6-ad1d-0f8df2637ed9.png "list files")

hmmm, `users.txt` seems suspicious, you can go to `http://natas2.natas.labs.overthewire.org/files/users.txt`

```
# username:password
alice:BYNdCesZqW
bob:jw2ueICLvT
charlie:G5vCxkVV3m
natas3:sJIJNW6ucpu6HPZ1ZAchaDtwd7oGrD14
eve:zo4mJWyNj2
mallory:9urtcpzBmH
```

> natas3: sJIJNW6ucpu6HPZ1ZAchaDtwd7oGrD14

---

## natas3

> No more information leaks!! Not even Google will find it this time...\
> http://natas3.natas.labs.overthewire.org/

**how to solve:**

_Not even Google will find_ it sounds like he’s telling us about `robots.txt` http://natas3.natas.labs.overthewire.org/robots.txt ![](CE592C4E-582C-40BE-B4A0-FC3B0E9D4081.png)

```
User-agent: *
Disallow: /s3cr3t/
```

check `http://natas3.natas.labs.overthewire.org/s3cr3t/` , it will tell you the path of `users.txt` on the page after that verify and check `http://natas3.natas.labs.overthewire.org/s3cr3t/users.txt`

> natas4: Z9tkRkWmpt9Qr7XrR5jWRkgOU901swEZ

---

## natas4

> Access disallowed. You are visiting from "" while authorized users should come only from http://natas5.natas.labs.overthewire.org/

**how to solve:**

try to access the website using curl (-u flag specify username:password for curl):

```shell
curl -u natas4:Z9tkRkWmpt9Qr7XrR5jWRkgOU901swEZ http://natas4.natas.labs.overthewire.org/

# response: Access disallowed. You are visiting from "" while authorized users should come only from "http://natas5.natas.labs.overthewire.org/"
```

it means that we have to specify the referer to `http://natas5.natas.labs.overthewire.org` fortunantely, curl comes with `--referer` flag to specify request’s referer.

```shell
# try to run this command
curl -u natas4:Z9tkRkWmpt9Qr7XrR5jWRkgOU901swEZ http://natas4.natas.labs.overthewire.org/ --referer http://natas5.natas.labs.overthewire.org/

#response: Access granted. The password for natas5 is iX6IOfmpN7AYOQGPwtn3fXpbaJVJcHfq
```

> natas5: iX6IOfmpN7AYOQGPwtn3fXpbaJVJcHfq

---

## `natas5`

> Access disallowed. You are not logged in\
> http://natas5.natas.labs.overthewire.org/

**how to solve:**

there is no clue after viewing page source and doing inspect element. So what is it? cookie? ![](0CEEB6CF-9F73-4C3B-8A1B-CFA09278693A.png) i have this extension called `EditThisCookie` on chrome so i just have to click the extension and see this cookie, there is key of the cookie `loggedIn` and it set the value to `0` so i just change the value to `1`.

```
Access granted. The password for natas6 is aGoY4q2Dc6MgDq4oL4YtoKtyAg9PeHa1
```

> natas6: aGoY4q2Dc6MgDq4oL4YtoKtyAg9PeHa1

---

## natas6

> http://natas5.natas.labs.overthewire.org/

**how to solve:**

see the source code:

```php
<?
include "includes/secret.inc";
    if(array_key_exists("submit", $_POST)) {
        if($secret == $_POST['secret']) {
        print "Access granted. The password for natas7 is <censored>";
    } else {
        print "Wrong secret";
    }
    }
?>
```

we can see there is _include_ command that include external file, it seems we can access it directly `http://natas6.natas.labs.overthewire.org/includes/secret.inc` and you get `FOEIUWGHFEEUHOFUOIU` as the correct key.

> natas7: 7z3hEENjQtflzgnT29q7wAvMNfZdh0i9

---

## Next Natas

// TODO
