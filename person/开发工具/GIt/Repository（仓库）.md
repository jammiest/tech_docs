# Repository 仓库

## 应用场景一

想要提交到多个远程仓库，如分别提交 Github 和公司的服务器上

## 1、创建空仓库

> 如果公司代码仓库已经有提交，可以不用创建为新的仓库

```shell
git init --bare
```

假设公司仓库地址为：`git@git.your-company.com:you_code.git`

## 2、克隆仓库到本地

```shell
git clone git@git.your-company.com:you_code.git
```

此时可在本地利用`git remote`或`git remote -v`查看仓库信息

## 3、在 Github 上创建一个空仓库

假设创建后的地址为 `https://github.com/jammiest/tech_docs.git`

## 4、添加 Github 远程仓库及配置信息

```shell
git remote add github https://github.com/jammiest/tech_docs.git
```

此时可在本地利用`git remote`或`git remote -v`查看仓库最新信息

## 5、推送本地代码到GitHub仓库

```shell
git push -u github master
```

## 6、此时本地Git仓库对应关系为

- `orgin` -> 公司
- `github` -> GitHub

> 之后的推送以及仓库只要指定好远程的仓库名就可以完成想要的操作了，如

- 拉取GitHub的master代码到本地分支: `git pull github master`
- 推送代码本地代码到公司仓库：`git push origin`
- 通过Github的dev分支创建一个本地分支：`git checkout -b dev github/dev`

## 相关操作

`git remote` 或 `git remote -v` ：查看仓库情况

`git remote add ...` : 创建一个远程仓库

`git remote set-url ...` : 设置某个远程仓库地的地址
