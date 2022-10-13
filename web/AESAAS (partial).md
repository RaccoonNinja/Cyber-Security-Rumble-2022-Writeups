## Skills required: JSON, basic command injection

A simple read-the-documentation challenge. I failed to solve it because it was my 22th hour awake when I first attempted it 🥲.<br/>I also didn't know the advanced functionalities offered by the `http.server` package and just used it to serve static files.

## Solution:

On the surface, the service uses 2 `POST` endpoints `/encrypt` and `/decrypt`.
They receive the request body and `X-AES-KEY` and encrypt/decrypt the text using `AES-128-ECB`. 
While the encryption is weak, that is not our focus now since it isn't a crypto challenge.

In the `POST` endpoint, the `X-AES-KEY` is checked against a strict regex `^[A-Z0-9]{32}$`, which works as expected without multiline issues.
This means we MUST bypass the regex check, i.e. the `do_POST` endpoint entirely.

In CTF challenges, it is very common to see unfamiliar frameworks. A quick way to gain understanding is to [read the documentation](https://docs.python.org/3/library/http.server.html).

![image](https://user-images.githubusercontent.com/114584910/195480063-30d58c12-5f7f-4c77-a996-152d9481bda2.png)

We can use the `-X` option to change the method to `decrypt` to get an easy command injection.

An example payload would be `curl http://0:1337/encrypt -X decrypt --data-binary 123 -H "X-AES-KEY: 5468617473206D79204B756E6720467A; ls"`

I used localhost because the server was already closed by the time this write up is written.
And because of this, I cannot know if there is additional privilege escalation tricks to the recon.
