---
layout: post
title:  "When to keep the pipe open"
date:   2018-08-21 19:14:00 +0200
categories: 
---

When exploiting binaries, usually on some Linux box, you are feeding input in some way to the application. 

Sometimes this is as an argument to the binary, e.g. `argv[1]`, or an environment variable. In this case, you would call the vulnerable program in some way like:

```
$ ./vuln $(python -c 'print "My_exploitation_string"')
```

At other times it will take input from STDIN, and you would exploit it for example like this:

```
$ python -c 'print "My_exploitation_string"' | ./vuln
```

Often you will use some kind of shellcode that does an `execve` of `/bin/sh` to spawn a shell. As already (hardly) learned multiple times, sometimes you have to keep "the pipe open", like this:

```
$ (python -c 'print "My_exploitation_string"; cat) | ./vuln
```

But when do I have to do this and when is it not required? (I believe it will never harm).

The answer is you have to keep the pipe open, only if you opened the pipe.

This means that here you do not have to do it, because no input is taken from STDIN:

```
$ ./vuln $(python -c 'print "My_exploitation_string"')
```

However, as here you are feeding stuff via STDIN, you have to keep it open:

```
$ (python -c 'print "My_exploitation_string"; cat) | ./vuln
```

This also means that the following command will not work (exploitation might work but shell will immediatly close): 

```
$ python -c 'print "My_exploitation_string"' | ./vuln
```

Of course, if all your payload is in your shellcode and you do not require any interaction with a shell, you do not care about this.

As a small demo, you can use this program to execute shellcode from `argv[1]`:

```
#include <stdio.h>

int main(int argc, char* argv[]) {
	(*(void(*)()) argv[1])();  
	return 0;
}
```

and this to execute shellcode from STDIN:

```
#include <stdio.h>
#include <unistd.h>

int main(){
	char shellcode[128];
	printf("Feed me some shellcode via STDIN\n");
	read(0, shellcode, 128);
	(*(void(*)()) shellcode)();  
	return 0;
}
```

Do not forget to compile with `-f execstack`, otherwise the shellcode will not be executable:
```
$ gcc -z execstack -m32 -o sc_from_argv1 sc_from_argv1.c
$ gcc -z execstack -m32 -o sc_from_input sc_from_input.c 
```

And now for the demo. This works:

```
admin@exploits1:/tmp/tmp.i3uK8lgKuz$ ./sc_from_argv1 $(python -c 'print "\x31\xc0\x50\xb0\x0b\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\x51\x53\x89\xe1\x31\xd2\xcd\x80"')
$ echo $0 
/bin//sh
```

Does not work:

```
admin@exploits1:/tmp/tmp.i3uK8lgKuz$ python -c 'print "\x31\xc0\x50\xb0\x0b\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\x51\x53\x89\xe1\x31\xd2\xcd\x80"' | $(pwd)/sc_from_input
Feed me some shellcode via STDIN
admin@exploits1:/tmp/tmp.i3uK8lgKuz$ 
```

But this does work:

```
admin@exploits1:/tmp/tmp.i3uK8lgKuz$ (python -c 'print "\x31\xc0\x50\xb0\x0b\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\x51\x53\x89\xe1\x31\xd2\xcd\x80"'; cat) | $(pwd)/sc_from_input
Feed me some shellcode via STDIN
echo $0
/bin//sh
```

Now I know when to `cat` :)


