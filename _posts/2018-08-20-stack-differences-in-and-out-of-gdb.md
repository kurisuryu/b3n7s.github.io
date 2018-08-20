---
layout: post
title:  "Stack layout differences in and out of GDB"
date:   2018-08-20 16:52:00 +0100
categories: 
---

# Stack layout differences in and out of GDB

Sometimes, when developping an exploit, you need the exact same stack layout (and addresses) in and outside of GDB.

It is well known that to obtain this, you need to:

* Make sure the environment variables are identical in- and outside of GDB. Generally speaking, GDB adds 2 environment variables `LINES` and `COLUMNS`.
* Call the binary outside of GDB with the absolute path, as this is the default behavior of GDB.

I always apply this technique with the shell `sh`, and to my knowledge always worked.

Something like this:

```
bash$ env - sh
sh$ export LINES=22
sh$ export COLUMNS=233
sh$ gdb binary
(gdb) 
... find required stack addresses ...
(gdb) quit
sh$ /absolute/path/to/binary $(python -c 'print "EXPLOITSTRING"')
```

Recently, someone told me this did not work in the bash shell. Why? Let's try to answer that question.

Let's do some initial testing with the following code:

```
$ cat stack_layout.c
#include <stdio.h>

int main(){
	int local_var;
	printf("Address of stack variable: 0x%08x\n", &local_var);
	return 0;
}
```

and compile it with:

```
gcc -m32 -o stack_layout stack_layout.c
```

Now lets do a first test in an `sh` shell with a minimum of environment variables:

```
admin@exploits1:/tmp/tmp.i3uK8lgKuz$ env - sh
$ echo $0
sh
$ gdb -q stack_layout
Reading symbols from stack_layout...(no debugging symbols found)...done.
(gdb) r
Starting program: /tmp/tmp.i3uK8lgKuz/stack_layout 
Address of stack variable: 0xffffddec
[Inferior 1 (process 6490) exited normally]
(gdb) show env
PWD=/tmp/tmp.i3uK8lgKuz
LINES=44
COLUMNS=158
(gdb) q
$ export LINES=44
$ export COLUMNS=158
$ env
COLUMNS=158
PWD=/tmp/tmp.i3uK8lgKuz
LINES=44
$ $(pwd)/stack_layout
Address of stack variable: 0xffffddec
```

As we respected our 2 previous conditions (call with absolute path and equalize environment variables), we obtain identical stack addresses.

Let's move on.

Let us try the same thing in an `sh` shell, but without removing all environment variables:

