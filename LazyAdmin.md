## https://tryhackme.com/room/lazyadmin
## Title: LazyAdminFinal
### IP: 10.10.57.236

###Answer the questions below:

###What is the user flag?
Answer format: ***{********************************}

###What is the root flag?
Answer format: ***{********************************}


###nmap:
22 open: SSH
80 open: Apache 2.4.18

Navigating to 10.10.57.236:80 is just the default Apache page.

Using dirb to recursively scan and locate any interesting directories or files:

dirb http://10.10.57.236 -R

Found directories:
http://10.10.57.236/content
  http://10.10.57.236/content/index.php
  http://10.10.57.236/content/_themes
  http://10.10.57.236/content/as/
  http://10.10.57.236/content/attachment/
  http://10.10.57.236/content/images/
  http://10.10.57.236/content/inc/
  http://10.10.57.236/content/js/
http://10.10.57.236/index.html
http://10.10.57.236/server-status

Visiting http://10.10.57.236/content/index.php we get:

"Welcome to SweetRice" and a link to https://www.sweetrice.xyz/docs/5-things-need-to-be-done-when-SweetRice-installed/
"SweetRice save all important file in the inc directory,there are two kinds of format ?:.txt (link.txt , htaccess.txt, lastest.txt) and .db"
Note: Look around for .db file.
  
Checking exploitDB for Sweetrice vulns:

Backup disclosure - php/webapps/40718.txt
"You can access to all mysql backup and download them from this directory: http://localhost/inc/mysql_backup"

Navigating to http://10.10.57.236/inc/mysql_backup gives us a 404, however earlier we found the /content/inc/ directory, so maybe that'll work.

Navigating to http://10.10.57.236/content/inc/mysql_backup/ gives us mysql_backup_20191129023059-1.5.1.sql

Opening this file we get the admin user: manager
We also get an MD5 hash of the user's password: 42f749ade7f9e195bf475f37a44cafcb
Using Crackstation we can find the user's password is: Password123

After some poking around I found the management page at /content/as and logged in to the admin portal.
On the Ads section of the admin portal we're able to create a new advert. Going back to exploitDB this was also listed:

CSRF + PHP Code Exec - php/webapps/40700.html
This exploit works by injecting PHP into an advert's HTML file from the admin portal. It also allows this PHP to be executed by going to the injected PHP file.
I went to Pentestmonkey's reverse shell cheat sheet for the PHP reverse shell, and changed the IP and port.
Instead of using the example in the exploit above, which created an alert box with PHP, I used:
<html>
<body>
[exploit goes here]
</body>
</html>

After searching in /content/inc (the same directory in which the mysql_backup file was found), I opened up the ads directory.
Inside was the PHP file I had added.
Before running the file I loaded up a netcat listener with:
nc -lvnp 4444

Once I clicked on the php file it started running and I gained a shell on the web server.

First thing to check was what could be run, I did this with:
sudo -l

"User www-data may run the following commands on THM-Chal:
  /home/itguy/backup.pl"
  
Unfortunately we don't have write access to this file.
 
I then cd'd into /home/itguy and obtained the user.txt flag.

Looking at backup.pl shows that it simply runs the copy.sh script in /etc/.
We do have write access to copy.sh, and opening it shows that it already contains a reverse shell, so we just need to change the IP and port included in the file.
This can be done with:
echo "[reverse_shell]" > copy.sh

Then we start a netcat listener again with:
nc -lvnp 4445

And run the backup.pl script as sudo:
sudo usr/bin/perl /home/itguy/backup.pl

We now have a shell as root.
cd ~/
cat root.txt
