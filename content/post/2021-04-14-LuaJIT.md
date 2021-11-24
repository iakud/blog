---
categories:
- program
date: "2021-04-14T18:34:31Z"
title: LuaJIT和Sublime Text配置
typora-root-url: ..
---

简单介绍LuaJIT和Sublime Text开发环境的配置。

<!--more-->

#### LuaJIT配置

```bash
tar zxf LuaJIT-XX.YY.ZZ.tar.gz
cd LuaJIT-XX.YY.ZZ
make
sudo make install
```

**macOS**

```bash
MACOSX_DEPLOYMENT_TARGET=XX.YY make
```

**symlink**

```bash
sudo ln -sf luajit-XX.YY.ZZ /usr/local/bin/luajit
```

#### Sublime配置

在`Tool->Build System->New Build System`中编辑：

```json
{
	"cmd": ["luajit", "$file"],
	"file_regex": "^(?:lua:)?[\t](...*?):([0-9]*):?([0-9]*)",
	"selector": "source.lua"
}
```

保存`LuaJIT.sublime-build`在默认路径即可。

在`Tool->build System`，选择`LuaJIT`为默认，`Ctrl+B`即可运行。