---
title: LearnVim
date: 2022-09-14 16:50:13
tags: Vim
categories: Editor
math: false
---

#### onoremap Is :\<c-u>normal! F]vi]\<cr>

用 onoremap 映射一个 motion，上面代码的意思为将 Is 按键映射为找到本行光标前的 ']' 符号，并选择在 '[]' 内的内容。按 dIs 就可以删除本行内光标位置之前的 '[]' 中的内容。

#### :\<c-r>\<c-w>

在命令模式下使用 ctrl + r 加 ctrl + w，代表把当前光标下的字符串写入命令行。

#### 10i\#

插入 10 个 '#'

#### normal 和 normal!

例如给 F 添加一个映射：nnoremap F x

这时 F 就有了两个含义：

* vim 的出厂设置定义：反向查找
* 用户的自定义：x 代表删除

normal F 代表用户的自定义用法，即 x 删除。normal! F 代表 vim 的出厂设置定义，即反向查找。

#### s 和 r

s 是替换当前光标下的字母并进入插入模式，r 是替换当前光标下的字母并保持在当前模式。

#### vnoremap . :normal! .\<cr>

可视模式下的映射，将 "." 映射为普通模式下的 "." 操作，普通模式下的 "." 操作为重复执行上一步的操作。例如：

asfdasdf aslfk sdf
asdfklsdf fklasdjf
sladfjaslas sdffasd
asdkfjksdjfksjdf sdfj

在第一行按 "A;"，给第一行添加了分号，然后在可视模式下选中剩下的三行，按下 "."，就可以给剩下三行也在最后添加分号。

#### 改变数字

当光标在数字之前，可以按 ctrl + a 来增加数字的大小，按 ctrl + x 来减小数字的大小，然而光标在数字之后不可以如此操作。因此我们可以先按 0 来将光标置于行首来改变数字大小。

如果将一列数字改变，如下：

array[0] = 0;   
array[0] = 0;   
array[0] = 0;   
array[0] = 0;   
array[0] = 0;   
array[0] = 0;   
array[0] = 0;   

可以使用可视块模式，即 ctrl + v 选中方括号中的 0 这一列，然后按 ctrl + a 来增加选中的全部数字的大小，按 ctrl + x 来减小选中的全部数字的大小。

如果想让数字递增，第一行的数字为 1，第二行的数字为 2 等，可以按 g ctrl + a 来实现，如果想让数字递减，可以按 g ctrl + x 来实现。

#### 生成数字

:put =range(1, 10) 可以从当前光标所在行的下一行开始，每一行插入一个数字，第一行为 1，第二行为 2，等等。

:0put =range(1, 10) 可以从首行开始，每一行插入一个数字，第一行为 1，第二行为 2，等等。

:for i in range(1, 10) | put ='196.168.0.' . i | endfor 可以生成十行字符串:
196.168.0.1  
196.168.0.2  
196.168.0.3  
196.168.0.4  
196.168.0.5  
196.168.0.6  
196.168.0.7  
196.168.0.8  
196.168.0.9  
196.168.0.10  

#### 录制宏及替代
**普通宏：**

| q          | a                 | \<operation> | q            |
| ---------- | ----------------- | ------------ | ------------ |
| 记录宏命令 | 记录宏到 a 寄存器 | 宏操作       | 记录结束退出 |

qa\<operation>q

使用宏：@a 或者 \<number>@a 来重复执行 number 次。

**录制套娃宏：**

| q          | a                 | \<operation> | @a   | q            |
| ---------- | ----------------- | ------------ | ---- | ------------ |
| 记录宏命令 | 记录宏到 a 寄存器 | 宏操作       | 套娃 | 记录结束退出 |

qa\<operation>@aq

使用宏：@a 

**替代方案：**

nnoremap \<leader>nl :%s/^/\\=printf('%-4d', line('.'))\<cr> 给所有行添加行标。

vnoremap \<leader>vn : s/^/\\=printf("%d. ", line(".") - line("'<") + 1)\<cr> 可视模式下给选中的行加行标。

#### set path+=**

