# covertutils
## A framework for Backdoor programming!

[![Documentation Status](https://readthedocs.org/projects/covertutils/badge/?version=latest)](http://covertutils.readthedocs.io/en/latest/?badge=latest) [![PyPI version](https://badge.fury.io/py/covertutils.svg)](https://pypi.python.org/pypi/covertutils)          [![GitHub version](https://badge.fury.io/gh/operatorequals%2Fcovertutils.svg)](https://github.com/operatorequals/covertutils) [![Build Status](https://travis-ci.org/operatorequals/covertutils.svg?branch=master)](https://travis-ci.org/operatorequals/covertutils)

[Documentation Page](https://covertutils.readthedocs.io)

[Blog Post in Securosophy describing some internals](https://securosophy.com/2017/04/22/reinventing-the-wheel-for-the-last-time-the-covertutils-package/)

[Arranged Con Presentation about the Package 
(DefCamp #8 | November 9-10)](https://def.camp/speaker/john-torakis/)

### What is it?
This python package automatically handles all communication channel options, like **encryption**, **chunking**, **steganography**, etc.

With all those set with a few lines of code, a programmer can spend time creating the *actual payloads*, *persistense mechanisms*, *shellcodes* and generally **more creative stuff!**!

The security programmers can stop *re-inventing the wheel* by implementing encryption mechanisms both agent-side and handler-side to spend their time to develop more versatile *agents*, and generally feature-full shells!

### Python?
Yes, python, and more specifically **Python2.7** only, for the time being...

### But why Python2?
Several reasons. Mostly because Python2 is **more popular among devices** (*IoT devices*, *old Linux servers*, etc), and backdoor code could run *as-is* on them, without `Freezing`, `Packing`, `PyInstalling`, etc. Backdoors are valuable when they are as cross-platform as possible.
Macs, for example, do not have Python3 installed by default. If you want `covertutils` in Python3, do not complain, read [this reddit flame war dodging](https://www.reddit.com/r/netsec/comments/6rj7b0/a_python_package_for_creating_backdoors_coverutils/) and start PRing...


### Dependencies?
NO! Absolutely no dependencies, only pure python built-ins! The `entropy` package is required for the `tests` though.
This is a package's requirement, to ensure good flow when compiling in executable binaries.


# Summary

## The Entities

### The `Message`
Messages are all things that mean something to the listener. Messages travel through communication channels, and they have to be unaware of the channel they are travelling in. In other words, messages have to be independent of the mean of their transportation.
 *  If the communication channel can handle low length byte-chunks per "burst", the message has to be chunked.
 *  If the communication channel filters certain byte arrays (IDS/IPS, NextGen Firewalls).
 

### The `Stream`
The Stream is a tag that gives certain context to the message. Can be defined and used for arbitrary reasons. Streams, for example, can be used to separate `Shell Commands` from `shellcode` messages.

## The Organizers

### The `Orchestrator`
Orchestrators are the core of data manipulation in `covertutils`. They handle all data transformation methods to translate raw chunks of data into Stream-Message pairs.

### The `Handler`
Handlers tie together the raw byte input/output with the `orchestrators` to provide an interface of:
* `onChunk()`
* `onMessage()`
* `onNotRecognized()`

#### Example :
```python
def onMessage( message, stream ) :
  if stream == 'shell' :
    os.system( message )
```

### The `Shell`
A shell interface with prompt and `stream` control can be spawned from a `Handler` instance with:
``` python

shell = StandardShell(handler, prompt = "(%s:%d)> " % client_addr )
shell.start()
```
```bash
(127.0.0.5:8081)> 
# <Ctrl-C>
Available Streams:
	[ 0] - control
	[ 1] - python
	[ 2] - os-shell
	[99] - Back
Select stream: 2
[os-shell]> uname -a
Linux hostname 4.9.0-kali4-amd64 #1 SMP Debian 4.9.25-1kali1 (2017-05-04) x86_64 GNU/Linux
[os-shell]> !control sysinfo
General:
	Host: hostname
	Machine: x86_64
	Version: #1 SMP Debian 4.9.25-1kali1 (2017-05-04)
	Locale: en_US-UTF-8
	Platform: Linux-4.9.0-kali4-amd64-x86_64-with-Kali-kali-rolling-kali-rolling
	Release: 4.9.0-kali4-amd64
	System: Linux
	Processor: 
	User: unused

Specifics:
	Windows: ---
	Linux: glibc-2.7

[os-shell]> 
# <Ctrl-C>
(127.0.0.5:8081)> q
[!]	Quit shell? [y/N] y
Aborted by the user...

```

### The `Encryption Schemes`
Custom _Stream Ciphers_ are used, designed and implemented from scratch in the `covertutils.crypto` subpackage. Currently a custom _scrambling_ function (`std`) and the standard `CRC32` (`crc`) functions are used to generate the _stream keys_.

The crypto and scrambling algorithms can be tried in the below CLI implementations:

#### Scrambling
``` bash
$ python -m covertutils.crypto.algorithms --length 16 std message_to_digest
f3c7de5e591d2eb7fba938847430e2c0
$ python -m covertutils.crypto.algorithms --length 20 std message_to_digest
413928828205d7af0a5f415f6c0a5014e49c7250
$ python -m covertutils.crypto.algorithms std message_to_digest --length 31
6d9dd92f9eada2611c04a29da18b8b845638aec85d0783617f51dfc72e62ae
$ python -m covertutils.crypto.algorithms std message_to_digest --length 32 --cycles 10
252f9b7175399bae1cb2b02c36f4dbefd5ae6d4971b10f16b25631e45a4efc6c
$ python -m covertutils.crypto.algorithms std message_to_digest --length 32 --cycles 20
4fd94b21d6ee742e7426de512d1565bf1dd1031a1aa9ddd9de263773cfc8888c
$ python -m covertutils.crypto.algorithms std message_to_digest
4fd94b21d6ee742e7426de512d1565bf1dd1031a1aa9ddd9de263773cfc8888c
```

#### Encryption/Decryption
``` bash
$ python -m covertutils.crypto.keys crc keyphrase message_to_encrypt --output b64
SkonjSa1pat95PVhAG9U3DHO
$
$ python -m covertutils.crypto.keys crc keyphrase SkonjSa1pat95PVhAG9U3DHO --input b64 --decrypt
message_to_encrypt
$ #	Change the keyphrase and try to decrypt:
$ python -m covertutils.crypto.keys crc keyphrase2 SkonjSa1pat95PVhAG9U3DHO --input b64 --decrypt
����R��M8�A�q�/�
```

**The `std` algorithm is used by default in all communications.**

## Networking
Networking is not handled by `covertutils`, as python provides great built-in networking API (directly inherited from C). The only requirements for `covertutils` `Handler` instances are **2 functions wrapping the raw data sending and receiving**.

Just pass a `send( raw )` and a `recv()` function to a `Handler` and you have a working *One-Time-Pad* encrypted, bandwidth aware, protocol independent, *password protected*, *multi-usable* channel.

# Further Examples:
Sample TCP/UDP Reverse Shells and TCP Bind Shell scripts can be found in `examples/` directory.

Tutorial and explanation of the architecture can be found in the [CovertUtils Tutorial Restaurant](http://covertutils.readthedocs.io/en/latest/assembling_backdoor.html)!


# Pull Requests?
Certainly! All pull requests that are tested and do not break the existing tests will be accepted!
Especially Pull Requests towards Python2/Python3 compatibility will be greatly appreciated!




# Disclaimer
Usage of `covertutils` for attacking infrastructures without prior mutual consistency can be considered as an illegal activity. It is the final user's responsibility to obey all applicable local, state and federal laws. Authors assume no liability and are not responsible for any misuse or damage caused by this package.
