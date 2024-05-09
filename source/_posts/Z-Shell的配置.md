---
title: Z-Shell的配置
date: 2024-05-06 15:46:27
description: Z-Shell的配置过程记录
categories:
- Linux
tags:
- Linux
- Z-Shell
- Shell
---

# Z-Shell配置

## 前置准备

创建一个文件夹来存放zsh的配置

```sh
$ mkdir ~/.config/zsh/plugins
```

## 基础配置

在第一次使用Z-Shell时，由于配置文件.zshrc不存在，会出现提示，按`1`进入配置选项

Please pick one of the following options:

```
(1)  Configure settings for history, i.e. command lines remembered
     and saved by the shell.  (Recommended.)

(2)  Configure the new completion system.  (Recommended.)

(3)  Configure how keys behave when editing command lines.  (Recommended.)

(4)  Pick some of the more common shell options.  These are simple "on"
     or "off" switches controlling the shell's features.  

(0)  Exit, creating a blank ~/.zshrc file.

(a)  Abort all settings and start from scratch.  Note this will overwrite
     any settings from zsh-newuser-install already in the startup file.
     It will not alter any of your other settings, however.

(q)  Quit and do nothing else.  The function will be run again next time.
--- Type one of the keys in parentheses --- 
```

### 历史设置

按1进入历史选项设置，我的设置如下

```
History configuration
=====================

# (1) Number of lines of history kept within the shell.
HISTSIZE=100000                                                                      (set but not saved)
# (2) File where history is saved.
HISTFILE=~/.zsh_history                                                              (set but not saved)
# (3) Number of lines of history to save to $HISTFILE.
SAVEHIST=100000                                                                      (set but not saved)

# (0)  Remember edits and return to main menu (does not save file yet)
# (q)  Abandon edits and return to main menu

--- Type one of the keys in parentheses ---
```

### 配置补全系统

```
The new completion system (compsys) allows you to complete
commands, arguments and special shell syntax such as variables.  It provides
completions for a wide range of commonly used commands in most cases simply
by typing the TAB key.  Documentation is in the zshcompsys manual page.
If it is not turned on, only a few simple completions such as filenames
are available but the time to start the shell is slightly shorter.

You can:
  (1)  Turn on completion with the default options.

  (2)  Run the configuration tool (compinstall).  You can also run
       this from the command line with the following commands:
        autoload -Uz compinstall
        compinstall
       if you don't want to configure completion now.

  (0)  Don't turn on completion.

--- Type one of the keys in parentheses ---
```


按`2`进行自动补全功能的自定义

```
If you have some already defined by compinstall, edit the name of the
file where these can be found.  Note that this will only work if they
are exactly the form in which compinstall inserted them.  If you leave
the line as it is, or empty, I won't search.
file> /home/user/.zshrc
```

直接回车即可

```
               *** compinstall: main menu ***
Note that hitting `q' in menus does not abort the set of changes from
lower level menus.  However, quitting at top level will ensure that nothing
at all is actually written out.

1.  Completers:  choose completion behaviour for tasks such as
    approximation, spell-checking, expansion.

2.  Matching control: set behaviour for case-insensitive matching,
    extended (partial-word) matching and substring matching.

3.  Styles for changing the way completions are displayed and inserted.

4.  Styles for particular completions.

c.  Change context (plus more information on contexts).

q.  Return without saving.
0.  Save and exit.

```

#### Compliter配置

按`1`，进入completers配置
```
Current context: :completion:*

The following completers are available.  Those marked `(*)' are already
set for the context shown above.  If none are selected, the completers will
not be set for this context at all.

1. (*) Basic completion.
2. (*) Approximate completion:  completion with correction of existing word.
3. (*) Correction:  correct existing word, no completion.
4. (*) Expansion: use globbing and parameter substitution, if possible.

o.     Set options for the completers above.
m.     Set completers that modify the behaviour of the four main ones above.
q.     Return without saving.
0.     Done setting completers.
```

按`1`,`2`,`3`,`4`选择四个completer的选项，然后按`o`

##### 设置:completion:\*选项

