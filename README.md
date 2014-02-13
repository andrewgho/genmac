genmac - generate valid looking random MAC address
==================================================

This program generates a random MAC address, suitable for testing, or to
change your local MAC address for whatever purpose. It differs from
other, similar, programs in that it optionally looks up an actual
manufacturer prefix (Organizational Unique Identifier) to make the MAC
address look more like ones that would occur in the real world.

Getting Started
---------------

Download the source from GitHub:

* <https://github.com/andrewgho/genmac>

Install the program by just copying it anywhere in your `$PATH`.

For usage:

    $ genmac -h

To generate a single, random MAC address:

    $ genmac

Description
-----------

MAC addresses are generally 48-bit unique identifiers assigned to
network interfaces on a local, physical network, most commonly, an
Ethernet network, or a wireless (Wi-Fi) network that acts like one. MAC
addresses are usually represented as six octets (bytes), each one
printed as a two-digit hexadecimal number, separated by colons, or
sometimes hyphens, for example:

    d4:01:29:9d:10:e2

MAC addresses are typically set in hardware, for example, in read-only
memory on a network interface card. However, most operating systems
allow the MAC address to be overridden at the OS network layer.

This program assists in generating a MAC address that can be used to
simulate real-world MAC addresses in testing, or to use to override a
hardware-set MAC address, whether for testing, or as part of a layered
effort to mask your identity.

It is pretty straightforward to generate a random MAC address from the
command line, or with a scripting language:

    $ openssl rand -hex 6 | sed 's/\(..\)/\1:/g; s/.$//'

    $ ruby -e 'puts (1..6).collect { "%02x" % [rand 255] }.join(":")'

However, there are two non-random aspects to real-world MAC addresses.
First, there is a special meaning to the least significant two bits in
the first octet; and second, every organization (usually manufacturer)
in the world that creates network interfaces is assigned an IEEE
registered "Organizational Unique Identifier" (OUI), which forms the
first three octets of the MAC address.

`genmac` can take both of these non-random aspects into account when
creating an otherwise random MAC address. To generate a mostly-random
MAC address, run `genmac` by itself:

    $ genmac

In the MAC address printed to stdout, all values are random, except that
the first octet has the second-least significant bit, which determines
whether a MAC address is universally or locally administered, set to the
former (universally administered), just as it would be for an actual
manufacturer MAC address.

To mark your MAC address as locally administered, which is conventional
for addresses that are purposely fake or random, pass the `-l` or
`--local` flag:

    $ genmac -l

To generate a large number of MAC addresses, pass `-n` with a numeric
argument, to print that many addresses:

    $ genmac -n 10

To generate a more realistic MAC address, the first three octets should
be taken from a real-world OUI. To do this, you need to download a copy
of the IEEE maintained list of Organizational Unique Identifiers, which
is updated daily:

    $ wget http://standards.ieee.org/develop/regauth/oui/oui.txt

Save the file to the same directory `genmac` lives in (or, pass the `-f`
command line argument to point at where the file lives). Then, run
`genmac` with a string that matches an organization name. For example,
to generate a single MAC address that could plausibly have been assigned
to a Broadcom device:

    $ genmac broadcom

The organization argument is interpreted as a case-insensitive Ruby
regular expression, anchored at the left, with a word boundary on the
right. The following example of creates MAC addresses from multiple
possible organizations:

    $ genmac -n 20 'apple|motorola|nokia|samsung'

To see what organization names match, pass the `-v` (`--verbose`) flag:

    $ genmac -v google

This program simply prints out the random MAC address. If you wish to
change your local network interface MAC address, you will need to look
for instructions specific to your OS and version.

See Also
--------

* <https://en.wikipedia.org/wiki/MAC_address>
* <https://en.wikipedia.org/wiki/Organizationally_Unique_Identifier>
* <https://standards.ieee.org/develop/regauth/oui/public.html>

### Prior Art ###

* [GNU MAC Changer](https://github.com/alobbs/macchanger)
  is a more fully featured program that ships with its own OUI database.
  `genmac` differs in that it parses the raw IEEE OUI file, and is a
  single-file Ruby script that can be run without compilation.
* [SpoofMAC](https://github.com/feross/SpoofMAC)
  is a Python script and library to show and set MAC addresses.
  `genmac` differs in that it considers real world OUIs instead of
  using a single hardcoded one, and is a standalone program that can
  be run without being installed as a library.

Author
------

Andrew Ho (<andrew@zeuscat.com>)

License
-------

    Copyright (c) 2014, Andrew Ho
    All rights reserved.
    
    Redistribution and use in source and binary forms, with or without
    modification, are permitted provided that the following conditions
    are met:
    
    1. Redistributions of source code must retain the above copyright
       notice, this list of conditions and the following disclaimer.
    
    2. Redistributions in binary form must reproduce the above copyright
       notice, this list of conditions and the following disclaimer in the
       documentation and/or other materials provided with the distribution.
    
    3. Neither the name of Andrew Ho nor the names of its contributors may
       be used to endorse or promote products derived from this software
       without specific prior written permission.
    
    THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS
    IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED
    TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A
    PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT
    HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
    SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED
    TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR
    PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF
    LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
    NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
    SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
