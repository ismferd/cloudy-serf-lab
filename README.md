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

- You can check it in your navigator and you shoud watch something like:

![cloudy](images/home.png)

## 2- Playing with Serf

As I said before, Cloudy use Serf in order to get Failure detection, Service Discovery with Gossip and Custom Events.

- Firs of all, I will put the first member of me cluster as Bootsrap Server.

```sh
pi@raspberrypi:~ $ serf agent -bind=192.168.1.141
==> Starting Serf agent...
==> Starting Serf agent RPC...
==> Serf agent running!
         Node name: 'raspberrypi'
         Bind addr: '192.168.1.141:7946'
          RPC addr: '127.0.0.1:7373'
         Encrypted: false
          Snapshot: false
           Profile: lan

==> Log data will now stream in as it occurs:

    2019/03/09 16:46:57 [INFO] agent: Serf agent starting
    2019/03/09 16:46:57 [INFO] serf: EventMemberJoin: raspberrypi 192.168.1.141
    2019/03/09 16:46:58 [INFO] agent: Received event: member-join
    2019/03/09 16:50:02 [INFO] agent.ipc: Accepted client: 127.0.0.1:41974
    2019/03/09 16:50:02 [INFO] serf: EventMemberUpdate: raspberrypi
    2019/03/09 16:50:03 [INFO] agent: Received event: member-update
```

Then I see the node inside the cluster:

```sh
pi@raspberrypi:~ $ serf members
raspberrypi  192.168.1.141:7946  alive  
```

- Joininng more nodes to cluster:

```sh
pi@raspberry-1:~ $ serf agent -join=192.168.1.141 -bind=192.168.1.140
==> Starting Serf agent...
==> Starting Serf agent RPC...
==> Serf agent running!
         Node name: 'raspberry-1'
         Bind addr: '192.168.1.140:7946'
          RPC addr: '127.0.0.1:7373'
         Encrypted: false
          Snapshot: false
           Profile: lan
==> Joining cluster...(replay: false)
    Join completed. Synced with 1 initial agents

==> Log data will now stream in as it occurs:

    2019/03/09 17:05:36 [INFO] agent: Serf agent starting
    2019/03/09 17:05:36 [INFO] serf: EventMemberJoin: raspberry-1 192.168.1.140
    2019/03/09 17:05:36 [INFO] agent: joining: [192.168.1.141] replay: false
    2019/03/09 17:05:36 [INFO] serf: EventMemberJoin: raspberrypi 192.168.1.141
    2019/03/09 17:05:36 [INFO] agent: joined: 1 nodes
    2019/03/09 17:05:37 [INFO] agent: Received event: member-join
```
**NOTE:** You can see in the command the flag -join, is the reference to bootstrap server (you could do it from config files too).

```sh
pi@raspberrypi:~ $ serf members
raspberrypi  192.168.1.141:7946  alive
raspberry-1  192.168.1.140:7946  alive
```

I have added another one:

```sh
pi@raspberrypi:~ $ serf members
raspberrypi  192.168.1.141:7946  alive
raspberry-1  192.168.1.140:7946  alive
raspberry-1  192.168.1.143:7946  alive
```

Thanks to Cloudy and using [avahi-ps script](https://github.com/Clommunity/avahi-ps/blob/master/avahi-ps) I can publish or unpublish service through Serf really easy:

```sh
pi@raspberrypi:~ $ avahi-ps
avahi-ps (Avahi Publish and Search) is a system to publish local services and
discover remote ones using Avahi and other available modules plugged-in.

Usage: /usr/sbin/avahi-ps publish|unpublish|search|info <options>

Examples:

 - Publishing a local service to the network:
   /usr/sbin/avahi-ps publish <description> <type> <port> [txt]
      <description>: a short text describing the service
      <type>: service type
      <port>: service port
      <txt>: additional information, formatted as
                'attribute1=value1&attribute2=value2&...&attributeN=valueN'

 - Unpublishing a local service to the network
   /usr/sbin/avahi-ps unpublish [type] [port]

 - Searching for services on the network:
   /usr/sbin/avahi-ps search [type] [hostname]

 - Showing available information:
   /usr/sbin/avahi-ps info <variable>
       <variable>: ip|cloud|tincdev|communitydev
```