```
Current context: :completion:*

The following options are available.  Note that these require the relevant
completers to be present, as set in the menu above this one.

a.     Set options for approximation or correction.
e.     Set options for expansion.
q.     Return without saving.

0.     Done setting options.
```

###### 模糊和修正补全

按a，进入模糊和改正匹配的设置

```
Approximation and correction can correct the errors in what you have typed,
up to a maximum number of errors which you can specify.  Each `error'
is the omission of a character, the addition of a superfluous character,
the substitution of one character by an incorrect one, or transposition of
two different characters.

Current context: :completion:*

To have different values for approximation and correction, you should
change the context appropriately.  For approximation, use
`:completion:*:approximate:*' and for correction use
`:completion:*:correct:*'.

Enter maximum number of errors allowed:

number>3
```

输入修正的错误字符个数

然后设置数字前缀行为的选项，用于指定在自动补全或近似补全时如何处理数字前缀：
```
Select behaviour of numeric prefix.

1.     Numeric prefix is not used by approximation or completion.
2.     Numeric prefix, if provided, gives max number of errors allowed,
       replacing the number you just typed for that one completion.
3.     Numeric prefix, if provided, prevents approximation or completion
       from taking place at all for that one completion.

--- Hit selection ---
```

我选择的是2，可以在前面输入前缀表明允许的错误个数

###### 扩展选项

按`e`进入扩展选项，选项：
- 通配符扩展
- 替换功能
- 显示所有补全选项

```
The _expand completer can be tuned to perform any of globbing (filename
generation), substitution (anything with a `$' or backquote), or
normal completion (which is useful for inserting all possible completions
into the command line).  For each feature, a 1 turns it on, while a 0 turns
it off; if the feature is unset, that expansion will *not* be performed.

You can also give more complicated mathematical expressions, which can use
the parameter NUMERIC to refer to the numeric argument.  For example, the
expression `NUMERIC == 2' means that the expansion takes effect if you
type ESC-2 (Emacs mode) or 2 (Vi command mode) before the expansion.
Quotes will be added automatically as needed.

g.     Set condition to perform globbing: 1
s.     Set condition to perform substitution: 1
c.     Set condition to perform completion: 1
0.     Done setting conditions (will not be saved until you leave options)

--- Enter selection ---
```

然后返回这个界面
```
              *** compinstall: completer menu ***

Current context: :completion:*

The following completers are available.  Those marked `(*)' are already
set for the context shown above.  If none are selected, the completers will
not be set for this context at all.

1. (*) Basic completion.
2. (*) Approximate completion:  completion with correction of existing word.
3. (*) Correction:  correct existing word, no completion.
4. (*) Expansion: use globbing and parameter substitution, if possible.

o.     Set options for the completers above.
m.     Set completers that modify the behaviour of the four main ones above.
q.     Return without saving.
0.     Done setting completers.
```

##### 设置备用completers

选择`m`设置备用Completers

```
              *** compinstall: minor completer menu ***

Current context: :completion:*

The following completers are available.  Those marked `(*)' are already
set for the context shown above.  Note none of these are required for
normal completion behaviour.

1. (*) _ignored: Use patterns that were previously ignored if no matches so far.
2. (*) _list:    Only list matches until the second time you hit TAB.
3.     _oldlist: Keep matches generated by special completion functions.
4. (*) _match:   If completion fails, retry with pattern matching.
5.     _prefix:  If completion fails, retry ignoring the part after the cursor.

o.     Set options for the completers above.
q.     Return without saving.
0.     Done setting minor completers.
```

按`0`退出备用completer设置

#### 匹配控制

设置了四个匹配器，按优先顺序：
1. 不匹配
2. 小写匹配大写
3. 大小写相互匹配
4. 部分单词匹配，用三个字符`.`、`_`、`-`来表示省略部分进行匹配。实现这个功能需要在匹配的单词后面加上.，比如输入`c.u`不会匹配`comp.source.unix`，而输入`c.u.`会匹配。

```
              *** compinstall: matcher menu ***

`Matchers' compare the completion code with the possible matches in some
special way.  Numbers in parentheses show matchers to be tried and the order.
The same number can be assigned to different matchers, meaning apply at the
same time.  Omit a sequence number to try normal matching at that point.
A `+' in the first line indicates the element is added to preceding matchers
instead of replacing them; toggle this with `t'.  You don't need to set
all four, or indeed any matchers --- then the style will not be set.

   (    )   `+' indicates add to previous matchers, else replace
n. (1   ) No matchers; you may want to try this as the first choice.
c. ( 2  ) Case-insensitive completion (lowercase matches uppercase)
C. (  3 ) Case-insensitive completion (lower/uppercase match each other)
p. (   4) Partial-word completion:  expand 'f.b' to 'foo.bar', etc., in one go.
          You can choose the separators (here `.') used each time.
s. (    ) Substring completion:  complete on substrings, not just initial
          strings.  Warning: it is recommended this not be used for element 1.

t.        Toggle replacing previous matchers (` ' at top) or add (`+')
q.        Return without saving.
0.        Done setting matchers.

--- Hit selection --- p
Set/unset for element number (1234)? 4
Edit the set of characters which terminate partial words.  Typically
these are punctuation characters, such as `.', `_' and `-'.
The expression will automatically be quoted.

characters> ._-

You can allow the partial-word terminators to be matched in the pattern,
too:  then  for example `c.u' would expand to `comp.source.unix', whereas
usually you would need to type an extra intervening dot.  Do you wish the
terminators to be matched in this way? (y/n) [n] y
```

#### 设置自动完成显示和插入的样式

Styles for changing the way completions are displayed and inserted.

```
         *** compinstall: display and insertion options ***

1.  Change appearance of completion lists:  allows descriptions of
    completions to appear and sorting of different types of completions.

2.  Change how completions are inserted: includes options for sorting,
    and keeping the original or an unambiguous prefix with correction etc.

3.  Configure coloured/highlighted completion lists, selection of items
    and scrolling.

4.  Change whether old-style `compctl' completions will be used.

q.  Return without saving.
0.  Done setting display and insertion options.

--- Hit selection --- 
```

##### 自动完成列表的外观

```
       *** compinstall: order and descriptions in completion lists ***
Type the appropriate number for more information on how this would affect
listings.

1.  Print a message above completion lists describing what is being
    completed.

2.  Make different types of completion appear in separate lists.

3.  Make completion verbose, using option descriptions etc. (on by default).

4.  Make single-valued options display the value's description as
    part of the option's description.

q.  Return without saving.
0.  Done setting options for formatting of completion lists.
```

1. 输出正在进行的补全操作

```
You can set a string which is displayed on a line above the list of matches
for completions.  A `%d' in this string will be replaced by a brief
description of the type of completion.  For example, if you set the
string to `Completing %d', and type ^D to show a list of files, the line
`Completing files' will appear above that list.  Enter an empty line to
turn this feature off.  If you enter something which doesn't include `%d',
then `%d' will be appended.  Quotation will be added automatically.

description> Compliting %d
```

2. 不同类型的补全出现在单独的列表中

```
Normally, all possible completions are listed together in a single list, and
if you have set a description with 1) above, the descriptions are listed
together above that.  However, you can specify that different types of
completion appear in separate lists; any description appears above its
own list.  For example, external commands and shell functions would appear
in separate lists when you are completing a command name.  Do you
want to turn this on?

[y]es, [n]o, [k]eep old setting? y
```

3. 自动补全显示详细信息，比如选项描述

```
By default, completion uses a `verbose' setting.  This
affects different completions in different ways.  For example,  many
well-known commands have short, uninformative option names; in some cases,
completion will indicate what the options do when offering to complete them.
If you prefer shorter listings you can turn this off.  What setting to
you want?

[v]erbose, [n]ot verbose, [k]eep old setting? 
```

4. 当选项的值是单一的时候，使值的描述显示在选项的描述中。

```
Many commands have options which take a single argument.  In some cases,
completion is not set up to describe the option even though it has a
description for the argument.  You can enter a string containing `%d',
which will be replaced by the description for the option.  For
example, if you enter the string `specify: %d', and an option -ifile
exists which has an argument whose description is `input file', then the
description `specify: input file' will appear when the option itself
is listed.  As this long explanation suggests, this is only occasionally
useful.  Enter an empty line to turn this feature off.  If you enter
something which doesn't include `%d', then `%d' will be appended.
Quotation will be added automatically.

auto-description> %d
```

##### 更改自动补全的插入方式

```
          *** compinstall: options for inserting completions ***

1.   In completers that change what you have already typed, insert any
     unambiguous prefix rather than go straight to menu completion.

2.   In completers which correct what you have typed, keep what you
     originally typed as one of the list of possible completions.

q.   Return without saving.
0.   Done setting options for insertion.
```

1. 在你已经输入一部分内容后，如果系统能够确定你输入的内容在自动完成列表中有一个明确的前缀，那么系统会直接将这个前缀插入到你的输入中。这样你就不需要再从自动完成列表中选择完整的选项了。(启用)
2. 当系统纠正你输入的内容后，它会将你最初输入的内容保留在自动完成列表中，作为一个备选选项。这样，即使系统纠正了你的输入，你还是可以选择保留原始输入的内容。(启用)

##### 配置彩色和高亮显示的自动完成列表，光标选择项目和滚动浏览列表

```
     *** compinstall: options for colouring and selecting in lists ***

1.   Use coloured lists for listing completions.

2.   Use cursor keys to select completions from completion lists.

3.   Allow scrolling of long selection lists and set the prompt.

q.   Return without saving.
0.   Done setting options for insertion.
```

1. 设置补全列表的颜色 <a id="section1"></a>
    - 默认颜色
    - 通过`$LS_COLORS`环境变量已经设定好的颜色
    - 关闭颜色(*)
2. 使用光标键(方向键)在自动补全菜单中选择补全项
    - 无条件启用该功能
    - 补全项超过指定数量时启用该功能(*, 3）
        - `Edit a prompt below. It can contain '%l' to show the number of matches as 'current_number/total_number', '%p' to show the fraction of the way down the list, or font-control sequences such as %B, %U, %S and the corresponding %b, %u, %s; quotes will be added automatically. Delete the whole line to turn it off. Hit return to keep the current value. prompt> %SMatches: %l, Progress: %p%`
        - 显示效果：`Matches: [当前匹配项数量]/[总匹配项数量], Progress: [当前列表位置的百分比]%`
    - 列表无法完全显示在屏幕上时启用此功能
    - 对于只进行补全项立标的命令，也会在列表无法完全显示在屏幕上时启用此功能。
3. 允许选择列表很长时使用滚动条浏览列表并设置提示信息(关闭这个功能，保持空白即可）
    - 显示提示：prompt> %SAt %p: Hit TAB for more, or the character to insert

##### 4. 是否使用旧式的compctl配置方式

不使用旧式`compctl`方式，直接跳过不用管

#### 特定补全项的样式

##### 文件补全选项

```
      *** compinstall: options for filename completion ***

1.  Choose how to sort the displayed list of filename matches.(File Name)

2.  In expressions with .., don't include directories already implied.(p c d)

3.  Allow completion of . and .. for the bone idle.(y 允许在自动补全时包括当前目录 . 和父目录 ..)

4.  When expanding paths, `foo//bar' is treated as `foo/bar'.(y，把多于的斜杠合为一条)

5.  Configure how multiple paths are expanded and displayed, 
    e.g. /f/b -> /foo/bar
    (1. y /f/b允许匹配/foo/bar、/foo/basket，也允许匹配/foo/test/bar（关闭后则不匹配）
     2. n 匹配到不含歧义的部分，如果存在/foo/bar和/food/basket，只补全到/foo，然后进行选择/foo还是/food，而不是显示所有匹配/f/b的项，比如列出/foo/bar和/food/basket
     3. y
        对于目录/foo/bar /foo/bad /failed/bag，第三个功能如果选y 则会显示
        /foo/bar
        /foo/bad
        /failed/bag
        如果选n，则会显示
        /foo
        /foo
        /failed
     )

6.  Keep certain prefixes unchanged, such as `//resource/'.

q.  Return without saving.
0.  Done setting options for filename completion.
```

### 选择默认的编辑配置

```
Default editing configuration
=============================

The keys in the shell's line editor can be made to behave either
like Emacs or like Vi, two common Unix editors.  If you have no
experience of either, Emacs is recommended.  If you don't pick one,
the shell will try to guess based on the EDITOR environment variable.
Usually it's better to pick one explicitly.

# (1) Change default editing configuration
bindkey -e                                                                         (set but not saved)

# (0)  Remember edits and return to main menu (does not save file yet)
# (q)  Abandon edits and return to main menu

--- Type one of the keys in parentheses --- 1
Pick a keymap (set of keys) to use when editing.
Type:
  (e) for Emacs keymap (recommended unless you are vi user)
  (v) for Vi keymap
  (n) not to set a keymap (allow shell to choose)
  (k) to keep the current setting, (e):
--- Type one of the keys in parentheses --- 
```

这里我选择v，即Vi的键位设置

终于，一些基础配置和compinstall已经完善了

配置完后可以将~/.zshrc移动到~/.config/zsh/目录中方便管理，建立一个链接即可

```sh
$ mv ~/.zshrc ~/.config/zsh/zshrc
$ ln -sf ~/.config/zsh/.zshrc ~/.zshrc
```

### 命令纠正

配置功能：当输入错误命令时，询问是否为某一个命令

添加下列配置

```zsh $HOME/.zshrc
setopt correct
```

效果如下，把sudo输入成了`sudi`

```sh
$ sudi pacman -Syyu
zsh: correct 'sudi' to 'sudo' [nyae]? y
```

输入`y`以后就会执行这个命令

### 创建别名

通过`alias`指令设置别名

创建`~/.config/zsh/alias.zsh`文件进行别名的配置

```zsh $HOME/.config/zsh/alias.zsh
alias cnpm="npm --registry=https://registry.npmmirror.com --cache=$HOME/.npm/.cache/cnpm --disturl=https://npmmirror.com/mirrors/node --userconfig=$HOME/.cnpmrc"
alias e="exit"
alias ls="lsd"
alias la="ls -a"
alias ll="ls -llha"
alias gia="git add"
alias gic="git commit"
alias gicfg="git config"
```

加载配置

```zsh $HOME/.zshrc
source ~/.config/zsh/alias.zsh
```

### 问题

#### 需要按两次tab才能补全无歧义的内容

原因：

与下面这行配置后面选项的配置有关，要把`_complete`放在第一个

```zsh $HOME/.zshrc
zstyle ':completion:*' completer _complete _list _expand _ignored _match _correct _approximate
```

## Vi-Mode

在上面的[基础配置](#选择默认的编辑配置)部分，已经设置了Vi-Mode键位，现在需要设置Vi-Mode的光标，可以将Vi-Mode相关的配置从`.zshrc`移动到`vi.zsh`，方便管理

`~/.config/zsh/vi.zsh`配置如下，主要配置了Inserte Mode和Common Mode的光标显示效果：

```zsh $HOME/.config/zsh/vi.zsh
bindkey -v
function zle-keymap-select {
	if [[ ${KEYMAP} == vicmd ]] || [[ $1 = 'block' ]]; then
		echo -ne '\e[1 q'
	elif [[ ${KEYMAP} == main ]] || [[ ${KEYMAP} == viins ]] || [[ ${KEYMAP} = '' ]] || [[ $1 = 'beam' ]]; then
		echo -ne '\e[5 q'
  fi
}
zle -N zle-keymap-select

# Use beam shape cursor on startup.
echo -ne '\e[5 q'

# Use beam shape cursor for each new prompt.
preexec() {
	echo -ne '\e[5 q'
}

_fix_cursor() {
	echo -ne '\e[5 q'
}
precmd_functions+=(_fix_cursor)

KEYTIMEOUT=1
```

加载配置

```zsh $HOME/.zshrc
source ~/.config/zsh/vi.zsh
```

### 键位配置

键位配置的语法如下：

```zsh
# Set keymaps for Vi-Mode
bindkey -M vicmd keys action
# 'keys' is the key combination to bind, and 'action' is the corresponding action or command.
```

具体可以查看 [Zsh Line Editor - Movements](https://zsh.sourceforge.io/Doc/Release/Zsh-Line-Editor.html#Movement)

例如

```zsh
# Map 'k' to backward-char
bindkey -M vicmd 'k' backward-char
```

## 输入历史记录

将上下方向键绑定到历史记录搜索功能。按上下方向键时，zsh 将会显示与当前输入行匹配的过去命令，直到当前光标位置为止。

```zsh $HOME/.zshrc
autoload -Uz up-line-or-beginning-search down-line-or-beginning-search
zle -N up-line-or-beginning-search
zle -N down-line-or-beginning-search

[[ -n "${key[Up]}"   ]] && bindkey -- "${key[Up]}"   up-line-or-beginning-search
[[ -n "${key[Down]}" ]] && bindkey -- "${key[Down]}" down-line-or-beginning-search
```

## 插件

### 主题插件

这次选择了[Powerlevel10k](https://github.com/romkatv/powerlevel10k)主题，通过手动方式安装

主题样式如下：
<img src="https://raw.githubusercontent.com/romkatv/powerlevel10k-media/master/prompt-styles-high-contrast.png" style="max-width:50%">

安装方法：

```sh
$ git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ~/.config/zsh/plugins/powerlevel10k
$ echo 'source ~/.config/zsh/plugins/powerlevel10k/powerlevel10k.zsh-theme' >>~/.zshrc
```

如果速度慢可以使用Gitee上的官方镜像进行加速

```sh
$ git clone --depth=1 https://gitee.com/romkatv/powerlevel10k.git ~/.config/zsh/plugins/powerlevel10k
$ echo 'source ~/.config/zsh/plugins/powerlevel10k/powerlevel10k.zsh-theme' >>~/.zshrc
```

在终端中输入`source ~/.zshrc`，就会出现初始配置提示，按照提示进行操作

将配置文件移动到`~/.config/zsh`中方便管理，创建链接

```sh
$ mv ~/.p10k.zsh ~/.config/zsh/
$ ln -sf ~/.config/zsh/.p10k.zsh ~/.p10k.zsh
```

配置效果如下：

<img src="P10K.png" style="max-width:50%">

### 命令补全

在[基础配置](#配置补全系统)部分已经配置了大部分的补全功能，而这个插件提供了补全的扩展功能

**ghost text提示和补全**

安装[zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)插件实现ghost text提示和补全

```sh
$ git clone https://github.com/zsh-users/zsh-autosuggestions ~/.config/zsh/plugins/zsh-autosuggestions
```
加载插件

```zsh $HOME/.zshrc
source ~/.config/zsh/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh
```

在输入命令时，会在光标的后面以柔和的灰色显示补全，可以在配置中用`ZSH_AUTOSUGGEST_HIGHLIGHT_STYLE`变量来更改颜色，如果按右方向键`⇒`或`end`键，会补全灰色部分的内容。

### 命令行高亮显示

通过插件[zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting)实现命令行的高亮效果

```sh
$ git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ~/.config/zsh/plugins/zsh-syntax-highlighting
```

加载插件

```zsh $HOME/.zshrc
source ~/.config/zsh/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
```

**注意**

这个插件好像跟[列表颜色的配置](#section1)有冲突，建议把颜色选项关闭，`zshrc`对应的配置如下，在前面添加`#`注释掉该配置
```zsh $HOME/.zshrc
zstyle ':completion:*' list-colors ''
```

效果如下：
<img src="https://raw.githubusercontent.com/zsh-users/zsh-syntax-highlighting/master/images/after2.png" style="max-width:50%">

### 提示显示别名

接下来这个插件在你输入了一个命令，而这个命令有对应存在的别名时，提醒你应当去使用别名

[You Should Use](https://github.com/MichaelAquilina/zsh-you-should-use) 

```sh
$ git clone https://github.com/MichaelAquilina/zsh-you-should-use.git ~/.config/zsh/plugins/zsh-you-should-use
```

加载插件

```zsh $HOME/.zshrc
source ~/.config/zsh/plugins/zsh-you-should-use/you-should-use.plugin.zsh
```
<img src="https://raw.githubusercontent.com/MichaelAquilina/zsh-you-should-use/master/img/alias.png" style="max-width:50%">

## 显示效果

<img src="zsh-screenshot.png" style="max-width:50%">
