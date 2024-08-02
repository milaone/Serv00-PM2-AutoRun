# Serv00-PM2-AutoRun

本项目借鉴了kuoihao仓库Serv00_Auto_Run的思路利用自动工作流进行远程运行脚本

原库地址https://github.com/kuoihao/Serv00_Auto_Run


# 功能
旨在通过判断Web服务是否运行，否则通过远程方式登录Serv00的ssh运行脚本以恢复PM2快照 
本项目分三种方式来解决

## 1.方法一：利用vps远程监控,并保活serv00中的PM2

### 1.1 Serv00服务器端
#### 1.1.1在Serv00中编写PM2恢复快照脚本
```
cd ~
touch run.sh
chmod +x run.sh


#run.sh中粘贴下面脚本

#!/bin/sh
~/.npm-global/bin/pm2 kill
/home/dino/.npm-global/bin/pm2 resurrect >/dev/null 2>&1
sleep 10
/home/dino/.npm-global/bin/pm2 restart all
~/.npm-global/bin/pm2 save

```
#### 1.1.2 Serv00中生成密钥对
```
ssh-keygen -t rsa -b 4096 -C "dino@milaone.app"
```
#### 1.1.3 将公钥并添加到authorized_keys中
    
```
cat ~/.ssh/id_rsa.pub | tee -a ~/.ssh/authorized_keys
```
#### 1.1.4打印私钥内容，复制私钥全部内容
```
cat ~/.ssh/id_rsa
```
复制私钥内容,需要包括全部内容
```
-----BEGIN OPENSSH PRIVATE KEY-----
xxxxxxxxxxxxxxxxxxx
xxxxxxxxxxxxxxxxxxx
xxxxxxxxxxxxxxxxxxx
xxxxxxxxxxxxxxxxxxx
xxxxxxxxxxxxxxxxxxx
xxxxxxxxxxxxxxxxxxx
xxxxxxxxxxxxxxxxxxx
xxxxxxxxxxxxxxxxxxx
-----END OPENSSH PRIVATE KEY-----
```

- -----BEGIN OPENSSH PRIVATE KEY-----
- -----END OPENSSH PRIVATE KEY-----

这两行也要包含进来
### 1.2 VPS端
#### 1.2.1配置私钥
```
cd ~
nano .ssh/serv00key
#把私钥中内容粘贴进来，保存退出
chmod 600 .ssh/serv00key
```
#### 1.2.2远程探测、PM2恢复快照脚本

```
cd ~
touch keep.sh
chmod +x keep.sh

```

keep.sh的内容，直接复制粘贴
```
#!/bin/bash

# 目标URL
URL="https://memos.milaone.app"

# 远程运行的脚本命令
RUN_SCRIPT="ssh -i .ssh/serv00key dino@s4.serv00.com '/home/dino/run.sh'"

# 检查服务是否运行
curl --head --silent --fail $URL > /dev/null
SERVICE_STATUS=$?

if [ $SERVICE_STATUS -eq 0 ]; then
    echo "Service is running."
else
    echo "Service is not running. Starting the service..."
    $RUN_SCRIPT
fi

```
#### 1.2.3设置Cron计划任务执行脚本
```
sudo crontab -e
#加入下面任务，keep.sh文件完整路径
*/2 * * * * /home/ubuntu/keep.sh >/dev/null 2>&1
```
---
---

## 2. 方法二：利用openwrt远程check https://memos.milaone.app 的运行状态，出错就ssh登录Serv00的ssh运行脚本
整体思路跟方法一是一样的，只是openwrt中的ssh是dropbear提供的，它不能使用serv00生成的证书
其实说这个方法，我就是为了说一下openwrt下的ssh是特殊的，大家注意一下
### 2.1 在Serv00中部分
#### 2.1.1 openwrt中生成密钥对
```
dropbearkey -f ~/.ssh/id_dropbear -t rsa -s 2048
```
默认私钥就已经保存到了/root/.ssh/id_dropbear中了，后面我们直接使用即可

显示公钥就出现在屏幕中，我们复制下来，

#### 2.1.2 Serv00中添加公钥
编辑~/.ssh/authorized_keys文件，粘贴在文件末尾即可

#### 2.1.3 在Serv00中编写恢复快照脚本
```
cd ~
touch run.sh
chmod +x run.sh


#把下面代码粘贴进去

#!/bin/sh
~/.npm-global/bin/pm2 kill
/home/dino/.npm-global/bin/pm2 resurrect >/dev/null 2>&1
sleep 10
/home/dino/.npm-global/bin/pm2 restart all
~/.npm-global/bin/pm2 save

```

