# 4d-plugin-file-information

Read file DLL/EXE information on windows.

##Platform

| carbon | cocoa | win32 | win64 |
|:------:|:-----:|:---------:|:---------:|
|🚫|🚫|🆗|🆗|

Commands
---

```c
// --- File
FILE_Get_information
```

**Note**: ~~The [GetFileVersionInfoEx](https://msdn.microsoft.com/en-us/library/windows/desktop/aa969434(v=vs.85).aspx) function exists on Windows Vista and above, Windows Server 2008 and above~~.

Now using old [GetFileVersionInfo](https://msdn.microsoft.com/en-us/library/windows/desktop/ms647003(v=vs.85).aspx) instead.

Example
---

![](https://github.com/miyako/4d-plugin-file-information/blob/master/images/1.png)
