# Shell 编码规范

> 作者：Matthew Wang &lt;matt&#x77;yl&#x40;gmai&#x6c;.c&#x6f;m&gt;<br>
> 链接：[github.com/ymattw/shell-style-guide](https://github.com/ymattw/shell-style-guide/blob/master/shell-style-guide-cn.md)

本文为作者结合自身多年 shell 编码经验并参考 Google Shell Style Guide <sup>[1]</sup> 完成，**文中 shell 特指 bash**。转载随意（CC0 1.0 通用版权）。

## 命名规范

### 文件

- 文件名：全小写，扩展名为 `.sh`，文件名要有明确含义
  - 主文件的文件名必要时采用连接符分隔，如 `build-foobar.sh`
  - 函数库文件名以 `lib` 开始，如 `liblog.sh`
- 文件编码：utf-8，以 LF (\n) 分隔行，CR (\r) 将引起错误
- 权限位：主文件加执行权限（chmod +x），但函数库文件不加

### 全局变量

定义在所有函数之外的变量为全局变量。

- 全部大写，单词间以下划线连接
- 定义后不会修改的以 `readonly` 修饰

例如：

```bash
readonly TOP_DIR="/opt/myapp"

COMMAND_ARGS="$*"
```

### 函数名

全部小写，单词间以下划线连接。

### 局部变量

定义在函数内部的变量为“局部变量”，但如果不加 `local` 修饰将**全局可见**。

- 全部小写，单词间以下划线连接
- 在函数开始处统一以 `local` 先声明后使用
- 整型数额外加 `-i` 修饰
- 数组额外加 `-a` 修饰

例如：

```bash
function status_check
{
    local prog=${1:?}
    local -i counter
    local -a args

    ...
}
```

## 格式

### 解释器（shebang）

用 `#!/bin/bash`，无空格，不带选项。

### 注释

注释用 `#` 紧跟一个空格开始，用英文。

紧接 shebang 后之后要撰写头部注释：

- 解释文件的用途
- 简要解释如何使用（详细的写 usage 函数提供）
- 公司版权声明（如果有）

其他约定：

- 代码中的注释，多行的在最后一行注释后额外加入一个空注释行
- 行尾注释，`#` 符号之前至少留出两个空格
- 必要时使用特殊标记：`TODO`，`FIXME`，`XXX`，多数编辑器能高亮显示它们

例如：

```bash
# This file stores the pid of the parent process
PID_FILE="/var/run/foo_daemon.pid"

# This function probes a tcp port by trying to establish
# a connection without sending any data. Returns 0 on
# success, 1 otherwise. TODO: implement an udp version
#
function tcp_port_test
{
    local timeout=5  # in second
    ...

    # FIXME: detect whether nc is installed
    nc -w $timeout $host $port

    # XXX: on older system '-w' is not for connection timeout
    ...
```

提示：`vimrc` 中设置 `set formatoptions=tcroqnmMB` 后，编辑多行注释时可在
Normal 模式下按 `V` 进入行模式，`j` 和 `k` 选择多行，然后 `gq` 即可按当前行宽重排注释。

### 行宽

尽可能保持在 80 列以内，以便适应并排对比和打印输出的场合，使用 backslash `\` 断行。参考一些断行的写法：

```bash
echo "Here we output a very very long message need to break into multiple" \
     "lines with backslashes"

tar -Pzcf $backup_dir/host-config.tar.gz \
    /etc/ssh/ssh_host* \
    /etc/ssh/sshd_config \
    /etc/hosts \
    /etc/postfix/main.cf \
    /etc/postfix/relay_passwd
```

以下 `vimrc` 配置可将第 80 列的字符标成红色。更多请见 [参考 vimrc 配置](
https://github.com/ymattw/profiles/blob/1.1/vimrc#L134-L137)。

```vim
highlight! link CharAtCol80 WarningMsg
match CharAtCol80 /\%80v/
```

### 缩进

用四个空格缩进。以下示例 `vimrc` 配置设定 tab 为 soft tab 四个空格，为 Makefile
保持 hard tab。更多请见 [参考 vimrc 配置
](https://github.com/ymattw/profiles/blob/1.1/vimrc#L121-L129)。

```vim
set expandtab softtabstop=4 shiftwidth=4 tabstop=8
autocmd! BufEnter *[Mm]akefile*,[Mm]ake.*,*.mak,*.make setlocal filetype=make
autocmd! FileType make setlocal noexpandtab shiftwidth=8
```

### 空行和空格

使用单行空行分隔代码中的小段逻辑增加可读性。不留多余空格。使用下面 vim
配置可以高亮显示去多余空白，且 Normal 模式下按空格开关显示。更多请见 [参考
vimrc 配置](https://github.com/ymattw/profiles/blob/1.1/vimrc#L113-L117)。

```vim
set list listchars=tab:▸\ ,trail:▌
nmap <Space> :set list!<CR>
```

注：Windows 系统下对 utf-8 字符显示支持不完善，可考虑更换上面的两个特殊字符为 `>` 和 `_`。

### 全局选项

- 始终使用 `set -o nounset` 确保变量已经定义
- 在主文件且只在主文件中使用 `set -o errexit` 第一时间捕获错误

注：参考后面的“陷阱”一节。

### 函数定义

采用 `function func_name` 并将开括弧放在下一行，不用多余的 `()` 记号。这样至少有两个好处：

- 从文件中查找所有函数定义只需要 `grep ^function` 即可
- Vim 中设置 `smartindent` 之后下一行会正确自动缩进

```bash
function func_name
{
    ...
}
```

### 分支

将 `then` 放在和 `if` 同一行。

```bash
if condition 1; then
    ...
elif condition 2; then
    ...
else
    ...
fi
```

### 循环

将 `do` 放在和 `for` 和 `while` 同一行。

```bash
for x in "foo" "bar" "quz"; do
    ...
done

while true; do
    ...
done
```

### Case

- 分支缩进四格
- 结束符 `;;` 单独一行

```bash
case "$OS" in
    Linux)
        echo "This is a Linux system"
        ;;
    Darwin)
        echo "This is a Mac system"
        ;;
    *)
        echo "Unknown OS $OS" >&2
        ;;
