# 1.下载相关工具，创建sudo账号，换源

# Softwares Setup
## Pre-installion
- Create new user
    - `sudo useradd yourname`
    - `sudo passwd abc123_`
- Configure sudoer
    - `sudo chmod u+w /etc/sudoers`
    - `sudo vim /etc/sudoers`
    - add new line "yourname ALL=(ALL:ALL) ALL" to the end of the file
    
```
deb http://mirrors.aliyun.com/ubuntu trusty main restricted
deb-src http://mirrors.aliyun.com/ubuntu trusty main restricted
deb http://mirrors.aliyun.com/ubuntu trusty-updates main restricted
deb-src http://mirrors.aliyun.com/ubuntu trusty-updates main restricted
deb http://mirrors.aliyun.com/ubuntu trusty universe
deb-src http://mirrors.aliyun.com/ubuntu trusty universe
deb http://mirrors.aliyun.com/ubuntu trusty-updates universe
deb-src http://mirrors.aliyun.com/ubuntu trusty-updates universe
deb http://mirrors.aliyun.com/ubuntu trusty multiverse
deb-src http://mirrors.aliyun.com/ubuntu trusty multiverse
deb http://mirrors.aliyun.com/ubuntu trusty-updates multiverse
deb-src http://mirrors.aliyun.com/ubuntu trusty-updates multiverse

deb http://mirrors.aliyun.com/ubuntu trusty-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu trusty-backports main restricted universe multiverse
sudo apt-get update
```

- Make sure home directory exists, if not create user home and make default.
    - `ls /home/`

## Mail reader

- 选择使用已存在用户
- mail2, pop端口101

## Vim
- Install
    - `sudo apt-get install vim`
- Verify
    - `vim --version`
- Configuration
    - `export EDITOR=/usr/bin/vim`
    - `alias vi=/usr/bin/vim`
    
# Git
- Install
    - `sudo apt-get install git`
    - `sudo apt-get install git-gui`
- Verify
    - `git version`
- Configure
    - `git config --global user.name "John Doe“ `
    - `git config --global user.email “johndoe@example.com"`

## Chromium
- Install
    - sudo apt-get chromium-browser

## Chinese input method
- Install
    - `sudo apt-get install fcitx-pinyin`
    - 在网上搜 安装sougou拼音

# 2. 拉取代码

git clone git:  workspace

# 3. 安装phpbrew

- 先安装依赖
    - https://github.com/phpbrew/phpbrew/wiki/Requirement#ubuntu-1304--1404-requirement

- Install
```
curl -L -O https://github.com/phpbrew/phpbrew/raw/master/phpbrew
chmod +x phpbrew
sudo mv phpbrew /usr/local/bin/phpbrew
phpbrew init
[[ -e ~/.phpbrew/bashrc ]] && source ~/.phpbrew/bashrc
phpbrew update
```

- Install php5.6
```
phpbrew install 5.6.13 +default+fpm+gd
phpbrew switch 5.6.13
phpbrew ext install mongo
phpbrew ext install redis
phpbrew fpm start
```

# 4. 安装nginx， sublime

## Install sublime text plugin for editorconfig

