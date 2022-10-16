## Skills required: IDA

The first challenge I solved in this CTF. I cannot run the program due to dependency issues but IDA was more than enough.

## Solution

It is not hard to see string fragments of the flag with `strings`.

![image](https://user-images.githubusercontent.com/114584910/196041092-01907a21-b5b0-4e9c-aef0-4d6f1268f94c.png)

But we need to disassemble/decompile the executable to know how the flag works.

``` c
__int64 __fastcall door_lock(int a1)
{
  __int64 result; // rax
  __int64 v2; // rax
  __int64 v3; // rax
  __int64 v4; // rax
  __int64 v5; // rax
  __int64 v6; // rax

  result = (unsigned int)(char)(a1 >> 7);
  if ( (_DWORD)result == 1337 )
  {
    v2 = std::operator<<<std::char_traits<char>>(&std::cout, "CSR{");
    v3 = std::ostream::operator<<(v2, (unsigned int)(a1 % 37));
    v4 = std::operator<<<std::char_traits<char>>(v3, "_submarines_");
    v5 = std::ostream::operator<<(v4, (unsigned int)(268 * a1 - 7));
    v6 = std::operator<<<std::char_traits<char>>(v5, "_solved_n1c3!}");
    return std::ostream::operator<<(v6, &std::endl<char,std::char_traits<char>>);
  }
  return result;
}
```

From my memory I guessed a1 = 1337<<7 and got the flag, even if it doesn't work the search space is only $2^7=128$

P.S. `./rev: /lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.34' not found (required by ./rev)` likely meant that [I need to update my OS to a newer distribution](https://askubuntu.com/questions/1143268/how-to-install-a-libc6-version-2-29).
I couldn't really spend time on it during the CTF though. Maybe I'll just spin up another VM later.
