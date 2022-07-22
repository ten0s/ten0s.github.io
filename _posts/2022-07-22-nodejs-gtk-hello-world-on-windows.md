---
layout: post
title: Node.js GTK Hello World on Windows
date: 2022-07-22
---

Recently I helped porting [Node-Gtk](https://github.com/romgrk/node-gtk) to Windows and since
[VeLisp](https://github.com/ten0s/velisp) is probably the first project that use it on Windows
I'm in a good position to share what I have learned along the way.

The below you will find the steps how to make [GTK](https://www.gtk.org/) Hello World application
using [Node.js](https://nodejs.org/). In the [next post](TODO) we will know how to determine dependencies
(DLLs and TypeLibs) needed for distribution and how to start the application without showing the terminal prompt.

## Table of contents

- [Prerequisites](#prerequisites)
  - [Install Node.js](#install-nodejs)
  - [Install MSYS2](#install-msys2)
  - [Install Windows Terminal](#install-windows-terminal)
  - [Install GTK development dependencies](#install-gtk-development-dependencies)
- [GTK Hello World inside MinGW64](#gtk-hello-world-inside-mingw64)
- [What's next?](#whats-next)


## Prerequisites

### Install Node.js

The below steps assume that Node.js 16.x is installed to `C:/nodejs16`.

![](/assets/images/nodejs-gtk-hello-world-on-windows/nodejs16-path.png)

Since there's no pre-built [Node-Gtk](https://github.com/romgrk/node-gtk) native module available yet you have to build it yourself.

For this we need to install the Tools for Native Modules.

![](/assets/images/nodejs-gtk-hello-world-on-windows/nodejs16-tools.png)

If you happen to have Node.js already installed, but without the Tools for Native Modules, fear not, simply go to the
installation directory and run `install_tools.bat`.


### Install MSYS2

The below steps assume that MSYS2 is installed to `C:/msys64`.


![](/assets/images/nodejs-gtk-hello-world-on-windows/msys2-path.png)

Otherwise, follow the instructions from [https://www.msys2.org/#installation](https://www.msys2.org/#installation)


### Install Windows Terminal

At this point I strongly suggest you to install the [Windows Terminal from the Microsoft Store](https://aka.ms/terminal).
It is not strictly necessary, but makes the life a bit easier on Windows.

After installing the terminal, follow the instructions from [https://www.msys2.org/docs/terminals/#windows-terminal](https://www.msys2.org/docs/terminals/#windows-terminal), but paste the settings below that add Windows PATH to the MinGW64 and MSYS shells:

```
{
    "commandline": "C:/msys64/msys2_shell.cmd -use-full-path -defterm -here -no-start -mingw64",
    "guid": "{17da3cac-b318-431e-8a3e-7fcdefe6d114}",
    "icon": "C:/msys64/mingw64.ico",
    "name": "MINGW64 / MSYS2",
    "startingDirectory": "C:/msys64/home/%USERNAME%"
},
{
    "commandline": "C:/msys64/msys2_shell.cmd -use-full-path -defterm -here -no-start -msys",
    "guid": "{71160544-14d8-4194-af25-d05feeac7233}",
    "icon": "C:/msys64/msys2.ico",
    "name": "MSYS / MSYS2",
    "startingDirectory": "C:/msys64/home/%USERNAME%"
}
```

### Install GTK development dependencies

Start the MinGW64 shell and run:


```
$ pacman -S --needed --noconfirm git mingw-w64-$(uname -m)-{gtk3,gobject-introspection,pkg-config,cairo}
```

### Check everything is available

Start the MinGW64 shell and run:

```
$ node --version
v16.16.0
```

```
$ python --version
Python 3.10.5
```

```
$ gtk3-demo
```

![](/assets/images/nodejs-gtk-hello-world-on-windows/gtk3-demo.png)

Ok, you're all set!


## GTK Hello World inside MinGW64

You can find the code [here](https://github.com/ten0s/blog-code/tree/main/nodejs-gtk-hello-world-on-windows).

Start the MinGW64 shell and run:


```
$ git clone https://github.com/ten0s/blog-code
$ cd blog-code/nodejs-gtk-hello-world-on-windows
```

Build [Node-Gtk](https://github.com/romgrk/node-gtk)

```
$ npm install --ignore-scripts
$ node_modules/node-gtk/windows/mingw_include_extra.sh
$ npm rebuild node-gtk
```

Run the application


```
$ cat index.js
```

```javascript
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

```
$ node index.js
```

![](/assets/images/nodejs-gtk-hello-world-on-windows/hello-gtk-mingw64.png)

It works! At this point you've got everything to start developing
[GTK](https://www.gtk.org/) applications using [Node.js](https://nodejs.org/) on Windows.



## What's next?

In the [next post](TODO) we will know how to determine dependencies (DLLs and TypeLibs) needed for distribution and
how to start the application without showing the terminal prompt.
