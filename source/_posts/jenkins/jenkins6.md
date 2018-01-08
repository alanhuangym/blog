---
title: Jenkins学习笔记6-搭建gitlabs仓库&Gtilab实现代码质量管理（提交到代码仓库前）
date: 2017/12/23
tags: 
- jenkins
- git
categories:
- jenkins
---

### 1. 搭建gitlab仓库

##### 1.1 安装前准备工作

由于需要实现代码仓库的质量管理操作，所以我们需要搭建一个gitlab的仓库。

> GitLab是一个利用 `Ruby on Rails` 开发的开源应用程序，实现一个自托管的Git项目仓库，可通过Web界面进行访问公开的或者私人项目。 

> 　　GitLab拥有与Github类似的功能，能够浏览源代码，管理缺陷和注释。可以管理团队对仓库的访问，它非常易于浏览提交过的版本并提供一个文件历史库。它还提供一个代码片段收集功能可以轻松实现代码复用，便于日后有需要的时候进行查找。

在[之前的教程](http://pirrla.cn/2017/12/21/jenkins/jenkins4/)里，我们已经装好了docker，现在有了docker，所以搭建运行环境都比较方便，我们这里直接使用docker进行gitlab的搭建。

使用以下命令拉取所需要的docker镜像

```
docker pull sameersbn/redis
docker pull sameersbn/postgresql
docker pull sameersbn/gitlab
```

这里由于docker的官方镜像服务器在国外，所以访问速度比较慢。我们可以选择使用阿里云的镜像服务器进行加速。

http://www.cnblogs.com/anliven/p/6218741.html

##### 1.2 PostgreSQL介绍

docker运行命令：

```shell
docker run --name postgresql -d \
-e 'DB_NAME=gitlabhq_production' \
-e 'DB_USER=gitlab' \
-e 'DB_PASS=password' \
-e 'DB_EXTENSION=pg_trgm' \
-v /Users/alan/postgresql/data:/var/lib/postgresql \
sameersbn/postgresql
```

##### 1.3 Redis介绍

docker运行命令：

```shell
docker run --name redis -d \
-v /Users/alan/redis/data:/var/lib/redis \
sameersbn/redis
```

##### 1.4 搭建Gitlab

当我们将PostgreSQL和redis跑起来之后，我们就可以进行gitlab的启动了。

docker运行命令：

```
docker run --name gitlab -d \
--link postgresql:postgresql --link redis:redisio \
-p 10022:22 -p 10080:80 \
-e 'GITLAB_PORT=10080' \
-e 'GITLAB_SSH_PORT=10022' \
-e 'GITLAB_SECRETS_DB_KEY_BASE=long-and-random-alpha-numeric-string' \
-e 'GITLAB_SECRETS_SECRET_KEY_BASE=long-and-random-alpha-numeric-string' \
-e 'GITLAB_SECRETS_OTP_KEY_BASE=long-and-random-alpha-numeric-string' \
-e 'GITLAB_HOST=服务器地址' \
-e 'GITLAB_EMAIL=邮箱地址' \
-e 'SMTP_ENABLED=true' \
-e 'SMTP_DOMAIN=www.sina.com' \
-e 'SMTP_HOST=smtp.sina.com' \
-e 'SMTP_STARTTLS=false'  \
-e 'SMTP_USER=邮箱地址' \
-e 'SMTP_PASS=邮箱密码' \
-e 'SMTP_AUTHENTICATION=login' \
-e 'GITLAB_BACKUP_SCHEDULE=daily' \
-e 'GITLAB_BACKUP_TIME=10:30' \
-v /home/root/opt/gitlab/data:/home/git/data \
sameersbn/gitlab
```

##### 1.5gitlab配置

当我们把gitlab运行起来之后，就可以访问默认的10080端口访问gitlab了。

<font color="red">注意到可能初次访问有时会出现502错误，多试几次即可</font>

当初次进入gitlab的页面时，需要重置管理员密码，然后跳转到登录界面。

默认用户名是root，密码为刚刚设置的密码。

登录完成之后就可以进入gitlab页面，具体使用跟github网站类似。

首先需要将本机的公钥输入到设置当中进行身份验证。

右上角找到**settings**

![](http://ondsf10qe.bkt.clouddn.com/jenkins26.png)

左侧找到**SSH Keys**

![](http://ondsf10qe.bkt.clouddn.com/jenkins27.png)

将本机的公钥填入即可，生成公钥可参考[如何生成ssh公钥](http://blog.csdn.net/qaz13177_58_/article/details/27544177)

![](http://ondsf10qe.bkt.clouddn.com/jenkins28.png)

现在即可如github一般使用gitlab了，可参考我的[GIT学习笔记](http://pirrla.cn/2017/03/16/other/GitTutorial_ByLiaoxuefeng/)，除了有一点不同

在github时，我们对github的访问是

```
git@github.com:alanhuangym/learngit.git
```

现在使用gitlab我们使用ssh（10022端口）

```
ssh://git@localhost:10022/root/test.git
```

当然也可以使用http连接（10080端口）

```
http://localhost:10080/root/test.git
```



### 2.搭建基于git的代码质量管理

对于git，我们主要使用的是git hook对git进行代码质量管理。

代码质量管理不仅仅是指希望对代码可以执行，没有语法错误，我们更希望能实现例如代码注释率、代码重复率等高级指标，从而提高代码仓库的代码质量的目的，避免低质量、错误的代码入库。

> Git 钩子是在 Git 仓库中特定事件发生时自动运行的脚本。它可以让你自定义 Git 内部的行为，在开发周期中的关键点触发自定义的行为。

> Git 钩子最常见的使用场景包括推行提交规范，根据仓库状态改变项目环境，和接入持续集成工作流。但是，因为脚本可以完全定制，你可以用 Git 钩子来自动化或者优化你开发工作流中任意部分。

我们知道，git有两个仓库，一个是本地仓库，一个是远程仓库。所以对git的代码质量管理也有两个层面，一个是本地层面的**Client-Side Hooks**，另一个则是服务器层面的**Server-Side Hooks**。

##### 2.1 本地仓库管理

不论使用的是gitlab或者github，都有本地仓库hook管理。

在每一个git的项目中，都有一个`.git`隐藏文件夹，进入到里面的`hook`文件夹就可以看到一些已经预设好的hook脚本。

![](http://ondsf10qe.bkt.clouddn.com/jenkins29.png)

他们都带着`.sample`的后缀，证明他们是供我们参考的，拓展名防止它们默认被执行，如果希望执行这些hook，可以将`.sample`后缀去除。

这里钩子的脚本语言没有限制，一般的shell，ruby，python都可以。

**命名方式**是决定他们在Git的哪一步被执行，所以命名方式需要注意，例如`pre-commit`钩子就是在用户进行commit操作的时候，在进行commit操作之前运行，详细的每个钩子的执行位置可以查看[官方文档](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks)，这里介绍一些比较实用的。

- `pre-commit`是我们希望实现的代码质量管理的一个好钩子，它在用户在进行`git commit -m "注释"`提交修改到本地仓库时执行，在提交操作之前执行脚本，如果脚本没有通过，则不会执行提交修改操作，返回错误。

这样的话我们就可以通过`pre-commit`钩子实现代码质量管理了，在`pre-commit`中写入代码质量管理的脚本，当质量过关时，返回`0`，即脚本已通过，可以执行`commit`操作，当质量不过关时，返回`非0数值`，此时脚本未通过，`commit`操作未成功，即实现了阻止错误、低质量代码进入代码库的操作。

<font color="red">但是</font>我们需要注意到，`.git`文件是不会提交到远程仓库的，所以`pre-commit`脚本是不能同步到所有的协作者`.git`文件夹中的。

即是`pre-commit`脚本的作用域只在本机，且不受服务器控制（可以被本机删除而不执行）。所以，如果希望通过`pre-commit`进行版本质量管理，只能将脚本放在`.git`文件夹外，提交到远程仓库，同时需要复制文件进每一个协作者的`.git/hook`文件夹中。这样是一种**鼓励形式**的代码质量管理，可行性不高。

##### 2.2 远程仓库管理（重点）

由于本地仓库的钩子不能实现我们希望的效果，所以我们选择在服务端进行钩子脚本构建。在服务器端构建钩子有以下好处：

- 无法被本地修改，强制进行质量管理
- 可以统一在服务器进行对质量管理阈值的修改

同时也有部分的缺点：

- 没有一个通用的代码质量管理软件，只能对应每一个项目建立相对语言的代码质量管理

###### 2.2.1 全局钩子

当使用docker构建gitlab时，远程仓库的脚本存在于docker容器内的`/home/git/gitlab-shell/hooks`路径当中。

进入docker容器内可以使用命令`docker exec -it 容器ID /bin/bash`

这个路径下存放有三个钩子，分别是`pre-receive`，`post-receive`和`update`。

这三个是<font color="red">gitlab全局的钩子脚本</font>，就是这三个脚本对gitlab内的所有项目启用。他们三个的运行顺序如下

![](http://ondsf10qe.bkt.clouddn.com/jenkins32.png)

他们各自代表的步骤是：

- pre-receive

  在服务器收到用户推送的代码之前，进行操作


- update

  服务器已经收到代码之后，对分支进行更新的操作，如果有多个分支则会运行多次update，且update操作之间<font color="red">相互独立</font>，失败不影响其他


- post-receive

  在服务器已经更新完服务器端的代码之后的操作，一般是对客户返回信息



所以如果我们需要对代码进行质量管理，应该选择的是`update`钩子。

###### 2.2.2 项目钩子

但是如果我们想用这三个全局钩子对所有的项目进行质量管理是不科学的，首先是因为没有一个通用的全代码质量管理工具，所以不能对全局使用同一个脚本，需要对不同的编程语言使用不同的脚本，其次，每一个项目可能会有特殊的需求，使用全局脚本不现实。所以我们需要找到每个脚本各自的`pre-receive`钩子。

每一个项目各自的钩子可以在路径`/home/git/data/repositories/用户名/项目名.git`或者`/home/git/repositories/<group>/<project>.git`路径中找到。

文件夹当中有一个`hooks`文件夹，<font color="red">但是</font>这个是一个超链接，跳转到的是全局钩子的地址`/home/git/gitlab-shell/hooks`。

我们就可以知道，项目是通过这个超链接去调用全局脚本的，所以如果我们希望项目能调用自己的脚本，那么我们可以创建一个同名的`hooks`文件夹替代，从而实现项目的个性化处理。

<font color="red">不过需要注意的是</font>，当我们使用自定义的脚本的时候 ，如果将全局脚本当做模板进行改写的时候，全局脚本内有一些关联路径的语句，例如

```
require_relative '../lib/gitlab_custom_hook
```

这里是定义了全局脚本的<font color="red">相对地址</font>，所以我们需要更改为<font color="red">绝对地址</font>，例如改为

```
require_relative '/home/git/gitlab-shell/lib/gitlab_custom_hook
```

为了实现我们的项目钩子的编写，查看[官方文档](https://docs.gitlab.com/ee/administration/custom_hooks.html)，我们了解到自定义钩子的写法是在`/home/git/repositories/<group>/<project>.git`文件夹内创建一个名为`custome_hooks`的文件夹，然后再文件夹内放置脚本，放置脚本的方法也有两种：

- 直接创建同名的<font color="red">可执行</font>脚本

  例如直接创建一个名为`pre-receive`的<font color="red">可执行</font>脚本


- 创建同名后缀为`.d`的文件夹，文件夹内放置脚本（命名随意）

  例如创建一个`pre-receive.d`文件夹，在文件夹内放入一些<font color="red">可执行</font>脚本，则会按照命名顺序，顺序执行

![](http://ondsf10qe.bkt.clouddn.com/jenkins35.png)

<font color="red">\*\*</font>这里我们遇到的可执行脚本的问题，可以通过命令行解决，只需要一行命令`chmod +x 文件名`即可以将脚本变为可执行脚本

<font color="red">需要注意的是</font>，如果没有将脚本变为可执行，那么脚本将不会在自定义钩子中生效。

我们的自定义钩子是在官方的脚本运行结束后运行的，但是因为自定义钩子的写法也有不同，所以运行自定义钩子的顺序也有所不同，通过实验（通过对所有脚本加上一个命令行输出查看），我们了解到顺序如下：

![](http://ondsf10qe.bkt.clouddn.com/jenkins33.png)

![](http://ondsf10qe.bkt.clouddn.com/jenkins34.png)

### 3.钩子脚本编写

当我们了解完全局钩子和自定义钩子的写法和执行顺序之后，我们就可以开始部署我们的代码质量检查钩子的编写了。由于没有一个统一的软件对全部代码进行统一的代码检测，所以我们将针对不同语言的项目进行不同的代码质量管理部署。

##### 3.1 服务器代码存放问题

首先，由于在gitlab上存储的是每一个项目的.git文件夹，即<font color="red">服务器端没有存放除.git文件夹内容外真实存在的代码</font>，数据文件只有.git文件夹的objects文件（文件包含提交信息、项目树信息，代码信息、标签信息）。

所以如果我们希望在服务器端实现代码质量审查，那么我们就首先需要将代码从objects文件夹中提取出来，再进行代码质量审查。

##### 3.2 Git的数据存放原理

Git 存储数据内容的方式──为每份内容生成一个文件，取得该内容与头信息的 SHA-1 校验和，创建以该校验和前两个字符为名称的子目录，并以 (校验和) 剩下 38 个字符为文件命名 (保存至子目录下)。每当文件有改动之后，SHA-1值都会变动，所以我们可以通过SHA-1值找到不同版本的代码。

![](http://ondsf10qe.bkt.clouddn.com/jenkins37.jpg)

对于SHA-1数据类型的文件，我们主要使用`git cat-file`命令对文件进行解析读取。

Git中有四种基本的对象类型，这四种基本类型组成了Git更高级的数据结构。

- blob

  每一个blob文件储存着一个（版本的）文件，blob文件只包含文件的数据，而忽略文件的其他元数据，如名字、路径、格式等。


- tree

  tree代表了<font color="red">一个目录</font>的信息，包含了该目录下的blob文件和相应的元文件信息和子目录（对应的子tree）。git对项目的管理就是使用嵌套的tree结构对文件进行存储。

- commit

  每一个commit提交记录都记录着该次提交的一些内容，例如该次提交后的tree的信息，父commit的信息，作者和提交时间的信息，提交日志。

- tag

  tag通常用于为commit命名一个易于记忆和分辨的名字。

下面用一个例子简单解释一下以上的数据类型

在根目录的tree对象里我们可以看到，这里有两个文件 README和Rakefile都是blob对象，lib是一个子目录，所以是tree对象。

```
$ git cat-file -p master^{tree}
100644 blob a906cb2a4a904a152e80877d4088654daad0c859      README
100644 blob 8f94139338f9404f26296befa88755fc2598c289      Rakefile
040000 tree 99f1a6d12cb4b6f19c8655fca46c3ecf317074e0      lib
```

再进入子树lib的查看，可以看到有一个simplegit.rb的rubu文件的blob对象

```
$ git cat-file -p 99f1a6d12cb4b6f19c8655fca46c3ecf317074e0
100644 blob 47c6340d6459e05787f644c2447d2595f5d3a54b      simplegit.rb
```

结构图如下

![](http://ondsf10qe.bkt.clouddn.com/jenkins38.png)



##### 3.3 通用代码存放脚本

由于在服务器端没有存放真实的代码，所以我们需要在receive到用户push上来的commit文件之后，对文件进行解析，并将代码读取保存在服务器的临时文件夹中。

所以我们对钩子中的update进行编写，脚本使用的是shell脚本，主要就是解析提交文件，然后将<font color="red">有改动或新增</font>的代码放置入服务器的临时文件夹中。

```bash
#!/bin/bash

# 提交分支的名字
ref_name=$1
# 上一个commit的ID
old_value=$2
# 现在这个commit的ID
new_value=$3

# 共有四种状态
# M-修改，A-新增，D-删除，RXXX-重命名 XXX是指重命名前后代码相似度
# 所以这里选择修改和新增和重命名的进行代码审查
# （因为重命名会覆盖掉D和M的状态
new_and_modify=`git diff $old_value $new_value --raw | grep "\sM\s\|\sA\s" | awk '{print $4"\t"$6}' |  uniq`
rename=`git diff $old_value $new_value --raw | grep "\sR.*\s" | awk '{print $4"\t"$7}' |  uniq`
# 检查文件夹和文件的名字是否有空格
# 有空格则直接提交失败
count=`git show $new_value --name-only | grep "\s" -c`
if [[ $count -gt 4 ]]
then
	echo "文件/文件夹 名字包含空格，提交失败"
	exit 1
fi

# 将字符串变为数组
new_and_modify_array=($new_and_modify)
rename_array=($rename)

collect_array=(${new_and_modify_array[@]} ${rename_array[@]})

# n用于遍历数组
n=0
# 遍历数组，提取出需要代码质量管理的文件的blob id和原本的文件名
while [ $n -lt ${#collect_array[@]} ]
do
	# blob id 用于提取代码
	blob_id=${collect_array[$n]:0:7}
	# 文件名，由于带有路径，所以将文件名中的/斜杠替换为_下划线
	f_name=${collect_array[${n}+1]////_}
	# 将代码保存到临时文件夹中
	git show $blob_id > temp_files/code_${f_name}
	# n自增，用于遍历
	let n+=2
done

# 该标记用于识别是否有代码没用通过，全部通过则为0，只要有一个代码没通过就为1
flag=0

# 转换工作路径到临时文件夹
cd temp_files
# 获取临时文件夹里的文件
files=`ls`
# 如果临时文件夹里没有内容，则证明没有进行代码修改
# 不需要进行代码管理，返回0
if [[ $files == "" ]]
then 
	exit 0
fi

# 将文件转换成数组，遍历数组，对每一个文件分别进行代码质量管理操作
files_array=($files)
# m用于遍历数组
m=0
files_count=${#files_array[@]}
echo "此次提交共有 $files_count 个文件进行过修改/新增/重命名"

while [ $m -lt ${#files_array[@]} ]
do
	# 文件名
	file_name=${files_array[$m]}
	let m+=1


#########################################################
#此处，不同语言应对不同的代码质量管理程序
#主要操作是 `操作语句 $file_name` 
#例如python语言使用的是pylint，那么语句就是`pylint $file_name`
#主要通过设定flag的值操作代码是否通过
#########################################################





# 文件循环结束
done

# 移除临时生成的文件
rm *
# 判断标记是否为0
if [[ $flag == 0 ]]
then 
	echo "代码全部通过，提交成功"
else
	echo "有代码未通过，提交失败"
	exit 1
fi

exit 0
```



##### 3.4 Python代码质量管理

脚本如下，使用的是pylint对代码进行审查。pylint会返回一个报告供我们进行修改，然后有一个评分，我们可以根据评分进行阈值设定，例如这里这里设定的是四舍五入之后的评分大于等于5分则通过审查，可以提交成功，flag值为0。

```shell
#########################################################
#Python 文件
if [[ $file_name = *.py ]]
then

# 运行pylint脚本，对代码进行审查
# 将报告输出在临时文件夹的report.txt中
pylint $file_name  > report.txt

# 检测是否代码有语法错误，如果有语法错误直接退出
# （因为有语法错误的话没有评分，无法与阈值比较
error=`cat report.txt | grep "syntax-error"`
if [[ $error != "" ]]
then
	echo $file_name "有语法错误，提交失败"
	exit 1
fi

# 获取评分的语句
score_senten=`cat report.txt | grep "rated" | awk '{print $7}'`

# 从语句中提取分数
if [[ "$score_senten" =~ (.*)/10 ]] 
then
	score="${BASH_REMATCH[1]}"
else
	score=0
fi

echo $file_name 程序得分为 $score_senten
echo 程序分析报告:
cat report.txt

# 如果分数大于5分，则通过，否则失败
# 需要更改为单独计分，即每一个.py运行一次pylint
# 对分数进行四舍五入（因bash不能处理浮点数）
k=`echo $score|awk '{ printf("%.0f\n", $0); }'`
if [[ ${k} -ge 5 ]]
then
	echo "该代码通过"
else
	echo "该代码评分未通过"
	let flag=1
fi

fi

#Python 文件结束
#########################################################
```

##### 3.5 Javascript 代码



References:

\[1\] [持续集成 by www.abcdocker.com](http://blog.csdn.net/abcdocker/article/category/6638595)

\[2\] [使用docker搭建gitlab初体验+数据备份](https://www.jianshu.com/p/060e7223e211?open_source=weibo_search)

\[3\] [Gitlab 服务器端 custom hook 配置](https://www.jianshu.com/p/5531a21afa68)

\[4\] [Git 钩子](https://github.com/geeeeeeeeek/git-recipes/wiki/5.4-Git-%E9%92%A9%E5%AD%90%EF%BC%9A%E8%87%AA%E5%AE%9A%E4%B9%89%E4%BD%A0%E7%9A%84%E5%B7%A5%E4%BD%9C%E6%B5%81)

\[5\] [Custom Git Hooks ](https://docs.gitlab.com/ee/administration/custom_hooks.html)

\[6\] https://www.zhihu.com/question/65604891/answer/232935144

\[7\] https://git-scm.com/book/zh/v2/Git-%E5%86%85%E9%83%A8%E5%8E%9F%E7%90%86-Git-%E5%AF%B9%E8%B1%A1

\[8\] http://blog.jobbole.com/26209/