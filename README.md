                                                  记录搭建fabric1.1搭建过程
1.Docker安装
如果在服务器上有旧版的docker，需要先执行卸载操作，如下：
sudo yum remove docker  docker-common  docker-selinux  docker-engine
随后开始安装Docker CE，命令如下：
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum-config-manager --enable docker-ce-edge
sudo yum-config-manager --enable docker-ce-test
sudo yum-config-manager --disable docker-ce-edge
sudo yum makecache fast
sudo yum install docker-ce
执行查询docker版本号，看是否安装成功
docker --version
然后将docker启动：service docker start，可设置docker为开机自启动：chkconfig docker on

2.Docker-Compose安装
需要服务器支持curl功能，如果服务器不支持curl，需要执行如下操作安装curl依赖：yum install curl
执行如下操作下载docker-compose：
curl -L https://github.com/docker/compose/releases/download/1.15.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
该下载目录为/usr/local/bin/docker-compose，且权限已经给出，再执行docker-compose --version检查版本号

3.Go语言安装
执行以下操作下载最新版Go语言包：
curl -O https://storage.googleapis.com/golang/go1.8.3.linux-amd64.tar.gz
解压go1.8.3.linux-amd64.tar.gz至/usr/local目录下，执行如下操作：
tar -C /usr/local -xzf go1.8.3.linux-amd64.tar.gz
配置go环境变量，修改/etc/profile文件使其永久性生效，并对所有系统用户生效，在文件末尾加上如下两行代码：
export PATH=$PATH:/usr/local/go/bin
export GOPATH=/opt/gopath
执行修改后，继续执行：source profile 使其修改生效。随后可通过命令：echo $PATH 查看是否添加成功。最后可通过命令go version查看当前go版本信息
至此整个Fabric所需的基础环境都已经搭建起来了，这种搭建方式是有网络的情况下成立的。

为了加速docker拉取镜像，可配置国内镜像：
$ mkdir -p /etc/docker/
$ cat <<EOF > /etc/docker/daemon.json
{
  "registry-mirrors": ["https://registry.docker-cn.com"]
}
EOF

$ systemctl enable docker
$ systemctl restart docker
$ docker info
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: 18.03.0-ce
....
Registry Mirrors:
 https://registry.docker-cn.com  # ------- 确认这里生效
Live Restore Enabled: false

4.下载二进制文件和docker镜像
准备工作目录为 fabric
$ cd ~
$ mkdir fabric
$ cd fabric
# 下载1.1 版本的bootstrap.sh 脚本
$ wget https://raw.githubusercontent.com/hyperledger/fabric/release-1.1/scripts/bootstrap.sh
$ bash bootstrap.sh

bootstrap 脚本会执行如下动作：
1. 下载预编译好的fabric和fabric-ca
2. 拉取fabric 1.1.0 的docker镜像
3. 拉取fabric 依赖的第三方软件镜像，如couchdb，kafka，zookeeper，版本号都是0.4.6

执行完成后查看当前目录的bin，应该会有如下文件
ls bin/ -lh
总用量 146M
-rwxrwxr-x 1 1001 1001 23M 3月  16 06:13 configtxgen
-rwxrwxr-x 1 1001 1001 24M 3月  16 06:13 configtxlator
-rwxrwxr-x 1 1001 1001 12M 3月  16 06:13 cryptogen
-rwxrwxr-x 1 1001 1001 20M 3月  16 09:33 fabric-ca-client
-rwxrwxr-x 1 1001 1001 757 4月  11 16:00 get-docker-images.sh
-rwxrwxr-x 1 1001 1001 31M 3月  16 06:14 orderer
-rwxrwxr-x 1 1001 1001 38M 3月  16 06:14 peer

5.运行 fabric-sample
$ yum install -y git
$ git clone https://github.com/hyperledger/fabric-samples.git
$ mv bin fabric-samples/
$ cd fabric-samples
$ git checkout v1.1.0
$ cd first-network/
接下来的内容和官方教程就很像了：
./byfn.sh -m generate
Generating certs and genesis block for with channel 'mychannel' and CLI timeout of '10' seconds and CLI delay of '3' seconds
Continue? [Y/n]
proceeding ...
/root/fabric/fabric-samples/bin/cryptogen

