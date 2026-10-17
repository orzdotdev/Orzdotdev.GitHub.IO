---
title: 博客日常更新以及更换设备之后的操作
tags:
  - hexo
  - 博客
top: false
copyright: true
date: 2025-06-04 02:12:36
updated:
---

<!-- 正文从此处开始 -->

## 一、账号管理

有时候要负责Git多个账号的登录，或者多平台的操作，如github.com 或者gitee.com 下面是解决方案!

## 二、指定位置生成密钥

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"   -f ~/.ssh/user_id_rsa
```



| 参数                          | 含义                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| `-t ed25519`                  | 使用 Ed25519 算法生成密钥（比 RSA 更快更安全）               |
| `-C "your_email@example.com"` | 添加注释，通常用于标识该密钥（如邮箱或用途）                 |
| `-f ~/.ssh/user_id_rsa`       | 指定密钥文件保存位置（不加 `.pub`，公钥会自动生成 `.pub` 文件） |

生成结果：
私钥：~/.ssh/user_github_id_rsa

公钥：~/.ssh/user_github_id_rsa.pub

这已经可以满足单平台了

## 三、多平台

### 密钥生成管理

可以通过文件夹方式来管理密钥如 github 以及gitee，在生成密钥是时候  用-f 参数来指定生成的文件夹地址&名字

github

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"   -f ~/.ssh/github/user1_id_rsa
ssh-keygen -t ed25519 -C "your_email@example.com"   -f ~/.ssh/github/user2_id_rsa
```

gitee

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"   -f ~/.ssh/gitee/user1_id_rsa
ssh-keygen -t ed25519 -C "your_email@example.com"   -f ~/.ssh/gitee/user2_id_rsa
```
<!--more-->


### 本地配置

密钥生成了，就要通过config文件来区分账户跟平台了

1.创建config文件

```bash
cd ~/.ssh  #进入.ssh根目录
touch config #创建config配置文件
```

2.配置config文件

```bash
# [github.com]

## 第一个账号，默认使用的账号
Host github.com
 HostName github.com
 User git
 IdentityFile ~/.ssh/github/user1_id_rsa
## 第二个账号 your_name
Host your_name.github.com # your_name为前缀名，可以任意设置
 HostName github.com
 User git
 IdentityFile ~/.ssh/github/user2_id_rsa
 
# [gitee.com] 

## 第一个账号，默认使用的账号
Host gitee.com
 HostName gitee.com
 User git
 IdentityFile ~/.ssh/gitee/user1_id_rsa
## 第二个账号 your_name
Host your_name.gitee.com # your_name为前缀名，可以任意设置
 HostName gitee.com
 User git
 IdentityFile ~/.ssh/gitee/user2_id_rsa
 

 
```

> 原理分析:
>
> 1.ssh 客户端是通过类似 git@github.com:githubUserName/repo.git 的地址来识别使用本地的哪个私钥的， 地址中的 User 是@前面的git， Host 是@后面的github.com。
>
> 2.如果所有账号的 User 和 Host 都为 git 和 github.com，那么就只能使用一个私钥。 所以要对User 和 Host 进行配置，让每个账号使用自己的 Host，每个 Host 的域名做 CNAME 解析到 github.com， 如上面配置中的Host your_name.github.com。
>
> 3.配置了别名之后，新的地址就是git@your_name.github.com:githubUserName/repo.git。 这样 ssh 在连接时就可以区别不同的账号了。
>
> 4.其他平台也是同样理解

3.验证配置

输入指令,验证配置是否成功，以github为例，其他平台一样的道理

3.1 默认ssh_key验证

```bash
ssh -T git@github.com
```

3.2 新ssh_key验证, 新秘钥名替换your_name

```bash
ssh -T git@your_name.github.com
```

(注意)如果提示如下信息, 则配置成功

Hi your_name! You've successfully authenticated, but GitHub does not provid

## 四、项目配置

1.设置单独用户名以及邮箱

```bash
# 取消全局 用户名/邮箱 配置
git config  --global --unset user.name
git config  --global --unset user.email

