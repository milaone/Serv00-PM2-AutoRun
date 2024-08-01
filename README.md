# Serv00-PM2-Autorun

本项目借鉴了kuoihao仓库Serv00_Auto_Run的思路利用自动工作流进行远程运行脚本

原库地址https://github.com/kuoihao/Serv00_Auto_Run


### 功能
旨在通过判断Web服务是否运行，否则通过远程方式登录Serv00的ssh运行脚本以恢复PM2快照 
本项目分三种方式来解决

### 利用vps远程运行serv00中的pm2保活命令

#### -Serv00中生成密钥对
```
ssh-keygen -t rsa -b 4096 -C "dino@milaone.app"
```
#### -将公钥并添加到authorized_keys中
    
```
cat ~/.ssh/id_rsa.pub | tee -a ~/.ssh/authorized_keys
```
#### -打印私钥内容，复制
```
cat ~/.ssh/id_rsa
```
复制私钥内容作为SSH_PRIVATE_KEY的值,需要包括
-----BEGIN OPENSSH PRIVATE KEY-----
-----END OPENSSH PRIVATE KEY-----这两行


### 利用openwrt远程check https://memos.milaone.app 的运行状态，出错就ssh登录Serv00的ssh运行脚本



### 利用github的Actions中自动工作流脚本每5分钟check一下 https://memos.milaone.app 的运行状态，出错就ssh登录运行脚本