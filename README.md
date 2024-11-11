# ZKWork Subspace farmer


## zkwork pool address
```shell
pool_address=ai3.asia.zk.work:10020
```

A git repository for ZKWork Subspace farmer release versions

* Download releases : https://github.com/6block/zkwork_subspace_farmer/releases
* Discord Group :  https://discord.com/invite/pKufwyjGFF
* Twitter : https://x.com/ZKWorkHQ

## We are on mainnet with official version `mainnet-2024-nov-06`

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

## Start Farming

### 1. Choose and download the corresponding version on release page

### 2. Unzip
    tar zxvf <YOUR DIR>/zkwork-mainnet-2024-nov-06.tar.gz

### 3. Start local node

    1. keep .env file and subspace-node binary in the same directory, then open .env file and set POOL_ADDR_IPV4 with the pool address:
     POOL_ADDR_IPV4="ai3.asia.zk.work:10020"

    2. start subspace-node
     ./subspace-node run --base-path <NODE_DATA_PATH> --farmer --rpc-listen-on <LOCAL_IP>:30003 --rpc-cors "all" --sync full

### 4. Start farming
   The farming on single machine is designed for only one machine used to farm.

   The farming cluster is designed for larger-scale farmers.

#### 4.1 Farmering on single machine
    1. start farmer in one command:
     ./subspace-farmer farm --reward-address <YOUR_REWARD_ADDRESS> --node-rpc-url ws://<NODE_IP>:30003 --custom-name <YOUR_NAME> path=<FARMER_DATA_PATH>,size=<SIZE> --cuda-gpus 0,1

#### 4.2 Farmering on cluster
   Notes: farming cluster and subspace local node need be started within the same intranet.

   ##### 4.2.1 instatll and start NATS server
   Before start farming cluster, the NATS.io shoule be started fristly:
      https://docs.autonomys.xyz/farming/advanced-cli/cluster/#core-messaging-technology-natsio

   ##### 4.2.2 start farming cluster
   The cluster consists of four distinct components, and they should be started in the following order.

    1. start Controller
     ./subspace-farmer cluster --nats-server nats://<NATS_IP>:4222 controller --base-path <FARMER_DATA_PATH> --node-rpc-url ws://<NODE_IP>:30003

    2. start Cache
     ./subspace-farmer cluster --nats-server nats://<NATS_IP>:4222 cache path=<FARMER_DATA_PATH>,size=<SIZE>

     200 GB is a good size to use for cache based on the current state of Mainnet.

    3.  start Farmer
     ./subspace-farmer cluster --nats-server nats://<NATS_IP>:4222 farmer --reward-address <YOUR_REWARD_ADDRESS> --custom-name <YOUR_NAME> path=<FARMER_DATA_PATH>,size=<SIZE>

     The <SIZE> can be a little smaller than the free size of the <FARMER_DATA_PATH>.

    4. start Plotter
     ./subspace-farmer cluster --nats-server nats://<NATS_IP>:4222 plotter --cuda-gpus 0,1

  Reference case for farming cluster

  100 machines with 3T hard disk and GPU 3090, all in one intranet, 10G network bandwidth.

  Suggestion: One local node, nats.io, controller and cache run on the same machine, then run one farmer and one plotter on each of other machines.

  If every machine have more than 5T hard disk OR want plotter faster, more caches is need,  e.g. we can run one local node, but need two or more nats.ios, controllers and caches, and they should be run on two or more machines respectively.
