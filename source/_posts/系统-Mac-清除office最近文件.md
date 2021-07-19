---
title: 清除office最近文件
date: 2020-05-23 18:20:35
categories:
    - 系统
    - Mac
tags:
    - office
---

批量删除Mac系统下office最近文件

```bash
rm -rf ~/Library/Containers/com.microsoft.Word/Data/Library/Preferences/com.microsoft.Word.securebookmarks.plist

rm -rf ~/Library/Containers/com.microsoft.Excel/Data/Library/Preferences/com.microsoft.Excel.securebookmarks.plist

rm -rf ~/Library/Containers/com.microsoft.Powerpoint/Data/Library/Preferences/com.microsoft.Powerpoint.securebookmarks.plist
```

