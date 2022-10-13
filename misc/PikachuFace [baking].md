## Skills required: Python, knowledge on different serialization methods, working in an organized way

I regret spending hours over this, but maybe it was better than staring at WASM.
I ended up only recovering 2 letters with a folder full of meme pics.

## Solution:

I received a .zst compressed file for this challenge, which can be easily decompressed into a huge SVG (415MB).

For me, opening the svg file in Firefox made the whole VM unresponsive due to relentless RAM requirements.
(Post event: opening it in Burp Chromium returned all black cells regardless of window size)

I knew at that time I need to program a custom SVG renderer - I didn't really like the idea but that seemed a challenge that can be overcome as long as I spent enough time.

### Basic research

- There are 10000 cells with most of them (9046) having media query
- There are 3249 distinct types of pixels
- There are 3210 distinct types of media queries

### First iteration

I wrote a [Python program](https://gist.github.com/RaccoonNinja/e54b317e580b62e8aa1454d0a57cd7d9) to parse the whole SVG, subbing all media queries with regexes and render the png.
It took whooping **90 secs** for one pic (post-event speed evaluation) so total it'll take 3000\*1.5 = 75 hours without parallelization.

### Second iteration

I decided to [clean](https://gist.github.com/RaccoonNinja/d06fd229ae1adbc8f1b14dcafed5b425) up the data, which can be deserialized and worked with.
I extracted the rows I needed and then changing them to Python compatible type that can be *eval*-ed and directly [processed](https://gist.github.com/RaccoonNinja/9d3c3b54ae58863b7ec48b1f85d321d5)

It turned out **using eval is the mistake** because [eval is slow](https://stackoverflow.com/questions/9949533/python-eval-vs-ast-literal-eval-vs-json-decode)
Post-event speed evaluation revealed that it took around **60 secs** for one pic so it'll still be 50 hours.

During the CTF I tried different sizes:
- Blindly choosing 400,400; 400,500, etc (I landed on a letter but it was a signal that encouraged me to work badly)
- Using the given intervals, the results are all just memes. Plus I'm worried that there will be overlapping intervals.

I gave up at this point and turned to RobertIsAGangsta, which I did solve. The whole difficulty rank thing really tripped me.

### Third iteration (post-event)

This time I used [clean](https://gist.github.com/RaccoonNinja/8ef15cbeda69cddec77405f59d2b0623) the data into JSON.
I didn't serialize all the lines into a big array because dumping >80MB JSON into a Python program may not be a good idea, but with some basic tweaking of the code I was able to get **9~10 seconds** per picture!
It'll still take 3000/6 = 500 minutes without parallelization though.

<img src="https://user-images.githubusercontent.com/114584910/195672463-a924ae84-32e0-40cc-896c-09b2a423629e.png" alt="happy_meme" height="200"/>

### Fourth iteration

To avoid reparsing the json file, this time I [stored a lookup object on the tuples separately](https://gist.github.com/RaccoonNinja/cb63cac6d731550d9ff84e981b3bc75b) so that I can further compress the JSON data and hopefully load the whole thing into the python program.

I end up having <20MB data file with a 0.1MB lookup object file with some space-saving tricks.

Remembering there are only 3249 distinct types of pixels, implementing a [pixel-level lookup](https://gist.github.com/RaccoonNinja/7acd1d94cfc874dd61992310099cafb3) will also be fast. Now it takes **only 4 seconds** for 1 pic and I can produce the pics without re-loading the files.

### Now what?

We need to throw away the memes and keep only the goodies.

*TBC*