递归搜索目录，即搜索所有的目录，包括它们的子目录，子目录的子目录，一直循环下去，意味着目录下所有的文件和文件夹都要搜索。以 find 命令为例，添加这个设置后，可以找到子目录下的文件。

#### set textwidth=70 和 set fo+=Mm

set textwidth=70 可以让一行显示70个字母，多的会自动换行，set fo+=Mm 可以让 textwidth 设置同样应用到中文，一行只显示 70 个汉字。

#### 搜索 / 和 ?

用 / 是从上往下搜索，按 n 是从上往下搜索下一个，按 N 是从下往上搜索上一个。用 ? 搜索，方向与 / 相反。

#### 搜索 * 和 \#

从上到下：

* 精确搜索：*
* 模糊搜索：g*

从下到上：

* 精确搜索：#
* 模糊搜索：g#

#### 正则表达式

| 表达式 | 含义                  |
| ------ | --------------------- |
| [ab1]  | a 或者 b 或者 1       |
| [0-9]  | 0 到 9 里的任意数字   |
| [a-z]  | a 到 z 里的任意字母   |
| [A-Z]  | A 到 Z 里的任意字母   |
| [^0-9] | 非 0-9 数字的任意字符 |

| 符号      | 含义          |
| --------- | ------------- |
| .         | 任意一个字符  |
| ?         | 没有或者一个  |
| *         | 没有或者多个  |
| +         | 一个或者多个  |
| [^a]      | 不是 a 的字符 |
| ^         | 行首          |
| $         | 行末          |
| \\d       | [0-9]         |
| \\D       | [^0-9]        |
| \\w       | [a-zA-Z0-9_]  |
| \\W       | [^a-zA-Z0-9_] |
| \\s       | 空格          |
| \\S       | 非空格        |
| {min,max} | 重复          |
| ()        | 组            |
| (A\|B)    | 组 A 或者组 B |
#### 计算混合运算

inoremap \<leader>js \<C-O>yiW\<End>=\<C-R>=\<C-R>0\<CR>
在插入模式下按 \<leader>js 就可以计算式子，其中\<C-O>为一次性使用命令，使用后仍回到插入模式，\<C-R>为调用寄存器

#### 窗口管理

命令行下

vim -o \<filename1>\<filename2> 依次水平打开窗口  
vim -O \<filename1>\<filename2> 依次垂直打开窗口

\<c-w>| 最大化宽度
\<c-w>_ 最大化高度

:on 或者 :only 关闭除当前光标所在窗口外的所有窗口

:mksession\<filename> 将当前窗口的 layout 记录到文件里，想要回到这样的 layout 就使用 :so\<filename> 命令

#### 替换搜索到的字符

nnoremap \<leader>s :%s/\\<\<C-R>\<C-W>\>//g\<left>\<left>

#### cnoreabbrev Q! q!

在命令模式下，将 Q! 映射为 q!

* c：命令行指令，也就是说该命令只在命令行里起作用
* nore：不循环映射
* abbrev：映射命令

#### 在命令行快速移动及修改

**移动**

| 快捷键                   | 含义                   |
| ------------------------ | ---------------------- |
| <-                       | 向左移动一个位置       |
| ->                       | 向右移动一个位置       |
| \<S-Left> or \<C-Left>   | 向左移动一个 word 位置 |
| \<S-Right> or \<C-Right> | 向右移动一个 word 位置 |
| CTRL-B or \<Home>        | 到行首位置             |
| CTRL-E or \<End>         | 到行尾位置             |

**删除**

| 快捷键 | 含义                 |
| ------ | -------------------- |
| CTRL-w | 删除一个 word        |
| CTRL-u | 从光标位置删除到行首 |

**覆盖**

\<Insert>: 进入覆盖模式，输入的字符覆盖以前的字符，再次按 \<Insert> 退出覆盖模式。

**取消输入，回到 Normal 模式**

* CTRL-c
* \<Esc>

#### 命令行窗口

在命令行窗口可以使用 normal 模式下的移动，可以使用各种 vimrc 里的映射。

可以通过两种方式打开这个窗口：

* 如果已经在命令行：'ctrl-f'
* 如果在 normal 模式：'q:'
  
退出 command-line window：

