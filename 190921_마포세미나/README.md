# Vagrant를 이용하여 하이퍼레저 패브릭 VM 구성하기

[TOC]

## 0. Vagrant?

Vagrant는 개발 환경을 구축하고 배포하기 위한 도구입니다.

Vagrant에서 관리하는 개발 환경은 VirtualBox 또는 VMware와 같은 로컬 가상화 플랫폼, AWS 또는 OpenStack을 통한 클라우드 또는 Docker 또는 원시 LXC와 같은 컨테이너에서 실행할 수 있습니다.

Vagrant는 완전한 휴대용 개발 환경을 만들고 관리하기 위한 프레임워크와 구성 형식을 제공합니다. 이러한 개발 환경은 컴퓨터나 클라우드에서 사용할 수 있으며, Windows, Mac OS X 및 Linux 간에 이동 가능합니다.

* https://www.vagrantup.com



## 1. VirtualBox 설치

아래 링크에서 본인 노트북 OS에 맞는 프로그램을 다운로드 받아 설치합니다.

- https://www.virtualbox.org/wiki/Downloads



## 2. Vagrant 설치

아래 링크에서 본인 노트북 OS에 맞는 프로그램을 다운로드 받아 설치합니다.

- https://www.vagrantup.com/downloads.html



## 3. Vagrant 설치 확인 및 플러그인 설치

정상적으로 설치가 되었는 지 버전 확인을 합니다. 2.x 버전을 권장합니다.

```shell
$ vagrant version
Installed Version: 2.2.5
Latest Version: 2.2.5
```

VM에 할당하는 디스크 용량을 지정하기 위해 vagrant-disksize 플러그인을 설치합니다.

기본적으로 10G이상이 자동적으로 할당되며 vagrant-disksize 플러그인을 통해서 임의의 디스크 용량을 할당할 수 있습니다. 단, 줄일 수는 없습니다.

- https://github.com/sprotheroe/vagrant-disksize

```shell
$ vagrant plugin install vagrant-disksize
Installing the 'vagrant-disksize' plugin. This can take a few minutes...
Fetching: vagrant-disksize-0.1.3.gem (100%)
Installed the plugin 'vagrant-disksize (0.1.3)'!
```

 

<u>**※ VMWare가 이미 설치되어있는 경우 vagrant 명령어에 --provider virtualbox 옵션을 추가해야 합니다.**</u> 

예> $ vagrant box add **<u>*--provider virtualbox*</u>** bento/ubuntu-18.04

## 4. Box(Image) 다운로드

Ubuntu 18.04 LTS Box(Image)를 다운 받습니다. Docker Container구동을 위해 Docker Image를 다운 받는것과 동일합니다.

```shell
$ vagrant box add --provider virtualbox bento/ubuntu-18.04
==> box: Loading metadata for box 'bento/ubuntu-18.04'
    box: URL: https://vagrantcloud.com/bento/ubuntu-18.04
==> box: Adding box 'bento/ubuntu-18.04' (v201906.18.0) for provider: virtualbox
    box: Downloading: https://vagrantcloud.com/bento/boxes/ubuntu-18.04/versions/201906.18.0/providers/virtualbox.box
    box: Download redirected to host: vagrantcloud-files-production.s3.amazonaws.com
==> box: Successfully added box 'bento/ubuntu-18.04' (v201906.18.0) for 'virtualbox'!

# Box(Image) 목록 확인
$ vagrant box list
bento/ubuntu-18.04 (virtualbox, 201906.18.0)
```



## 5. Vagrantfile 다운로드

Vagrantfile에 VM 형식, 설정, 프로비전등을 설정하는 설정 파일입니다. (Dockerfile과 비슷하다고 보면 됩니다.)

아래 파일을 본인 노트북 특정 위치에 다운로드 받습니다. 

* <u>***본 가이드에서는 Hyperledger Fabric 1.4.x 의 Fabric Samples 설치를 대상으로 설명합니다. 다른 버전 설치가 필요한 경우 해당 버전에 맞는 Vagrantfile을 사용하시면 됩니다.***</u> 
* Vagrantfile
  * https://github.com/hlkug/meetup/tree/master/000000/vagrant/hyperledger_fabric/1.4.x/Vagrantfile
  * https://github.com/hlkug/meetup/tree/master/000000/vagrant/hyperledger_fabric/2.0.x/Vagrantfile
