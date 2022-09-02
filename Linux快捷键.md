Shell命令
- Linux 快速打开终端
  - 1.进入设置-硬件-键盘-自定义快捷键。
  - 2.添加一个快捷键。名称：打开终端；命令：gnome-terminal；
  - 3.点击添加后，设置快捷键，例：Ctrl+Alt+T.
- 搜索
  -  find / -inname "filename" -type d -empty
  -  locate
  -  which 打印可执行文件的完整路径
  -  whereis 命令用于搜索给定命令的二进制、源码和手册页文件
- 字符串分割|文件分割|剪切|截取
  - echo $name | awk '{split($0, arr, " "); print arr[2]}'
  - echo $name | awk 'print $1'
- 文件相关
  - touch 创建文件
  - mk
  - ln python python2 # 软链接
  - mv python python # 移动、重命名
  - scp file root@192.168.1.194:/root/ws/file # 复制文件到另一台
  - rm -f file
  - mkdir dictory
  - rmdir dictory
  - cp file file1 (-r)
  - ll python* 列举python开头的文件
  - cat filename 查看filename内容
- 文件权限
  - chmod [mode] filename
  - chmod 764 filename
  - 0表示没有权限，1表示可执行权限， 2表示可写权限，4表示可读
  - 文件属主（u）与文件属主同组用户（g）和其他用户（o）
- 进程
  - ps -aux列出目前所有的正在内存当中的程序来显示全面的信息:
  - ps -aux --sort -pcpu | less 依据 CPU 使用来升序排序
  - ps -aux --sort -pmem | less
  - ps -ef 显示所有进程信息，连同命令行
- 过滤
  - grep -v "leo" /etc/passwd 反查
  - grep -n -c leo /etc/passwd 展示行号和统计行数
  - grep -A -B -C 1 leo passwd 环顾四周
  - grep -i 不区分大小写
  - grep -E "^root|bash$" pwd 等同于egrep命令实现了两个条件的搜索
- 校验
  - md5sum file.txt 求文件的MD5值
- 机器登录
  - ssh 10.0.0.112
  - shutdown now
  - exit
  - reboot
- gcc
  - gcc -o main main.c 编译
- 安装软件
  - yum search all libnuma-devel
  - yum install libnuma-devel
  - rpm -ivh apache-1.3.6.i386.rpm 安装rpm，i安装软件包v显示附加信息h安装时输出哈希标记
  - rpm -Uvh apache-1.3.6.i386.rpm
  - rpm -e apache-1.3.6.i386.rpm
  - rpm --nodeps
- 挂载
  - mount 查看当前挂载的所有文件系统

VIM配置文件
~~~
set nocompatible  关闭 vi 兼容模式
syntax on  自动语法高亮
set number  显示行号
set cursorline  突出显示当前行
set ruler  打开状态栏标尺
set shiftwidth=4  设定 << 和 >> 命令移动时的宽度为 4
set softtabstop=4  使得按退格键时可以一次删掉 4 个空格
set tabstop=4  设定 tab 长度为 4
set nobackup  覆盖文件时不备份
set autochdir  自动切换当前目录为当前文件所在的目录
filetype plugin indent on  开启插件
set backupcopy=yes  设置备份时的行为为覆盖
set ignorecase smartcase  搜索时忽略大小写，但在有一个或以上大写字母时仍保持对大小写敏感
set nowrapscan  禁止在搜索到文件两端时重新搜索
set incsearch 输入搜索内容时就显示搜索结果
set hlsearch  搜索时高亮显示被找到的文本

set showmatch  插入括号时，短暂地跳转到匹配的对应括号
set matchtime=2  短暂跳转到匹配括号的时间
set magic " 设置魔术
set hidden  允许在有未保存的修改时切换缓冲区，此时的修改由 vim 负责保存
set smartindent  开启新行时使用智能自动缩进
set backspace=indent,eol,start  不设定在插入状态无法用退格键和 Delete 键删除回车符
set cmdheight=1  设定命令行的行数为 1
set laststatus=2  显示状态栏 (默认值为 1, 无法显示状态栏)
set statusline=\ %<%F[%1*%M%*%n%R%H]%=\ %y\ %0(%{&fileformat}\ %{&encoding}\ %c:%l/%L%)\   设置在状态行显示的信息
set foldenable  开始折叠
set foldmethod=syntax  设置语法折叠
set foldcolumn=0  设置折叠区域的宽度
setlocal foldlevel=1  设置折叠层数为
set foldclose=all  设置为自动关闭折叠 
nnoremap <space> @=((foldclosed(line('.')) < 0) ? 'zc' : 'zo')<CR>  用空格键来开关折叠
~~~
VIM控制模式
~~~
ctrl + v    块模式，可选中同一列多行的字符
ctrl + v 进入列编辑模式，向下或向上移动光标，把需要注释的行的开头标记起来，然后按I，再插入注释符，再按Esc，就会全部注释
G   // 文件最底部
456gg 或者 456G   // 跳转到456行
zf456G 或者 zf456gg   // 从当前行折叠到456行
zm    // 关闭折叠区
zr    // 打开折叠区
zd    // 删除折叠
zj    // 向下一个折叠移动
~~~