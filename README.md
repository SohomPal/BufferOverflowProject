Solutions for buflab 2025 for Network Centric Programming
Solutions will only work for my netid

Notes
Level 0 - Just run the payload with this command: 
```
cat payload0 | ./hex2raw | ./bufbomb -u sp2160
```

Level 1 - Just run the payload with this command: 
```
cat payload1 | ./hex2raw | ./bufbomb -u sp2160
```

Level 2 - When submitting the payload normally, it might not work. In this case, compile the assembly source file and disassemble it as so:
```
gcc -m32 -c exploit.s -o exploit.o
objdump -d exploit.o
```
The payload is structured as such <0x90> * 24, <Machine Code> (16 Bytes), Buffer Address (4 Bytes) twice
The buffer should be such that the machine code instructions end with 40 bytes and the buffer address is written twice to overwrite the program return address

