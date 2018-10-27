---
layout: post
title:  "AppCert DLL persistence"
date:   2018-10-27 19:14:00 +0200
categories: 
---

This persistence method is mentioned in the [ATT&CK framework](https://attack.mitre.org/techniques/T1182/). It is also mentioned in a great blog post of [EndGame](https://www.endgame.com/blog/technical-blog/ten-process-injection-techniques-technical-survey-common-and-trending-process). However, no specific details are provided on how to implement it.

So I tried it and indeed it works. These are the steps I have followed:

1. Create a 64-bit DLL which does this in the DllMain function:

```
BOOL APIENTRY DllMain( HMODULE hModule,
                       DWORD  ul_reason_for_call,
                       LPVOID lpReserved
                     )
{
    switch (ul_reason_for_call)
    {
    case DLL_PROCESS_ATTACH:
MessageBox(NULL, L"Hello world from DLL!", L"Hello world", 0x0);
    case DLL_THREAD_ATTACH:
    case DLL_THREAD_DETACH:
    case DLL_PROCESS_DETACH:
        break;
    }
    return TRUE;
}
```

Of course, as an attacker, you would place your code here.

I have compiled it to 64-bit.

2. Create a registry key:

```
HKLM\System\CurrentControlSet\Control\Session Manager\AppCertDlls
```

3, Add in that registry key a value name `yadayada` with string data `PATH_TO_DLL`.

4. It does not work at every `CreateProcess` strangely. I still have to figure out why. But the DLL can be triggered by e.g.:
  * Launch cmd.exe
  * From cmd.exe, launch notepad.exe
