# Libreoffice Basic

- [Libreoffice Basic](#libreoffice-basic)
  - [Simple example](#simple-example)

## Simple example

```
sub AddDxDate

document   = ThisComponent.CurrentController.Frame
rem UNO is the LibreOffice API (see https://api.libreoffice.org)
dispatcher = createUnoService("com.sun.star.frame.DispatchHelper")

rem array declaration; primitive data types don't require declaration
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