* 执行命令退出回到编辑窗口：\<CR>
* 通常使用的方式：':q'，':qa'，':qa!'
* ctrl-c：进入命令行，'q'，'qa'，'qa!'

甚至搜索模式下，也可以使用以下方式来打开 command-line 窗口来进行编辑：

* 从上到下的搜索：'q/'
* 从下到上的搜索：'q?'

如果在命令行的命令比较长，或者避免使用箭头按键来移动，就可以使用命令行窗口。

#### IdeaVimrc

```
" ================================================================================================
" Extensions 
" ================================================================================================
"下列插件需要在IDEA中下载
"ideaVim
"IdeaVim-EasyMotion
"IdeaVimExtension
"CodeGlance Pro

" ================================================================================================
" Basic settings
" ================================================================================================
"设置在光标距离窗口顶部或底部一定行数时，开始滚动屏幕内容的行为
set scrolloff=5

set easymotion

set cursorline

set showmatch

set nobackup

set surround

set ruler

set clipboard^=unnamed,unnamedplus

"--递增搜索功能：在执行搜索（使用 / 或 ? 命令）时，
"Vim 会在您输入搜索模式的过程中逐步匹配并高亮显示匹配的文本。
set incsearch

"--在搜索时忽略大小写
set ignorecase

"--将搜索匹配的文本高亮显示
set hlsearch

"--设置相对行号 和 当前行的绝对行号
set number relativenumber

"--设置返回normal模式时回到英文输入法
set keep-english-in-normal

" ================================================================================================
" No Leader Keymaps
" ================================================================================================

" go to somewhere (g in normal mode for goto somewhere)
nnoremap ga :<C-u>action GotoAction<CR>
nnoremap gb :<C-u>action JumpToLastChange<CR>
nnoremap gc :<C-u>action GotoClass<CR>
nnoremap gd :<C-u>action GotoDeclaration<CR>
nnoremap gs :<C-u>action GotoSuperMethod<CR>
nnoremap gi :<C-u>action GotoImplementation<CR>
nnoremap gf :<C-u>action GotoFile<CR>
nnoremap gm :<C-u>action GotoSymbol<CR>
nnoremap gu :<C-u>action ShowUsages<CR>
nnoremap gt :<C-u>action GotoTest<CR>
nnoremap gr :<C-u>action RecentFiles<CR>
nnoremap gh :<C-u>action Back<CR>
nnoremap gl :<C-u>action Forward<CR>

nnoremap ta :<C-u>action Annotate<cr>
nnoremap tb :<C-u>action ToggleLineBreakpoint<cr>
nnoremap tm :<C-u>action ToggleBookmark<cr>
nnoremap tp :<C-u>action ActivateProjectToolWindow<CR>

nnoremap <C-f> :<C-u>action ReformatCode<CR>
nnoremap J :<C-u>action MethodDown<CR>
nnoremap K :<C-u>action MethodUp<CR>
nnoremap <S-Down> :<C-u>action EditorCloneCaretBelow<CR>
nnoremap <S-Up> :<C-u>action EditorCloneCaretAbove<CR>

" ================================================================================================
"️️ Leader Keymaps
" ================================================================================================
let mapleader = " "
nnoremap <Leader>o :<C-u>action RecentProjectListGroup<CR>
nnoremap <Leader>r :<C-u>action Replace<CR>
vnoremap <Leader>r :<C-u>action Replace<CR>
nnoremap <Leader>R :<C-u>action ReplaceInPath<CR>
vnoremap <Leader>R :<C-u>action ReplaceInPath<CR>
nnoremap <Leader>f :<C-u>action Find<CR>
vnoremap <Leader>f :<C-u>action Find<CR>
nnoremap <Leader>F :<C-u>action FindInPath<CR>
vnoremap <Leader>F :<C-u>action FindInPath<CR>
nnoremap <Leader>h gT
nnoremap <Leader>l gt
nnoremap <leader>ns :action NextSplitter<CR>
nnoremap <leader>ps :action PrevSplitter<CR>
nnoremap <leader>sh :action SplitHorizontally<CR>
nnoremap <leader>sv :action SplitVertically<CR>
```