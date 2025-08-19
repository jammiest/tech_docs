# anaconda使用指南

官方参考文档：<https://conda.io/projects/conda/en/latest/commands.html#conda-vs-pip-vs-virtualenv-commands>

## 使用对比

> conda vs. Pip vs. Virtualenv

| 操作                 | Conda包和环境管理工具                                 | Pip包管理命令                                                         | Virtualenv环境管理命令                                |
| -------------------- | ----------------------------------------------------- | --------------------------------------------------------------------- | ----------------------------------------------------- |
| 安装包               | `conda install $PACKAGE_NAME`                         | `pip install $PACKAGE_NAME`                                           | X                                                     |
| 更新一个包           | `conda update --name $ENVIRONMENT_NAME $PACKAGE_NAME` | `pip install --upgrade $PACKAGE_NAME`                                 | X                                                     |
| 更新包管理工具       | `conda update conda`                                  | Linux/macOS: `pip install -U pip` Win: `python -m pip install -U pip` | X                                                     |
| 卸载包               | `conda remove --name $ENVIRONMENT_NAME $PACKAGE_NAME` | `pip uninstall $PACKAGE_NAME`                                         | X                                                     |
| 创建一个环境         | `conda create --name $ENVIRONMENT_NAME python`        | X                                                                     | `cd $ENV_BASE_DIR; virtualenv $ENVIRONMENT_NAME`      |
| 激活一个环境         | `conda activate $ENVIRONMENT_NAME`*                   | X                                                                     | `source $ENV_BASE_DIR/$ENVIRONMENT_NAME/bin/activate` |
| 取消激活一个环境     | `source deactivate`                                   | X                                                                     | `deactivate`                                          |
| 查找可用的包         | `conda search $SEARCH_TERM`                           | `pip search $SEARCH_TERM`                                             | X                                                     |
| 从特定的源安装包     | `conda install --channel $URL $PACKAGE_NAME`          | `pip install --index-url $URL $PACKAGE_NAME`                          | X                                                     |
| 列出已经安装的包     | `conda list --name $ENVIRONMENT_NAME`                 | `pip list`                                                            | X                                                     |
| 创建requirements文件 | `conda list --export`                                 | `pip freeze`                                                          | X                                                     |
| 列出所有环境         | `conda info --envs`                                   | X                                                                     | Install virtualenv wrapper, then `lsvirtualenv`       |
| 安装其他包管理工具   | `conda install pip`                                   | `pip install conda`                                                   | X                                                     |
| 安装python           | `conda install python=x.x`                            | X                                                                     | X                                                     |
| 更新python           | `conda update python`*                                | X                                                                     | X                                                     |