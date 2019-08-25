# 하이퍼레저 2.0 구성 (Ubuntu 18.04)

1. sudo apt update
2. sudo apt install curl
3. sudo apt install docker.io
4. sudo usermod -aG docker $USER
5. su $USER
6. sudo apt install docker-compose
7. wget https://dl.google.com/go/go1.12.7.linux-amd64.tar.gz
8. tar xvf go1.12.7.linux-amd64.tar.gz 
9. sudo cp -r go /usr/local
10. curl -sSL http://bit.ly/2ysbOFE | bash -s -- 2.0.0-alpha 2.0.0-alpha 0.4.15
11. vi ~/.bashrc 맨 아래에 아래 명령어 추가
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin:$HOME/fabric-samples/bin

12. source ~/.bashrc
13. go version
14. cryptogen --help
15. cd fabric-samples/first-network/
16.  ./byfn.sh up