```
admin@exploits1:/tmp/tmp.i3uK8lgKuz$ sh
$ echo $0
sh
$ gdb -q stack_layout
Reading symbols from stack_layout...(no debugging symbols found)...done.
(gdb) run
Starting program: /tmp/tmp.i3uK8lgKuz/stack_layout 
Address of stack variable: 0xffffd32c
[Inferior 1 (process 6500) exited normally]
(gdb) show env
GJS_DEBUG_TOPICS=JS ERROR;JS LOG
USER=admin
XDG_SEAT=seat0
SSH_AGENT_PID=5955
XDG_SESSION_TYPE=x11
SHLVL=1
HOME=/home/admin
OLDPWD=/home/admin
DESKTOP_SESSION=default
GTK_MODULES=gail:atk-bridge
QT_LINUX_ACCESSIBILITY_ALWAYS_ON=1
DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1011/bus
COLORTERM=truecolor
QT_QPA_PLATFORMTHEME=qgnomeplatform
LOGNAME=admin
JOURNAL_STREAM=8:61683
WINDOWID=23068678
_=sh
USERNAME=admin
XDG_SESSION_ID=51
TERM=xterm-256color
GNOME_DESKTOP_SESSION_ID=this-is-deprecated
WINDOWPATH=2
PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
GDM_LANG=en_US.UTF-8
SESSION_MANAGER=local/exploits1:@/tmp/.ICE-unix/5905,unix/exploits1:/tmp/.ICE-unix/5905
XDG_MENU_PREFIX=gnome-
XDG_RUNTIME_DIR=/run/user/1011
DISPLAY=:0
LANG=en_US.UTF-8
XDG_CURRENT_DESKTOP=GNOME
LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=00:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.Z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.zst=01;31:*.tzst=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.cab=01;31:*.jpg=01;35:*.jpeg=01;35:*.mjpg=01;35:*.mjpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=00;36:*.au=00;36:*.flac=00;36:*.m4a=00;36:*.mid=00;36:*.midi=00;36:*.mka=00;36:*.mp3=00;36:*.mpc=00;36:*.ogg=00;36:*.ra=00;36:*.wav=00;36:*.oga=00;36:*.opus=00;36:*.spx=00;36:*.xspf=00;36:
XDG_SESSION_DESKTOP=default
XAUTHORITY=/run/user/1011/gdm/Xauthority
---Type <return> to continue, or q <return> to quit---
SSH_AUTH_SOCK=/run/user/1011/keyring/ssh
SHELL=/bin/bash
QT_ACCESSIBILITY=1
GDMSESSION=default
GJS_DEBUG_OUTPUT=stderr
GPG_AGENT_INFO=/run/user/1011/gnupg/S.gpg-agent:0:1
XDG_VTNR=2
PWD=/tmp/tmp.i3uK8lgKuz
XDG_DATA_DIRS=/usr/share/gnome:/usr/local/share/:/usr/share/
VTE_VERSION=4601
LINES=44
COLUMNS=158
(gdb) q
$ export LINES=44
$ export COLUMNS=158
$ env
GJS_DEBUG_TOPICS=JS ERROR;JS LOG
USER=admin
XDG_SEAT=seat0
SSH_AGENT_PID=5955
XDG_SESSION_TYPE=x11
SHLVL=1
HOME=/home/admin
OLDPWD=/home/admin
DESKTOP_SESSION=default
GTK_MODULES=gail:atk-bridge
QT_LINUX_ACCESSIBILITY_ALWAYS_ON=1
DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1011/bus
COLORTERM=truecolor
QT_QPA_PLATFORMTHEME=qgnomeplatform
LOGNAME=admin
JOURNAL_STREAM=8:61683
WINDOWID=23068678
_=COLUMNS=158
USERNAME=admin
XDG_SESSION_ID=51
TERM=xterm-256color
COLUMNS=158
GNOME_DESKTOP_SESSION_ID=this-is-deprecated
WINDOWPATH=2
PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
GDM_LANG=en_US.UTF-8
SESSION_MANAGER=local/exploits1:@/tmp/.ICE-unix/5905,unix/exploits1:/tmp/.ICE-unix/5905
XDG_MENU_PREFIX=gnome-
XDG_RUNTIME_DIR=/run/user/1011
DISPLAY=:0
LANG=en_US.UTF-8
XDG_CURRENT_DESKTOP=GNOME
LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=00:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.Z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.zst=01;31:*.tzst=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.cab=01;31:*.jpg=01;35:*.jpeg=01;35:*.mjpg=01;35:*.mjpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=00;36:*.au=00;36:*.flac=00;36:*.m4a=00;36:*.mid=00;36:*.midi=00;36:*.mka=00;36:*.mp3=00;36:*.mpc=00;36:*.ogg=00;36:*.ra=00;36:*.wav=00;36:*.oga=00;36:*.opus=00;36:*.spx=00;36:*.xspf=00;36:
XDG_SESSION_DESKTOP=default
XAUTHORITY=/run/user/1011/gdm/Xauthority
SSH_AUTH_SOCK=/run/user/1011/keyring/ssh
SHELL=/bin/bash
QT_ACCESSIBILITY=1
GDMSESSION=default
GJS_DEBUG_OUTPUT=stderr
GPG_AGENT_INFO=/run/user/1011/gnupg/S.gpg-agent:0:1
XDG_VTNR=2
PWD=/tmp/tmp.i3uK8lgKuz
XDG_DATA_DIRS=/usr/share/gnome:/usr/local/share/:/usr/share/
LINES=44
VTE_VERSION=4601
$ $(pwd)/stack_layout
Address of stack variable: 0xffffd32c
```

Again we obtain the same addresses in- and outside of GDB.

So far, so good, Let us now test this with `bash` and a minimum of environment variables:

```
admin@exploits1:/tmp/tmp.i3uK8lgKuz$ env - bash
admin@exploits1:/tmp/tmp.i3uK8lgKuz$ echo $0
bash
admin@exploits1:/tmp/tmp.i3uK8lgKuz$ env
LS_COLORS=
PWD=/tmp/tmp.i3uK8lgKuz
SHLVL=1
_=/usr/bin/env
admin@exploits1:/tmp/tmp.i3uK8lgKuz$ gdb -q stack_layout
Reading symbols from stack_layout...(no debugging symbols found)...done.
(gdb) r
Starting program: /tmp/tmp.i3uK8lgKuz/stack_layout 
Address of stack variable: 0xffffddcc
[Inferior 1 (process 6517) exited normally]
(gdb) show env
LS_COLORS=
PWD=/tmp/tmp.i3uK8lgKuz
SHLVL=1
_=/usr/bin/gdb
LINES=44
COLUMNS=158
(gdb) q
admin@exploits1:/tmp/tmp.i3uK8lgKuz$ export LINES=44
admin@exploits1:/tmp/tmp.i3uK8lgKuz$ export COLUMNS=158
admin@exploits1:/tmp/tmp.i3uK8lgKuz$ env     
LS_COLORS=
PWD=/tmp/tmp.i3uK8lgKuz
LINES=44
COLUMNS=158
SHLVL=1
_=/usr/bin/env
admin@exploits1:/tmp/tmp.i3uK8lgKuz$ $(pwd)/stack_layout
Address of stack variable: 0xffffddbc
admin@exploits1:/tmp/tmp.i3uK8lgKuz$ 
```

