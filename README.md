# cloudylab-serf

Firstly, I'm going to describe what is cloudy, [Cloudy](http://cloudy.community/) is a distribution based on Debian GNU/Linux, aimed at end users, to foster the transition and adoption of the Community Network cloud environment.

# How it works

Cloudy use [Serf](https://www.serf.io/)  in order to add or delete nodes from Cloudy.

# What am I going to do? 

- Transform 4 raspberrys pi on Cloudy distro [Cloudyning](https://github.com/Clommunity/cloudynitzar) these.
- Use 1 raspberry as Bootsrap Server.
- Join the other raspberrys to cluster.
- Expose services from raspberrys and watch how Serf through [gossip](https://www.serf.io/docs/internals/gossip.html) will be able to communicate changes to other nodes. 

# Let's do it 
## 1- Cloudynizing raspberrys:
```sh
sudo apt-get update; sudo apt-get install -y curl lsb-release
curl -k https://raw.githubusercontent.com/Clommunity/cloudynitzar/master/cloudynitzar.sh |sudo  bash -
```
- When the process is finished you will be able to connect to 127.0.0.1:7000
```sh
cloudy:~$ nc -zv 127.0.0.1 7000
Connection to 127.0.0.1 7000 port [tcp/afs3-fileserver] succeeded!
```

- You can check it in your navigator and you shoud watch some like:
![cloudy][images/home.png]