* Vagrantfile 설명
  * VM 수: vm_num
  * VM 형식: config.vm.box
  * VM CPU(core수): node_cpu
  * VM 메모리(G): node_memroy
  * VM Network: node_network, 예> 10.10.10.0/24
  * VM 호스트명(Prefix): node_prefix + index, 예> node1-1, node1-2
  * VM IP: node_network + index, 예> 10.10.10.1, 10.10.10.2
  * VM 디스크: config.disksize.size, 10G이상만 가능합니다. 

아래는 1Core, 2G 메모리, 10G 디스크를 가지는 VM 한개를 생성하는 샘플 Vagrantfile입니다.(Hyperledger Fabric 1.4.x)

```ruby
ENV["LC_ALL"] = "en_US.UTF-8"

Vagrant.configure("2") do |config|
  vm_num = 1
  node_cpu = 1 # 1Core
  node_memory = "2048" # 2G Memory
  node_network = "10.10.10"
  node_prefix = "node"
  
  config.vm.box = "bento/ubuntu-18.04"
  config.vm.box_check_update = false
  config.disksize.size = "10GB" # > 10GB

  (1..vm_num).each do |i|
    config.vm.define "#{node_prefix}1-#{i}" do |node|
      hostname = "#{node_prefix}1-#{i}"
      hostip = "#{node_network}.#{i + 1}"

      node.vm.hostname = hostname
      node.vm.network "private_network", ip: hostip

      node.vm.provider "virtualbox" do |vb|
        vb.name = "#{node_prefix}1-#{i}"
        vb.gui = false
        vb.cpus = node_cpu
        vb.memory = node_memory
      end
    end
  end

  config.vm.provision "shell", inline: <<-EOF
    	apt-get update
      	apt-get upgrade

    	# Install Go
    	wget https://dl.google.com/go/go1.12.9.linux-amd64.tar.gz
            tar zxf go1.12.9.linux-amd64.tar.gz
            mv go /usr/local
            rm go1.12.9.linux-amd64.tar.gz

    	# Install Docker
            apt-get -y install apt-transport-https ca-certificates curl software-properties-common
    	curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    	add-apt-repository \
    	   "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
    	apt-get update
    	apt-get -y install docker-ce
            usermod -aG docker vagrant

    	# Install Docker Compose
    	curl -L https://github.com/docker/compose/releases/download/1.24.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
    	chmod +x /usr/local/bin/docker-compose
            echo "PATH=$PATH:/usr/local/go/bin" >> /etc/profile
    	
    	# Install Hyperledger Fabric Samples, Binaries and Docker Images
            curl -sSL http://bit.ly/2ysbOFE | bash -s -- 1.4.3 1.4.3 0.4.15
    	chown -R vagrant:vagrant fabric-samples

      # Install Node.js 8.x
      curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
      sudo apt-get install -y nodejs
  EOF
end
```



## 6. VM(Guest Machine) 생성

Vagrantfile을 기반으로 VM(Guest Machine)을 생성합니다. VM 생성 중 Hyperledger Fabric 1.4.x 에 포함된 fabric-samples 구동을 위한 환경까지 설정됩니다. VM생성시 몇분이 소요됩니다. 

* Go, Docker, Docker Compose, Hyperledger Fabric Docker Image & Sample(1.4.x) 설치

