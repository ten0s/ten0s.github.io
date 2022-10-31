---
layout: post
title: Package Node.js GTK Application on Windows
date: 2022-07-27
---

In the [previous post]({{ site.baseurl }}{% post_url 2022-07-25-find-dlls-and-typelibs-dependencies-for-nodejs-gtk-application-on-windows %})
we have found the dependencies (DLLs and Typelibs) needed for distribution.
In this post we will package the application for distribution and learn how start it without showing the terminal prompt.

## Table of contents

- [House cleaning](#house-cleaning)
- [Packaging](#packaging)
- [Zipping](#zipping)
- [Create the shortcut](#create-the-shortcut)
- [What's next?](#whats-next)


## House cleaning

Alright, we have found all the dependencies, but what the mess we've got in the directory?

Open the MSYS shell and run.

```
$ ls -l
total 23952
-rwxr-xr-x 1 someo someo    1520 Jul 27 05:41 copy-mingw64-deps.sh
-rw-r--r-- 1 someo someo     419 Jul 22 09:18 index.js
drwxr-xr-x 1 someo someo       0 Jul 27 05:41 lib
-rwxr-xr-x 1 someo someo  158785 Jul 27 05:42 libatk-1.0-0.dll
-rwxr-xr-x 1 someo someo  143397 Jul 27 05:42 libbrotlicommon.dll
-rwxr-xr-x 1 someo someo   51852 Jul 27 05:42 libbrotlidec.dll
-rwxr-xr-x 1 someo someo   99146 Jul 27 05:42 libbz2-1.dll
-rwxr-xr-x 1 someo someo 1154776 Jul 27 05:41 libcairo-2.dll
-rwxr-xr-x 1 someo someo   34691 Jul 27 05:42 libcairo-gobject-2.dll
-rwxr-xr-x 1 someo someo   32570 Jul 27 05:42 libdatrie-1.dll
-rwxr-xr-x 1 someo someo 1737259 Jul 27 05:42 libepoxy-0.dll
-rwxr-xr-x 1 someo someo  203594 Jul 27 05:42 libexpat-1.dll
-rwxr-xr-x 1 someo someo   30354 Jul 27 05:41 libffi-7.dll
-rwxr-xr-x 1 someo someo  316608 Jul 27 05:41 libfontconfig-1.dll
-rwxr-xr-x 1 someo someo  749498 Jul 27 05:42 libfreetype-6.dll
-rwxr-xr-x 1 someo someo  146395 Jul 27 05:42 libfribidi-0.dll
-rwxr-xr-x 1 someo someo  107699 Jul 27 05:41 libgcc_s_seh-1.dll
-rwxr-xr-x 1 someo someo  167499 Jul 27 05:42 libgdk_pixbuf-2.0-0.dll
-rwxr-xr-x 1 someo someo 1296073 Jul 27 05:42 libgdk-3-0.dll
-rwxr-xr-x 1 someo someo 1716832 Jul 27 05:42 libgio-2.0-0.dll
-rwxr-xr-x 1 someo someo  232980 Jul 27 05:41 libgirepository-1.0-1.dll
-rwxr-xr-x 1 someo someo 1346542 Jul 27 05:41 libglib-2.0-0.dll
-rwxr-xr-x 1 someo someo   24997 Jul 27 05:41 libgmodule-2.0-0.dll
-rwxr-xr-x 1 someo someo  334386 Jul 27 05:41 libgobject-2.0-0.dll
-rwxr-xr-x 1 someo someo  154163 Jul 27 05:42 libgraphite2.dll
-rwxr-xr-x 1 someo someo 7501875 Jul 27 05:42 libgtk-3-0.dll
-rwxr-xr-x 1 someo someo 1116228 Jul 27 05:42 libharfbuzz-0.dll
-rwxr-xr-x 1 someo someo   88053 Jul 27 05:42 libharfbuzz-gobject-0.dll
-rwxr-xr-x 1 someo someo 1114369 Jul 27 05:42 libiconv-2.dll
-rwxr-xr-x 1 someo someo  136724 Jul 27 05:41 libintl-8.dll
-rwxr-xr-x 1 someo someo  375913 Jul 27 05:42 libpango-1.0-0.dll
-rwxr-xr-x 1 someo someo   73627 Jul 27 05:42 libpangocairo-1.0-0.dll
-rwxr-xr-x 1 someo someo  100512 Jul 27 05:43 libpangoft2-1.0-0.dll
-rwxr-xr-x 1 someo someo   93444 Jul 27 05:42 libpangowin32-1.0-0.dll
-rwxr-xr-x 1 someo someo  281695 Jul 27 05:42 libpcre-1.dll
-rwxr-xr-x 1 someo someo  684803 Jul 27 05:41 libpixman-1-0.dll
-rwxr-xr-x 1 someo someo  243078 Jul 27 05:41 libpng16-16.dll
-rwxr-xr-x 1 someo someo 2024733 Jul 27 05:41 libstdc++-6.dll
-rwxr-xr-x 1 someo someo   67411 Jul 27 05:42 libthai-0.dll
-rwxr-xr-x 1 someo someo   58109 Jul 27 05:42 libwinpthread-1.dll
drwxr-xr-x 1 someo someo       0 Jul 22 09:19 node_modules
-rw-r--r-- 1 someo someo     626 Jul 27 05:31 noprompt.vbs
-rw-r--r-- 1 someo someo     212 Jul 22 09:18 package.json
-rw-r--r-- 1 someo someo   87428 Jul 22 09:19 package-lock.json
-rw-r--r-- 1 someo someo     563 Jul 27 04:51 README.md
-rwxr-xr-x 1 someo someo  119026 Jul 27 05:42 zlib1.dll
```

We can do better by exploiting two facts:

1. Windows DLLs search uses the directories from the PATH environment variable.
2. We control when the [Node-Gtk](https://github.com/romgrk/node-gtk) native module gets loaded.

Move the dependencies in the `mingw64` directory.

```
$ mkdir -p mingw64/bin
$ mv *.dll mingw64/bin/
$ mv lib mingw64/
```

```
$ ls -l
total 128
-rwxr-xr-x 1 someo someo  1520 Jul 27 05:41 copy-mingw64-deps.sh
-rw-r--r-- 1 someo someo   419 Jul 22 09:18 index.js
drwxr-xr-x 1 someo someo     0 Jul 27 06:03 mingw64
drwxr-xr-x 1 someo someo     0 Jul 22 09:19 node_modules
-rw-r--r-- 1 someo someo   626 Jul 27 05:31 noprompt.vbs
-rw-r--r-- 1 someo someo   212 Jul 22 09:18 package.json
-rw-r--r-- 1 someo someo 87428 Jul 22 09:19 package-lock.json
-rw-r--r-- 1 someo someo   563 Jul 27 04:51 README.md
```

Now the application should stop working.

```
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

Right! Prepend the `mingw64\bin` directory to the PATH environment variable:


**index.js**:

```javascript
const path = require('path')
const rootdir = __dirname.includes('snapshot') ? path.dirname(process.argv[0]) : __dirname
process.env['PATH'] = `${rootdir}\\mingw64\\bin;` + process.env['PATH']

const gi = require('node-gtk')
const Gtk = gi.require('Gtk', '3.0')

gi.startLoop()
Gtk.init()

const win = new Gtk.Window({
  title: 'Hello',
  window_position: Gtk.WindowPosition.CENTER
})

win.on('show', Gtk.main)
win.on('destroy', Gtk.mainQuit)
win.setDefaultSize(200, 80)
win.add(new Gtk.Label({label: 'Hello from Gtk!'}))
win.showAll()
```

Run the application again.

![Hello Gtk in MSYS Image](/assets/images/package-nodejs-gtk-application-on-windows/hello-gtk-msys.png)

And it indeed works! Move to packaging...


## Packaging

At this point we have everything in place and we can simply zip the whole directory and send the archive to our users.
But a couple of problems arise: your users might not have Node.js installed or they might have a different Node.js
version installed, i.e. not the one you have built [Node-Gtk](https://github.com/romgrk/node-gtk) with.
In either case they won't be able to run the application.
The solution I propose is to use [pkg](https://www.npmjs.com/package/pkg). It includes appropriate Node.js,
the node_modules/ directory and index.js into a self-contained package fixing the problems above.

**pkg.json**:

```
{
    "pkg": {
        "assets": [
            "package.json"
        ],
        "options": [],
        "scripts": []
    }
}
```

```
$ NODE=$(node --version | sed -E 's/v([0-9]+)\..*/node\1/')
$ npx pkg -c pkg.json -t ${NODE}-win-x64 -o hello-gtk.exe index.js
> pkg@5.8.0
> Fetching base Node.js binaries to PKG_CACHE_PATH
  fetched-v16.15.0-win-x64            [====================] 100%
```

Run the package

```
./hello-gtk.exe
```

![Hello Gtk Pkg in MSYS Image](/assets/images/package-nodejs-gtk-application-on-windows/hello-gtk-pkg-msys.png)


## Zipping

Let's make a distribution archive. First install `zip` if you don't have it yet.

```
$ pacman -S zip
```

Create the zip archive.

```
$ zip -r hello-gtk.zip hello-gtk.exe noprompt.vbs mingw64/
```


## Create the shortcut

Now we have the distribution archive `hello-gtk.zip`. It's time for a final test.

Copy it to some place like `C:\Users\Public` and extract it there.

![Extract Hello Gtk Image](/assets/images/package-nodejs-gtk-application-on-windows/extract-hello-gtk.png)

If you run `hello-gtk.exe` you will also see the Command Prompt.

![Hello Gtk Prompt Image](/assets/images/package-nodejs-gtk-application-on-windows/hello-gtk-prompt.png)

It's ugly, let's fix it.

**noprompt.vbs**

```visualbasic
REM
REM Run program without showing the Command Prompt
REM Useful for creating Shortcuts
REM

If (WScript.Arguments.Count = 0) Then
    WScript.Echo "Usage: " & WScript.ScriptName & " Prog [Arg...]"
    WScript.Quit 1
End If

REM Windows 10 does NOT allow starting unknown programs in
REM the hidden state. We trick it here by first running
REM something it knows really well: cmd.exe /C.

ReDim Cmd(2)
Cmd(0) = "cmd.exe"
Cmd(1) = "/C"

ReDim Args(WScript.Arguments.Count)
For i = 0 To WScript.Arguments.Count - 1
   Args(i) = """" & WScript.Arguments(i) & """"
Next

Command = Join(Cmd) & """" & Join(Args) & """"
REM WScript.Echo Command

Set WshShell = WScript.CreateObject("WScript.Shell")
HideWindow = 0
WaitOnReturn = True
WshShell.Run Command, HideWindow, WaitOnReturn
```

Create shortcut for `hello-gtk.exe` and rename it to `hello-gtk.lnk`.

Change the properties of `hello-gtk.lnk` to:

- Target: C:\Users\Public\hello-gtk\noprompt.vbs hello-gtk.exe
- Start in: C:\Users\Public\hello-gtk

![Hello Gtk Link Image](/assets/images/package-nodejs-gtk-application-on-windows/hello-gtk-lnk.png)

Run the `hello-gtk.lnk` shortcut.

![Hello Gtk No Prompt Image](/assets/images/package-nodejs-gtk-application-on-windows/hello-gtk-noprompt.png)

It's done!

## What's next?

To be a good OSS (Open-source software) citizen you should create NOTICE file(s) for
Node.js and MinGW64 dependencies. Also, for a serious application you might want to
create an installer package. This is your homework:)
