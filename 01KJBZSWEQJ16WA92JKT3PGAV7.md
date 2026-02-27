---
title: 20250522 - 在windows调试RPA
confluence_page_id: 3997739
created_at: 2025-05-23T05:16:35+00:00
updated_at: 2025-05-23T05:16:35+00:00
---

# pywinauto

pip install -U pywinauto

运行脚本: 

```
from pywinauto.application import Application
app = Application().start("notepad.exe")

app.UntitledNotepad.menu_select("帮助(&H)->关于记事本(&A)")
app['关于“记事本”']["确定"].click()
app.UntitledNotepad.Edit.type_keys("pywinauto Works!", with_spaces = True)
``` 

可成功
