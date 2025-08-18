# GitFlow

?> 参考 [Git之GitFlow工作流 | Gitflow Workflow（万字整理，已是最详）](https://blog.csdn.net/sunyctf/article/details/130587970)

## 常用分支说

| 分支名称   | 分支说明                                                                                                       |
| ---------- | -------------------------------------------------------------------------------------------------------------- |
| Production | 生产分支，即 Master分支。只能从其他分支合并，不能直接修改                                                      |
| Release    | 发布分支，基于 Develop 分支创建，待发布完成后合并到 Develop 和 Production 分支去                               |
| Develop    | 主开发分支，包含所有要发布到下一个 Release 的代码，该分支主要合并其他分支内容                                  |
| Feature    | 新功能分支，基于 Develop 分支创建，开发新功能，待开发完毕合并至 Develop 分支                                   |
| Hotfix     | 修复分支，基于 Production 分支创建，待修复完成后合并到 Develop 和 Production 分支去，同时在 Master 上打一个tag |