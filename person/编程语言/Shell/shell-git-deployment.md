# Shell脚本示例-部署git代码

> 该脚本是一个简单的部署代码的示例，执行命令`/bin/bash auto-git-pull.sh my_git_project 1> /var/www/auto-git-pull.log`

```shell
#!/bin/bash
echo ""
# 输出当前时间
date --date='0 days ago' "+%Y-%m-%d %H:%M:%S"
echo "-------开始-------"
# 项目名参数是否存在
if [ ! -n "$1" ];
then 
          echo "param参数错误"
          echo "End"
          exit
fi
# 要部署代码的位置
gitPath="/var/www/$1"
echo "部署代码路径：$gitPath"

# 你的项目的git仓库地址
gitHttp="git@gitsrever/$1.git"
echo "拉取的仓库地址：$gitHttp"

# 判断项目路径是否存在
if [ -d "$gitPath" ]; then
        cd $gitPath
        # 判断是否存在git目录
        if [ ! -d ".git" ]; then
                echo "在$gitPath下克隆仓库$gitHttp"
                git clone $gitHttp gittemp
                mv gittemp/.git .
                rm -rf gittemp
        fi
        # 拉取最新的项目文件
        git reset --hard origin/master
        # git clean -f
        git pull origin master
        echo "拉取完成"
        # 执行npm
        # 执行编译
        # npm run build
        # 设置目录权限
        echo "设置$gitPath目录权限www:www"
        chown -R www:www $gitPath
        echo "-------结束--------"
        exit
else
        echo "创建目录"
        sudo mkdir $gitPath
        cd $gitPath
        echo "在$gitPath下克隆仓库$gitHttp"
        sudo git clone $gitHttp gittemp
        cd gittemp
        echo "切换master分支"
        sudo git checkout master
        cd ..
        echo "移动+删除文件夹"
        sudo mv gittemp/* .
        sudo mv gittemp/.[^.]* .
        sudo rm -rf gittemp
        # echo "配置文件"
        # sudo cp .env.test .env
        # 执行npm
        # 执行编译
        # npm run build
        # 设置目录权限
        echo "设置$gitPath目录权限www:www"
        sudo chown -R www:www $gitPath
        echo "End"
        exit
fi
```
