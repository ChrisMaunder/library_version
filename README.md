# Determining the version number of the Windows system libraries

A simple way to determine the version of the Comctl32.dll, Shell32.dll and Shlwapi.dll system libraries

## Introduction

With each Windows operating system update, and each update of Internet Explorer, 
we are issued with a new version of the system libraries Comctl32.dll, Shell32.dll, 
and Shlwapi.dll. These libraries contain the bulk of the code that handles the
common control and shell functionality of windows. The problem is, each version is 
different, and it can sometimes be critical to know which platform you are 
targeting when writing software that specifically uses functionality in these 
libraries.

Below is a list of library versions, and the platforms you would expect to find
these versions.

| <!----> | <!----> |
| --- | --- |
| **Version** | **Distribution platform** |
| 4.00 | Microsoft Windows 95/Windows NT 4.0 |
| 4.70 | Microsoft Internet Explorer 3.x |
| 4.71 | Microsoft Internet Explorer 4.0 |
| 4.72 | Microsoft Internet Explorer 4.01 and Windows 98 |
| 5.00 | Microsoft Windows 2000 and Internet Explorer 5 |

There are a couple of points here:

Firstly, For Windows 95, Internet Explorer 4.0 can be installed without the integrated
shell. Thus, you may have either version 4.71 or 4.72 of Comctl32.dll or Shlwapi.dll 
on a Windows 95 box, but the version of Shell32.dll may be different. On a Windows 98
system all three libraries will have the same value.

Secondly, Internet Explorer will update Comctl32.dll and Shlwapi.dll to version 5.0,
but will not update the Shell32.dll library. Thus, if you install IE 5 on a non-Windows
2000 system, your Shell32.dll version will again be different to your Comctl32.dll and
 Shlwapi.dll versions.

## So how do you test?

Microsoft kindly provides us with a handy function to test for the various
 flavours.

```cpp
#define PACKVERSION(major,minor) MAKELONG(minor,major)

DWORD GetDllVersion(LPCTSTR lpszDllName)
{
    HINSTANCE hinstDll;
    DWORD dwVersion = 0;

    hinstDll = LoadLibrary(lpszDllName)

    if(hinstDll)
    {
        DLLGETVERSIONPROC pDllGetVersion;

        pDllGetVersion = (DLLGETVERSIONPROC) GetProcAddress(hinstDll, "DllGetVersion");

        /*Because some DLLs may not implement this function, you
         *must test for it explicitly. Depending on the particular 
         *DLL, the lack of a DllGetVersion function may
         *be a useful indicator of the version.
        */
        
        if(pDllGetVersion)
        {
            DLLVERSIONINFO dvi;
            HRESULT hr;

            ZeroMemory(&dvi, sizeof(dvi));
            dvi.cbSize = sizeof(dvi);

            hr = (*pDllGetVersion)(&dvi);

            if(SUCCEEDED(hr))
            {
                dwVersion = PACKVERSION(dvi.dwMajorVersion, dvi.dwMinorVersion);
            }
        }
        
        FreeLibrary(hinstDll);
    }
    return dwVersion;
}
```

It's noted that the above will work only for version 4.71 and above of the libraries.
Oh well...

So - to use the above function in your code, you would do something like the following:

```cpp
if (GetDllVersion(TEXT("comctl32.dll")) >= PACKVERSION(4,71))
{
    // We have at least version 4.71
}
else
{
    // Version 4.70 or below...
}
```
