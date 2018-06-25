### Security by obscurity / MISC / OffHub

Flag: `CTF{CompressionIsNotEncryption}`

> Reading the contents of the screenshot you find that some guy named "John" created the firmware for the OffHub router and stored it on an iDropDrive cloud share. You fetch it and find "John" packed the firmware with an unknown key. Can you recover the package key?

`attachment`

John, john.. How many times can you read John before thinking ['John the ripper' the password cracker](http://openwall.com/john/)? 

Anyways let's first look at the attachment:

```
$ unzip -l 2cdc6654fb2f8158cd976d8ffac28218b15d052b5c2853
Archive:  2cdc6654fb2f8158cd976d8ffac28218b15d052b5c2853232e4c1bafcb632383.zip
  Length      Date    Time    Name
---------  ---------- -----   ----
    11100  00-00-1980 00:00   password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p
---------                     -------
    11100                     1 file
```

So the ZIP contains a ZIP. And that ZIP contains a ZIP. Continue for ~10 levels. Then the ZIP contains a XZ compressed archive. Continue for ~10 levels. Then the XZ contains a BZIP2 archive. More levels. Then GZIP...

I created the following script to automate the unpacking:

```
#!/bin/bash
filename="password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p"

set -e
work=$filename
while [ "$work" != "password" ]; do
  next_work=$(rev <<<$work|cut -c3-|rev)
  sig=$(file $work)
  echo "$sig"
  if [[ $sig =~ 'Zip archiv' ]]; then
    unzip -f $work
  elif [[ $sig =~ 'XZ compressed data' ]]; then
    xzcat $work > $next_work
  elif [[ $sig =~ 'bzip2 compressed data' ]]; then
    bzcat $work > $next_work
  elif [[ $sig =~ 'gzip compressed data' ]]; then
    gzcat $work > $next_work
  else
    echo "Unknown sig: $sig"
    exit
  fi
  work=$next_work
done
```

Finally you end up with `password.x` which is a ZIP:

```
$ unzip -l password.x
Archive:  password.x
  Length      Date    Time    Name
---------  ---------- -----   ----
       32  06-14-2018 13:53   password.txt
---------                     -------
       32                     1 file

$ unzip password.x
Archive:  password.x
[password.x] password.txt password:
```

Ah so we have a password protected ZIP archive...

Cue John the ripper:

```
$ ./zip2john ../password.zip > ../zip.hashes
ver a  efh 5455  efh 7875  password.zip->password.txt PKZIP Encr: 2b chk, TS_chk, cmplen=44, decmplen=32, crc=4341BA5D

$ ./john ../zip.hashes
Loaded 1 password hash (PKZIP [32/64])
Press 'q' or Ctrl-C to abort, almost any other key for status
asdf             (password.zip)
...
```

Within a few seconds we cracked it. The password is `asdf`. Now let's use it:

```
$ unzip -p password.zip
[password.zip] password.txt password:
CTF{CompressionIsNotEncryption}
```