esac
```

### 变量引用

可能引起阅读困难时使用 `${VAR}`，例如对比下面两种写法，使用第一种。

```bash
# Good
LOG_FILE="${LOG_PREFIX}-${PROG_NAME}-${LOG_SUFFIX}"

# Bad
LOG_FILE="$LOG_PREFIX-$PROG_NAME-$LOG_SUFFIX"
```

### 引号

因为需要引用变量的场合占多数，字符串使用双引号，除非下面两种情况：

- 需要规避变量展开
- 字符串内部有多处双引号需要转义

```bash
readonly TOP_DIR="/opt/myapp"
readonly CONFIG="$TOP_DIR/myapp.conf"
```

使用单引号场合示例：

```bash
function start
{
    local banner='Welcome to the "Awesome Portal"'
    local pattern='loaded$'
```

### 导入其他文件

用 `source` 而不是点 `.` 因为前者可读性更好。

### 命令替换（subshell）

使用 `$(..)` 而不是 \`..\` 因为前者可读性更好并且能嵌套。

```bash
SELF_DIR=$(cd $(dirname $0) && pwd)
```

### 条件测试

使用 `[[ .. ]]` 而不是 `[ .. ]` 或 `test`，因为第一种容错性最好。

```bash
if [[ -n "$FOO" ]]; then
    ..
fi
```

不用 `[[ "x$FOO" == "x" ]]` 这种传承自古老 Unix 系统的写法，改用 `[[ -z "$FOO" ]]`。

### 算术运算

使用 `(( .. ))`，注意类似 `[[ .. ]]` 内部前后各留一个空格。括弧内的变量不使用 `$` 引用。

```bash
(( COUNTER += 1 ))

if (( COUNTER > TIMEOUT )); then
    ..
fi
```

### Here document

需要写大段文本或模板时采用 here document。无需变量展开时结束符使用双引号。

```bash
function usage
{
    cat << EOT

Usage: $0 [options]

--help          Display this message and exit
--file FILE     Specify target filename, default is /etc/myapp.conf

EOT
}

MESSAGE=$(cat << "EOT"
Prices list:

- Foo: $2.99
- Bar: $1.49
EOT)
```

## 惯例

### 环境

注意很多系统下 `/usr/sbin` 和 `/sbin` 不在默认路径中，如果程序里引用到它们下的命令，将它们加进
`PATH`。安全性敏感的脚本，应显式设置 PATH，如：

```bash
export PATH="/usr/local/bin:/usr/bin:/bin:/usr/local/sbin:/usr/sbin:/sbin"
```

小心处理一些常见的环境变量，例如启动一个 Java 应用时，如果依赖系统默认安装的
Java 环境，应 `unset JAVA_HOME` 等避免受用户环境干扰。

### 错误输出

错误信息输出到 `&2` (stderr)，仅当条件测试时才考虑重定向至 `/dev/null`。

```bash
if ! which nc > /dev/null 2>&1; then
    echo "ERROR: nc is not installed, aborting" >&2
    exit 1
fi
```

### 当前目录

不要改变当前目录，也不采用 `pushd` 和 `popd`，因为会给维护者增加负担。

如果需要得到当前脚本的绝对路径，采用下面方法：

```bash
SELF_DIR=$(cd $(dirname $0) && pwd)
```

尽量使用一些程序自己的特性改变目录，如 `tar -C`，`patch -d`。如果需要临时改变当前目录，采用子 shell：

```bash
(
    cd /tmp/foo
    ...
)
```

### 临时文件

使用 `mktemp` 创建临时文件和目录，采用 `trap EXIT` 做清理。

```bash
TMP_DIR=$(mktemp -d /tmp/foo.XXXXXXXXXX)
TMP_FILE=$(mktemp /tmp/bar.XXXXXXXXXX)
trap "rm -rf $TMP_DIR $TMP_FILE" EXIT
```

### 检查输入

- 采用全局选项 `set -o nounset` 确保变量展开时已定义
- 使用 `${VAR:?}` 或 `${VAR:?"Error mssage"}` 确保关键变量已经定义且非空
- 用 `${VAR:-"default value"}` 设置变量默认值

考虑下面的例子，如果变量名不慎写错，运行时前一种就会及时报错，后一种则将会发生惨剧 :-)。

```bash
# Good
rm -rf ${TEMPORARY_DATA_DIR:?}/*

# Bad
rm -rf $TEMPORARY_DATA_DIR/*
```

检查函数输入参数的例子：

```bash
function status_check
{
    local prog_name=${1:?}
    local timeout=${2:-5}    # In second

    ...
}
```

### Main 函数

尽量用函数组织代码中的独立逻辑，函数外的代码留给全局变量定义，其他代码变多时放进 `main` 函数，然后按如下方式调用：

```bash
main "$@"
```

### '&&' 和 '||'

仅将 `&&` 和 `||` 用于连接多个测试条件。`||` 适用于“断言（assertion）”时方可接命令语句。一些例子如下：

```bash
# Good: assertion
(( $# > 1 )) || { usage; exit 1; }

# Good
if [[ -r $conf ]] && grep -q "foobar" $conf; then
    load_config $conf
fi

# Bad: may be suffered by "set -o errexit" when the config file does not exist
[[ -r $config ]] && grep -q "foobar" $conf && load_config $conf
```

### Exit vs return

函数内部只使用 `return`，返回 `0` 代表成功，返回 `1` 代表失败。`exit` 只用于主程序。

### Builtin vs 外部命令

优先使用 builtin (内置) 命令，如

```bash
# Good
SUM=$(( x + y ))
FILE_SUFFIX=${PATHNAME%%*.}

# Bad
SUM=$(expr $x + $y)
FILE_SUFFIX=$(echo "$PATHNAME | sed -e 's/.*\.//')
```

## 陷阱

### 选项 errexit

启用了 `set -o errexit` 时小心每条命令的返回值，它的作用是任何命令返回值 `$?` 非 0 时即让程序将直接退出。但它在下面几种情形时不起作用：

- 命令跟在 while, until, if 关键字之后
- 命令是 `&&` 或 `||` 语句的一部分
- `! foo`

应当显示中断执行，例如 `! foo` 该写成：

```bash
! foo || return 1
```

对于不会输出错误信息的命令，还要显式输出，以便容易知道程序终止在何处。

```bash
nc -w 5 $host $port || {
    echo "$host:$port seems unreachable" >&2
    exit 1
}
```

### Subshell 给 local 变量赋值

注意 `local output=$(foo_command)` 赋值始终会成功（返回 0），若依赖命令返回值，要分开两行写：

```bash
    local output

    output=$(foo_command)  # errexit will capture the return value
```

### Shift

小心 `shift 2` 在参数个数小于 2 时的行为将是一个也不移，在某些系统上并不返回错误，应改成 `shift; shift`。

## 参考

[1] [Google Shell Style Guide](https://google-styleguide.googlecode.com/svn/trunk/shell.xml)
