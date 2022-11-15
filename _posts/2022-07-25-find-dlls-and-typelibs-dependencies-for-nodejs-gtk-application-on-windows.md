---
layout: post
title: Find DLLs and Typelibs dependencies for Node.js GTK Application on Windows
date: 2022-07-25
---

In the [previous post]({{ site.baseurl }}{% post_url 2022-07-22-nodejs-gtk-hello-world-on-windows %})
we have made a [GTK](https://www.gtk.org/) Hello World application using [Node.js](https://nodejs.org/).
In this post we will find the dependencies (DLLs and Typelibs) needed for distribution and
in the [next post]({{ site.baseurl }}{% post_url 2022-07-27-package-nodejs-gtk-application-on-windows %})
we will package the application for distribution and learn how start it without showing the terminal prompt.

## Table of contents

- [Prerequisites](#prerequisites)
  - [Install Debugging Tools for Windows](#install-debugging-tools-for-windows)
- [Find needed DLLs](#find-needed-dlls)
- [Find needed Typelibs](#find-needed-typelibs)
- [Automate the dependencies search](#automate-the-dependencies-search)
- [Tear down](#tear-down)
- [What's next?](#whats-next)


## Prerequisites

### Install Debugging Tools for Windows

If you followed along the [previous post]({{ site.baseurl }}{% post_url 2022-07-22-nodejs-gtk-hello-world-on-windows %})
you should have the Windows SDK already installed. We also need to install the
[Debugging Tools for Windows](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/debugger-download-tools)

1. **Start** -> **Settings** -> **Apps**
2. Inside **Search this list** type: **devel**
3. Select **Windows Software Development Kit** -> **Modify**

   ![Find SDK Image](/assets/images/find-dlls-and-typelibs-dependencies-for-nodejs-gtk-application-on-windows/find-sdk.png)

4. **Change** -> **Next**

   ![SDK Change Image](/assets/images/find-dlls-and-typelibs-dependencies-for-nodejs-gtk-application-on-windows/sdk-change.png)

5. Check **Debugging Tools for Windows** -> **Change**

   ![Add Debugging Tools Image](/assets/images/find-dlls-and-typelibs-dependencies-for-nodejs-gtk-application-on-windows/add-debug-tools.png)

6. You're done!

See [here](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/debugger-download-tools#adding-the-debugging-tools-for-windows-if-the-sdk-is-already-installed) if you need more detail.


## Find needed DLLs

In the [previous post]({{ site.baseurl }}{% post_url 2022-07-22-nodejs-gtk-hello-world-on-windows %}) we were able to run the application inside the MinGW64 shell.

![Hello Gtk in MinGW64 Image](/assets/images/find-dlls-and-typelibs-dependencies-for-nodejs-gtk-application-on-windows/hello-gtk-mingw64.png)

Now it's time to find out what DLLs are needed to run in clear environment.

You might want to review [Debugging DLL Loading Errors]({{ site.baseurl }}{% post_url 2022-07-01-debugging-dll-loading-errors %})
for a short introduction.

It is possible to use the Command Prompt or the PowerShell for some steps below. But since I'm going to automate the whole
process and I'm not familiar enough with the Windows shells, I'm going to use the MSYS shell. It has the same Unix tools available
like the MinGW64 shell, but doesn't have the needed dependencies.

![Open MSYS shell Image](/assets/images/find-dlls-and-typelibs-dependencies-for-nodejs-gtk-application-on-windows/open-msys-shell.png)

Now let's run the application inside the MSYS shell:

```
$ cd blog-code/nodejs-gtk-hello-world-on-windows/
$ node.exe index.js
node:internal/modules/cjs/loader:1189
  return process.dlopen(module, path.toNamespacedPath(filename));
                 ^

Error: The specified module could not be found.
\\?\C:\msys64\home\someo\blog-code\nodejs-gtk-hello-world-on-windows\node_modules\node-gtk\lib\binding\node-v93-win32-x64\node_gtk.node
    at Object.Module._extensions..node (node:internal/modules/cjs/loader:1189:18)
    at Module.load (node:internal/modules/cjs/loader:981:32)
    at Function.Module._load (node:internal/modules/cjs/loader:822:12)
    at Module.require (node:internal/modules/cjs/loader:1005:19)
    at require (node:internal/modules/cjs/helpers:102:18)
    at Object.<anonymous> (C:\msys64\home\someo\blog-code\nodejs-gtk-hello-world-on-windows\node_modules\node-gtk\lib\native.js:11:17)
    at Module._compile (node:internal/modules/cjs/loader:1105:14)
    at Object.Module._extensions..js (node:internal/modules/cjs/loader:1159:10)
    at Module.load (node:internal/modules/cjs/loader:981:32)
    at Function.Module._load (node:internal/modules/cjs/loader:822:12) {
  code: 'ERR_DLOPEN_FAILED'
}
```

It fails because [Node-Gtk](https://github.com/romgrk/node-gtk) native module is unable to load some DLLs. But which ones?
To start debugging which DLLs are needed exactly first we need to enable Show Loader Snaps (sls) for **node.exe** to get extended DLLs loading errors in the debugger.

Add **Debugging Tools for Windows** to **PATH**

```
$ export PATH="/c/Program Files (x86)/Windows Kits/10/Debuggers/x64/":$PATH
```


Unfortunately, the command below doesn't work in the MSYS shell by default (run it as administrator otherwise)

```
$ gflags -i node.exe.exe +sls
```

Anyway, run the command below or go to **Start** -> **Windows Kits** -> **Global Flags (X64)**

```
$ cmd.exe /C glags
```

![Gflags enable SLS Image](/assets/images/find-dlls-and-typelibs-dependencies-for-nodejs-gtk-application-on-windows/gflags-enable-sls.png)


Now run the application under the debugger. I found that using **CDB** to find and copy the missing DLLs is much more convenient than using **WinDbg**, since you don't leave the terminal.
The commands "g;q" mean start debugging (go) and quit (q) on error.
It also creates an opportunity to automate the whole searching process, see [below](#automate-the-dependencies-search).

```
$ cdb -c "g;q" node.exe index.js 2>&1 | grep 'Unable to load DLL'
1ed8:0c80 @ 12843406 - LdrpProcessWork - ERROR: Unable to load DLL: "libglib-2.0-0.dll", Parent Module: "\\?\C:\msys64\home\someo\blog-code\nodejs-gtk-hello-world-on-windows\node_modules\node-gtk\lib\binding\node-v93-win32-x64\node_gtk.node", Status: 0xc0000135
1ed8:1bfc @ 12843406 - LdrpProcessWork - ERROR: Unable to load DLL: "libgmodule-2.0-0.dll", Parent Module: "\\?\C:\msys64\home\someo\blog-code\nodejs-gtk-hello-world-on-windows\node_modules\node-gtk\lib\binding\node-v93-win32-x64\node_gtk.node", Status: 0xc0000135
1ed8:22e8 @ 12843406 - LdrpProcessWork - ERROR: Unable to load DLL: "libgobject-2.0-0.dll", Parent Module: "\\?\C:\msys64\home\someo\blog-code\nodejs-gtk-hello-world-on-windows\node_modules\node-gtk\lib\binding\node-v93-win32-x64\node_gtk.node", Status: 0xc0000135
1ed8:1fb8 @ 12843406 - LdrpProcessWork - ERROR: Unable to load DLL: "libffi-7.dll", Parent Module: "\\?\C:\msys64\home\someo\blog-code\nodejs-gtk-hello-world-on-windows\node_modules\node-gtk\lib\binding\node-v93-win32-x64\node_gtk.node", Status: 0xc0000135
```

We see that **libglib-2.0-0.dll**, **libgmodule-2.0-0.dll**, **libgobject-2.0-0.dll** and **libffi-7.dll** are immediate DLL dependencies.

Copy the reported DLLs to the current directory.

```
$ cp /mingw64/bin/libglib-2.0-0.dll .
$ cp /mingw64/bin/libgmodule-2.0-0.dll .
$ cp /mingw64/bin/libgobject-2.0-0.dll .
$ cp /mingw64/bin/libffi-7.dll .
```

Run the application under the debugger again.

```
$ cdb -c "g;q" node.exe index.js 2>&1 | grep 'Unable to load DLL'
1574:2320 @ 13696515 - LdrpProcessWork - ERROR: Unable to load DLL: "libgirepository-1.0-1.dll", Parent Module: "\\?\C:\msys64\home\someo\blog-code\nodejs-gtk-hello-world-on-windows\node_modules\node-gtk\lib\binding\node-v93-win32-x64\node_gtk.node", Status: 0xc0000135
1574:1690 @ 13696515 - LdrpProcessWork - ERROR: Unable to load DLL: "libcairo-2.dll", Parent Module: "\\?\C:\msys64\home\someo\blog-code\nodejs-gtk-hello-world-on-windows\node_modules\node-gtk\lib\binding\node-v93-win32-x64\node_gtk.node", Status: 0xc0000135
1574:1304 @ 13696515 - LdrpProcessWork - ERROR: Unable to load DLL: "libintl-8.dll", Parent Module: "C:\msys64\home\someo\blog-code\nodejs-gtk-hello-world-on-windows\libglib-2.0-0.dll", Status: 0xc0000135
1574:1f88 @ 13696515 - LdrpProcessWork - ERROR: Unable to load DLL: "libintl-8.dll", Parent Module: "C:\msys64\home\someo\blog-code\nodejs-gtk-hello-world-on-windows\libgmodule-2.0-0.dll", Status: 0xc0000135
```

We see that there is a different set of DLLs reported. Repeating the debug-and-copy loop long enough we will reach the point where there is no more DLL loading errors reported.

```
$ cdb -c "g;q" node.exe index.js 2>&1 | grep 'Unable to load DLL'
```


## Find needed Typelibs

Run the application without the debugger.

```
$ node.exe index.js
C:\msys64\home\someo\blog-code\nodejs-gtk-hello-world-on-windows\node_modules\node-gtk\lib\bootstrap.js:10
const GI = internal.Bootstrap();
                    ^

Error: Typelib file for namespace 'GIRepository' (any version) not found
    at Object.<anonymous> (C:\msys64\home\someo\blog-code\nodejs-gtk-hello-world-on-windows\node_modules\node-gtk\lib\bootstrap.js:10:21)
    at Module._compile (node:internal/modules/cjs/loader:1105:14)
    at Object.Module._extensions..js (node:internal/modules/cjs/loader:1159:10)
    at Module.load (node:internal/modules/cjs/loader:981:32)
    at Function.Module._load (node:internal/modules/cjs/loader:822:12)
    at Module.require (node:internal/modules/cjs/loader:1005:19)
    at require (node:internal/modules/cjs/helpers:102:18)
    at Object.<anonymous> (C:\msys64\home\someo\blog-code\nodejs-gtk-hello-world-on-windows\node_modules\node-gtk\lib\index.js:9:19)
    at Module._compile (node:internal/modules/cjs/loader:1105:14)
    at Object.Module._extensions..js (node:internal/modules/cjs/loader:1159:10)
```

Now we see that the typelib **GIRepository** is not found.

Create Typelibs directory and copy the reported Typelib there.

```
$ mkdir -p lib/girepository-1.0/
$ cp /mingw64/lib/girepository-1.0/GIRepository-2.0.typelib lib/girepository-1.0/
```

Run the application again.

```
$ node.exe index.js
C:\msys64\home\someo\blog-code\nodejs-gtk-hello-world-on-windows\node_modules\node-gtk\lib\bootstrap.js:10
const GI = internal.Bootstrap();
                    ^

Error: Typelib file for namespace 'GObject', version '2.0' not found
    at Object.<anonymous> (C:\msys64\home\someo\blog-code\nodejs-gtk-hello-world-on-windows\node_modules\node-gtk\lib\bootstrap.js:10:21)
    at Module._compile (node:internal/modules/cjs/loader:1105:14)
    at Object.Module._extensions..js (node:internal/modules/cjs/loader:1159:10)
    at Module.load (node:internal/modules/cjs/loader:981:32)
    at Function.Module._load (node:internal/modules/cjs/loader:822:12)
    at Module.require (node:internal/modules/cjs/loader:1005:19)
    at require (node:internal/modules/cjs/helpers:102:18)
    at Object.<anonymous> (C:\msys64\home\someo\blog-code\nodejs-gtk-hello-world-on-windows\node_modules\node-gtk\lib\index.js:9:19)
    at Module._compile (node:internal/modules/cjs/loader:1105:14)
    at Object.Module._extensions..js (node:internal/modules/cjs/loader:1159:10)
```

We see that there is a different Typelib reported. Repeating the run-and-copy loop long enough we will reach the point where there is no more Typelib loading errors reported,
but again there are DLL loading errors.

```
$ node.exe index.js

** (process:2216): WARNING **: 10:46:46.213: Failed to load shared library 'libgdk_pixbuf-2.0-0.dll' referenced by the typelib: 'libgdk_pixbuf-2.0-0.dll':The specified module could not be found.
Couldn't load GdkPixbuf.Pixbuf: 'libgdk_pixbuf-2.0-0.dll': The specified module could not be found.
Couldn't load GdkPixbuf.PixbufAnimation: (NULL)
Couldn't load GdkPixbuf.PixbufAnimationIter: (NULL)
Couldn't load GdkPixbuf.PixbufLoader: (NULL)
Couldn't load GdkPixbuf.PixbufNonAnim: (NULL)
Couldn't load GdkPixbuf.PixbufSimpleAnim: (NULL)
Couldn't load GdkPixbuf.PixbufSimpleAnimIter: (NULL)

** (process:2216): WARNING **: 10:46:46.228: Failed to load shared library 'libharfbuzz-gobject-0.dll' referenced by the typelib: 'libharfbuzz-gobject-0.dll': The specified module could not be found.
** (process:2216): WARNING **: 10:46:46.244: Failed to load shared library 'libcairo-gobject-2.dll' referenced by the typelib: 'libcairo-gobject-2.dll': The specified module could not be found.
** (process:2216): WARNING **: 10:46:46.244: Failed to load shared library 'libpango-1.0-0.dll' referenced by the typelib: 'libpango-1.0-0.dll': The specified module could not be found.
```

Run the application under the debugger.

```
$ cdb -c "g;q" node.exe index.js 2>&1 | grep 'Unable to load DLL'
1e48:1e00 @ 15502968 - LdrpProcessWork - ERROR: Unable to load DLL: "libgdk_pixbuf-2.0-0.dll", Parent Module: "(null)", Status: 0xc0000135
1e48:1e00 @ 15503078 - LdrpProcessWork - ERROR: Unable to load DLL: "libharfbuzz-gobject-0.dll", Parent Module: "(null)", Status: 0xc0000135
1e48:1e00 @ 15503265 - LdrpProcessWork - ERROR: Unable to load DLL: "libcairo-gobject-2.dll", Parent Module: "(null)", Status: 0xc0000135
1e48:1e00 @ 15503328 - LdrpProcessWork - ERROR: Unable to load DLL: "libpango-1.0-0.dll", Parent Module: "(null)", Status: 0xc0000135
```

At this point, if you repeat the debug-and-copy loop discussed [above](#find-needed-dlls) you will find all the needed DLLs and run the application successfully.
And this is exactly the same process I followed to create the initial list of [dependencies](https://github.com/ten0s/velisp/blob/0.7.1/windows/copy-mingw64-deps.sh)
for [VeLisp](https://github.com/ten0s/velisp). But can we do better?

## Automate the dependencies search

Let's start from scratch.

Remove all the dependencies found.

```
$ rm -rf *.dll lib/
```

**copy-mingw64-deps.sh**:

```bash
#!/bin/bash

function is-sls-enabled() {
    IMAGE=$1
    reg query "HKLM\\SOFTWARE\\Microsoft\\Windows NT\\CurrentVersion\\Image File Execution Options\\$IMAGE" | grep -Eq "GlobalFlag\s+REG_SZ\s+0x00000002"
}

function is-cdb-available() {
    cdb &>/dev/null
    if [[ $? -eq 2 ]]; then
        return 0
    else
        return 1
    fi
}

function copy-dlls() {
    LOG=$1
    SRC=$2
    DST=$3
    cat $LOG | sed -En 's/.*Unable to load DLL: "([^"]*\.dll)".*/\1/p' | while read dll; do
        if [[ -f $SRC/$dll ]]; then
            cp -v $SRC/$dll $DST/
        fi
    done
}

function copy-typelibs() {
    LOG=$1
    SRC=$2
    DST=$3
    cat $LOG | sed -En "s/.*Typelib file for namespace '([^']+)'.*/\1/p" | while read typelib; do
        cp -v $SRC/$typelib* $DST/
    done
}

if [[ $# -lt 1 ]]; then
    echo "Usage: $(basename $0) PROG [ARG...]"
    exit 1
fi

PROG=$1
shift
ARGS="$@"

if ! is-sls-enabled $PROG; then
    echo "Enable Show Loader Snaps (sls) for $PROG and try again"
    exit 1
fi

if ! is-cdb-available; then
    echo "CDB is not found"
    echo "Run the command below and try again"
    echo 'export PATH="/c/Program Files (x86)/Windows Kits/10/Debuggers/x64/":$PATH'
    exit 1
fi

mkdir -p ./lib/girepository-1.0/

TEMP=$(mktemp)
while true; do
    cdb -c "g;q" $PROG $ARGS &>$TEMP
    if [[ $? -ne 0 ]]; then
        copy-dlls $TEMP /mingw64/bin/ ./
        copy-typelibs $TEMP /mingw64/lib/girepository-1.0/ ./lib/girepository-1.0/
    else
        break
    fi
done
rm $TEMP

exit 0
```

The script first checks that Show Loader Snaps (sls) are enabled for a given program (node.exe in our case) and CDB is available and
then it is basically a loop on every iteration it runs the application under the debugger and looks for DLL and Typelib errors,
in such case it copies either DLL or Typelib locally.


Run the script:

```
$ ./copy-mingw64-deps.sh node.exe index.js
'/mingw64/bin//libgmodule-2.0-0.dll' -> './libgmodule-2.0-0.dll'
'/mingw64/bin//libgobject-2.0-0.dll' -> './libgobject-2.0-0.dll'
'/mingw64/bin//libglib-2.0-0.dll' -> './libglib-2.0-0.dll'
'/mingw64/bin//libffi-7.dll' -> './libffi-7.dll'
'/mingw64/bin//libcairo-2.dll' -> './libcairo-2.dll'
'/mingw64/bin//libglib-2.0-0.dll' -> './libglib-2.0-0.dll'
'/mingw64/bin//libgirepository-1.0-1.dll' -> './libgirepository-1.0-1.dll'
'/mingw64/bin//libintl-8.dll' -> './libintl-8.dll'
'/mingw64/bin//libgio-2.0-0.dll' -> './libgio-2.0-0.dll'
'/mingw64/bin//libgcc_s_seh-1.dll' -> './libgcc_s_seh-1.dll'
'/mingw64/bin//libfontconfig-1.dll' -> './libfontconfig-1.dll'
'/mingw64/bin//libstdc++-6.dll' -> './libstdc++-6.dll'
'/mingw64/bin//libiconv-2.dll' -> './libiconv-2.dll'
'/mingw64/bin//libfreetype-6.dll' -> './libfreetype-6.dll'
'/mingw64/bin//libpixman-1-0.dll' -> './libpixman-1-0.dll'
'/mingw64/bin//zlib1.dll' -> './zlib1.dll'
'/mingw64/bin//libpng16-16.dll' -> './libpng16-16.dll'
'/mingw64/bin//libpcre-1.dll' -> './libpcre-1.dll'
'/mingw64/bin//libwinpthread-1.dll' -> './libwinpthread-1.dll'
'/mingw64/bin//libbrotlidec.dll' -> './libbrotlidec.dll'
'/mingw64/bin//libbz2-1.dll' -> './libbz2-1.dll'
'/mingw64/bin//libexpat-1.dll' -> './libexpat-1.dll'
'/mingw64/bin//libbrotlidec.dll' -> './libbrotlidec.dll'
'/mingw64/bin//libharfbuzz-0.dll' -> './libharfbuzz-0.dll'
'/mingw64/bin//libbrotlicommon.dll' -> './libbrotlicommon.dll'
'/mingw64/bin//libgraphite2.dll' -> './libgraphite2.dll'
'/mingw64/lib/girepository-1.0//GIRepository-2.0.typelib' -> './lib/girepository-1.0/GIRepository-2.0.typelib'
'/mingw64/lib/girepository-1.0//GObject-2.0.typelib' -> './lib/girepository-1.0/GObject-2.0.typelib'
'/mingw64/lib/girepository-1.0//GLib-2.0.typelib' -> './lib/girepository-1.0/GLib-2.0.typelib'
'/mingw64/lib/girepository-1.0//Gtk-3.0.typelib' -> './lib/girepository-1.0/Gtk-3.0.typelib'
'/mingw64/lib/girepository-1.0//Gdk-3.0.typelib' -> './lib/girepository-1.0/Gdk-3.0.typelib'
'/mingw64/lib/girepository-1.0//GdkPixbuf-2.0.typelib' -> './lib/girepository-1.0/GdkPixbuf-2.0.typelib'
'/mingw64/lib/girepository-1.0//GdkPixdata-2.0.typelib' -> './lib/girepository-1.0/GdkPixdata-2.0.typelib'
'/mingw64/lib/girepository-1.0//GdkWin32-3.0.typelib' -> './lib/girepository-1.0/GdkWin32-3.0.typelib'
'/mingw64/lib/girepository-1.0//cairo-1.0.typelib' -> './lib/girepository-1.0/cairo-1.0.typelib'
'/mingw64/lib/girepository-1.0//Pango-1.0.typelib' -> './lib/girepository-1.0/Pango-1.0.typelib'
'/mingw64/lib/girepository-1.0//PangoCairo-1.0.typelib' -> './lib/girepository-1.0/PangoCairo-1.0.typelib'
'/mingw64/lib/girepository-1.0//PangoFc-1.0.typelib' -> './lib/girepository-1.0/PangoFc-1.0.typelib'
'/mingw64/lib/girepository-1.0//PangoFT2-1.0.typelib' -> './lib/girepository-1.0/PangoFT2-1.0.typelib'
'/mingw64/lib/girepository-1.0//PangoOT-1.0.typelib' -> './lib/girepository-1.0/PangoOT-1.0.typelib'
'/mingw64/lib/girepository-1.0//HarfBuzz-0.0.typelib' -> './lib/girepository-1.0/HarfBuzz-0.0.typelib'
'/mingw64/lib/girepository-1.0//freetype2-2.0.typelib' -> './lib/girepository-1.0/freetype2-2.0.typelib'
'/mingw64/lib/girepository-1.0//Gio-2.0.typelib' -> './lib/girepository-1.0/Gio-2.0.typelib'
'/mingw64/lib/girepository-1.0//GModule-2.0.typelib' -> './lib/girepository-1.0/GModule-2.0.typelib'
'/mingw64/lib/girepository-1.0//Atk-1.0.typelib' -> './lib/girepository-1.0/Atk-1.0.typelib'
'/mingw64/bin//libgdk_pixbuf-2.0-0.dll' -> './libgdk_pixbuf-2.0-0.dll'
'/mingw64/bin//libharfbuzz-gobject-0.dll' -> './libharfbuzz-gobject-0.dll'
'/mingw64/bin//libcairo-gobject-2.dll' -> './libcairo-gobject-2.dll'
'/mingw64/bin//libpango-1.0-0.dll' -> './libpango-1.0-0.dll'
'/mingw64/bin//libfribidi-0.dll' -> './libfribidi-0.dll'
'/mingw64/bin//libthai-0.dll' -> './libthai-0.dll'
'/mingw64/bin//libdatrie-1.dll' -> './libdatrie-1.dll'
'/mingw64/bin//libgdk-3-0.dll' -> './libgdk-3-0.dll'
'/mingw64/bin//libatk-1.0-0.dll' -> './libatk-1.0-0.dll'
'/mingw64/bin//libgtk-3-0.dll' -> './libgtk-3-0.dll'
'/mingw64/bin//libpangowin32-1.0-0.dll' -> './libpangowin32-1.0-0.dll'
'/mingw64/bin//libepoxy-0.dll' -> './libepoxy-0.dll'
'/mingw64/bin//libpangocairo-1.0-0.dll' -> './libpangocairo-1.0-0.dll'
'/mingw64/bin//libpangocairo-1.0-0.dll' -> './libpangocairo-1.0-0.dll'
'/mingw64/bin//libpangowin32-1.0-0.dll' -> './libpangowin32-1.0-0.dll'
'/mingw64/bin//libepoxy-0.dll' -> './libepoxy-0.dll'
'/mingw64/bin//libpangoft2-1.0-0.dll' -> './libpangoft2-1.0-0.dll'
'/mingw64/bin//libpangoft2-1.0-0.dll' -> './libpangoft2-1.0-0.dll'
'/mingw64/bin//libgdk-3-0.dll' -> './libgdk-3-0.dll'
```

![Hello Gtk in MSYS Image](/assets/images/find-dlls-and-typelibs-dependencies-for-nodejs-gtk-application-on-windows/hello-gtk-msys.png)

It works! At this point you've got all the needed dependencies for the application.

To prove it, let's run the application inside the Command Prompt.

![Hello Gtk in Command Prompt Image](/assets/images/find-dlls-and-typelibs-dependencies-for-nodejs-gtk-application-on-windows/hello-gtk-cmd.png)

And it indeed works!

## Tear down

Don't forget to disable Show Loader Snaps (sls) for **node.exe**

## What's next?

In the [next post]({{ site.baseurl }}{% post_url 2022-07-27-package-nodejs-gtk-application-on-windows %})
we will package the application for distribution and learn how to start it without showing the terminal prompt.
