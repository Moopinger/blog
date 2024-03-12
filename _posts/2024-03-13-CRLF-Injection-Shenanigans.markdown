---
layout: post
title:  "CRLF Injection Shenanigans"
date:   2024-03-12 07:00:02 +0200
categories: CRLF Injection
---

{:refdef: style="text-align: center;"}
![Main banner](/blog/assets/crlfbanner.png)
{: refdef}

## CRLF Injection

The recently unveiled compilation of the ['Top 10 Web Hacking Techniques of 2023'](https://portswigger.net/research/top-10-web-hacking-techniques-of-2023) is a treasure trove of cutting-edge research and ingenious hacks crafted by industry leaders. As I delved into the list, [no.6 caught my attention](https://offzone.moscow/upload/iblock/11a/sagouc86idiapdb8f29w41yaupqv6fwv.pdf) I realized that I had completely missed last year. Astonishingly, it delves into a technique I had assumed was already thoroughly explored and kinda done—CRLF injection— boy was I wrong, as detailed in this insightful research.

[Sergey Bobrov](https://twitter.com/black2fan?lang=en) unveiled a series of innovative exploitation methods for 'request splitting.' And bringing the attack vectors client-side. This underscores the notion that hacking often demands not only technical expertise, of which he possesses abundantly, but also a creative mindset. One particularly intriguing technique involves the manipulation of multipart/form-data upload functionality, standing out as my personal favorite among his inventive approaches.

Sergey knew that when the message is reassembled by the server backend, the actual POST parameters will be appended to the end. Allowing him to control not just the start of a hijacked request via CRLF, but also the end. Combining this with Multipart requests he knew he could craft a malicious page (CSRF), that when viewed by a victim would upload the victim's session information to his account signature (or anywhere accepting multipart uploads). Another amazing part is CSRF-tokens can't save you here.

There is a lot more to [his research and you should check it out.](https://offzone.moscow/upload/iblock/11a/sagouc86idiapdb8f29w41yaupqv6fwv.pdf)

## Detection

This made me revisit CRLF injection within my own bounty hunting programs, the outcomes exceeded my expectations significantly. The success has been so remarkable that I am compelled to share my insights in this post, complete with my detection methods and the Python script I employed to uncover these vulnerabilities

I used a single HTTP request containing two detection methods, that if successfully triggered will return a different HTTP status code. The first being the method Sergey highlights above by simply stating a nonexistant HTTP Version (HTTP/9).

`https://www.moopinger.com/%20HTTP/9%0D%0Ax-end:%20a`

Should generate a ["505/HTTP Version Not Supported"](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/505) response.

The next was a method I have seen because I generated this error often throughout my smuggle testing- When specifying a nonexistant 'Transfer-Encoding' method the server will respond with ["501 Not Implemented"](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/501) So to trigger this we just include 'Transfer-Encoding: nonexistant' so now we have:

`https://www.moopinger.com/%20HTTP/9%0D%0Transfer-Encoding:%20nonexistant%0D%0Ax-end:%20a`

I have created a python script to automate this, and can be found on my GitHub: [https://github.com/Moopinger/crlf-detection-script.](https://github.com/Moopinger/crlf-detection-script) Just add your targets to hosts.txt and run the Python script - It does not offer multiple routines/threads. And, you are damn right there is green console text.

The script first attempts to conduct the detection over HTTP/2 (You know they are all being downgraded to HTTP/1.1 anyway and may allow additional vulnerable target discovery), and if HTTP/2 is not supported is will switch to HTTP/1.1. This has proven a bit more successful throughout my testing, than just testing over HTTP/1.1.

I hope the script helps and you are able to rain down some bounties. Once you have identified a vulnerable endpoint, consult the research mentioned above to begin your exploitation journey. Remember, to have a good working PoC when you submit - demonstrate the risk not the vulnerability. 
