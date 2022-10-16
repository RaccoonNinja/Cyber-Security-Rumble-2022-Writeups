## Skills required: Ghidra WASM plugin

I couldn't solve this challenge because I didn't know there are Ghidra plugins that parses WASMs.
I could only work on `wasm-decompile` and `wasm2wat` results, which is hard given my lack of knowledge and CTF time restrictions.

## Solution:

Upon inspecting the challenge web page source, we can download a **.wasm** file.
Using [ghidra-wasm](https://github.com/nneonneo/ghidra-wasm-plugin) was the suggested toolchain so I'll proceed on using it.

All functions are unnamed so I'll list my findings here:

```
04: errorHandler
09: isDigit
10: isLowerCase
11: isUpperCase
12: isLetter
13: isVowelOfEitherCase
14: isAlphaNum
15: isDigitAndConvertToNumber
16: is4AlphaNum
17: isSerialFormat (xxxx-xxxx-xxxx-xxxx-xxxx)
20: innerErrorHandler
24: strcmp
28: strlen (not certain given the mess in 27)
```

The main key-checking logic is in function 18 (liberally tidied-up):

``` c
int unnamed_function_18(char *param1,byte *param2){
  bool bVar1;
  int keylen;
  int l;
  byte bStack33;
  int k;
  byte bStack26;
  byte bStack25;
  int j;
  int i;
  byte bStack14;
  byte bStack13;
  int iStack4;
  
  if (param1 == (char *)0x0) return 1;

  keylen = func28_maybeStrLen(param1,0x18);
  if ((keylen != 0x18) || (param1[0x18] != '\0')) return 1;

  if (!unnamed_function_5(param1)) return 1;

  if (!func17_isSerialFormat(param1)) return 1;

  bStack14 = 0;
  for (i = 0; i < 4; i++) {
    if (func15_isDigitAndConvertToNumber(param1[i],&bStack13)) {
      bStack14 += bStack13;
    }
  }
  if (bStack14 != 13) return 1;

  *param2 = *param2 | 1;
  for (j = 0; j < 4; j++) {
    if (!func12_isLetter(param1[j + 5])) return 1;
  }

  *param2 = *param2 | 4;
  bStack25 = 0;
  bStack26 = 0;
  for (k = 0; k < 4; k++) {
    bVar1 = func12_isLetter(param1[k + 10]);
    if (bVar1 != 0) {
      bStack25++;
    }
    bVar1 = func09_isDigit(param1[k + 10]);
    if (bVar1 != 0) {
      bStack26++;
    }
  }

  if (bStack25 != bStack26) return 1;

  *param2 = *param2 | 0x30;
  for (l = 0; l < 4; l++) {
    keylen = func15_isDigitAndConvertToNumber(param1[l + 0xf],&bStack33);
    if ((keylen != 0) && ((uint)bStack33 % 2 != 0)) {
      return 1;
    }
    if (func12_isLetter(param1[l + 0xf]) && !func13_isVowelOfEitherCase(param1[l + 0xf])) {
      return 1;
    }
  }
  *param2 = *param2 | 0xc0;
  return 0;
}
```

I only have time for these at the moment.
