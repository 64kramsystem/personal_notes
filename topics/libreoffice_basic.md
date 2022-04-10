# Libreoffice Basic

- [Libreoffice Basic](#libreoffice-basic)
  - [Basic concepts](#basic-concepts)
  - [Record macros](#record-macros)
  - [Simple example](#simple-example)

## Basic concepts

Documents include "modules", which are groups of macros.

Macros recorded without defining modules etc. are global, and they're stored under `$HOME/.config/libreoffice/4/user/basic/Standard/Module1.xba`.

## Record macros

Enable recording: `Tools -> Options -> LibreOffice -> Advanced -> Enable macro recording`.

Record: `Tools -> Macros -> Record Macro`

## Simple example

From `Tools -> Macros -> Edit Macros`:

```
sub AddDxDate

rem "UNO" is the LibreOffice API (see https://api.libreoffice.org)

controller = ThisComponent.CurrentController
document   = controller.Frame
dispatcher = createUnoService("com.sun.star.frame.DispatchHelper")

rem Backup the position

cursor = controller.getViewCursor()
cursorPrevPos = cursor.Start

rem Update TOC before adding the new line; since we can't fully add the new entry anyway, do what is
rem possible - update the existing entries.

dispatcher.executeDispatch(document, ".uno:UpdateAllIndexes", "", 0, Array())

rem We need to move the cursor twice, as updating the TOC does sort of locks the cursor.
rem Any movement (also gotoEnd()) will do for the first invocation.

cursor.goToRange(cursorPrevPos, false)
cursor.goToRange(cursorPrevPos, false)

rem Array declaration; primitive data types don't require declaration.

dim textArgs(5) as new com.sun.star.beans.PropertyValue
textArgs(2).Name = "Text"
textArgs(2).Value = Format(Now(),"NN DD/MM/YYYY HH:MM ")

dispatcher.executeDispatch(document, ".uno:InsertText", "", 0, textArgs())

dim formatArgs(2) as new com.sun.star.beans.PropertyValue
formatArgs(0).Name = "Template"
formatArgs(0).Value = "Heading 2"
formatArgs(1).Name = "FamilyName"
formatArgs(1).Value = "ParagraphStyles"
formatArgs(2).Name = "Style"
formatArgs(2).Value = "Heading 2"

dispatcher.executeDispatch(document, ".uno:StyleApply", "", 0, formatArgs())

end sub
```