##########################################################
##### Generate certificates using cryptogen tool #########
##########################################################
+ cryptogen generate --config=./crypto-config.yaml
org1.example.com
org2.example.com
+ res=0
+ set +x

/root/fabric/fabric-samples/bin/configtxgen
##########################################################
#########  Generating Orderer Genesis block ##############
##########################################################
+ configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block
2018-04-11 16:21:21.523 CST [common/tools/configtxgen] main -> INFO 001 Loading configuration
2018-04-11 16:21:21.531 CST [msp] getMspConfig -> INFO 002 Loading NodeOUs
2018-04-11 16:21:21.531 CST [msp] getMspConfig -> INFO 003 Loading NodeOUs
2018-04-11 16:21:21.531 CST [common/tools/configtxgen] doOutputBlock -> INFO 004 Generating genesis block
2018-04-11 16:21:21.532 CST [common/tools/configtxgen] doOutputBlock -> INFO 005 Writing genesis block
+ res=0
+ set +x
....

$ ./byfn.sh -m up -i 1.1.0
Starting with channel 'mychannel' and CLI timeout of '10' seconds and CLI delay of '3' seconds
Continue? [Y/n]
proceeding ...
2018-04-11 09:03:16.069 UTC [main] main -> INFO 001 Exiting.....
LOCAL_VERSION=1.1.0
DOCKER_IMAGE_VERSION=1.1.0
/root/fabric/fabric-samples/bin/cryptogen

##########################################################
##### Generate certificates using cryptogen tool #########
##########################################################
+ cryptogen generate --config=./crypto-config.yaml
...
...
===================== Query on peer1.org2 on channel 'mychannel' is successful =====================

========= All GOOD, BYFN execution completed ===========


 _____   _   _   ____
| ____| | \ | | |  _ \
|  _|   |  \| | | | | |
| |___  | |\  | | |_| |
|_____| |_| \_| |____/

查看运行的docker容器:docker ps

5.使用cli
docker exec -it cli /bin/bash -c "export COLUMNS=`tput cols`; export LINES=`tput lines`; exec bash"
查看已经安装的chaincode：
# peer chaincode list --installed
Get installed chaincodes on peer:
Name: mycc, Version: 1.0, Path: github.com/chaincode/chaincode_example02/go/, Id: 476fca1a949274001971f1ec2836cb09321f0b71268b3762d68931c93f218134
2018-04-11 09:14:26.305 UTC [main] main -> INFO 001 Exiting.....
查询账户余额：
# peer chaincode query -C mychannel -n mycc -c '{"Args": ["query", "a"]}'
2018-04-11 09:15:07.853 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-04-11 09:15:07.853 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
Query Result: 90
2018-04-11 09:15:07.857 UTC [main] main -> INFO 003 Exiting.....

# peer chaincode query -C mychannel -n mycc -c '{"Args": ["query", "b"]}'
2018-04-11 09:15:39.147 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-04-11 09:15:39.147 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
Query Result: 210
2018-04-11 09:15:39.152 UTC [main] main -> INFO 003 Exiting.....

转账
# peer chaincode invoke   --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem  -C mychannel -n mycc -c '{"Args":["invoke","a","b","90"]}'
2018-04-11 09:16:44.028 UTC [chaincodeCmd] InitCmdFactory -> INFO 001 Get chain(mychannel) orderer endpoint: orderer.example.com:7050
2018-04-11 09:16:44.032 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default escc
2018-04-11 09:16:44.032 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 003 Using default vscc
2018-04-11 09:16:44.038 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 004 Chaincode invoke successful. result: status:200
2018-04-11 09:16:44.039 UTC [main] main -> INFO 005 Exiting.....
查询a账户余额
# peer chaincode query -C mychannel -n mycc -c '{"Args": ["query", "a"]}'
2018-04-11 09:17:23.511 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-04-11 09:17:23.511 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
Query Result: 0
2018-04-11 09:17:23.516 UTC [main] main -> INFO 003 Exiting.....
