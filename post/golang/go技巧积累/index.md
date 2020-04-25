
# 一、交叉编译


linux下编译windows的可执行文件
```bash
GOOS="windows" GOEXE=".exe" go build
```

win下编译linux的可执行文件
```bash
set GOOS="linux"
set GOARCH="amd64"
go build
```


# 二、条件编译
[条件编译](https://studygolang.com/articles/154)

## 1 使用编译标签 

放到源代码文件头部，注意空一行
```go
// +build darwin freebsd netbsd openbsd

package mypkg // correct
```
这个将会让这个源文件只能在支持kqueue的BSD系统里编译


```go
// +build linux darwin
// +build 386
```
这个将限制此源文件只能在 linux/386或者darwin/386平台下编译

## 2 文件后缀


```go
mypkg_linux.go         // only builds on linux systems
mypkg_windows_amd64.go // only builds on windows 64bit platforms
```