```shell
$ ls 
Vagrantfile
$ vagrant up
Bringing machine 'node1-1' up with 'virtualbox' provider...
==> node1-1: Importing base box 'bento/ubuntu-18.04'...
==> node1-1: Matching MAC address for NAT networking...
==> node1-1: Setting the name of the VM: node1
==> node1-1: Clearing any previously set network interfaces...
==> node1-1: Preparing network interfaces based on configuration...
    node1-1: Adapter 1: nat
    node1-1: Adapter 2: hostonly
==> node1-1: Forwarding ports...
    node1-1: 22 (guest) => 2222 (host) (adapter 1)
==> node1-1: Running 'pre-boot' VM customizations...
==> node1-1: Booting VM...
==> node1-1: Waiting for machine to boot. This may take a few minutes...
    node1-1: SSH address: 127.0.0.1:2222
...
node1-1: ===> List out hyperledger docker images
    node1-1: hyperledger/fabric-javaenv     1.4.3               1cd707531ce7        3 weeks ago         1.76GB
    node1-1: hyperledger/fabric-javaenv     latest              1cd707531ce7        3 weeks ago         1.76GB
    node1-1: hyperledger/fabric-ca          1.4.3               f289675c9874        3 weeks ago         253MB
    node1-1: hyperledger/fabric-ca          latest              f289675c9874        3 weeks ago         253MB
    node1-1: hyperledger/fabric-tools       1.4.3               0abc124a9400        3 weeks ago         1.55GB
    node1-1: hyperledger/fabric-tools       latest              0abc124a9400        3 weeks ago         1.55GB
    node1-1: hyperledger/fabric-ccenv       1.4.3               fc0f502399a6        3 weeks ago         1.43GB
    node1-1: hyperledger/fabric-ccenv       latest              fc0f502399a6        3 weeks ago         1.43GB
    node1-1: hyperledger/fabric-orderer     1.4.3               362021998003        3 weeks ago         173MB
    node1-1: hyperledger/fabric-orderer     latest              362021998003        3 weeks ago         173MB
    node1-1: hyperledger/fabric-peer        1.4.3               d79f2f4f3257        3 weeks ago         178MB
    node1-1: hyperledger/fabric-peer        latest              d79f2f4f3257        3 weeks ago         178MB
    node1-1: hyperledger/fabric-zookeeper   0.4.15              20c6045930c8        4 months ago        1.43GB
    node1-1: hyperledger/fabric-zookeeper   latest              20c6045930c8        4 months ago        1.43GB
    node1-1: hyperledger/fabric-kafka       0.4.15              b4ab82bbaf2f        4 months ago        1.44GB
    node1-1: hyperledger/fabric-kafka       latest              b4ab82bbaf2f        4 months ago        1.44GB
    node1-1: hyperledger/fabric-couchdb     0.4.15              8de128a55539        4 months ago        1.5GB
    node1-1: hyperledger/fabric-couchdb     latest              8de128a55539        4 months ago        1.5GB
    node1-1: Unpacking nodejs (8.16.1-1nodesource1) ...
    node1-1: Setting up nodejs (8.16.1-1nodesource1) ...
    node1-1: Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
    
# VM 상태 확인
$ vagrant status
Current machine states:

node1-1                     running (virtualbox)

The VM is running. To stop this VM, you can run `vagrant halt` to
shut it down forcefully, or you can run `vagrant suspend` to simply
suspend the virtual machine. In either case, to restart it again,
simply run `vagrant up`.
```



## 7. VM(Guest Machine) 접속

```shell
$ vagrant ssh node1-1
Welcome to Ubuntu 18.04.2 LTS (GNU/Linux 4.15.0-51-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon Sep 16 21:23:54 UTC 2019

  System load:  0.8               Users logged in:        0
  Usage of /:   8.0% of 61.80GB   IP address for eth0:    10.0.2.15
  Memory usage: 15%               IP address for eth1:    10.10.10.2
  Swap usage:   0%                IP address for docker0: 172.17.0.1
  Processes:    94


92 packages can be updated.
55 updates are security updates.


vagrant@node1-1:~$ ls
fabric-samples
vagrant@node1-1:~$ exit

$
```



## 8. VM(Guest Machine) 중지

```shell
$ vagrant halt node1-1
==> node1-1: Attempting graceful shutdown of VM...

$ vagrant status
Current machine states:

node1-1                     poweroff (virtualbox)

The VM is powered off. To restart the VM, simply run `vagrant up`
```



## 9. VM(Guest Machine) 재시작

```shell
$ vagrant up node1-1
Bringing machine 'node1-1' up with 'virtualbox' provider...
==> node1-1: Clearing any previously set forwarded ports...
==> node1-1: Clearing any previously set network interfaces...
==> node1-1: Preparing network interfaces based on configuration...
    node1-1: Adapter 1: nat
    node1-1: Adapter 2: hostonly
==> node1-1: Forwarding ports...
    node1-1: 22 (guest) => 2222 (host) (adapter 1)
==> node1-1: Running 'pre-boot' VM customizations...
==> node1-1: Booting VM...
==> node1-1: Waiting for machine to boot. This may take a few minutes...
    node1-1: SSH address: 127.0.0.1:2222
    node1-1: SSH username: vagrant
    node1-1: SSH auth method: private key
==> node1-1: Machine booted and ready!
==> node1-1: Checking for guest additions in VM...
==> node1-1: Setting hostname...
==> node1-1: Configuring and enabling network interfaces...
==> node1-1: Mounting shared folders...
    node1-1: /vagrant => /Users/yunho.chung/Vagrant/fabric
==> node1-1: Machine already provisioned. Run `vagrant provision` or use the `--provision`
==> node1-1: flag to force provisioning. Provisioners marked to run always will still run.

$ vagrant status
Current machine states:

node1-1                     running (virtualbox)

The VM is running. To stop this VM, you can run `vagrant halt` to
shut it down forcefully, or you can run `vagrant suspend` to simply
suspend the virtual machine. In either case, to restart it again,
simply run `vagrant up`.
```