### 2.2 下面部分OpenWrt中创建脚本，添加crontab计划任务
#### 2.2.1 远程探测、PM2恢复快照脚本

```
cd ~
touch keep.sh
chmod +x keep.sh

```

#keep.sh的内容，直接粘贴进去
```
#!/bin/bash

# 目标URL
URL="https://memos.milaone.app"

# 远程运行的脚本命令
RUN_SCRIPT="ssh -i /root/.ssh/id_dropbear dino@s4.serv00.com '/home/dino/run.sh'"

# 检查服务是否运行
curl --head --silent --fail $URL > /dev/null
SERVICE_STATUS=$?

if [ $SERVICE_STATUS -eq 0 ]; then
    echo "Service is running."
else
    echo "Service is not running. Starting the service..."
    $RUN_SCRIPT
fi

```
#### 2.2.2 设置Cron计划任务执行脚本
```
crontab -e
#加入下面任务，keep.sh文件完整路径
*/2 * * * * sh /root/keep.sh >/dev/null 2>&1
```

---
---


## 3. 方法三：利用github的Actions中自动工作流脚本每5分钟check一下 https://memos.milaone.app 的运行状态，出错就ssh登录运行脚本

### 3.0 在Serv00中编写PM2恢复快照脚本
```
cd ~
touch run.sh
chmod +x run.sh


#run.sh中粘贴下面脚本

#!/bin/sh
~/.npm-global/bin/pm2 kill
/home/dino/.npm-global/bin/pm2 resurrect >/dev/null 2>&1
sleep 10
/home/dino/.npm-global/bin/pm2 restart all
~/.npm-global/bin/pm2 save

```
### 3.1 serv00生成密钥对，添加公钥到服务器、并且拿到私钥
serv00中运行，生成密钥对
```
ssh-keygen -t rsa -b 4096 -C "dino@milaone.app"
``` 
将公钥并添加到authorized_keys中
    
```
cat ~/.ssh/id_rsa.pub | tee -a ~/.ssh/authorized_keys
```

打印私钥内容，复制私钥全部内容
```
cat ~/.ssh/id_rsa
```
复制私钥内容作为SSH_PRIVATE_KEY的值,需要包括全部内容
```
-----BEGIN OPENSSH PRIVATE KEY-----
xxxxxxxxxxxxxxxxxxx
xxxxxxxxxxxxxxxxxxx
xxxxxxxxxxxxxxxxxxx
xxxxxxxxxxxxxxxxxxx
xxxxxxxxxxxxxxxxxxx
xxxxxxxxxxxxxxxxxxx
xxxxxxxxxxxxxxxxxxx
xxxxxxxxxxxxxxxxxxx
-----END OPENSSH PRIVATE KEY-----
```

- -----BEGIN OPENSSH PRIVATE KEY-----
- -----END OPENSSH PRIVATE KEY-----
这两行也要包含进来

### 3.2 fork本仓库到您的github账户下
### 3.3 配置密钥
在fork后的仓库中settings--> Security --> Secrets and variables --> Actions-->New repository secret按钮
```
Name：SSH_PRIVATE_KEY
Secret： 这里填刚复制出来的私钥
```
Add secret 按钮提交

### 3.4 修改仓库中
Serv00-PM2-Autorun/.github/workflows目录下的main.yml
其中需要修改的地方我都进行注释。
```
name: autorun

on:
  workflow_dispatch:
  schedule:
    # 5分钟运行一次
    - cron: '0/5 * * * *'
    

jobs:
  ssh-and-run:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout
      uses: actions/checkout@v2

    - name: Setup SSH
      uses: webfactory/ssh-agent@v0.5.3
      with:
        ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

    - name: Check if service is running
      id: check_service
      run: |
        if curl --head --silent --fail https://memos.milaone.app > /dev/null; then #这里改成你的Web服务的地址
          echo "service_status=running" >> $GITHUB_ENV
        else
          echo "service_status=not_running" >> $GITHUB_ENV
        fi

    - name: Run script on remote server if service is not running
      if: env.service_status == 'not_running'
      run: ssh -o StrictHostKeyChecking=no dino@s4.serv00.com "/home/dino/run.sh" #这里改成你的用户名@你的ssh服务器地址，以及/home/你的用户名/run.sh
```

### 3.5 打开Actions中的自动工作流即可