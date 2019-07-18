# 子模块

参考：[git-scm - 子模块]

## 本地初始化子模块

```git
git submodule update --init --recursive
```

## 从远程克隆创建子模块

```git
git submodule add https://github.com/PrismJS/prism.git asset/PrismJS
```

## 删除子模块

步骤1、删除子模块需要将子模块的跟踪文件全部从索引树中删除  

```git
git rm -rf asset/PrismJS
```

步骤2、删除该子模块的信息

```shell
# Linux
rm -rf .git/modules/asset/PrismJS
# Windfow
mkdir .git/modules/asset/PrismJS
```

[git-scm - 子模块]:https://git-scm.com/book/zh/v2/Git-工具-子模块