### Firmware / RE / OffHub

Flag: `CTF{I_kn0W_tH15_Fs}`

> After unpacking the firmware archive, you now have a binary in which to go hunting. Its now time to walk around the firmware and see if you can find anything.

`attachment`

The ZIP contains a big `challenge.ext4.gz` file. EXT4 is the name of a filesystem, so my first guess is we have a disk image.

Let's do a loop mount, so we can explore the filessystem. In the root of this filesystem we see a interesting file:

```
$ zcat .mediapc_backdoor_password.gz
CTF{I_kn0W_tH15_Fs}
```
