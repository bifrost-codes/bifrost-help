# Bifrost Asgard CC2 Node Running Tutorial

> Writer: 安_change, Lurpis
> 
> Date: 2020-07-18

## Tutorial
### Recommended Configurations

```
CPU: 4 core
RAM: (running memory) 8 GB
DISK: (disk size) 100 GB
NET: (network bandwidth) 10 MB
System: Linux (Centos, Ubantu)
```

### Docker self-running mode
#### 1. Docker Installation
- Linux：<https://www.runoob.com/docker/centos-docker-install.html>
- Windows：<https://www.runoob.com/docker/windows-docker-install.html>
- Mac：<https://www.runoob.com/docker/macos-docker-install.html>

> Tips:
> 
> - It is recommended to use cloud servers to run nodes.
> - If it is a new cloud server, `It is recommended to install the CoreOS system with Docker`, this step can be skipped

#### 2. Run a node

##### 2.1 First time run

Order:

```sh
docker run -it -p 30333:30333 -p 9944:9944 bifrostnetwork/bifrost:asgard-v0.5.0 --name "NodeName" --rpc-cors 'all' --unsafe-ws-external --validator
```

<img :src="$withBase('/zh/node-tutorials/node-tutorials-01.png')" alt="" />

> Parameter Description:
> 
> - `-p 30333:30333 -p 9944:9944` is the node port number 
>     - Node restart can use docker restart container_id
> - `bifrostnetwork/bifrost:asgard-v0.5.0`: the image that the container depends on;
> - `--name "NodeName"`: where NodeName is the node name;

#### 2.2 Run again

First check the status of the previously running container:

```sh
$ docker ps -a
```

<img :src="$withBase('/zh/node-tutorials/node-tutorials-02.png')" alt="" />

If the computer or server restarts and the container exits, restart it with the following command:

```sh
$ docker restart 66e31
```

> Order description:
> 
> - `docker restart`: Restart the existing container, the Node ID will not change at this time
> - `66e31`: It is the container ID that was viewed earlier, the first 5 digits are enough; **Don’t copy, use your own**.

#### 2.3 Check node operation

Node monitoring: <https://telemetry.polkadot.io/#/Bifrost> Asgard CC2

Node reward: <https://rewards.bifrost.finance>

<img :src="$withBase('/zh/node-tutorials/node-tutorials-03.png')" alt="" />

### Ankr Hosting Operation Method
#### 1. Visit and register Ankr account <https://www.ankr.com/>
#### 2. Run the node
##### 2.1 Click "Deploy a Node" to enter the node market.
##### 2.2 Select Bifrost Validator Node.
##### 2.3 Enter the payment page.
Fill in the node name of the application name | the first 10 digits of the Bifrost address, and select Validator for the operation mode

##### 2.4 Wait for the node to automatically complete the deployment after payment.
##### 2.5 Check node operation.

Node monitoring: <https://telemetry.polkadot.io/#/Bifrost> Asgard CC2 Node reward: <https://rewards.bifrost.finance>

> The above has completed the task of participating in the node running, and then you can view "Validator tutorial"

## Q & A
#### 1. Windows Toolbox failed to pull boot2docker.iso

The following error occurred when installing Docker Toolbox, indicating that the boot2docker.iso failed to pull.

<img :src="$withBase('/zh/node-tutorials/node-tutorials-04.png')" alt="" />

solution:

There is a downloaded image in the Bifrost community group, first need it in the group; then copy it to the cache path marked in red in the figure above;

After copying, double-click to start Docker Quickstart Terminal

> Tips:
> 
> - Add customer service WeChat to join the group: bifrost00
> - boot2docker.iso download https://lanzous.com/iaqdpmb

#### 2. What should I do if I forget the node ID after docker run or docker restart?

Excuting an order:

```sh
$ docker logs 66e31
```

<img :src="$withBase('/zh/node-tutorials/node-tutorials-05.png')" alt="" />

### 3. Other questions?

Add the WeChat customer service bifrost00 and enter the WeChat group for discussion.