---
title: Automating First Responder Kit
categories:
- automate
- ssms
excerpt: |
  You use First Responder Kit and
feature_text: |
  ## When all fail, use the dummiest idea
  Do I really look like a guy with a plan?
feature_image: "https://picsum.photos/2560/600?image=734"
image: "https://picsum.photos/2560/600?image=735"
---

I use the _First Responder Kit_. It is sooo helpful to troubleshoot TSQL performance problems. `sp_BlitzIndex` is the call I make most oftenly. I was looking for a way to type the command directly from the Object Browser, Templates Explorer, writing a stored proc then using my keyboard macro settings to transform the full object name into `exec dbatools.dba.sp_BlitzIndex @DatabaseName = 'DB1', @SchemaName = 'dbo', @TableName = 'MyTable'`. None of it was satisfiying. But I then found _AutoHotKey_, boy I was amazed at how easy it was to implement.

## The first implementation

AutoHotKey can implement a hot key that can be used anywhere. So I decided to implement `Ctrl+Shift+I` to transform `DB1.dbo.MyTable` into `exec dbatools.dba.sp_BlitzIndex @DatabaseName = 'DB1', @SchemaName = 'dbo', @TableName = 'MyTable'`

To ease the pain, you can drag and drop a table from the Object Browser into the Query Editor. It would produce something like `[dbo].[MyTable]`. This text would already be selected so I could implement this HotKey to implement :

1. Save the original ClipBoard content in a variable;
2. Clear the ClipBoard, then capture the selected text from the Query Editor in the ClipBoard;
3. Make the transformation and paste it back;
4. Recall the original ClipBoard content;

## Try it out

First, you need to install _AutoHotKey_. Then you need to create the script file with the AHL extension, ex: Blitz.AHK and adapt it to your need. Then run it.

```autohotkey
#SingleInstance Force

; --------------------------------------------------------------------
; Transform [dbo].[MyTable] into
; EXEC AMF_OutilsDBA..sp_BlitzIndex @DatabaseName = '?', @SchemaName = 'dbo', @TableName = 'MyTable';
; When pressing Crtl+Shift+I
; --------------------------------------------------------------------
^+i::
ClipSave := ClipboardAll
Clipboard := ""  ; Start off empty to allow ClipWait to detect when the text has arrived.
Send ^C
ClipWait
Array := StrSplit(Clipboard, ".", "[]", 4)
If (Array.MaxIndex() == 3)
  Clipboard := "EXEC AMF_OutilsDBA..sp_BlitzIndex @DatabaseName = '"Array[1]"', @SchemaName = '"Array[2]"', @TableName = '"Array[3]"';"
Else
  Clipboard := "EXEC AMF_OutilsDBA..sp_BlitzIndex @DatabaseName = '?', @SchemaName = '"Array[1]"', @TableName = '"Array[2]"';"
Send ^V
Clipboard := ClipSave
Return
```
