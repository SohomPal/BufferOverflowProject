Solutions for buflab 2025 for Network Centric Programming
Solutions will only work for my netid

Report:

**Level 0** - Just run the payload with this command: 
```
cat payload0 | ./hex2raw | ./bufbomb -u sp2160
```
Design: The goal at this level was just to call the smoke() function. I designed a payload to be 48 bytes. The buffer is 40 bytes. After the buffer is EBP and then RET, I wanted to overwrite RET (which contains the return address back to test from getbuf). So my payload overwrites the buffer and EBP, and then overwrites RET with the memory address for the function smoke. That way when returning, the program will go to smoke() rather than returning to test. 

<img width="1440" alt="Level0" src="https://github.com/user-attachments/assets/95691ef3-0c1f-4ee8-bc16-f039f9e70a95" />

**Level 1** - Just run the payload with this command: 
```
cat payload1 | ./hex2raw | ./bufbomb -u sp2160
```
Design: This level I had to call fizz with an argument (my unique cookie) passed in. Similar to level 0, I wrote thru the buffer and overwrote EBP. I then put in the address for fizz() in the RET area. Above this are the local variables. I want to overwrite the second local variable with my cookie. This is the equivalent of calling fizz() with my cookie. Doing this allowed fizz to run believing it was called with my cookie. 

<img width="1439" alt="Level1" src="https://github.com/user-attachments/assets/13446885-8ae3-4456-8850-6ca848b06052" />

**Level 2** - When submitting the payload normally, it might not work. In this case, compile the assembly source file and disassemble it as so:
```
gcc -m32 -c exploit.s -o exploit.o
objdump -d exploit.o
```
The payload is structured as such <0x90> * 24, <Machine Code> (16 Bytes), Buffer Address (4 Bytes) twice
The buffer should be such that the machine code instructions end with 40 bytes and the buffer address is written twice to overwrite the program return address

You can then run this exploit with:
```
cat payload2 | ./hex2raw | ./bufbomb -u sp2160
```

Design: This level was different than the first two. I now had to run machine code that would set the global value to my cookie before returning to test. I created the machine code. My machine code below explains what it needed to do
```
  movl $0x70a12a36, 0x0804d1ec  // Set global_value = 0x70a12a36
  push $0x08049012 // Push the address of bang
  ret //Return to bang
```
After compiling and using objdump to get the hex values of it I was pasted it into my payload. In my payload I used a NOP sled to reach my machine code. However this exploit could also work with the machine code at the start of the buffer. The important thing is setting the return address to where the machine code begins. I set mine to the start of the buffer which was a NOP instruction. This does nothing but move onto the next instruction. This continues until it gets to my machine code which will run, setting the global value and returning back to the program. 

<img width="1440" alt="Level2" src="https://github.com/user-attachments/assets/37fcfcfa-5961-46aa-ac40-f78d01ee8dd8" />

**Level 3** - When submitting the payload normally, it might not work. In this case, compile the assembly source file and disassemble it as so:
```
gcc -m32 -c exploit.s -o exploit.o
objdump -d exploit.o
```
The payload is structured as such <0x90> * 24, <Machine Code> (16 Bytes), Buffer Address (4 Bytes) twice
The buffer should be such that the machine code instructions end with 40 bytes and the buffer address is written twice to overwrite the program return address

You can run this exploit with:
```
cat payload3 | ./hex2raw | ./bufbomb -u sp2160
```

Design: This level was similar to level two. The only difference was some additional steps in the assembly code to hide the manipulation I did in the stack. 
```
  main:
    movl $0x70a12a36, %eax
    movl 4(%esp), %ebx
    movl $0x55683360, %ebp
    pushl $0x8048c93
    ret
```
Essentially, I set the eax register with my cookie, which will be needed in validation by test to pass. I then set ebx with its old value, set the value of my base pointer to its original value and then push the original instruction pointer address back. Now when I return from my machine code, my program will continue on as normal and not detect any changes to my stack. 

Once again, after compiling and using objdump to get the hex values of my machine code, I pasted it into my payload. In my payload I once again used a NOP sled to reach my machine code. However this exploit could also work with the machine code at the start of the buffer. The important thing is setting the return address to where the machine code begins. The NOP sled just allows my machine code to be set anywhere within the buffer. I set my instruction pointer to the start of the buffer which was a NOP instruction, which eventually gets to my machine code which will run. Now I can set my cookie in the eax register and return back to the program stealthily.  

<img width="1440" alt="Level3" src="https://github.com/user-attachments/assets/10785689-0b49-4a7e-a4e3-67609ebda892" />
