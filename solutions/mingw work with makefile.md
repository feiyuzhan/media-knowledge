
First check if mingw32-make is installed on your system. Use mingw32-make.exe command in windows terminal or cmd to check, else install the package mingw32-make-bin.

then go to bin directory default ( C:\MinGW\bin) create new file make.bat

```c
@echo off
"%~dp0mingw32-make.exe" %*
```

add the above content and save it

set the env variable in powershell

```c
$Env:CC="gcc"
```

then compile the file

```c
make hello
```

where hello.c is the name of source code


Reference:
https://stackoverflow.com/questions/11772602/how-to-compile-makefile-using-mingw
