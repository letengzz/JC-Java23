# Git 代码托管平台

## GitHub

GitHub官网地址：https://github.com/

### GitHub 基础操作

首先需要访问[官网](https://github.com/)，注册登录账号。

#### 创建新的仓库

- 两种方式，任选其一：

  ![image-20230513235452404](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140104273.png)

- 输入仓库的相关信息：

  ![image-20230513235459573](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140104185.png)

#### 基本操作指令

![image-20230513235613316](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140105568.png)

#### SSH 免密操作

github支持两种同步方式“https”和“ssh”。如果使用https很简单基本不需要配置就可以使用，但是每次提交代码和下载代码时都需要输入用户名和密码。ssh模式比https模式的一个重要好处就是，每次push、pull、fetch等操作时，不用重复填写遍用户名密码。前提是必须是这个项目的拥有者或者合作者，且配好了ssh key。

- 本地生成SSH密钥：`ssh-keygen -t rsa -C GitHub邮箱`

  - 需要把邮件地址换成你自己的邮件地址，然后一路回车，使用默认值即可，由于这个Key无需设置密码。

- 执行命令完成后，在window本地用户 .ssh 目录 C:\Users\用户名\.ssh 下面生成公钥和私钥：

  ![image-20230513235656181](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140105198.png)

- 集成用户公钥：将id_rsa.pub文件内容复制到GitHub仓库中

  ![image-20230513235711239](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140105423.png)

  ![image-20230513235716328](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140105262.png)

#### 设定全局用户

```bash
git config --global user.name '202088'
# 这里的邮箱地址需要为GitHub网站的注册账号
git config --global user.email '202088@163.com'
```

#### 创建本地库以远程地址

```bash
# 初始化本地仓库
git init
# 设置远程仓库
git remote add origin git@github.com:Joe-Clion/git-study.git
```

#### 新增，提交本地仓库文件

```bash
# 新增文件
git add test.txt
# 提交文件
git commit test.txt
```

#### 推送到GitHub远程仓库

```bash
# 推送文件
git push origin master
```

SSH警告：

当第一次使用Git的 clone 或者 push 命令连接GitHub时，会得到一个警告：

> The authenticity of host 'github.com (xx.xx.xx.xx)' can't be established.
> RSA key fingerprint is xx.xx.xx.xx.xx.
> Are you sure you want to continue connecting (yes/no)?

这是因为Git使用SSH连接，而SSH连接在第一次验证GitHub服务器的Key时，需要确认GitHub的Key的指纹信息是否真的来自GitHub的服务器，输入 yes 回车即可。
Git会输出一个警告，告诉已经把GitHub的Key添加到本机的一个信任列表里了：这个警告只会出现一次，后面的操作就不会有任何警告了。

#### 从远程库克隆

- 新建仓库，勾选Add a README file，这样GitHub会自动创建一个README.md 文件：

  ![image-20230514000030061](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140106249.png)

- 创建完毕后，可以看到 README.md 文件：

  ![image-20230514000038799](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140106846.png)

- 使用命令 git clone 克隆一个本地库，查看文件已经存在了：

  ![image-20230514000043963](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140106585.png)

- GitHub给出的地址不止一个，还可以用http 地址。实际上，Git支持多种协议，默认的 git:// 使用ssh，但也可以使用 https 等其他协议。

  使用 https 除了速度慢以外，还有个最大的麻烦是每次推送都必须输入口令，但是在某些只开放http端口的公司内部就无法使用 ssh 协议而只能用 https 。

### 团队操作

#### 增加合作伙伴

- ![image-20230514000127591](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140106217.png)

- 选择合作账号,发出合作申请：

  ![image-20230514000136928](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140106298.png)

#### 合作伙伴确认

- 合作伙伴收到确认后，点击Join按钮继续：

  ![image-20230514000146898](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140110334.png)

- 点击Accept Invitation按钮，进行确认：

  ![image-20230514000152437](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140110775.png)

### 远程仓库fork操作

如果项目存在大量合作伙伴，对于版本库的管理明显是一个特别大的风险，所以如果不想要选择大量的合作伙伴，但依然有人想要对项目代码进行维护，更新和扩展的话，此时，可以使用fork功能。

#### 操作步骤

- ![image-20230514000239625](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140110030.png)

- ![image-20230514000243690](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140110238.png)

  这样就等同于创建了一个自己的远程仓库。但是这个远程仓库等同于是一个分支远程仓库，可以随便操作，并不会影响源仓库，但是如果你的修改，更新想要融合到源仓库中，就需要提交申请了

- **提交申请操作**：

   1. 修改文件内容
      - ![image-20230514000330023](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140110604.png)
      - ![image-20230514000333861](./assets/image-20230514000333861.png)


   2. 发送提交申请
      - ![image-20230514000345625](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140110627.png)
      - ![image-20230514000349763](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140109341.png)
      - ![image-20230514000353709](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140109791.png)

   3. 合并修改请求
      - ![image-20230514000407529](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140109946.png)
      - ![image-20230514000411696](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140109072.png)
      - ![image-20230514000415394](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140109932.png)
      - ![image-20230514000419502](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140109979.png)

   4. 修改请求确认
      - ![image-20230514000432842](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140109763.png)


### GitHub 客户端

- [官方地址：https://desktop.github.com/](https://desktop.github.com/)

#### 安装GitHub Desktop

下载链接：https://central.github.com/deployments/desktop/desktop/latest/win32

无安装过程，安装完成后，弹出应用界面

#### 配置

##### 配置用户名及邮箱

- 点击软件得File菜单后，选择Options, 设定软件得操作用户名称及对应得邮箱地址：

  ![image-20230514000745901](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140109517.png)

#### 设置主题样式

- 可以根据自己得偏好设定软件主题样式：

  ![image-20230514000756689](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140109455.png)

##### 设置全屏

- 如果觉得软件界面比较小，可以适当进行调整或全屏：

  ![image-20230514000801530](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140109293.png)

#### 本地仓库操作

##### 新建仓库

- ![image-20230514000934583](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140109134.png)
- ![image-20230514000937969](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140109045.png)

##### 操作区域

- ![image-20230514000942375](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140109268.png)

##### 切换仓库

- ![image-20230514000946745](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140109071.png)

##### 删除仓库

- ![image-20230514000951015](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140109131.png)
- ![image-20230514000954863](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140109453.png)

#### 文件操作

##### 新增文件

- 当工作区域创建了一份新文件，工具可以自动识别并进行对应得显示：

  ![image-20230514001002555](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140109746.png)

- 此时Git仓库中并没有这份文件，所以需要执行commit操作，将文件保存到Git仓库中：

  ![image-20230514001009106](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140109527.png)

##### 忽略文件

- 如果某一个文件或某一类得文件，不想被Git软件进行管理。可以在忽略文件中进行设定：

  ![image-20230514001017278](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140109488.png)

- 修改文件

  修改文件只是将工作区域得文件进行修改，但是对于Git软件来讲，其实本质上还是提交，因为底层会生成新得文件

  ![image-20230514001026194](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140109392.png)

##### 删除文件

- 删除文件对于Git软件来讲，依然是一个提交

  ![image-20230514001037001](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140109287.png)

- 提交后，最新版本得文件也会被“删除”

##### 历史记录

- 如果存在多次得提交操作得话，可以查看提交得历史记录

  ![image-20230514000903123](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140109024.png)

#### 分支操作

##### 默认分支

软件创建仓库时，默认创建得分支为main

![image-20230514001148016](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140109564.png)

点击右键可以改名

![image-20230514001154841](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140109056.png)

##### 创建分支

- ![image-20230514001159121](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140109439.png)
- ![image-20230514001203414](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140109815.png)

##### 切换分支

- ![image-20230514001207595](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140108632.png)

##### 删除分支

- ![image-20230514001211116](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140108623.png)

##### 合并分支

- ![image-20230514001215769](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140108936.png)
- ![image-20230514001219782](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140108708.png)

#### 标签操作

##### 创建标签

- ![image-20230514001226860](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140108494.png)
- ![image-20230514001232010](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140108106.png)
- ![image-20230514001236866](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140108716.png)

##### 删除标签

- ![image-20230514001240933](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140108740.png)

#### 远程仓库

##### 克隆仓库

- 选择克隆仓库：

  ![image-20230514001248813](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140108189.png)

- 可以选择登录GitHub账号，也可以使用GitHub企业版 同时也可以使用URL：

  ![image-20230514001412701](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140108373.png)

- ![image-20230514001417272](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140108823.png)

- ![image-20230514001420934](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140108073.png)

- ![image-20230514001424800](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140108952.png)

##### 拉取文件

- 远程仓库更新了文件：

  ![image-20230514001429120](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140108014.png)

- 拉取远程仓库文件：

  ![image-20230514001434216](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140108114.png)

- ![image-20230514001439590](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140108092.png)

- ![image-20230514001442657](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140108463.png)

- ![image-20230514001447106](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140112960.png)

##### 推送文件

- 本地创建新文件：

  ![image-20230514001455569](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140108899.png)

- 提交新文件到本都仓库：

  ![image-20230514001459645](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140108118.png)

- 向远程仓库推送文件：

  ![image-20230514001505855](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140108853.png)

- ![image-20230514001539897](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140108208.png)

## Gitee

相对于GitHub来讲，由于网络的原因，在连接时不是很稳定，所以在采用第三方远程仓库时，也可以选择国内的Gitee平台。

![image-20230514001609954](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140112949.png)

- Gitee官网地址：https://gitee.com/

### Gitee 基础操作

#### 注册网站会员

- 注册并登录Gitee：

  ![image-20230514001949044](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140108715.png)

#### 用户中心

- 登录成功后跳转到用户中心

  ![image-20230514002037048](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140108200.png)

#### 创建远程仓库

- 右上角选择新建仓库：

  ![image-20230514002051523](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140107533.png)

- 填写仓库信息并创建：

  ![image-20230514002101335](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140107562.png)

- 创建成功

  ![image-20230514002107558](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140107151.png)

#### 远程仓库简易操作指令

![image-20230514002127575](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140107118.png)

#### SSH 免密操作

同github一样Gitee支持两种同步方式“https”和“ssh”。如果使用https很简单基本不需要配置就可以使用，但是每次提交代码和下载代码时都需要输入用户名和密码。ssh模式比https模式的一个重要好处就是，每次push、pull、fetch等操作时，不用重复填写遍用户名密码。前提是必须是这个项目的拥有者或者合作者，且配好了ssh key。

- 本地生成SSH密钥：

  `ssh-keygen -t rsa -C Gitee邮箱`

  需要把邮件地址换成你自己的邮件地址，然后一路回车，使用默认值即可，由于这个Key无需设置密码。

- 执行命令完成后，在window本地用户 .ssh 目录 C:\Users\用户名\.ssh 下面生成公钥和私钥：

  ![image-20230514002244992](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140107229.png)

- 集成用户公钥：将id_rsa.pub文件内容复制到Gitee仓库中

  - ![image-20230514002254440](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140107317.png)
  - ![image-20230514002259614](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140107029.png)

#### 设定全局用户

```bash
git config --global user.name '202088'
# 这里的邮箱地址需要为远程仓库简易操作指令内容
git config --global user.email '202088@163.com'
```

#### 创建本地库以远程地址

```bash
# 初始化本地仓库
git init
# 设置远程仓库
git remote add origin git@gitee.com:letengzz/git-repo.git
```

#### 新增，提交本地仓库文件

```bash
# 新增文件
git add test.txt
# 提交文件
git commit test.txt
```

#### 推送到 Gitee 远程仓库

```bash
# 推送文件
git push -u origin "master"
```

#### 从远程库克隆

- 新建仓库，设置README文件，这样Gitee会自动创建一个README.md 文件：

  ![image-20230514002916289](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140107271.png)

- 创建完毕后，可以看到 README.md 文件：

  ![image-20230514002921857](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140107213.png)

- 使用命令 `git clone` 克隆一个本地库，查看文件已经存在了：

  ![image-20230514002925784](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140107085.png)

- Gitee给出的地址不止一个，还可以用http 地址。实际上，Git支持多种协议，默认的 git:// 使用ssh，但也可以使用 https 等其他协议。

  使用 https 除了速度慢以外，还有个最大的麻烦是每次推送都必须输入口令，但是在某些只开放http端口的公司内部就无法使用 ssh 协议而只能用 https 。

### Gitee 团队操作

- ![image-20230514002941700](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140107947.png)

- 选择合作账号,发出合作申请

  ![image-20230514002946687](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140107262.png)

- ![image-20230514002950731](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140107523.png)

- ![image-20230514002953471](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140107666.png)

## GitLab

GitLab是由GitLabInc开发，使用MIT许可证的基于网络的Git仓库管理工具，且具有wiki和issue跟踪功能。使用Git作为代码管理工具，并在此基础上搭建起来的Web服务。
GitLib由乌克兰程序员DmitriyZaporozhets和ValerySizov开发，它使用Ruby语言写成。后来，一些部分用Go语言重写。GitLab被IBM，Sony，JulichResearchCenter，NASA，Alibab，Invincea，O'ReillyMedia，Leibniz-Rechenzentrum(LRZ)，CERN，SpaceX等组织使用。

GitLab可以用于搭建自己的代码托管平台。

官网地址：https://about.gitlab.com/

### 搭建步骤

#### 下载GitLab

  - 下载地址：https://packages.gitlab.com/gitlab/gitlab-ce

  - 如果下载不了，或下载比较慢，可以根据提示在在linux系统中直接采用wget指令下载：

    ![image-20230514003127782](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140107182.png)

#### 安装GitLab

  - 安装GitLab：直接采用下载的RPM软件包安装即可：
    
    ```
    sudo rpm -ivh /opt/module/software/gitlab-ce-15.7.0-ce.0.el7.x86_64.rpm
    ```
    
    ![image-20230514003143828](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140107080.png)

    出现 `error: Failed dependencies: policycoreutils-python is needed by gitlab-ce-15.9.6-ce.0.el7.x86_64` 问题：
    
    **解决办法**：
    
    ```bash
    yum install policycoreutils-python
    ```

#### 安装配置依赖

在CentOS 7上，下面的命令也会在系统防火墙中打开HTTP、HTTPS和SSH访问。这是一个可选步骤，如果您打算仅从本地网络访问极狐GitLab，则可以跳过它

- **firewall http 服务开启**：

  ```bash
  firewall-cmd --query-service http ##查看http服务是否支持，返回yes或者no
  firewall-cmd --add-service=http ##临时开放http服务
  firewall-cmd --add-service=http --permanent ##永久开放http服务
  firewall-cmd --reload ##重启防火墙生效
  ```
  
- **firewall https 服务开启**：

  ```bash
  firewall-cmd --add-service=https --permanent
  firewall-cmd --reload
  ```

#### 初始化GitLab

  - 配置软件镜像：

    ```
    curl -fsSL https://packages.gitlab.cn/repository/raw/scripts/setup.sh | /bin/bash
    ```

  - 安装：

    ```bash
    sudo EXTERNAL_URL="https://centos" yum install -y gitlab-ce
    ```

  - 初始化：

    ```bash
    sudo gitlab-ctl reconfigure
    ```

#### 启动GitLab

  - 启动：

    ```bash
    gitlab-ctl start
    ```

  - 停止：

    ```bash
    gitlab-ctl stop
    ```

#### 访问GitLab 进入：http://centos/users/sign_in

```bash
sudo yum install -y curl policycoreutils-python openssh-server perl
sudo systemctl enable sshd
sudo systemctl start sshd
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo systemctl reload firewalld
# 为了演示方便，我们也可以直接关闭防火墙
sudo systemctl stop firewalld  
```

	- 初始化时，软件会提供默认管理员账户：root，但是密码是随机生成的。
	
	在/etc/gitlab/initial_root_password文件中查找密码

**注意**：需在host中设置主机名及ip

![image-20230514003740918](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140107910.png)

#### 修改密码

默认的密码是随机的，且不容易记忆，还会在系统初始化后24小时被删除，所以需要先修改一下密码

- ![image-20230514003756915](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140107749.png)
- ![image-20230514003802544](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140107587.png)

#### 创建项目

- ![image-20230514003813146](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140107243.png)
- ![image-20230514003817553](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140107096.png)
- ![image-20230514003821768](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140106782.png)
- ![image-20230514003826117](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140106428.png)