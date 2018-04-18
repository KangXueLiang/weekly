## Centos搭建gitlab及持续集成

### 解决官方无法安装的情况

> * [Gitlab Community Edition 镜像使用帮助](https://mirror.tuna.tsinghua.edu.cn/help/gitlab-ce/)
> * [在阿里云上通过Omnibus一键安装包安装Gitlab](https://github.com/hehongwei44/my-blog/issues/19)

### 编辑源

##### 使用清华大学 TUNA 镜像源 打开网址将内容复制到gitlab-ce.repo文件中，编辑路径vim /etc/yum.repos.d/gitlab-ce.repo

```
[gitlab-ce]
name=gitlab-ce
baseurl=http://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el6
repo_gpgcheck=0
gpgcheck=0
enabled=1
gpgkey=https://packages.gitlab.com/gpg.key

```
### 更新本地 YUM 缓存

```
sudo yum makecache
```

### 安装 GitLab 社区版

```
sudo yum install gitlab-ce #(自动安装最新版)
sudo yum install gitlab-ce-8.8.4-ce.0.el6 #(安装指定版本)
```

### 配置并启动GitLab

```
# 打开`/etc/gitlab/gitlab.rb`,
# vim /etc/gitlab/gitlab.rb
# 将`external_url = 'http://git.example.com'`修改为自己的IP地址或域名：`http://xxx.xx.xxx.xx`,
# 然后执行下面的命令, 对GitLab进行编译.
sudo gitlab-ctl reconfigure
```

### 登录GitLab

```
# 访问已修改后的ip地址或域名
# 会提示更新密码
# UserName: root
# PassWord: 已设置的密码
```

### GitLab头像无法正常显示
    * 如果能正常显示头像忽略此配置

原因: gravatar被墙
解决办法:
编辑 /etc/gitlab/gitlab.rb, 将

```
# gitlab_rails['gravatar_plain_url'] = 'http://gravatar.duoshuo.com/avatar/%{hash}?s=%{size}&d=identicon'
修改为:
gitlab_rails['gravatar_plain_url'] = 'http://gravatar.duoshuo.com/avatar/%{hash}?s=%{size}&d=identicon'

```
然后在命令行执行：

```
sudo gitlab-ctl reconfigure 
sudo gitlab-rake cache:clear RAILS_ENV=production
```

### nginx配置

解决 80 端口被占用(没占用忽略此配置)
```
upstream gitlab {
     server 114.55.111.111:8081 ;
}
server {
    #侦听的80端口
    listen       80;
    server_name  git.diggg.cn;
    location / {
        proxy_pass   http://gitlab;    #在这里设置一个代理，和upstream的名字一样
        #以下是一些反向代理的配置可删除
        proxy_redirect             off;
        #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
        proxy_set_header           Host $host;
        proxy_set_header           X-Real-IP $remote_addr;
        proxy_set_header           X-Forwarded-For $proxy_add_x_forwarded_for;
        client_max_body_size       10m; #允许客户端请求的最大单文件字节数
        client_body_buffer_size    128k; #缓冲区代理缓冲用户端请求的最大字节数
        proxy_connect_timeout      300; #nginx跟后端服务器连接超时时间(代理连接超时)
        proxy_send_timeout         300; #后端服务器数据回传时间(代理发送超时)
        proxy_read_timeout         300; #连接成功后，后端服务器响应时间(代理接收超时)
        proxy_buffer_size          4k; #设置代理服务器（nginx）保存用户头信息的缓冲区大小
        proxy_buffers              4 32k; #proxy_buffers缓冲区，网页平均在32k以下的话，这样设置
        proxy_busy_buffers_size    64k; #高负荷下缓冲大小（proxy_buffers*2）
        proxy_temp_file_write_size 64k; #设定缓存文件夹大小，大于这个值，将从upstream服务器传
    }
}
```
```
# 检查配置
/usr/local/nginx-1.5.1/sbin/nginx -tc conf/nginx.conf

# nginx 重新加载配置
/usr/local/nginx-1.5.1/sbin/nginx -s reload
```

### 运维

```
# 启动所有 gitlab 组件：
sudo gitlab-ctl start

# 停止所有 gitlab 组件：
sudo gitlab-ctl stop

# 重启所有 gitlab 组件：
sudo gitlab-ctl restart

# 查看服务状态
sudo gitlab-ctl status

# 启动服务
sudo gitlab-ctl reconfigure

# 修改默认的配置文件
sudo vim /etc/gitlab/gitlab.rb

# 查看版本
sudo cat /opt/gitlab/embedded/service/gitlab-rails/VERSION

# echo "vm.overcommit_memory=1" >> /etc/sysctl.conf
# sysctl -p
# echo never > /sys/kernel/mm/transparent_hugepage/enabled

# 检查gitlab
gitlab-rake gitlab:check SANITIZE=true --trace

# 查看日志
sudo gitlab-ctl tail
```

### 备份恢复

#### 1. Gitlab 创建备份
使用Gitlab一键安装包安装Gitlab非常简单, 同样的备份恢复与迁移也非常简单,用一条命令即可创建完整的Gitlab备份

```
gitlab-rake gitlab:backup:create
```

以上命令将在/var/opt/gitlab/backups目录下创建一个名称类似为xxxxxxxx_gitlab_backup.tar的压缩包, 这个压缩包就是Gitlab整个的完整备份, 其中开头的xxxxxx是备份创建的时间戳

#### 2. Gitlab 修改备份文件默认目录

修改/etc/gitlab/gitlab.rb来修改默认存放备份文件的目录:

```
gitlab_rails['backup_path'] = '/mnt/backups'
```
修改后使用gitlab-ctl reconfigure命令重载配置文件

#### 3. 备份

```
0 3 * * * /usr/bin/gitlab-rake gitlab:backup:create
0 3 * * * /opt/gitlab/bin/gitlab-rake gitlab:backup:create
```

#### 4. 恢复

首先进入备份 gitlab 的目录, 这个目录是配置文件中的 gitlab_rails['backup_path'], 默认为 /var/opt/gitlab/backups.
然后停止 unicorn 和 sidekiq, 保证数据库没有新的连接，不会有写数据情况

```
# 停止相关数据连接服务
# ok: down: unicorn: 0s, normally up
gitlab-ctl stop unicorn  
# ok: down: sidekiq: 0s, normally up
gitlab-ctl stop sidekiq

# 从xxxxx编号备份中恢复
# 然后恢复数据，1406691018为备份文件的时间戳
gitlab-rake gitlab:backup:restore BACKUP=xxxxxx

# 启动Gitlab
sudo gitlab-ctl start
```

### GitLab-Runner的安装与使用
* 添加yum源
```
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-ci-multi-runner/script.rpm.sh | sudo bash
```
* 安装
```
yum install gitlab-ci-multi-runner
```

### 使用gitlab-ci-multi-runner注册Runner

* 安装好gitlab-ci-multi-runner这个软件之后, 我们就可以用它向GitLab-CI注册Runner了.

* 向GitLab-CI注册一个Runner需要两样东西: GitLab-CI的url和注册token.
其中, token是为了确定你这个Runner是所有工程都能够使用的Shared Runner还是具体某一个工程才能使用的Specific Runner

* 如果要注册Shared Runner，你需要到管理界面的Runners页面里面去找注册token.管理界面的Runners页面大致信息如下:

> ### Setup a shared Runner manually
> * 1. 安装一个与 GitLab CI 兼容的 Runner (如需了解更多的安装信息，请查看 GitLab Runner)
> * 2. 在 Runner 设置时指定以下 URL： http://20.195.133.77/
> * 3. 在安装过程中使用以下注册令牌： czoa_iPHqHwbD7msYXcn
> * 4. 启动 Runner!

#### 注册
1. 执行
```
gitlab-runner register
```
根据提示分别输入url和token, 回车完成Runner注册

### 持续集成部署
* 进入搭建gitlab服务器设置免密登录

```
# 先设置用户root免密登录：
ssh-keygen -t rsa
ssh-copy-id -i ~/.ssh/id_rsa.pub root@ip（部署代码的服务器IP地址）

# 修改gitlab-runner用户的密码
passwd gitlab-runner
执行后输入密码即可
	
# 使用gitlab-runner帐号登录gitlab服务器
ssh gitlab-runner@id
输入设置后的密码

# 设置用户gitlab-runner免密登录
ssh-keygen -t rsa
ssh-copy-id -i ~/.ssh/id_rsa.pub root@ip(部署代码的服务器IP地址)
```

* 在项目脚手架添加.gitlan-ci.yml
```
# 定义 stages
stages:
  - install_deps
#  - test
  - build_uat
  - deploy_uat

cache:
  key: ${CI_BUILD_REF_NAME}
  paths:
    - node_modules/
    - dist_uat/

# 安装构建依赖
install_deps_job:
  stage: install_deps
  only:
    - dev
#    - regression
#    - release
  script:
    - pwd
    - npm i

# 构建uat环境src目录下应用
build_uat_job:
  stage: build_uat
  only:
    - dev
  script:
    - rm -rf dist_uat
    - npm run-script build-uat

# 部署uat环境
deploy_uat_job:
  stage: deploy_uat
  only:
    - dev
  script:
    - cd dist_uat/
    - pwd
    - whoami
    - ssh root@部署代码的服务器IP地址 "rm -rf /data/wwwroot/sls-application/uat-station/sls-pc/static/*"
    - scp -r -P 22 sls-pc/static/* root@xxx.xx.xx.xx:/data/wwwroot/sls-application/uat-station/sls-pc/static
    - ssh root@xxx.xx.xx.xx "rm -rf /data/wwwroot/sls-application/uat-station/sls-pc/*.html"
    - scp -r -P 22 *.html root@xxx.xx.xx.xx:/data/wwwroot/sls-application/uat-station/sls-pc
```

* 部署对应的文件夹要先去部署代码的服务器创建, 不然会提示匹配不到对应文件夹


### 配置邮箱
* GitLab中使用postfix进行邮件发送。因此，可以卸载系统中自带的sendmail。 使用yum list installed查看系统中是否存在sendmail，若存在，则使用yum remove sendmail指令进行卸载。

* 测试系统是否可以正常发送邮件。 echo "Test mail from postfix" | mail -s "Test Postfix" xxx@xxx.com

* 报错:
```
postfix: fatal: parameter inet_interfaces: no local interface found for ::1
```
* 修改配置
```
vim  /etc/postfix/main.cf
```
```
# 发现配置为：
inet_interfaces = localhost
inet_protocols = all
# 改成：
inet_interfaces = all
inet_protocols = all
```

### 重新启动

```
service postfix start
```

```
# 下一步执行
echo "Test mail from postfix" | mail -s "Test Postfix" xxx@xxx.com
邮件发送成功
```

当邮箱收到系统发送来的邮件时, 将系统的地址复制下来, 如: root@VM_52_12_centos.localdomain, 打开/etc/gitlab/gitlab.rb, 将
```
# gitlab_rails['gitlab_email_from'] = 'gitlab@example.com'
修改为
gitlab_rails['gitlab_email_from'] = 'root@VM_52_12_centos.localdomain'
```

保存后，执行sudo gitlab-ctl reconfigure重新编译GitLab. 如果邮箱的过滤功能较强,请添加系统的发件地址到邮箱的白名单中, 防止邮件被过滤


### 错误处理

待续


### gitlab安装参考文档

> * gitlab / https://packages.gitlab.com/gitlab/gitlab-ce
> * 官网下载：www.gitlab.cc/downloads
> * 官网安装说明：doc.gitlab.cc/ce/install/…
> * 开源版本和企业版本对比：www.gitlab.cc/features/#e…
> * https://www.cnblogs.com/weifeng1463/p/7714492.html
> * https://www.jianshu.com/p/2b43151fb92e