[Setup Guide](https://github.com/sindresorhus/editorconfig-sublime#readme)

```
cd ~/.config/sublime-text-3/Packages/
git clone https://github.com/inetfuture/sublime-Config User
cd ~/.config/sublime-text-3/Install\ Packages
wget https://sublime/wbond.net/Package%20Control.sublime-package
```
- 最后还要自己安装git, gitBuffer两个插件。

## Setup nginx

- 1. Install nginx
```sh
sudo apt-get install nginx
```

- 2. Config nginx
```sh
vi /etc/nginx/conf.d/baomi.conf
```

- 3. baomi.conf
```
server {
    listen 80;
    server_name localadmin.baomiding.com;
    root  /home/carlgao/workspace/src/frontend/web;
    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location /api {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_pass http://localapi.baomiding.com;
        access_log off;
    }

    # This config is only for wechat oauth.
    location /v1 {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_pass http://localapi.baomiding.com;
        access_log off;
    }

    location /webapp {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_pass http://localhost:82;
        access_log off;
    }

    location /webapp/build/ {
        alias /home/carlgao/workspace/src/webapp/web/build/;
    }

    location /vendor/ {
        alias /home/carlgao/workspace/src/vendor/;
    }

    location /dist/ {
        alias /home/carlgao/workspace/src/web/dist/;
    }

    location ~ .*\.(php|php5)?$ {
        fastcgi_pass   127.0.0.1:9000;
        include        fastcgi_params;
    }

    location ~ /\.(ht|svn|git) {
        deny all;
    }

    access_log /var/log/nginx/baomiding.com-access.log;
    error_log  /var/log/nginx/baomiding.com-error.log;
}

server {
    listen 80;
    server_name localapi.baomiding.com;
    root /home/carlgao/workspace/src/backend/web;
    index index.html index.htm index.php;

    set_real_ip_from 127.0.0.1;
    real_ip_header X-Real-IP;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ .*\.(php|php5)?$ {
        fastcgi_pass   127.0.0.1:9000;
        include        fastcgi_params;
    }

    location ~ /\.(ht|svn|git) {
        deny all;
    }

    access_log /var/log/nginx/baomiding.com-access.log;
    error_log  /var/log/nginx/baomiding.com-error.log;
}

server {
    listen 82;
    server_name localhost;
    root /home/carlgao/workspace/src/webapp/web;
    index index.php index.html index.htm;

    set_real_ip_from 127.0.0.1;
    real_ip_header X-Real-IP;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location /vendor/ {
        alias /home/carlgao/workspace/src/vendor/;
    }

    location /dist/ {
        alias /home/carlgao/workspace/src/web/dist/;
    }
    
    location ~ .*\.(php|php5)?$ {
        fastcgi_pass 127.0.0.1:9000;
        include fastcgi_params;
    }

    location ~ /\.(ht|svn|git) {
        deny all;
    }

    location ~ /.+\.(coffee|scss) {
        deny all;
    }

    access_log /var/log/nginx/baomiding-webapp-access.log;
    error_log /var/log/nginx/baomiding-webapp-error.log;
}

```

## Setup Mongo

### Setup with pecl

- 1. Install Mongo
   - Follow the [setup guide](http://docs.mongodb.org/manual/tutorial/install-mongodb-on-ubuntu/), and select the 2.6 version to install.

- 2. 把1中的步骤写下来(这里的mongo是3.2版本)
```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927
echo "deb http://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list
sudo apt-get update
sudo apt-get install -y mongodb-org
sudo apt-get install -y mongodb-org=3.2.7 mongodb-org-server=3.2.7 mongodb-org-shell=3.2.7 mongodb-org-mongos=3.2.7 mongodb-org-tools=3.2.7
sudo service mongod start
```

## Setup Redis

1. Install Redis
```sh
sudo apt-get install redis-server
```

## Setup SCSS

1. Install ruby (version > 1.9.1)
```sh
sudo apt-get install ruby
2. Use mirror for ruby
```
gem sources --remove http://rubygems.org/
gem sources -a https://ruby.taobao.org/
gem sources -l
*** CURRENT SOURCES ***
https://ruby.taobao.org

Ensure only ruby.taobao.org exists
```

3. Install SASS
```sh
sudo gem install sass
```

## Setup grunt and bower

1. Install nodejs
```sh
curl https://raw.githubusercontent.com/creationix/nvm/v0.25.1/install.sh | bash
. ~/.profile
nvm install v0.12.2
nvm use 0.12.2
```

2. Install grunt
```sh
sudo apt-get install npm
sudo npm install -g grunt-cli
```

## Initialize project

```sh
cd src
./initDev
cd ..
./updateModules
```

## Run grunt task

- 在运行grunt的时候要换一下node的版本
```sh
cd src
npm install
sudo apt-get install node
grunt
```

# 5. 安装supervisor

1. Install supervisor
```sh
sudo apt-get install supervisor
```

2. Config supervisor
```sh
sudo vi /etc/supervisor/conf.d/supervisor.conf
```

Add below configuration to the file, **Change the folder name to your own project (/home/user/aug-marketing)**

```sh
[program:scheduler]
process_name=%(program_name)s_%(process_num)02d
directory=/home/user/aug-marketing
command=php /home/user/aug-marketing/src/backend/modules/resque/components/bin/resque-scheduler
numprocs=1
redirect_stderr=True
autostart=True
autorestart= True
environment=QUEUE='global',LOGGING='1',APP_INCLUDE='/home/user/aug-marketing/src/backend/modules/resque/components/lib/Resque/RequireFile.php'
stdout_logfile=/var/log/supervisor/%(program_name)s-stdout.log
stderr_logfile=/var/log/supervisor/%(program_name)s-stderr.log

[program:global]
process_name=%(program_name)s_%(process_num)02d
directory=/home/user/aug-marketing
command=php /home/user/aug-marketing/src/backend/modules/resque/components/bin/resque
numprocs=5
redirect_stderr=True
autostart=True
autorestart= True
environment=QUEUE='global',LOGGING='1',APP_INCLUDE='/home/user/aug-marketing/src/backend/modules/resque/components/lib/Resque/RequireFile.php'
stdout_logfile=/var/log/supervisor/%(program_name)s-stdout.log
stderr_logfile=/var/log/supervisor/%(program_name)s-stderr.log
```

**Restart supervisor to make the configuration work**

```sh
sudo service supervisor {start|stop|restart|force-reload|status|force-stop}
```



