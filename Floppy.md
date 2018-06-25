### Floppy / MISC / Foobanizer9000

Flag: `CTF{qeY80sU6Ktko8BJW}`

> Using the credentials from the letter, you logged in to the Foobanizer9000-PC. It has a floppy drive...why? There is an .ico file on the disk, but it doesn't smell right..

`attachment`

The ZIP contains an .ico file:

```
$ file foo.ico
foo.ico: MS Windows icon resource - 1 icon, 32x32, 16 colors
```

But looking at the contents I see more data and the `PK` catches my eye twice! This is a signature of a ZIP file.

```
$ hexdump -C foo.ico
00000000  00 00 01 00 01 00 20 20  10 00 00 00 00 00 e8 02  |......  ........|
00000010  00 00 16 00 00 00 28 00  00 00 20 00 00 00 40 00  |......(... ...@.|
00000020  00 00 01 00 04 00 00 00  00 00 80 02 00 00 00 00  |................|
00000030  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
...
# below is our 1st PK signature, further down is `driver.txt`
000002f0  00 00 00 00 00 00 00 00  00 00 00 00 00 50 4b 03  |.............PK.|
00000300  04 14 00 00 00 08 00 8d  81 d6 4c fd ee 87 3e 7b  |..........L...>{|
00000310  00 00 00 88 00 00 00 0a  00 1c 00 64 72 69 76 65  |...........drive|
00000320  72 2e 74 78 74 55 54 09  00 03 ca 03 2d 5b fc 03  |r.txtUT.....-[..|
00000330  2d 5b 75 78 0b 00 01 04  4f 9a 01 00 04 53 5f 01  |-[ux....O....S_.|
00000340  00 1d 8b 41 0a c2 30 14  05 f7 39 c5 3b 80 16 57  |...A..0...9.;..W|
00000350  d2 ad 0a a2 76 5b 11 97  21 f9 6d 83 26 bf fe 24  |....v[..!.m.&..$|
00000360  0d 45 bc bb d5 61 36 b3  98 76 70 11 8b 69 20 58  |.E...a6..vp..i X|
00000370  71 13 09 3a 96 7f ee 9e  d9 bb 90 fd ba a1 19 27  |q..:...........'|
00000380  2d b6 68 21 8c 3a c6 c2  62 11 13 8b ee 97 8b 26  |-.h!.:..b......&|
00000390  67 a8 52 f8 71 68 8f ef  17 dd eb 4d bc 6e 9b f4  |g.R.qh.....M.n..|
000003a0  e0 7a 7f b9 7d 94 3a 07  18 1d 09 dc 81 3c 49 4f  |.z..}.:......<IO|
# and below another one
000003b0  c1 cc 2b 48 0e 28 a5 54  86 bd fa 02 50 4b 03 04  |..+H.(.T....PK..|
000003c0  14 00 00 00 08 00 01 81  d6 4c e6 85 84 66 d6 00  |.........L...f..|
000003d0  00 00 e1 00 00 00 07 00  1c 00 77 77 77 2e 63 6f  |..........www.co|
000003e0  6d 55 54 09 00 03 c1 02  2d 5b db 03 2d 5b 75 78  |mUT.....-[..-[ux|
000003f0  0b 00 01 04 4f 9a 01 00  04 53 5f 01 00 3d ca d1  |....O....S_..=..|
00000400  4e c2 30 14 00 d0 77 13  fe a1 18 54 50 aa 77 c9  |N.0...w....TP.w.|
...
```

Let's try unzipping the .ico file:

```
$ unzip -l foo.ico
Archive:  foo.ico
warning [foo.ico]:  765 extra bytes at beginning or within zipfile
  (attempting to process anyway)
  inflating: driver.txt
  inflating: www.com

$ cat driver.txt
This is the driver for the Aluminum-Key Hardware password storage device.
     CTF{qeY80sU6Ktko8BJW}

In case of emergency, run www.com
```

And there is our flag!

An alternative method is to use `binwalk` to extract contained stuff:

```
$ binwalk -e foo.ico

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
765           0x2FD           Zip archive data, at least v2.0 to extract, compressed size: 123, uncompressed size: 136, name: driver.txt
956           0x3BC           Zip archive data, at least v2.0 to extract, compressed size: 214, uncompressed size: 225, name: www.com
1392          0x570           End of Zip archive

$ ls
_foo.ico.extracted
foo.ico

$ find _foo.ico.extracted/
_foo.ico.extracted/
_foo.ico.extracted/driver.txt
_foo.ico.extracted/www.com
_foo.ico.extracted/2FD.zip
```
