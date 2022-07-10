---
layout: post
title: Git difftool and binary files
date: 2020-04-19
---
Recently I was asked how to see changes in Microsoft Word/Excel documents stored in git.

I researched the topic and below are my experiments.
I use **Linux** and [LibreOffice](https://www.libreoffice.org/discover/libreoffice/), but the steps
are generic enough to be ported to other platforms.

### Setup

Install [meld](https://meldmerge.org/), [pandoc](https://pandoc.org) and [unoconv](https://github.com/unoconv/unoconv):

```
$ sudo apt install meld pandoc unoconv
```

Create an empty git repo:

```
$ mkdir git-diff
$ cd git-diff
$ git init
```

In this directory create new Microsoft Word and Excel files to work with.

Create a new word.docx Microsoft Word file with the text "hello" and commit it:

```
$ git add word.docx
$ git commit -m "Add word.docx v1"
```

Change the text to "Hello world!", save the file and commit it:

```
$ git add word.docx
$ git commit -m "Change word.docx v2"
```

Create a new excel.xlsx Microsoft Excel file the text "1", "+", "1", "=", "2" in separate cells and commit it:

```
$ git add excel.xlsx
$ git commit -m "Add excel.xlsx v1"
```

Change the text in cells to "1", "+", "2", "=", "3", save the file and commit it:

```
$ git add excel.xlsx
$ git commit -m "Change excel.xlsx v2"
```

Now git log looks like this:

```
$ git log --oneline
31cb76d (HEAD -> master) Change excel.xlsx v2
07e90fe Add excel.xlsx v1
f02d3c0 Change word.docx v2
de8f24e Add word.docx v1
```

Let's see what changed:

```
$ git diff de8f24e 31cb76d
diff --git a/excel.xlsx b/excel.xlsx
new file mode 100644
index 0000000..4c8c59b
Binary files /dev/null and b/excel.xlsx differ
diff --git a/word.docx b/word.docx
index 5e522aa..d263873 100644
Binary files a/word.docx and b/word.docx differ
```

Not pretty, but anticipated.

### Git diff

Let's fix that using information from the [Git book](https://git-scm.com/book/en/v2/Customizing-Git-Git-Attributes#Binary-Files).

Add the below to .gitattributes file:

```
*.docx diff=docx
*.xlsx diff=xlsx
```

The above lines say that *.docx files should go through the **docx** filter and
*.xlsx files should go though the **xlsx** filter.

A filter is a program that reads a binary file and prints its textual presentation.

Let's setup the filters now.

I use [pandoc](https://pandoc.org) for Word and [unoconv](https://github.com/unoconv/unoconv) for Excel to keep things more interesting.

Add the below to .git/config:

```
[diff "docx"]
	textconv = pandoc --to=plain

[diff "xlsx"]
	textconv = unoconv --format=csv --stdout
```

Let's see the changes now:

```
$ git diff de8f24e f02d3c0
diff --git a/word.docx b/word.docx
index 5e522aa..d263873 100644
--- a/word.docx
+++ b/word.docx
@@ -1 +1 @@
-hello
+Hello world!
```

```
$ git diff 07e90fe 31cb76d
diff --git a/excel.xlsx b/excel.xlsx
index 547f86a..4c8c59b 100644
--- a/excel.xlsx
+++ b/excel.xlsx
@@ -1 +1 @@
-1,+,1,=,2
+1,+,2,=,3
```

It works!

### Git difftool

Textual [git diff](https://git-scm.com/docs/git-diff) is great, but what about [git difftool](https://git-scm.com/docs/git-difftool)?
I have [meld](https://meldmerge.org/) as my default difftool and running the command below shows binary gibberrish.

```
$ git difftool de8f24e f02d3c0
```

![](/assets/images/git-difftool-and-binary-files/difftool-auto-gibberish.png)

From the [Git documentation](https://git-scm.com/docs/gitattributes):

```
A textconv, by comparison, is much more limiting. You provide a transformation of the data into a line-oriented text format, and Git uses its regular difftools to generate the output.
```

Simply put, **textconv** only works for the internal **diff**, not for **difftool**.

To make **difftool** work, add the below to .git/config:

```
[difftool "docx"]
    cmd = t1=`mktemp` && `pandoc --to=plain $LOCAL --output=$t1` && t2=`mktemp` && `pandoc --to=plain $REMOTE --output=$t2` && meld $t1 $t2 && rm -f $t1 $t2

[difftool "xlsx"]
    cmd = t1=`mktemp` && `unoconv --format=csv --stdout $LOCAL >$t1` && t2=`mktemp` && `unoconv --format=csv --stdout $REMOTE >$t2` && meld $t1 $t2 && rm -f $t1 $t2
```

Now you can run:

```
$ git difftool --tool docx de8f24e f02d3c0
```

![](/assets/images/git-difftool-and-binary-files/difftool-docx.png)

```
$ git difftool --tool xlsx 07e90fe 31cb76d
```

![](/assets/images/git-difftool-and-binary-files/difftool-xlsx.png)

Let's add some more difftools.

Generic binary diff:

```
[difftool "xxd"]
   cmd = t1=`mktemp` && `xxd $LOCAL >$t1` && t2=`mktemp` && `xxd $REMOTE >$t2` && meld $t1 $t2 && rm -f $t1 $t2
```

```
$ git difftool --tool xxd de8f24e f02d3c0
```

![](/assets/images/git-difftool-and-binary-files/difftool-xxd.png)

Both *.docx and *.xlsx files are zipped archives of xml files and we can be compared using **unzip**:

```
[difftool "unzip-v"]
   cmd = t1=`mktemp` && `unzip -v $LOCAL >$t1` && t2=`mktemp` && `unzip -v $REMOTE >$t2` && meld $t1 $t2 && rm -f $t1 $t2

[difftool "unzip-c"]
   cmd = t1=`mktemp` && `unzip -c $LOCAL >$t1` && t2=`mktemp` && `unzip -c $REMOTE >$t2` && meld $t1 $t2 && rm -f $t1 $t2
```

```
$ git difftool --tool unzip-v de8f24e f02d3c0
```

![](/assets/images/git-difftool-and-binary-files/diftool-unzip-v.png)

```
$ git difftool --tool unzip-p de8f24e f02d3c0
```

![](/assets/images/git-difftool-and-binary-files/difftool-unzip-p.png)

You can list available difftools:

```
$ git difftool --tool-help
```

### Git auto difftool

Both **docx** and **xlsx** difftools work, but one question remains?
If we are happy with **textconv**(s) for **diff** why not use them for **difftool** without specifying --tool? It was explained above that it doesn't work out of the box, but let's see what can be done.

Add the below to .git/config:

```
[diff]
    tool = "auto"

[difftool "auto"]
    cmd = git-auto-diff.sh $LOCAL $REMOTE
```

Save the below bash script somewhere in your $PATH:

```
#!/bin/bash

DIFF=meld

if [[ $# -ne 2 ]]; then
    echo "Usage: $(basename $0) <FILE1.EXT> <FILE2.EXT>"
    exit 1
fi

FILE1=$1
FILE2=$2

EXT1=${FILE1##*.}
EXT2=${FILE2##*.}

if [[ $EXT1 != $EXT2 ]]; then
    echo "Extensions don't match, fallback to $DIFF"
    $DIFF $FILE1 $FILE2
    exit
fi

TEXTCONV=$(git config --get diff.$EXT1.textconv)
if [[ $? -ne 0 ]]; then
    echo "'$EXT1' textconv not found, fallback to $DIFF"
    $DIFF $FILE1 $FILE2
    exit
fi

$DIFF <($TEXTCONV $FILE1) <($TEXTCONV $FILE2)
```

The script checks that both file extensions are equal, there is a known textconv for it,
converts both file and runs the default diff tool, which is [meld](https://meldmerge.org/) in my case.
Otherwise, it falls back to the default diff tool.

Now you can run:

$ git difftool de8f24e f02d3c0

![](/assets/images/git-difftool-and-binary-files/difftool-auto-docx.png)

$ git difftool 07e90fe 31cb76d

![](/assets/images/git-difftool-and-binary-files/difftool-auto-xlsx.png)

And it works! I'm not going to claim that this is a production ready solution,
but as a prove of concept it works just fine.

Still, I found that [unoconv](https://github.com/unoconv/unoconv) is not stable for this use case and something simpler like [Gnumeric](http://www.gnumeric.org/) or [xlsx2csv](https://github.com/dilshod/xlsx2csv) should be used instead.

## Code

You can find the code [here](https://github.com/ten0s/blog-code/tree/main/git-difftool-and-binaries-files).

```
$ git clone https://github.com/ten0s/blog-code.git
$ cd blog-code/git-difftool-and-binaries-files
$ git config --local include.path $PWD/.gitconfig
```

```
$ sudo apt install meld pandoc unoconv
```

Copy git-auto-diff.sh to $PATH and you are all set!


### Resources

* [gitattributes](https://git-scm.com/docs/gitattributes)
* [git-diff](https://git-scm.com/docs/git-diff)
* [git-difftool](https://git-scm.com/docs/git-difftool)
* [8.2 Customizing Git - Git Attributes](https://git-scm.com/book/en/v2/Customizing-Git-Git-Attributes#Binary-Files)
* [Git diff images and pdfs](https://blog.oscarnajera.com/2017/11/git-diff-images-and-pdfs)