Indeed, the addresses in- and outside of GDB are different! They differ with 0x10 bytes in this case.

To find out what is different, let us dump the stack with the following little program:

```
#include <stdio.h>

int main(){
	int stack_var;
	int stack_address = (int)&stack_var;
	int end_of_stack = stack_address | 0x00000fff;
	printf("Stack variable is @: 0x%x\n", stack_address);
	printf("Stack end is @: 0x%x\n", end_of_stack);
	for (int i=stack_address; i <= end_of_stack; i += 0x10){
		printf("\n0x%08x: ", i);
		for (int j=i; j<i+0x10; j++){
			if (j > end_of_stack)
				break;
			printf("%02x ", *(unsigned char *)j);
		}
		for (int j=i; j<i+0x10; j++) {
			if (j > end_of_stack)
				break;
			if ((0x20 <= *(unsigned char*)j) && (*(unsigned char*)j < 0x80)) {
				printf("%c", *(unsigned char *)j);
			} else {
				printf(".");
			}
		}
	}
	printf("\n");
	return 0;
}

```

Basically, it will do a hex dump from the stack. We can compile it with:

```
admin@exploits1:/tmp/tmp.i3uK8lgKuz$ gcc -m32 -o dump_stack dump_stack.c
```

Let us now dump the stack in and outside of GDB and diff the results:

```
admin@exploits1:/tmp/tmp.i3uK8lgKuz$ env - bash
admin@exploits1:/tmp/tmp.i3uK8lgKuz$ echo $0
bash
admin@exploits1:/tmp/tmp.i3uK8lgKuz$ env
LS_COLORS=
PWD=/tmp/tmp.i3uK8lgKuz
SHLVL=1
_=/usr/bin/env
admin@exploits1:/tmp/tmp.i3uK8lgKuz$ gdb -q dump_stack
Reading symbols from dump_stack...(no debugging symbols found)...done.
(gdb) show env
LS_COLORS=
PWD=/tmp/tmp.i3uK8lgKuz
SHLVL=1
_=/usr/bin/gdb
LINES=44
COLUMNS=158
(gdb) r > in_gdb.hex
Starting program: /tmp/tmp.i3uK8lgKuz/dump_stack > in_gdb.hex
[Inferior 1 (process 6658) exited normally]
(gdb) q
admin@exploits1:/tmp/tmp.i3uK8lgKuz$ export LINES=44
admin@exploits1:/tmp/tmp.i3uK8lgKuz$ export COLUMNS=158
admin@exploits1:/tmp/tmp.i3uK8lgKuz$ $(pwd)/dump_stack > out_gdb.hex
```

If you compare closely the files `in_gdb.hex` and `out_gdb.hex`, you will find the problem. 

`bash` defines and `_` environment variable. This variable takes a different value in- and outside of GDB:

* In GDB: `/usr/bin/gdb`
* Outside GDB: `/absolute/path/to/executable`

If you want identical stack setups, the absolute path to your binary has to have the same length as the absolute path to GDB.

Let's test this:

```
admin@exploits1:/tmp/tmp.i3uK8lgKuz$ echo -n "/usr/bin/gdb" | wc -c
12
admin@exploits1:/tmp/tmp.i3uK8lgKuz$ echo -n "/tmp/AAAAAAA" | wc -c
12
admin@exploits1:/tmp/tmp.i3uK8lgKuz$ cp stack_layout /tmp/AAAAAAA
admin@exploits1:/tmp/tmp.i3uK8lgKuz$ cd ..
admin@exploits1:/tmp$ env - bash
admin@exploits1:/tmp$ gdb -q AAAAAAA
Reading symbols from AAAAAAA...(no debugging symbols found)...done.
(gdb) run
Starting program: /tmp/AAAAAAA 
Address of stack variable: 0xffffddfc
[Inferior 1 (process 7118) exited normally]
(gdb) show env
LS_COLORS=
PWD=/tmp
SHLVL=1
_=/usr/bin/gdb
LINES=44
COLUMNS=158
(gdb) q
admin@exploits1:/tmp$ export LINES=44
admin@exploits1:/tmp$ export COLUMNS=158
admin@exploits1:/tmp$ /tmp/AAAAAAA 
Address of stack variable: 0xffffddfc
```

Yay, it works!

BTW, the man page of bash explains it to us:

```
When bash invokes an external command, the variable _ is set to the full filename of the command and passed to that command in its environment.
```
