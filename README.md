# ZKWork Autonomys (Subspace) farmer
`Subspace-node` and `Subspace-farmer` software for farming on Autonomys mainnet.

## Latest version
`zkwork-mainnet-2024-dec-09`

## Requirements
- OS Version: Ubuntu 22.04 +

- Nvidia Driver Version: 555.42.02 +

## GPU Performance
The time spent on plotting one sector.

<table>
  <tr>
   <td><strong>GPU Version</strong>
   </td>
   <td><strong>6block</strong>
   </td>
   <td><strong>Official</strong>
   </td>
  </tr>
  <tr>
   <td>NVIDIA GeForce RTX 4090
   </td>
   <td>1.9s
   </td>
   <td>6.5s
   </td>
  </tr>
  <tr>
   <td>NVIDIA GeForce RTX 3090
   </td>
   <td>2.85s
   </td>
   <td>9.7s
   </td>
  </tr>
  <tr>
   <td>NVIDIA GeForce RTX 3080
   </td>
   <td>3.3s
   </td>
   <td>11s
   </td>
  </tr>
</table>

## Start farming

### 1. Login and get your apitoken for farming on [ZK.Work](https://zk.work/en/login)

To better support internal assets transfer feature, our farmer only supports to farm with apitoken since version `zkwork-mainnet-2024-nov-06`. The apitoken should start with `zkwork` like `zkworkxxxx`. You can get this token easily after login on [ZK.Work](https://zk.work/en/login) website.

### 2. Download the lateset version on release page

using cuda version for example:
```
wget https://github.com/6block/zkwork_subspace_farmer/releases/download/zkwork-mainnet-2024-dec-09/zkwork-mainnet-2024-dec-09-cuda.tar.gz
tar -zvxf zkwork-mainnet-2024-dec-09-cuda.tar.gz
```

### 3. Get Subspace-node ready

A node is needed to submit onchain solutions to the network and receive rewards. There are two options for ZK.Work farmers, you can choose one for yourself.

#### 3.1 Setup local node and connect to ZK.Work pool

```
NODE_DATA_PATH is used to store chain data, 10G is enough. Example: "/home/ubuntu/subspace/data"
LOCAL_IP is your server local ip, not public ip. Example: "0.0.0.0"
You can create a file name `.env` in this path then write POOL_ADDR_IPV4="ai3.asia.zk.work:10020" to the file, or you can start directly as follows,

POOL_ADDR_IPV4="ai3.asia.zk.work:10020" ./subspace-node run --base-path <NODE_DATA_PATH> --farmer --rpc-listen-on <LOCAL_IP>:30003 --rpc-cors "all"
```

#### 3.2 Use ZK.Work public node directly
  
  `ws://ai3.asia.zk.work:30003`

### 4. Get hardwares ready and make the decesion

Currently, there are two options to farm, one is `Standalone mode`, the other is `Cluster mode`. We recommend to use `Cluster mode` if you have more than 5 servers. `Standalone mode` is easy to but efficientless while `Cluster mode` is more complicated and efficient. 

#### 4.1 Standalone mode

```
YOUR_API_TOKEN is the one you get in step1
NODE_IP is the one you get in step3
YOUR_FARMER_NAME is the name for this farmer
FARMER_DATA_PATH is the path to store farming data
SIZE is human-readable free size for farming data
path=<FARMER_DATA_PATH>,size=<SIZE> can be repeated to load multiple disks

Example: 
./subspace-farmer farm --account zkworkxxxx --node-rpc-url ws://127.0.0.1:30003 --custom-name farmer1 path=/nvme/disk1,size=3T path=/nvme/disk2,size=3T --cuda-gpus 0,1
```

If you are farming on standalone mode, that's all here. You can try to search on [ZK.Work](https://zk.work/en/autonomys) website with your apitoken to monitor your farming status.

#### 4.2 Cluster mode
  
There are 5 farming components, [natsio](https://docs.autonomys.xyz/farming/advanced-cli/cluster/#core-messaging-technology-natsio), `controller`, `cache`, `farmer` and `plotter`. Since cluster farming is more complicated, we move it to part5.

### 5. Setup cluster farming

#### 5.1 Prepare hardware

We recommend to manage your servers in multiple smaller groups (cluster), instead of single large one. Recommended hardware configuration for cluster farmers (this can be updated) is as follows.

- About 300T ssd space for farming = 100 servers with 3T space = 50 servers with 6T space.
- High performance local network speed, 10G can be really enough.
- Plotting servers with GPUs, number of GPUs needed depends on plotting speed you want, 10 RTX3090 is recommended.

#### 5.2 Deploy ZK.Work cluster farming software

For each farming cluster, we need a server to run a nats.io service, a controller, and a cache. Then we should run farmer on each servers with free ssd space, and some plotters on GPU servers. The number of GPUs is depends on plotting speed you want.

* Start nats server with docker [natsio](https://docs.autonomys.xyz/farming/advanced-cli/cluster/#core-messaging-technology-natsio)
* Start controller
  
  Controller is the key component for cluster farming, which is responsible for task scheduling, start controller asap as follows.

  ```
  NATS_IP is local ip of your nats server.
  CONTROLLER_DATA_PATH is the path for saving controller identity, 5M free space is enough
  NODE_RPC is ws://NODE_IP:30003 if you are running your own node, or ws://ai3.asia.zk.work:30003 with ZK.Work public node

  ./subspace-farmer cluster --nats-server nats://<NATS_IP>:4222 controller --base-path <CONTROLLER_DATA_PATH> --node-rpc-url <NODE_RPC>

  Example:

  ./subspace-farmer cluster --nats-server nats://127.0.0.1:4222 controller --base-path "/home/ubuntu/subspace/controllerdata/" --node-rpc-url "ws://ai3.asia.zk.work:30003"
  ```

  * Start cache
  
  Cache is for storing and providing piece cache for plotting. It takes serval hours to keep cache fully synced in first-run.

  ```
  NATS_IP is local ip of your nats server.
  CACHE_DATA_PATH is the path for storing piece, we recommend to use 200G free disk for current network.

  ./subspace-farmer cluster --nats-server nats://<NATS_IP>:4222 cache path=<CACHE_DATA_PATH>,size=200G

  Example:

  ./subspace-farmer cluster --nats-server nats://127.0.0.1:4222 cache path=/home/ubuntu/subspace/cache,size=200G
  ```

  * Start farmer
  
  Farmer is server with ssd disk, responsible storing plotted data and farming to receive reward. Load all free disks to farmer to get more reward.

  ```
  NATS_IP is local ip of your nats server.
  YOUR_API_TOKEN is your zkwork mining apitoken you get on zkwork website.
  FARMER_NAME is name to distinguish different servers.
  FARMER_DATA_PATH is path to store plotted data.
  SIZE should be a smaller number than the free size of the FARMER_DATA_PATH.
  path=<FARMER_DATA_PATH>,size=<SIZE> can be repected multiple times to load more disks.

  ./subspace-farmer cluster --nats-server nats://<NATS_IP>:4222 farmer --account <YOUR_API_TOKEN> --custom-name <FARMER_NAME> path=<FARMER_DATA_PATH>,size=<SIZE>

  Example: 

  ./subspace-farmer cluster --nats-server nats://127.0.0.1:4222 farmer --account zkworkxxx --custom-name farmer1 path=/nvme/disk1,size=3T path=/nvme/disk2,size=5T path=/nvme/disk3,size=7T 

  ```

  * Start plotter
  
  Plotter is server with GPUs, responsible for plotting.

  ```
  NATS_IP is local ip of your nats server.
  GPU_INDEX is a string list of GPUs used for plotting, eg, "0", "0,1", "0,1,2"

  ./subspace-farmer cluster --nats-server nats://<NATS_IP>:4222 plotter --cuda-gpus <GPU_INDEX>

  Example:

  ./subspace-farmer cluster --nats-server nats://127.0.0.1:4222 plotter --cuda-gpus "0,1,2,3"
  ```

### 6. Keep it running

We recommend to use [supervisord](http://supervisord.org/) to manage all processes above. Here is an example on how to start controller.

#### 6.1 Install

`sudo apt install supervisor`

#### 6.2 Create a script to start controller.

Be sure to change `YOUR_USER` to your login user like `ubuntu`, `root` etc.

Create a shell file named `run_controller.sh` at `/home/YOUR_USER/subspace/script/run_controller.sh` with the following.

```
#!/bin/bash

cd /home/YOUR_USER/subspace/bin/

./subspace-farmer cluster --nats-server nats://127.0.0.1:4222 controller --base-path "/home/ubuntu/subspace/controllerdata/" --node-rpc-url "ws://ai3.asia.zk.work:30003"

wait
```

#### 6.3 Create config file

Create a configuration file named `subspace-controller.conf` at `/etc/supervisor/conf.d/` with the following. 

```
[program:subspace-controller]
command=/home/YOUR_USER/subspace/script/run_controller.sh
user=YOUR_USER

autostart=true
autorestart=true
stopwaitsecs=60
startretries=999
stopasgroup=true
killasgroup=true

redirect_stderr=true
stdout_logfile=/home/YOUR_USER/subspace/log/controller.log
stdout_logfile_maxbytes=256MB
```

#### 6.4 Start controller

`sudo supervisorctl update` on firstrun.

- Status check: `sudo supervisorctl status subspace-controller`.
- Stop controllet: `sudo supervisorctl stop subspace-controller`.
- Start controllet: `sudo supervisorctl stop subspace-controller`.

### Check farming status

Navigate to [ZK.Work](https://zk.work/en/autonomys) and search with your apitoken.

### Get help

Join [ZK.Work Discord](https://discord.gg/UfqdmPq7)

