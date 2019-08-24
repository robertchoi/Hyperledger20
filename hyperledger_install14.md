# 하이퍼레저 1.4 구성 (Ubuntu 18.04, HDD:40G, RAM:2G)

https://hyperledger-fabric.readthedocs.io/en/release-1.4/prereqs.html

~~~
sudo apt-get update
~~~
~~~
sudo apt-get upgrade
~~~
~~~
sudo apt-get install curl git docker.io docker-compose golang nodejs npm
~~~
~~~
sudo usermod -aG docker $USER
sudo systemctl start docker
sudo systemctl enable docker
su $USER
~~~

~~~
curl -sSL http://bit.ly/2ysbOFE | bash -s 1.4.0
~~~

~~~
cd fabric-samples/first-network
./byfn.sh generate
./byfn.sh up
~~~

~~~
./byfn.sh down
~~~

