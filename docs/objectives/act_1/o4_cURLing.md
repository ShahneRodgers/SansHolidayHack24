---
icon: material/text-box-outline
---

# cURLing

**Difficulty**: :fontawesome-solid-star::fontawesome-regular-star::fontawesome-regular-star::fontawesome-regular-star::fontawesome-regular-star:<br/>
**Direct link**: [Objective 1 terminal](https://.../)

## Objective

!!! question "Request"
    Team up with Bow Ninecandle to send web requests from the command line using Curl, learning how to interact directly with web servers and retrieve information like a pro!

??? quote "Bow Ninecandle"
    Well hello there! I'm Bow Ninecandle, bright as a twinkling star! Everyone's busy unpacking, but I've grown quite bored of that. Care to join me for a lovely game?

    Oh Joy! Today, We're diving into something delightful: the curling challenge—without any ice, but plenty of sparkle!

    No icy brooms here though! We're all about Curl, sending web requests from the command line like magic messages.

    So, have you ever wielded Curl before? If not, no worries at all, my friend!

    It's this clever little tool that lets you whisper directly to web servers. Pretty neat, right?

    Think of it like sending secret scrolls through the interwebs, awaiting a wise reply!

    To begin, you can type something like curl https://example.com. Voilà! The HTML of the page appears, like conjuring a spell!

    Simple enough, huh? But oh, there's a whole world of magic you can cast with Curl!

    We're just brushing the surface here, but trust me—it’s a hoot and a half!

    If you get tangled up or need help, just give me a shout! I’m here to help you ace this curling spectacle.

    So, are you ready to curl those web requests like a pro? Let’s see your magic unfold!

## Hints

??? tip "cURL Manual"
    The official cURL man page has tons of useful information on how to use cURL.

??? tip "Don't Squash"
    Take a look at cURL's "--path-as-is" option; it controls a default behavior that you may not expect!

## Solution

Silver can be solved just by using the cURL manual if you're not already familiar with cURL.
Gold took me a bit longer; Bow mentions just using three commands which I assumed means we need
to send the first answer:

1. `curl http://curlingfun:8080`
2. `curl https://curlingfun:9090/ -k`
But now we need to send the [final request](https://explainshell.com/explain?cmd=curl+https%3A%2F%2Fcurlingfun%3A9090%2F..%2F..%2Fetc%2Fhacks+-k+--data+%22skip%3Dalabaster%22+-b+%22end%3D3%22+-v+-H+%22Stone%3A+Granite%22+--path-as-is), and it won't accept that.

So I started to explore the container instead, and quickly found HARDMODE.txt:
```bash
$ cat HARD-MODE.txt
Prefer to skip ahead without guidance?  Use curl to craft a request meeting these requirements:

- HTTP POST request to https://curlingfun:9090/
- Parameter "skip" set to "bow"
- Cookie "end" set to "10"
- Header "Hack" set to "12ft"
```
That can be done with: 
[`curl https://curlingfun:9090 -k --data "skip=bow" -b "end=10" -H "Hack: 12ft"`](https://explainshell.com/explain?cmd=curl+https%3A%2F%2Fcurlingfun%3A9090+-k+--data+%22skip%3Dbow%22+-b+%22end%3D10%22+-H+%22Hack%3A+12ft%22)

Which garners the response:

!!! quote "Excellent!  Now, use curl to access this URL: https://curlingfun:9090/../../etc/button"

We can do that with [`curl https://curlingfun:9090/../../etc/button --path-as-is -k`](https://explainshell.com/explain?cmd=curl+https%3A%2F%2Fcurlingfun%3A9090%2F..%2F..%2Fetc%2Fbutton+--path-as-is+-k)

!!! quote "Great!  Finally, use curl to access the page that this URL redirects to: https://curlingfun:9090/GoodSportsmanship"

The `-L` argument tells curl to follow redirections, so: `curl -k https://curlingfun:9090/GoodSportsmanship -L`


## Response

!!! quote "Bow Ninecandle"
    Bravo! Look at you, curling through that like a true web wizard!

    You zipped through that challenge faster than a curling stone on enchanted ice!

    You know... rumor has it you can breeze through this with just three commands. Why don’t you give it a whirl?

    Wait... did you just slice and dice it all into three commands? My stars, you're a Curling conjurer!
