## Skills required: JSON, basic command injection

A simple challenge good for the score.

## Solution:

Upon inspecting the score, we know that we need to access the `admin` endpoint, which is only possible if the `userid` is greater than 90000000 with admin's userid initialized to 90010001.

The userid in generated on creation as the result of *parsing the result of "1" followed by 7 characters in our control as json*. This feels very suspicious because:
- whitelisting on groupid and userid are not performed
- the use of json seems artificial - I'll just use `1*10**7+int(groupid)*10**4+int(userid)` in python

The immediate idea that came to my mind was *scientific notation*, which is supported by many programming languages like [JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Lexical_grammar#numeric_literals) and [Python](https://docs.python.org/3/reference/lexical_analysis.html?highlight=exponent%20notation#floating-point-literals).
The only difference is that JSON string *allows* the use of such numbers while Python's `int()` doesn't:

![image](https://user-images.githubusercontent.com/114584910/195155659-9d9a66ca-e428-4ddc-b164-7c74f9663a6f.png)

![image](https://user-images.githubusercontent.com/114584910/195155491-7b5a6a0e-5044-4c3c-b819-62b055f288d6.png)

Then we have some `activation_code` challenge, which should be trivially solved by concatenating all strings from `0000` to `9999`. I used [de Bruijn sequence](https://en.wikipedia.org/wiki/De_Bruijn_sequence) because it was more fun.

Finally for the final part, there is obviously some kind of command injection but with some limitations:

```py
def validate_command(string):
    return len(string) == 4 and string.index("date") == 0
# ...
cmd = data["data"]["cmd"]
# currently only "date" is supported
if validate_command(cmd):
    out = subprocess.run(cmd, stdout=subprocess.PIPE, stderr=subprocess.STDOUT)
    return success_msg(out.stdout.decode())
# ...
```

While the `validate_command` function seem strict, in Python, both *strings* and *lists* are iterables and have length. My immediate idea was to use an array for `cmd`.
Still it doesn't hurt if we refer to some [documentation](https://docs.python.org/3/library/subprocess.html#frequently-used-arguments).

Now we know about the general direction and that:
- only the command `date` can be run but we can control the arguments
- all efforts to `;` or `|` will be quoted away by Python as we supplied a list to `subprocess.run`

Reading the [man page (aka documentation) of the `date` command](https://man7.org/linux/man-pages/man1/date.1.html) in linux, it's not hard to find out the `-f` option:

The final step is to order the arguments - I used `["date","-f","/usr/src/app/flag.txt","-u"]` to get the flag.
