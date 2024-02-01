# 【shell】使用getopts时踩到的坑

作者：wallace-lai </br>
发布：2024-02-02 </br>
更新：2024-02-02 </br>

## 错误版本
如下所示，该shell脚本的大意是想从命令行中捕获用户输入的`-t`选项的值，并将其存入`TARGET`变量中。

```sh
#!/bin/bash

set -e

function main()
{
    while getopts "t:" OPT; do
        case $OPT in
            t)
                TARGET="$OPTARG"
                ;;
            ?)
                echo "Unknow Argument : ${OPT}"
                exit 1
                ;;
        esac
    done

    echo "TARGET : ${TARGET}"
}

BUILD_BASE="$(cd "$(dirname $0)" && pwd)"

main
```

上述脚本的理想运行结果应该是以下的形式：

```
$ bash build.sh -t tool
TARGET : tool
```

然而实际情况是，脚本根本没有捕获到用户输入的`-t`选项的值：

```
$ bash build.sh -t tool
TARGET :
```

## 正确的版本
百思不得其解之际，将脚本塞给文心一言，问它到底哪里出错了。得知问题出在调用`main`函数的时候，**你没有把脚本参数传递给`main`函数，人家当然捕获不到了**！正确的版本如下所示，只要在`main`函数的调用后面加上`"$@"`将脚本参数传递给`main`函数即可。

```
main "$@"
```