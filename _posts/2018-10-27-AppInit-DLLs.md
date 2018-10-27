---
layout: post
title:  "AppInit DLL persistence"
date:   2018-10-27 19:14:00 +0200
categories: 
---

This one is a well known persistance mechanism on Windows for attackers, so I won't go into too much details.

However, here are some thing you should be aware of:

* The registry key for 64-bit applications is `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT \CurrentVersion\Windows`.
* The registry value is `AppInit_DLLs` and should contain the path to the DLL. Do not use spaces in the path.
* AppInit DLLs will never work if secure boot is enabled.
* Be sure to also create a registry value `RequireSignedAppInit_DLLs` and set it to 0, because by default it is set to 1.
* Be sure to check if `LoadAppInit_DLLs` is set to 1. On a Windows 10 box on which I did the test, it was set to 0.
* Your DLL may only use functions from `ntdll.dll` and `kernel32.dll`, because these are the only DLLs that are loaded when the AppInit DLLs, load.

An example code is here:

```
// dllmain.cpp : Defines the entry point for the DLL application.
#include "stdafx.h"

typedef int(__stdcall *pfMessageBox)(HWND    hWnd,LPCTSTR lpText,LPCTSTR lpCaption,UINT    uType);

BOOL APIENTRY DllMain( HMODULE hModule,
                       DWORD  ul_reason_for_call,
                       LPVOID lpReserved
                     )
{
HMODULE hUser32;
pfMessageBox pMessageBox;
    switch (ul_reason_for_call)
    {
    case DLL_PROCESS_ATTACH:
hUser32 = LoadLibrary(L"user32.dll");
pMessageBox = (pfMessageBox)GetProcAddress(hUser32, "MessageBoxW");
//pMessageBox(NULL, L"Hello world from DLL!", L"Hello world", 0x0);
MessageBox(NULL, L"Hello world from DLL!", L"Hello world", 0x0);
    case DLL_THREAD_ATTACH:
    case DLL_THREAD_DETACH:
    case DLL_PROCESS_DETACH:
        break;
    }
    return TRUE;
}
```
