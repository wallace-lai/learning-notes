# 解决cargo build出现timeout问题

作者：wallace-lai </br>
发布：2024-02-07 </br>
更新：2024-02-07 </br>

在给rust项目添加依赖后，使用`cargo build`重新构建项目出现了timeout问题，如下所示。

```
D:\WORKSPACE\learn-rust\CH02\guess>cargo build
    Updating `ustc` index
warning: spurious network error (3 tries remaining): [28] Timeout was reached (Failed to connect to xxx.xxx.xxx.xxx port xxxx after 21033 ms: Couldn't connect to server)
warning: spurious network error (2 tries remaining): [28] Timeout was reached (Failed to connect to xxx.xxx.xxx.xxx port xxxx after 21031 ms: Couldn't connect to server)
warning: spurious network error (1 tries remaining): [28] Timeout was reached (Failed to connect to xxx.xxx.xxx.xxx port xxxx after 21032 ms: Couldn't connect to server)
error: failed to get `rand` as a dependency of package `guess v0.1.0 (D:\WORKSPACE\learn-rust\CH02\guess)`

Caused by:
  failed to query replaced source registry `crates-io`

Caused by:
  download of config.json failed

Caused by:
  failed to download from `https://mirrors.ustc.edu.cn/crates.io-index/config.json`

Caused by:
  [28] Timeout was reached (Failed to connect to xxx.xxx.xxx.xxx port xxxx after 21043 ms: Couldn't connect to server)

```

看报错信息应该是`cargo build`的过程中自动连接了某个代理ip（xxx.xxx.xxx.xxx），但这个代理ip连接不成功。打开本地的代理软件，发现代理ip确实发生了变化，不再是之前的xxx.xxx.xxx.xxx了。

进入`%USERPROFILE%\.cargo`目录下（Windows平台），修改`config`文件。在`[http]`下添加最新的代理ip和端口。如下所示，问题解决。

```
[source.crates-io]
replace-with = 'ustc'

[source.ustc]
registry = "sparse+https://mirrors.ustc.edu.cn/crates.io-index/"

[http]
proxy="yyy.yyy.yyy.yyy:zzzz"
```
