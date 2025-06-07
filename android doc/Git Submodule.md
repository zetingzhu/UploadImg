

ZZTUtilCode

https://github.com/zetingzhu/ZZTUtilCode.git

1. 创建目录
$ mkdir zt-util
$ cd zt-util

2. 以submodule的方式注入到项目中
$ git submodule add https://github.com/zetingzhu/ZZTUtilCode.git
$ git submodule add https://github.com/zetingzhu/ZZTSubmodul.git

添加项目关联到当前project
include(':ztTools')
project(':ztTools').projectDir = new File("zt-u1/ZZTUtilCode/zztToolsLibrary")


# 创建
cd MainProject # 主模块
git submodule add  $submodule_url $submodule_dir  #添加子模块
git add .
git commit -m "add submodule" # 提交修改
git push origin master # 推送到主项目远程


3. 克隆跟新
1)git submodule update --init --recursive
2)git pull
3)git submodule update

# 克隆
git clone $main_project_url
cd MainProject # 进入主项目，发现submodule_dir是一个空目录
git submodule init # 初始化submodule配置，从.gitmodule读取相关配置
git submodule update # 拉取submodule代码到submodule_dir目录 #git submodule update --remote --merge 下面会讲到
# 上面2行代码可以用下面这1行代替：
git submodule update --init # 如果子模块还包含子模块，可以加--recursive


# 更新
cd MainProject
git pull # 主模块远程更新合并到本地。
git submodule update --remote --merge #子模块远程更新合并到本地
# 或者
cd $submodule_dir; git fetch; git merge origin/master # 拉取子模块更新


4. 查看当前submodule 版本提交信息
$ git submodule status

5. 更新最新版本
$ git submodule update zt-util/ZZTUtilCode


6. 卸载删除 submodule
a) 卸载 submodule
$ git submodule deinit -f ztSub1
b) 删除目录
$ git rm -r -f ztSub1
c)
删除 .gitmodules 文件
d）
删除 .git/modules 中的submodule目录
