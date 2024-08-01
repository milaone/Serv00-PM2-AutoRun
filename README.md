# Serv00-PM2-Autorun

本项目借鉴了kuoihao仓库Serv00_Auto_Run的思路利用自动工作流进行远程运行脚本

原库地址https://github.com/kuoihao/Serv00_Auto_Run


# 功能
旨在通过判断Web服务是否运行，否则通过远程方式登录Serv00的ssh运行脚本以恢复PM2快照 
本项目分三种方式来解决

## 方法一：利用vps远程运行serv00中的pm2保活命令

### Serv00服务器端
#### 在Serv00中编写恢复快照脚本
```
cd ~
touch run.sh
chmod +x run.sh

##写入命令（或者自己复制进去）：
cat << 'EOF' > run.sh
#!/bin/sh
~/.npm-global/bin/pm2 kill
/home/dino/.npm-global/bin/pm2 resurrect >/dev/null 2>&1
sleep 10
/home/dino/.npm-global/bin/pm2 restart all
~/.npm-global/bin/pm2 save
EOF
```
#### -Serv00中生成密钥对
```
ssh-keygen -t rsa -b 4096 -C "dino@milaone.app"
```
#### -将公钥并添加到authorized_keys中
    
```
cat ~/.ssh/id_rsa.pub | tee -a ~/.ssh/authorized_keys
```
#### -打印私钥内容，复制私钥全部内容
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
-----END OPENSSH PRIVATE KEY-----
```

- -----BEGIN OPENSSH PRIVATE KEY-----
- -----END OPENSSH PRIVATE KEY-----

这两行也要包含进来
### VPS端
#### -配置私钥
```
cd ~
nano .ssh/serv00key
#把私钥中内容粘贴进来，保存退出
chmod 600 .ssh/serv00key
```
#### -远程探测、PM2恢复快照脚本

```
cd ~
touch keep.sh
chmod +x keep.sh
```

keep.sh的内容，直接复制粘贴
```
#!/bin/bash
# 定义要检测的URL
URL="https://memos.milaone.app"

# 定义运行脚本的路径
RUN_SCRIPT="/home/dino/run.sh"

# 检查服务是否运行
curl --head --silent --fail $URL > /dev/null
SERVICE_STATUS=$?
if [ $SERVICE_STATUS -eq 0 ]; then
    echo "Service is running."
else
    echo "Service is not running. Starting the service..."
    bash $RUN_SCRIPT
fi

```




### 利用openwrt远程check https://memos.milaone.app 的运行状态，出错就ssh登录Serv00的ssh运行脚本



### 利用github的Actions中自动工作流脚本每5分钟check一下 https://memos.milaone.app 的运行状态，出错就ssh登录运行脚本