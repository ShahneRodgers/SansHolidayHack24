---
icon: material/text-box-outline
---

# Hardware Hacking

**Difficulty**: :fontawesome-solid-star::fontawesome-regular-star::fontawesome-regular-star::fontawesome-regular-star::fontawesome-regular-star:<br/>

## Objective

!!! question "Request"
    Ready your tools and sharpen your wits—only the cleverest can untangle the wires and unlock Santa’s hidden secrets!

??? quote "Jewel Loggins"
    Hello there! I’m Jewel Loggins.

    I hate to trouble you, but I really need some help. Santa’s Little Helper tool isn’t working, and normally, Santa takes care of this… but with him missing, it’s all on me.

    I need to connect to the UART interface to get things running, but it’s like the device just refuses to respond every time I try.

    I've got all the right tools, but I must be overlooking something important. I've seen a few elves with similar setups, but everyone’s so busy preparing for Santa’s absence.

    If you could guide me through the connection process, I’d be beyond grateful. It’s critical because this interface controls access to our North Pole access cards!

    We used to have a note with the serial settings, but apparently, one of Wombley’s elves shredded it! You might want to check with Morcel Nougat—he might have a way to recover it.

    Search the terminal thoroughly; passwords sometimes get left out in the open.

    Once you've found it, modify the entry for card number 42 to grant access. Sounds simple, right? Let’s get to it!


## Hints

??? tip "On the Cutting Edge"
    Hey, I just caught wind of this neat way to piece back shredded paper! It's a fancy heuristic detection technique—sharp as an elf’s wit, I tell ya! Got a sample Python script right here, courtesy of Arnydo. Check it out when you have a sec: [heuristic_edge_detection.py](https://gist.github.com/arnydo/5dc85343eca9b8eb98a0f157b9d4d719)."

??? tip "Shredded to Pieces"
    Have you ever wondered how elves manage to dispose of their sensitive documents? Turns out, they use this fancy shredder that is quite the marvel of engineering. It slices, it dices, it makes the paper practically disintegrate into a thousand tiny pieces. Perhaps, just perhaps, we could reassemble the pieces?

??? tip "It's in the signature"
    I seem to remember there being a handy HMAC generator included in CyberChef.

??? tip "Hidden in plain sight"
    It is so important to keep sensitive data like passwords secure. Often times, when typing passwords into a CLI (Command Line Interface) they get added to log files and other easy to access locations. It makes it trivial to step back in history and identify the password.


## Solution

### Silver
To begin this challenge, we first need to complete the Frosty Keypad in order to get 
[the shredded paper](https://holidayhackchallenge.com/2024/shreds.zip). We can then extract that so
that we have a `slices` directory with each paper fragment, and use the 
[recommended Python script](https://gist.github.com/arnydo/5dc85343eca9b8eb98a0f157b9d4d719) to put
it back together.

![The semi-reconstructed paper](../../img/hardware1.png)

While a bit hard to read, it's good enough to show us the settings we need:
```
Baud 115200
Parity even
Data: 7 bits
Stop bits 1 bit
Flow control rts
```

so we can connect up the UART, switch down to 3V and connect:

[The UART all connected up](../../img/hardware2.png)

!!! quote "Jewel Loggins"
    Fantastic! You managed to connect to the UART interface—great work with those tricky wires! I couldn't figure it out myself…

    Next, we need to access the terminal and modify the access database. We're looking to grant access to card number 42.

    Start by using the slh application—that’s the key to getting into the access database. Problem is, the ‘slh’ tool is password-protected, so we need to find it first.

After switching over to the "Hardware Hacking 102" terminal, we can look in the history to see the slh
tool has previously been run. Since the password is passed in through a CLI argument, we simply need
to update the card number.

!!! success "`slh --passcode CandyCaneCrunch77 --set-access 1 --id 42`"
    ...
    Card 42 granted access level 1.
    
### Gold

!!! quote "Jewel Loggins"
    Rumor has it you might be able to bypass the hardware altogether for the gold medal. Why not see if you can find that shortcut?

In main.js, we can see the following comment:
```javascript
// Build the URL with the request ID as a query parameter
// Word on the wire is that some resourceful elves managed to brute-force their way in through the v1 API.
// We have since updated the API to v2 and v1 "should" be removed by now.
// const url = new URL(`${window.location.protocol}//${window.location.hostname}:${window.location.port}/api/v1/complete`);
```

!!! success "To complete gold, we simply have to send the same request to /api/v1 instead of /api/v2"

!!! quote "Jewel Loggins"
    Wow! You're amazing at this! Clever move finding the password in the command history. It’s a good reminder about keeping sensitive information secure…

    There’s a tougher route if you're up for the challenge to earn the Gold medal. It involves directly modifying the database and generating your own HMAC signature.

    I know you can do it—come back once you've cracked it!

For gold, we can see that `slh` is using a SQLite database. We can use the following
to view the tables and their contents:

```
slh@slhconsole\> sqlite3
SQLite version 3.40.1 2022-12-28 14:03:47
Enter ".help" for usage hints.
Connected to a transient in-memory database.
Use ".open FILENAME" to reopen on a persistent database.
sqlite> .tables
sqlite> .open access_cards
sqlite> .schema config
CREATE TABLE config (
        	id INTEGER PRIMARY KEY,
        	config_key TEXT UNIQUE,
        	config_value TEXT
    	);
sqlite> .schema access_cards
CREATE TABLE access_cards (
        	id INTEGER PRIMARY KEY,
        	uuid TEXT,
        	access INTEGER,
        	sig TEXT
    	);

> SELECT * FROM config;
1|hmac_secret|9ed1515819dec61fd361d5fdabb57f41ecce1a5fe1fe263b98c0d6943b9b232e
2|hmac_message_format|{access}{uuid}
3|admin_password|3a40ae3f3fd57b2a4513cca783609589dbe51ce5e69739a33141c5717c20c9c1
4|app_version|1.0

> SELECT * FROM access_cards WHERE id == 42;
42|c06018b6-5e80-4395-ab71-ae5124560189|0|ecb9de15a057305e5887502d46d434c9394f5ed7ef1a51d2930ad786b02f6ffd
```

So presumably we need to use 42's uuid and 1 (to set the access to admin) to create
a new sig with the configured hmac_secret.

!!! warning "Testing the signatures"
    Initially, this confused me because using the admin password with known values to try
    to recreate the sig didn't work. So I tried solving it the silver way and then using
    the values dumped from the database.

    Naturally, that didn't work; the existing signatures won't recreate because the key
    we're provided is different from the key `slh` is using.

We can use [Cyberchef](https://gchq.github.io/CyberChef/#recipe=HMAC(%7B'option':'Latin1','string':'9ed1515819dec61fd361d5fdabb57f41ecce1a5fe1fe263b98c0d6943b9b232e'%7D,'SHA256'\)&input=MWMwNjAxOGI2LTVlODAtNDM5NS1hYjcxLWFlNTEyNDU2MDE4OQ)
to create our new signature, and then run the following SQL command to update the database:

!!! success ""
    ```
    UPDATE access_cards SET access=1, sig='135a32d5026c5628b1753e6c67015c0f04e26051ef7391c2552de2816b1b7096' WHERE id=42;
    ```


## Response

!!! quote "Jewel Loggins"
    Brilliant work! We now have access to… the Wish List! I couldn't have done it without you—thank you so much!
