---
layout: post
title: Debugging DLL Loading Errors
date: 2022-07-01
---

## Problem

You've made an application that works fine in your development environment,
but fails to start in clear environment.
You seemed to copy all the needed DLLs, but the error dialog like below keeps showing up.

![](/assets/images/debugging-dll-loading-errors/dll-loading-error.png)

## Prerequisites

1. Some application fails to start due to DLL loading errors.

2. You know where to get the missing DLL(s) from. For example, MSYS2/MinGW64/Cygwin/GTK DLLs, see [Example](#example)

3. You have
[Debugging Tools for Windows](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/debugger-download-tools)
already installed.


## Debugging algorithm

1. Run Command Prompt (**cmd.exe**)

2. Go to directory where **APP.exe** is located

   ```
   > cd PATH_TO\APP.exe
   ```

3. Add **gflags**, **WinDbg**, **CDB** to **PATH**

   ```
   > set PATH=C:\Program Files (x86)\Windows Kits\10\Debuggers\x64\;%PATH%
   ```

4. Enable loader snaps for **APP.exe**

   ```
   > gflags /i APP.exe +sls
   ```

5. Run **APP.exe** under **WinDbg** or **CDB**

   I found that using **CDB** to find and copy the missing DLL(s) to be much more convenient than
   using **WinDbg**, since you don't leave the terminal.

   ```
   > cdb -c "g;q" APP.exe
   ```

6. Look for the DLL loading errors like


   ```
   ... ERROR: Unable to load DLL: "some-name.dll",
   ```

7. Copy **some-name.dll** to the current directory

   ```
   > copy PATH_TO\some-name.dll .
   ```

8. Continue the steps 5 to 7 until there's no more DLL loading errors

9. Disable loader snaps for **APP.exe**

   ```
   > gflags /i APP.exe -sls
   ```

10. You're done

## Example


```
> cdb -c "g;q" node src\main.js 2>nul | findstr /R /c:"Unable to load DLL: .*\.dll"
... LdrpProcessWork - ERROR: Unable to load DLL: "libglib-2.0-0.dll", ...
... LdrpProcessWork - ERROR: Unable to load DLL: "libgmodule-2.0-0.dll", ...
```

```
> copy \msys64\mingw64\bin\libglib-2.0-0.dll .
        1 file(s) copied.
```

```
> copy \msys64\mingw64\bin\libgmodule-2.0-0.dll .
        1 file(s) copied.
```

You continue the above commands sequence until there's no more DLL loading errors.

## Post Scriptum

For simple cases [Process Monitor](https://docs.microsoft.com/en-us/sysinternals/downloads/procmon) or
even [Dependency Walker](https://dependencywalker.com/) might be easier to use.

## References

* [GFlags](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/gflags)
* [WinDbg](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/debugger-download-tools)