## 10. VM(Guest Machine) 삭제

```shell
$ vagrant destroy node1-1
    node1-1: Are you sure you want to destroy the 'node1-1' VM? [y/N] y
==> node1-1: Forcing shutdown of VM...
==> node1-1: Destroying VM and associated drives...

$ vagrant status
Current machine states:

node1-1                     not created (virtualbox)

The environment has not yet been created. Run `vagrant up` to
create the environment. If a machine is not created, only the
default provider will be shown. So if a provider is not listed,
then the machine is not created for that environment.
```


https://hyperledger-fabric.readthedocs.io/en/latest/whatsnew.html



# 3. Token CLI 설정 파일 생성

FabToken 실습에서 사용할 아래 3개 파일을 /tmp 디렉토리에 생성합니다.

* [configorg1.json](https://github.com/hlkug/meetup/tree/master/201906/config/configorg1.json)

```json
{
  "ChannelID":"",
  "MSPInfo":{
    "MSPConfigPath":"",
    "MSPID":"Org1MSP",
    "MSPType":"bccsp"
  },
  "Orderer":{
    "Address":"orderer.example.com:7050",
    "ConnectionTimeout":0,
    "TLSEnabled":true,
    "TLSRootCertFile":"/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem",
    "ServerNameOverride":""
  },
  "CommitterPeer":{
    "Address":"peer0.org1.example.com:7051",
    "ConnectionTimeout":0,
    "TLSEnabled":true,
    "TLSRootCertFile":"/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt",
    "ServerNameOverride":""
  },
  "ProverPeer":{
    "Address":"peer0.org1.example.com:7051",
    "ConnectionTimeout":0,
    "TLSEnabled":true,
    "TLSRootCertFile":"/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt",
    "ServerNameOverride":""
  }
}
```



* [configorg2.json](https://github.com/hlkug/meetup/tree/master/201906/config/configorg2.json)

```json
{
  "ChannelID":"",
  "MSPInfo":{
    "MSPConfigPath":"",
    "MSPID":"Org2MSP",
    "MSPType":"bccsp"
  },
  "Orderer":{
    "Address":"orderer.example.com:7050",
    "ConnectionTimeout":0,
    "TLSEnabled":true,
    "TLSRootCertFile":"/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem",
    "ServerNameOverride":""
  },
  "CommitterPeer":{
    "Address":"peer0.org2.example.com:9051",
    "ConnectionTimeout":0,
    "TLSEnabled":true,
    "TLSRootCertFile":"/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt",
    "ServerNameOverride":""
  },
  "ProverPeer":{
    "Address":"peer0.org2.example.com:9051",
    "ConnectionTimeout":0,
    "TLSEnabled":true,
    "TLSRootCertFile":"/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt",
    "ServerNameOverride":""
  }
}
```



* [shares.json](https://github.com/hlkug/meetup/tree/master/201906/config/shares.json)

```json
[
    {
    "recipient":"Org2MSP:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/User1@org2.example.com/msp",
    "quantity":"50"
    }
]
```



# 4. BYFN 구동

```shell
# FabToken 실습에서 사용할 설정 파일 확인
$ ls /tmp/*.json
/tmp/configorg1.json  /tmp/configorg2.json  /tmp/shares.json

# BYFN 구동
$ cd $HOME/meetup/1906/fabric-samples/first-network/
$ ./byfn.sh up
Starting for channel 'mychannel' with CLI timeout of '10' seconds and CLI delay of '3' seconds
Continue? [Y/n] y
proceeding ...
LOCAL_VERSION=2.0.0
DOCKER_IMAGE_VERSION=2.0.0-alpha
=================== WARNING ===================
  Local fabric binaries and docker images are
  out of  sync. This may cause problems.
...
========= All GOOD, BYFN execution completed ===========

```

CLI 컨테이너에서 FabToken 실습에서 사용할 설정 파일에 접근하기 위해 아래 디렉토리로 복사합니다.

※ BYFN 중지 시 *crypto-config* 디렉토리도 삭제됩니다. BYFN 재 구동시 아래 절차를 통해 설정파일을 복사해야 합니다.

```shell
$ cp /tmp/*.json $HOME/meetup/1906/fabric-samples/first-network/crypto-config
$ cd $HOME/meetup/1906/fabric-samples/first-network/crypto-config
$ ls *.json
configorg1.json  configorg2.json  shares.json
```



# 5. FabToken 실습

CLI 컨테이너 내에서 Token CLI를 통해 오퍼레이션을 합니다.

```shell
# CLI 컨테이너 접속
$ docker exec -it cli bash
bash-4.4# 
bash-4.4# ls
channel-artifacts/  crypto/             log.txt            mycc.tar.gz        mychannel.block    scripts/
bash-4.4# cd crypto/
# 설정 파일 확인
bash-4.4# ls
configorg1.json       configorg2.json       ordererOrganizations/  peerOrganizations/     shares.json
bash-4.4#
```

## 5.1 Issue

- 토큰명: BYFNcoins
- 수량: 100
- 소유자: [User1@org1.example.com]()

~~~shell
bash-4.4# token issue \
--config /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/configorg1.json \
--mspPath /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp \
--channel mychannel \
--type BYFNcoins \
--quantity 100 \
--recipient Org1MSP:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp

Orderer Status [SUCCESS]
Committed [true]
~~~

## 5.2 List

토큰 목록 조회로 출력되는 결과값 중 "tx_id"키를 포함하는 값은 Transfer,Redeem 오퍼레이션 시 Token ID로 사용됩니다.

```shell
bash-4.4# token list \
--config /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/configorg1.json \
--mspPath /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp \
--channel mychannel

{"tx_id":"0b6f347a438399c24c5aa00a78d58a20cd6a89c6700ad3ad0fdccf8f90ee1c54"}
[BYFNcoins,100]
```

## 5.3 Transfer

* Token ID: 목록 조회로 확인된 "tx_id"키를 포함하는 JSON
* 수신자: User1@org2.example.com
* 전송 수량: 50

```shell
bash-4.4# token transfer \
--config /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/configorg1.json \
--mspPath /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp \
--channel mychannel \
--tokenIDs '[{"tx_id":"0b6f347a438399c24c5aa00a78d58a20cd6a89c6700ad3ad0fdccf8f90ee1c54"}]' \
--shares /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/shares.json

Orderer Status [SUCCESS]
Committed [true]
```

아래 명령어로 User1@org1.example.com, User1@org2.example.com 가 보유하고 있는 토큰 수량을 확인합니다.

```shell
# User1@org1.example.com 토큰 조회
bash-4.4# token list \
--config /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/configorg1.json \
--mspPath /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp \
--channel mychannel

{"tx_id":"14d97dac90a9acc68432abaad7192eafb46887a2cb8b469b7db43ad07d27e5d1","index":1}
[BYFNcoins,50]

# User1@org2.example.com 토큰 조회
bash-4.4# token list \
--config /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/configorg2.json \
--mspPath /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/User1@org2.example.com/msp \
--channel mychannel

{"tx_id":"14d97dac90a9acc68432abaad7192eafb46887a2cb8b469b7db43ad07d27e5d1"}
[BYFNcoins,50]
```

## 5.4 Redeem

- Token ID: 목록 조회로 확인된 "tx_id"키를 포함하는 JSON
- 소유자: User1@org1.example.com
- 상환 수량: 25

```shell
bash-4.4# token redeem \
--config /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/configorg1.json \
--mspPath /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp \
--channel mychannel \
--tokenIDs '[{"tx_id":"14d97dac90a9acc68432abaad7192eafb46887a2cb8b469b7db43ad07d27e5d1","index":1}]' \
--quantity 25

Orderer Status [SUCCESS]
Committed [true]
bash-4.4# token list \
--config /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/configorg1.json \
--mspPath /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp \
--channel mychannel
{"tx_id":"921e4e8efdd8ff8246a3d6d19828b4e2665db6bfaefc1ae6bc73066d75dee1e6","index":1}
[BYFNcoins,25]
```

