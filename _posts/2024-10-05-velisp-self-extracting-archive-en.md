---
layout: post
language: en
title: How to make self-extracting archive for VeLisp dialog
description: How to make self-extracting archive for VeLisp dialog
created_date: 2024-10-05
updated_date: 2025-07-14
---

[Русская версия]({{ site.baseurl }}{% post_url 2024-10-05-velisp-self-extracting-archive-ru %})

Some people asked how to make self-extracting archive for a
[VeLisp](https://github.com/ten0s/velisp) dialog to make it possible to run it
as a standalone application without installing [VeLisp](https://github.com/ten0s/velisp)
to target machine and don't copy .lsp и .dcl files.

As a prove of concept, I suggest to use **Windows**'s native **IExpress** application,
which serves this exact purpose.

For example, let's create self-extracting archive for [Calculator](https://github.com/ten0s/velisp/tree/master?tab=readme-ov-file#run-code-from-file).

Copy **Calculator**'s source files **calc.lsp**, **calc.dcl** and **util.lsp** to some working directory.

Then, download to the working directory some version of [VeLisp](https://github.com/ten0s/velisp) [https://github.com/ten0s/velisp/releases/download/0.7.10/velisp-0.7.10-win-x64.zip](https://github.com/ten0s/velisp/releases/download/0.7.10/velisp-0.7.10-win-x64.zip).

Next, create two more files: **unzip.vbs** и **calc.sed** with the following content.

**unzip.vbs**

```visualbasic
REM
REM Unzip file
REM

If WScript.Arguments.Count = 0 Then
    WScript.Echo "Usage: " & WScript.ScriptName & " ZipPath [ExtractPath]"
    WScript.Quit 1
End If

Set Fs = CreateObject("Scripting.FileSystemObject")

ZipPath = WScript.Arguments(0)
If NOT Fs.FileExists(ZipPath) Then
    WScript.Echo "File Not Found: " & ZipPath
    WScript.Quit 1
Else
    ZipPath = Fs.GetAbsolutePathName(ZipPath)
End If

If WScript.Arguments.Count = 1 Then
    Set File = Fs.GetFile(ZipPath)
    ExtractPath = Fs.GetParentFolderName(File.Path)
Else
    ExtractPath = WScript.Arguments(1)
End If

If NOT Fs.FolderExists(ExtractPath) Then
    Fs.CreateFolder(ExtractPath)
    ExtractPath = Fs.GetAbsolutePathName(ExtractPath)
End If

REM WScript.Echo ZipPath & " -> " & ExtractPath

Set Shell = CreateObject("Shell.Application")
Set Files = Shell.NameSpace(ZipPath).items
NoProgress = 4
Shell.NameSpace(ExtractPath).CopyHere Files, NoProgress
```

**calc.sed**

```
[Version]
Class=IEXPRESS
SEDVersion=3
[Options]
PackagePurpose=InstallApp
ShowInstallProgramWindow=0
HideExtractAnimation=1
UseLongFileName=1
InsideCompressed=0
CAB_FixedSize=0
CAB_ResvCodeSigning=0
RebootMode=N
InstallPrompt=
DisplayLicense=
FinishMessage=
TargetName=calc.exe
FriendlyName=calc
AppLaunched="wscript.exe unzip.vbs velisp-%VeLispVersion%-win-x64.zip"
PostInstallCmd="wscript.exe velisp-%VeLispVersion%-win-x64\noprompt.vbs velisp-%VeLispVersion%-win-x64\velisp.exe calc.lsp"
AdminQuietInstCmd=
UserQuietInstCmd=
SourceFiles=SourceFiles
[Strings]
VeLispVersion=0.7.10
[SourceFiles]
SourceFilesUtil=.\
SourceFilesVelisp=.\
SourceFilesApp=.\
[SourceFilesUtil]
unzip.vbs=
[SourceFilesVelisp]
velisp-%VeLispVersion%-win-x64.zip=
[SourceFilesApp]
calc.lsp=
calc.dcl=
util.lsp=
```

**unzip.vbs** it's a VBS-script that allows to unpack a ZIP-archive without using any third-party utility.

**calc.sed** it's a **IExpress**'s configuration file that describes how to create the self-extracting archive and what commands to run on its startup.

Upon self-extracting archive startup its content is unpacked to a temporary directory.

Then, a command from **AppLaunched** is executed that unpacks **velisp-0.7.10-win-x64.zip** to the temporary directory.

```
AppLaunched="wscript.exe unzip.vbs velisp-%VeLispVersion%-win-x64.zip"
```

And finally, a command from **PostInstallCmd** is executed that starts **calc.lsp**.

```
PostInstallCmd="wscript.exe velisp-%VeLispVersion%-win-x64\noprompt.vbs velisp-%VeLispVersion%-win-x64\velisp.exe calc.lsp"
```

The name of the target self-extracting archive is located in **TargetName**.

```
TargetName=calc.exe
```

Source files are located under the section **SourceFilesApp**.

```
[SourceFilesApp]
calc.lsp=
calc.dcl=
util.lsp=
```

The working directory now should look like this.

![Source Files Image](/assets/images/velisp-self-extracting-archive/source-files.png)

Run the terminal, move to the working directory and run the command below.

```
> iexpress /N calc.sed
```

In the working directory **calc.exe** should be created.

Run **calc.exe** and after a while the **Calculator** window should appear.

![Calc App Started Image](/assets/images/velisp-self-extracting-archive/calc-app-started.png)

---

[Files source code](https://github.com/ten0s/blog-code/tree/main/velisp-self-extracting-archive)
