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

Below is a list of library versions for COMCTL32.DLL, and the platforms you would expect to find
these versions.

| **Version** | **Distribution platform** |
| --- | --- |
| 4.00 | Microsoft Windows 95/Windows NT 4.0 |
| 4.70 | Microsoft Internet Explorer 3.x |
| 4.71 | Microsoft Internet Explorer 4.0 |
| 4.72 | Microsoft Internet Explorer 4.01, Windows 98, late NT 4.0 |
| 5.00 | Microsoft Windows 98 SE and Internet Explorer 5 |
| 5.01 | Microsoft Windows ME and 2000, Internet Explorer 5, 5.5, 6.0 |
| 5.01 | Microsoft Windows XP, Vista, Server 2003, Server 2008, Windows 7 |
| 6.0	 |	Windows XP (SxS), Windows Server 2003 (SxS)	 |
| 6.10	|	Windows Vista (SxS), Windows Server 2008 (SxS), Windows 7 (SxS) |

SXS (versions 6 and 6.10) are side-by-side (SxS) assemblies and must be requested via an application manifest. See Geoff Chappell's [COMCTL32 Versions](https://www.geoffchappell.com/studies/windows/shell/comctl32/history/index.htm) for more details. 

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
