---
layout: post
language: ru
title: Как сделать самораспаковывающийся exe-архив для VeLisp диалога
description: Как сделать самораспаковывающийся exe-архив для VeLisp диалога
created_date: 2024-10-05
---

[English version]({{ site.baseurl }}{% post_url 2024-10-05-velisp-self-extracting-archive-en %})

Уже несколько человек интересовались как сделать самораспаковывающийся exe-архив для
[VeLisp](https://github.com/ten0s/velisp) диалога, чтобы его можно было запускать
как самостоятельную программу без установки [VeLisp](https://github.com/ten0s/velisp)
на целевой компьютер и необходимости копировать .lsp и .dcl файлы.

Как вариант решения я предлагаю использовать, встроенную в **Windows**, программу **IExpress**,
которая создана как раз для этой цели.

В качестве примера создадим exe-архив для [Калькулятора](https://github.com/ten0s/velisp/blob/master/README-ru.md#%D0%B7%D0%B0%D0%BF%D1%83%D1%81%D0%BA%D0%B0%D0%B5%D0%BC-%D0%BA%D0%BE%D0%B4-%D0%B8%D0%B7-%D1%84%D0%B0%D0%B9%D0%BB%D0%B0)    .

Копируем исходные файлы **Калькулятора** **calc.lsp**, **calc.dcl** и **util.lsp** в рабочую директорию.

Сюда же скачиваем последнюю версию [VeLisp](https://github.com/ten0s/velisp) [https://github.com/ten0s/velisp/releases/download/0.7.10/velisp-0.7.10-win-x64.zip](https://github.com/ten0s/velisp/releases/download/0.7.10/velisp-0.7.10-win-x64.zip).

Тут же нужно создать еще два файла: **unzip.vbs** и **calc.sed** со следующим содержимым.

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

**unzip.vbs** это VBS-скрипт, который позволяет распаковать ZIP-архив без использования сторонних утилит.

**calc.sed** это конфигурационный файл для **EIxpress** с описанием того, как создавать самораспаковывающийся exe-архив и какие команды выполнять при его запуске.

При старте exe-архива он сначала весь целиком распаковывается во временную директорию.

После чего выполняется команда, записанная в **AppLaunched**, которая распаковывает архив **velisp-0.7.10-win-x64.zip** в ту же временную директорию.

```
AppLaunched="wscript.exe unzip.vbs velisp-%VeLispVersion%-win-x64.zip"
```

И наконец, выполняется команда, записанная в **PostInstallCmd**, которая и запускает **calc.lsp**.

```
PostInstallCmd="wscript.exe velisp-%VeLispVersion%-win-x64\noprompt.vbs velisp-%VeLispVersion%-win-x64\velisp.exe calc.lsp"
```

Название результирующего exe-архива находится в **TargetName**.

```
TargetName=calc.exe
```

Исходный код диалога расположен в секции **SourceFilesApp**.

```
[SourceFilesApp]
calc.lsp=
calc.dcl=
util.lsp=
```

Рабочая директория на данный момент должна выглядеть вот так.

![Source Files Image](/assets/images/velisp-self-extracting-archive/source-files.png)

Заходим в терминал, переходим в рабочую директорию и выполняем команду.

```
> iexpress /N calc.sed
```

В рабочей директории должен появиться файл **calc.exe**.

Запускаем **calc.exe** и через некоторое время появляется окно **Калькулятора*.

![Calc App Started Image](/assets/images/velisp-self-extracting-archive/calc-app-started.png)

---

[Исходный код файлов](https://github.com/ten0s/blog-code/tree/main/velisp-self-extracting-archive)