# 单独设置每个repo 用户名/邮箱,进入到.git文件夹
git config user.name "your_name"  //在config后加上--global即全局
git config user.email "your_email@youremail.com"
```


2.远程仓库操作

2.1 绑定远程仓库(默认)

```bash
git remote add origin git@github.com:orzdotdev/Orzdotdev.GitHub.IO.git.git
```

2.2 绑定多git用户仓库地址

```bash
git remote add origin git@your_name.github.com:githubusername/repo.git
```

2.3 更换远程仓库（按需求）

```bash
git remote set-url origin git@github.com:githubusername/another-repo.git
```

2.4.查看远程仓库地址

```bash
git remote -v
```

2.5 删除仓库地址

```bash
git remote remove origin
```

2.6 推送到远程仓库（区分好main/master）

```bash
git push -u origin main
```

2.7 查看当前项目配置

```bash
git config --list
```
## 五、hexo搭建的流程

1. 创建仓库，[https://Orzdotdev.GitHub.IO.git](https://Orzdotdev.GitHub.IO.git/)；
2. 创建两个分支：master 与 orz-dev；
3. 设置orz-dev为默认分支（因为我们只需要手动管理这个分支上的Hexo网站文件）；
4. 使用git clone git@github.com:orzdotdev/Orzdotdev.GitHub.IO.git 拷贝仓库；
5. 在本地[https://Orzdotdev.GitHub.IO.git](https://Orzdotdev.GitHub.IO.git/)文件夹下通过Git bash依次执行npm install hexo、hexo init、npm install 和 npm install hexo-deployer-git（此时当前分支应显示为orz-dev,在执行hexo init时候尽量先把文件夹内容copy出来,否则容易清空.git文件）;
6. 修改_config.yml中的deploy参数，分支应为master；
7. 依次执行git add .、git commit -m “…“、git push origin hexo提交网站相关的文件；
8. 执行hexo g -d生成网站并部署到GitHub上。

这样一来，在GitHub上的[https://Orzdotdev.GitHub.IO.git](https://Orzdotdev.GitHub.IO.git/)仓库就有两个分支，一个orz-dev分支用来存放网站的原始文件，一个master分支用来存放生成的静态网页。完美( •̀ ω •́ )y！

## 六、日常的改动流程

在本地对博客进行修改（添加新博文、修改样式等等）后，通过下面的流程进行管理。

1. 依次执行git add .、git commit -m “…“、git push origin hexo指令将改动推送到GitHub（此时当前分支应为orz-dev）；
2. 然后才执行hexo g -d发布网站到master分支上。

虽然两个过程顺序调转一般不会有问题，不过逻辑上这样的顺序是绝对没问题的（例如突然死机要重装了，悲催….的情况，调转顺序就有问题了）。

## 七、本地资料丢失后的流程

当重装电脑之后，或者想在其他电脑上修改博客，可以使用下列步骤：
注意： 切换npm到国内淘宝镜像，安装依赖
npm config set registry [https://registry.npmmirror.com](https://registry.npmmirror.com)
npm install

1. 使用git clone git@github.com:orzdotdev/Orzdotdev.GitHub.IO.git拷贝仓库（默认分支为orz-dev）；
2. 在本地新拷贝的[https://Orzdotdev.GitHub.IO.git](https://Orzdotdev.GitHub.IO.git/)文件夹下通过Git bash依次执行下列指令：npm install hexo、npm install、npm install hexo-deployer-git（记得，不需要hexo init这条指令）。

## 八、其他(hexo语法)

**（1）`$ hexo g`或`$ hexo generate`** 该命令执行后在`hexo`站点根目录下生成`public`文件夹

**（2）`$ hexo clean`** 把项目中`public`的缓存文件删除

**（3）`$ hexo s`或`$ hexo server`** 启动服务预览

**（4）`$ hexo d`或`$ hexo deploy`** 部署站点，在本地生成`.deploy_git`文件夹，并将编译后的文件上传至 GitHub。

**（5）`$ hexo new [layout] <title>`** 例如：`$ hexo new photo “my-first-blog”` 上述指令执行时，Hexo 会尝试在 scaffolds 中寻找photo.md布局，若找到，则根据该布局新建文章；若未找到或指令中未指定该参数，则使用post.md新建文章。新建文章的名称在_config.yml中配置。

**（6）删除文章** 只需在本地把`source/_posts`文件夹下的文章源文件删除后，执行以下命令重新部署即可。


## git高级操作 main与master之间的切换（按需）

自此github 仓库默认分支改成main 之后，之前的老项目推送肯定有问题 ，会遇到本地分支名称跟仓库主分支名称不一致的情况

1.本地分支名称跟仓库分支名称一致

即情景A : 本地是master/仓库是master 或者 情景B: 本地是main / 仓库是main

```bash
git push -u origin master
```

或者

```bash
git push -u origin main
```

然后执行

```bash
git push
```

2.修改当前分支的默认上游（即重新绑定远程分支）

如果你想切换当前本地分支（比如 `main`）的推送目标为远程的 `master`：

```bash
git branch --set-upstream-to=origin/master main
```

或者

```bash
git push -u origin master  # 如果你本地当前分支是 master
```

3.修改远程默认分支（GitHub/Gitee）

远程仓库的默认分支（如 main 或 master）是由平台（GitHub、Gitee）设置的，可以通过网页端手动更改：

GitHub：进入仓库 → Settings → Branches → Default branch

Gitee：进入仓库 → 管理 → 默认分支设置

4.本地分支改名（main <-> master）

```bash
# master 改名为 main
git branch -m master main

