## 1 什么是fpm

fpm是一个命令行工具,帮助快速构建rpm包,dpkg包,osxpkg包等...

文档：[详细文档](http://fpm.readthedocs.io/en/latest/intro.html)

<!--more-->
## 2 centos6.8安装fpm
1. 使用rvm安装ruby[Linux在线升级ruby](http://blog.csdn.net/qq_14945847/article/details/77986900)

```bash
yum install -y rpm-build gcc-c++ patch readline readline-devel zlib zlib-devel libyaml-devel libffi-devel openssl-devel make bzip2 autoconf automake libtool bison iconv-devel
curl -L get.rvm.io | bash -s stable
source /etc/profile.d/rvm.sh
sed -i '$aruby_url=https://cache.ruby-china.org/pub/ruby' /usr/local/rvm/user/db
rvm install 2.4.1

```

2. 更换gem源[RubyGems](https://gems.ruby-china.org/)

```bash
gem sources --add https://gems.ruby-china.org/ --remove https://rubygems.org/
gem sources -l
https://gems.ruby-china.org
# 确保只有 gems.ruby-china.org
```

3. 下载fpm

```bash
gem install fpm
```
## 3 制作一个rpm包

制作一个rpm包安装之后在/usr/bin/目录下安装一个命令rpmtest

```bash
mkdir rpmbuild

cd rpmbuild

#rpmbuild 相当于安装的根

mkdir -p usr/bin 

echo "echo rpmtest" >> usr/bin/rpmtest && chmod +x usr/bin/rpmtest

fpm -s dir -t rpm -n rpmtest -v 0.0.1 -C .

Created package {:path=>"rpmtest-0.0.1-1.x86_64.rpm"}

rpm -ivh rpmtest-0.0.1-1.x86_64.rpm
Preparing...                ########################################### [100%]
1:rpmtest                ########################################### [100%]

rpmtest
rpmtest

rpm -qf /usr/bin/rpmtest
rpmtest-0.0.1-1.x86_64


rpm -ql rpmtest-0.0.1-1.x86_64
/usr/bin/rpmtest

#卸载
yum remove rpmtest

```
```html
-s          指定源类型
-t          指定目标类型，即想要制作为什么包
-n          指定包的名字
-v          指定包的版本号
-C          指定打包的相对路径  Change directory to here before searching forfiles
-d          指定依赖于哪些包
-f          第二次打包时目录下如果有同名安装包存在，则覆盖它
-p          输出的安装包的目录，不想放在当前目录下就需要指定
--post-install      软件包安装完成之后所要运行的脚本；同--after-install
--pre-install       软件包安装完成之前所要运行的脚本；同--before-install
--post-uninstall    软件包卸载完成之后所要运行的脚本；同--after-remove
--pre-uninstall     软件包卸载完成之前所要运行的脚本；同--before-remove

```

