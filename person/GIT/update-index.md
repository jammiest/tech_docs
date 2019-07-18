# update-index

> 该指令主要是跟gitignore文件互补:对已经跟踪的文件的忽略和跟踪做判断

```shell
# 忽略已经添加到索引树的指定文件改动，即改动不会被提交
git update-index --assume-unchanged
# 查找那些已经被添加忽略跟踪的文件（win命令行）
git ls-files -v | findstr ^h
# 重新跟踪指定文件的变化，及本地改动会被提交
git update-index --no-assume-unchanged
```