# 然后推送新分支并设置 upstream
git push -u origin main

# 可选：删除旧远程 master 分支（慎用）
git push origin --delete master
```

命令汇总

| 目的                           | 命令                                       |
| ------------------------------ | ------------------------------------------ |
| 推送当前分支并设置远程跟踪     | `git push -u origin main`                  |
| 更换当前分支绑定的远程分支     | `git branch --set-upstream-to=origin/main` |
| 查看所有分支及其跟踪信息       | `git branch -vv`                           |
| 修改本地分支名                 | `git branch -m old new`                    |
| 修改远程默认分支（网页上操作） | GitHub/Gitee 设置里更改                    |

## git高级操作 多账户/平台同步（按需）

### 配置文件

1.查看远程仓库

```bash
git remote -v
```

2.先删除已关联的名为origin的远程库：

```bash
git remote rm origin
```

3.然后，再关联GitHub的远程库：

```
git remote add github git@github.com:your_name/仓库名称.git     #your_name改成你的用户名
```

4.接着，再关联gitee的远程库：

```
git remote add gitee git@gitee.com:your_name/仓库名称.git     #your_name改成你的用户名
```

5.查看仓库配置 

```
git remote -v
```


gitee   git@gitee.com:your_name/仓库名称.git (fetch)

gitee   git@gitee.com:your_name/仓库名称.git (push)

github  git@github.com:your_name/仓库名称.git (fetch)

github  git@github.com:your_name/仓库名称.git (push)

如果出现以上结果说明配置成功

### 仓库更新

1.上传代码

```bash
git add .
git commit -m "update"
```

提交到github

```bash
git push github master
```

提交到码云

```bash
git push gitee master
```

2.更新代码

```bash
# 从github拉取更新
git pull github

# 从gitee拉取更新
git pull gitee
```
** 踩到的坑**

上述过程中，更新或提交代码时可能会遇到fatal:refusing to merge unrelated histories (拒绝合并无关的历史) 错误，解决办法：

首先将远程仓库和本地仓库关联起来。

```bash
git branch --set-upstream-to=origin/remote_branch  your_branch
```

其中，origin/remote_branch是你本地分支对应的远程分支，your_branch是你当前的本地分支。

然后使用git pull整合远程仓库和本地仓库。

git pull --allow-unrelated-histories    (忽略版本不同造成的影响)
重新更新、提交即可。

注： 如遇到 Git没有共同祖先的两个分支合并 的情形请自行查询！

