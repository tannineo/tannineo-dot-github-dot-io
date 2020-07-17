---
title: Perl项目管理
date: 2020-07-17
categories:
  - programming
tags:
  - perl
  - dependency
  - project
  - tool
top_image:
top: false
math: false
---

文章参考: [A gentle introduction to Perl-dependency management](http://christopher.rasch-olsen.no/perl-dependency-management), 参考的内容可能过时, 所以会有所出入.

本肥肥手头有一个别人给的`perl`代码, 利用它来完成自己的课题. 在尝试理解程序的情况下我在搬运代码到自己新建的项目里. 过程中碰到了一些问题让我注意到一条隔离`perl`项目依赖的工具链是有必要的.

<!-- more -->

如果是使用`node`尝试开发的同学一定会有所体会, `node_modules`的依赖版本控制冗余(node_modules blackhole)但是有效, 配合`nvm`也能锁定`node`运行时的版本. 而在`python`下进行开发, 基于`virtualenv`的沙箱环境配合一个配置文件(`environment.yml`之于`conda`, `Pipfile`之于`pipenv`, 或者无沙箱的`requirements.txt`之于`pip`), 可以很好的组织项目而互不影响. 类似的, `go`的`module`模式(`go.mod`)在解决依赖问题. 管理`java`的`jdk`版本有`jenv`这个工具.

那么对于`perl`呢?

- [需要解决的问题](#需要解决的问题)
- [使用`perlbrew`管理`perl`版本](#使用perlbrew管理perl版本)
- [使用`perlbrew`隔离项目依赖(沙箱)](#使用perlbrew隔离项目依赖沙箱)
- [使用`carton`和`cpanfile`管理依赖](#使用carton和cpanfile管理依赖)
- [总结](#总结)
- [之后](#之后)
  - [调整项目结构](#调整项目结构)
  - [背后的原理](#背后的原理)
  - [OSX下Homebrew和`perlbrew`的兼容问题](#osx下homebrew和perlbrew的兼容问题)
  - [并不是唯一方案 / 代码](#并不是唯一方案--代码)

## 需要解决的问题

我们需要解决的问题有:

- 固定`perl`的版本, 提供一个沙箱环境.
  - 尽可能不去使用**可能老旧的**系统自带`perl`, 也尽可能不对依赖于自带`perl`的系统组件造成影响.
- 隔离不同项目间的依赖.
  - 对于同一个包, 它们可能使用了不同版本.
- 锁定依赖的版本.
  - 新版本的依赖接口行为可能发生变化, 或者新版本可能会引入bug.

本肥肥会尝试使用[`perlbrew`](https://perlbrew.pl/)和官方的工具[`cpan`](https://www.cpan.org/)来解决上述问题. 之前本肥肥使用的是[`plenv`](https://github.com/tokuhirom/plenv), 某些地方也会有比较. 所有操作在OSX下进行.

## 使用`perlbrew`管理`perl`版本

遵照`perlbrew`[官网](https://perlbrew.pl/)的说明:

```text
\curl -L https://install.perlbrew.pl | bash
```

`\`在交互式的shell中禁止了alias, 直接运行`curl`的可执行程序. 替换为`| zsh`会使得安装失败.

提示在`~/.profile`加入`source ~/perl5/perlbrew/etc/bashrc`. 但是`zsh`默认不会执行`~/.profile`里的内容, 所以我们把`source ~/perl5/perlbrew/etc/bashrc`加入自己的`~/.zshrc`.

`perlbrew`查看用法, `perlbrew available`查看可安装版本. 本肥肥编辑时是`perl-5.32.0`.

```shell
perlbrew install perl-5.32.0
```

与`plenv`相同, `perlbrew`也是下载源码在本机编译, 需要等候一段时间.

`perlbrew switch perl-5.32.0`将这个版本设为登录后默认版本, `perlbrew use`加版本则会在当前session切换.

~~其实在日常使用`nvm`或者类似工具的时候, 都是偶尔想到了就升级到最新版本, 除非碰到问题才会使用某一特定版本或者降级,.可以把这些工具看作是快速安装运行时的工具.~~

## 使用`perlbrew`隔离项目依赖(沙箱)

对于某个有名字的项目, 比如`code`, 我们创建一个专门的环境用来保存依赖的lib:

```shell
perlbrew lib create code

perlbrew use perl-5.32.0@code
```

命名规则是`perl版本@名称`, 缺省`@`和之前的内容会使用当前环境的版本. 在这个环境下就可以进行开发了, 可以考虑在项目根目录写一个脚本来记忆和方便运行.

`perlbrew lib`查看具体用法.

`plenv`并没有自带项目依赖隔离, 这一功能通过插件[`plenv-contrib`](https://github.com/miyagawa/plenv-contrib)达成, 不是特别直接. [`local::lib`](https://metacpan.org/pod/local::lib)可能是这一功能更原始的实现.

输入下面的命令可以查看`@INC`(`perl`搜索依赖的路径):

```shell
perl -e "print qq(@INC)"
```

## 使用`carton`和`cpanfile`管理依赖

相比于沙箱, 一个更直接的办法是将依赖全部下载到在项目中, 然后在编写时include.

[`carton`](https://metacpan.org/pod/Carton)是能达成我们目标的工具.

比如我们想尝试这个`perl`的服务器框架[`Dancer`](http://perldancer.org/), `test.pl`:

```perl
#!/usr/bin/env perl

use strict;
use warnings;

use Dancer2;

get '/' => sub {
  "Hello World!";
};

dance;

1;
```

我们尝试运行(`perl test.pl`)会发现:

```text
Can't locate Dancer2.pm in @INC ...
```

新建`cpanfile`:

```text
# cpanfile
requires 'Dancer2', '0.300004';
```

接下来:

- 安装`carton`: `cpan Carton`.
- 安装依赖: `carton install`, 它会读取`./cpanfile`, 安装依赖并且声称快照文件`cpanfile.snapshot`.

运行, 可以:

- 使用`carton`运行命令: `carton exec -- perl test.pl`.
  - `--`能防止`carton`读取`exec`之后的一些配置flag.
- 在`test.pl`中, 加入`use lib "$FindBin::Bin/local/lib/perl5";`.
  - `FindBin::Bin`确认执行时脚本的路径.
  - 这样我们可以用沙箱环境的`perl`直接运行: `perl test.pl`.

还有其他的一些工具(都出自一个作者[miyagawa](https://github.com/miyagawa)):

- `cpanm` (替代`cpan`)
- `carmel` (实验性, 替代`carton`)

## 总结

1. `perlbrew`管理`perl`版本
2. `perlbrew`创建沙箱(类似`conda`) ~~可选~~
3. 编写`cpanfile`管理依赖版本, 使用`carton`安装到`./local/`并生成快照
4. `use lib "$FindBin::Bin/../local/lib/perl5";` ~~可选~~

## 之后

### 调整项目结构

我们可以调整一下目录结构, 更符合规范. 注意调整`use lib`的路径.

```text
.
├── README.md
├── bin
│   └── test.pl
├── cpanfile
├── cpanfile.snapshot
├── lib                 # use lib
│   └── TestLib
│       └── Tests.pm
└── local
    ├── bin
    ├── cache
    ├── lib             # use lib
    └── man
```

我们把`"Hello World!"`写成`sub`并且装进了自己的module里, 在`./lib/TestLib/Tests.pm`中:

```perl
package TestLib::Tests;

use Exporter;

@ISA = qw/Exporter/;

@EXPORT_OK = qw/hello/;    # use时必须显式声明hello

# @EXPORT    = qw/hello/;  # use时无需声明hello

sub hello {
  return "Hello World!";
}

1;

```

最后的`test.pl`:

```perl
#!/usr/bin/env perl

use strict;
use warnings;

use FindBin;
use lib "$FindBin::Bin/../local/lib/perl5";
use lib "$FindBin::Bin/../lib/";

use Dancer2;

use TestLib::Tests qw/hello/;

get '/' => sub {
  hello();
};

dance;

1;
```

### 背后的原理

大概率都是基于shell的环境变量更改.

之后需要详细了解`perl`, 或者说`perl5`的一些基于环境变量的路径配置.


### OSX下Homebrew和`perlbrew`的兼容问题

问题主要是Homebrew不会使用`perlbrew`所安装的`perl`, 若是完全交给Homebrew管理, 开发中也不会涉及Homebrew所安装的组件的话, 这个问题放置也没关系.

具体请查看这个[github wiki](https://github.com/gugod/App-perlbrew/wiki/Deploying-Perl-bindings-from-tools-installed-with-Homebrew-on-OS-X)

### 并不是唯一方案 / 代码

A Perl programming motto.

> TIMTOWTDI = There Is More Than One Way To Do It

See wiki page: [TIMTOWTDI](https://en.wikipedia.org/wiki/There%27s_more_than_one_way_to_do_it)
