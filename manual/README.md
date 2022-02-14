# Manual Deployment
These instructions have been created to manually deploy a 5G stack using Firecracker with Weave Ignite. For automated deployment check `ansible` directory.

# Pre-requisites
- Support for KVM
- Have the following installed: [Docker](https://docs.docker.com/engine/install/ubuntu/#installation-methods), [Firecracker](https://github.com/firecracker-microvm/firecracker/blob/main/docs/getting-started.md#getting-the-firecracker-binary), [Weave Ignite](https://github.com/weaveworks/ignite/blob/main/docs/installation.md)
- IP forwarding is enabled. Can follow this [link](https://www.ducea.com/2006/08/01/how-to-enable-ip-forwarding-in-linux/)

# Steps to Deploy

All commands are listed relative to project directory.

## 1. Build docker images
All the commands in this step can be run sequentially or in parallel in separate terminals.
```commandline
docker build -t open5gs-base ./dockerfiles/open5gs
docker build -t ueransim-gnb ./dockerfiles/ueransim-gnb
docker build -t ueransim-ue ./dockerfiles/ueransim-ue
```

## 2. Import images into Ignite
Run all ignite commands as root.

```commandline
ignite image import open5gs-base --runtime docker
ignite image import ueransim-gnb --runtime docker
ignite image import ueransim-ue --runtime docker
```

## 3. (Optional) Delete docker images
### Delete transient container
The import step above might create a container that is not required. Check to see if it exists:

```commandline
saurabh  # docker ps -a 
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS                     PORTS     NAMES
42368f96f812   3513133c9823   "/bin/sh -c 'wget -O…"   3 minutes ago   Exited (4) 3 minutes ago             affectionate_herschel
```
If such container exist it can be deleted using the container id or name:
```commandline
docker rm 42368f96f812
```

### Remove unecessary images
Since the images have now been imported into ignite, they can be deleted from Docker to save space.

**WARNING: This command will remove all unused images!**
```commandline
docker image prune -a
```

## 4. Setup VM(s)

### 4.1 Create Control Plane VM
The control plane will consist of all 5G core services except the UPF.
```commandline
ignite create open5gs-base \
--name o5s-cp \
--cpus 2 \
--memory 1GB \
--size 5GB \
--ssh
```

### 4.2 Create User Plane VM
The control plane will consist of all 5G core services except the UPF.
```commandline
ignite create open5gs-base \
--name o5s-up \
--cpus 2 \
--memory 1GB \
--size 5GB \
--ssh
```

### 4.3 Create GNB VM
```commandline
ignite create ueransim-gnb \
--name gnb \
--cpus 2 \
--memory 1GB \
--size 5GB \
--ssh
```

### 4.4 Create UE VM
```commandline
ignite create ueransim-ue \
--name ue \
--cpus 2 \
--memory 1GB \
--size 5GB \
--ssh
```

### 4.5 Start all VM(s)
```commandline
ignite start o5s-cp && \
ignite start o5s-up && \
ignite start gnb && \
ignite start ue
```
You can check if all VM(s) are up by doing `ignite ps`

```commandline
saurabh  # ignite ps               
VM ID			IMAGE			KERNEL					SIZE	CPUS	MEMORY		CREATED		STATUS	IPS		PORTS	NAME
1e4534a950645a88	open5gs-base:latest	weaveworks/ignite-kernel:5.10.51	5.0 GB	2	1024.0 MB	3m59s ago	Up 18s	10.61.0.23		o5s-up
329b23d100c7ce39	open5gs-base:latest	weaveworks/ignite-kernel:5.10.51	5.0 GB	2	1024.0 MB	35s ago		Up 23s	10.61.0.22		o5s-cp
4065bd96014466da	ueransim-ue:latest	weaveworks/ignite-kernel:5.10.51	5.0 GB	2	1024.0 MB	119s ago	Up 9s	10.61.0.25		ue
42f6d47995a32fb6	ueransim-gnb:latest	weaveworks/ignite-kernel:5.10.51	5.0 GB	2	1024.0 MB	2m6s ago	Up 14s	10.61.0.24		gnb
```

### 4.4 SSH into VM
You can now SSH into any VM using command `ignite ssh name_of_vm`. For example to get IP address of you control plane VM:
```commandline
ignite ssh o5s-cp
```

### 4.4 Find the IP address of the VM
Find the IP address of the VM running `ignite exec NAME_OF_VM ifconfig eth0`. It is important to find the IP address of all the VM(s) for next step.
```
firecracker-5g (main) # ignite exec o5s-cp ifconfig eth0
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.61.0.28  netmask 255.255.0.0  broadcast 10.61.255.255
        inet6 fe80::d8bb:9cff:fee7:1336  prefixlen 64  scopeid 0x20<link>
        ether da:bb:9c:e7:13:36  txqueuelen 1000  (Ethernet)
        RX packets 258  bytes 28069 (28.0 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 95  bytes 12963 (12.9 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```
In this example the IP address of this VM is 10.61.0.28


## 5. Configure 5G Stack
Find the IP address of control plane, user plane and gnb VM by using the instructions in step 4.4. Assuming the IP addresses of control plane is `10.61.0.28`, user plane is `10.61.0.29` and gnb is `10.61.0.30`, the following command should be executed.


### 5.1 Configure Control Plane

```commandline
sed -i 's/CHANGE_ME_TO_CP_VM_IP/10.61.0.28/g' config-cp/amf.yaml
sed -i 's/CHANGE_ME_TO_CP_VM_IP/10.61.0.28/g' config-cp/smf.yaml
sed -i 's/CHANGE_ME_TO_UP_VM_IP/10.61.0.29/g' config-cp/smf.yaml
```
Please make sure to change the IP address to your corresponding VM ip. Copy files to VM:

```commandline
ignite cp config-cp/amf.yaml o5s-cp:/etc/open5gs/ && \
ignite cp config-cp/smf.yaml o5s-cp:/etc/open5gs/ && \
ignite exec o5s-cp chown root:root /etc/open5gs/amf.yaml && \
ignite exec o5s-cp chown root:root /etc/open5gs/smf.yaml
```

### 5.2 Configure User Plane

```commandline
sed -i 's/CHANGE_ME_TO_UP_VM_IP/10.61.0.29/g' config-up/upf.yaml
ignite cp config-up/upf.yaml o5s-up:/etc/open5gs/ && \
ignite exec o5s-up chown root:root /etc/open5gs/upf.yaml
```

Setup user plane networking
```
ignite exec o5s-up "sysctl -w net.ipv4.ip_forward=1"
ignite exec o5s-up "ip addr add 10.45.0.1/16 dev ogstun"
ignite exec o5s-up ip link set ogstun up
ignite exec o5s-up "iptables -t nat -A POSTROUTING -s 10.45.0.0/16 ! -o ogstun -j MASQUERADE"
```

### 5.3 Configure GNB

```commandline
sed -i 's/CHANGE_ME_TO_CP_VM_IP/10.61.0.28/g' config-gnb/open5gs-gnb.yaml
sed -i 's/CHANGE_ME_TO_GNB_VM_IP/10.61.0.30/g' config-gnb/open5gs-gnb.yaml
ignite cp config-gnb/open5gs-gnb.yaml gnb:/root/open5gs-gnb.yaml
ignite exec gnb chown root:root /root/open5gs-gnb.yaml
```

### 5.4 Configure UE
```commandline
sed -i 's/CHANGE_ME_TO_GNB_VM_IP/10.61.0.30/g' config-ue/open5gs-ue.yaml
ignite cp config-ue/open5gs-ue.yaml ue:/root/open5gs-ue.yaml
ignite exec ue chown root:root /root/open5gs-ue.yaml
```

### 5.5 Start Control Plane
```commandline
ignite exec o5s-cp systemctl start open5gs-nrfd && \
sleep 5 && \
ignite exec o5s-cp systemctl start open5gs-smfd && \
ignite exec o5s-cp systemctl start open5gs-amfd && \
ignite exec o5s-cp systemctl start open5gs-ausfd && \
ignite exec o5s-cp systemctl start open5gs-udmd && \
ignite exec o5s-cp systemctl start open5gs-udrd && \
ignite exec o5s-cp systemctl start open5gs-pcfd && \
ignite exec o5s-cp systemctl start open5gs-nssfd && \
ignite exec o5s-cp systemctl start open5gs-bsfd
```
Get status
```commandline
ignite exec o5s-cp systemctl status open5gs-nrfd && \
ignite exec o5s-cp systemctl status open5gs-smfd && \
ignite exec o5s-cp systemctl status open5gs-amfd && \
ignite exec o5s-cp systemctl status open5gs-ausfd && \
ignite exec o5s-cp systemctl status open5gs-udmd && \
ignite exec o5s-cp systemctl status open5gs-udrd && \
ignite exec o5s-cp systemctl status open5gs-pcfd && \
ignite exec o5s-cp systemctl status open5gs-nssfd && \
ignite exec o5s-cp systemctl status open5gs-bsfd
```

### 5.6 Start User Plane

```commandline
ignite exec o5s-up systemctl start open5gs-upfd
ignite exec o5s-up systemctl status open5gs-upfd
```

### 5.7 Start GNB (In separate terminal)

```commandline
ignite exec gnb "nr-gnb -c /root/open5gs-gnb.yaml"
```

### 5.8 Start WebUI
Execute following commands inside control plane VM:

```commandline
curl -fsSL https://open5gs.org/open5gs/assets/webui/install | sudo -E bash -
vi /lib/systemd/system/open5gs-webui.service

# Add this after the other env variable
Environment="HOSTNAME=0.0.0.0"

# Save and then run the following
systemctl daemon-reload
systemctl start open5gs-webui
```
You can now visit the web ui from your browser by visting http://<Control_plane_vm_ip>:3000 admin/1423

Click 'Add Subscriber'
Copy paste IMSI from config-ue/open5gs-ue.yaml Should be `901700000000001` if you haven't changed it. Click `Save`.

### 5.9 Start UE (In seperate terminal)
```commandline
ignite exec ue "nr-ue -c /root/open5gs-ue.yaml"
```

## Test 5G stack
Run the following command to ensure the UE interface up and can reach the internet:
```commandline
ignite exec ue "ping -I uesimtun0 www.google.com"
```

Can capture the UPF data frowarding from UE by running `tcpdump` on the UPF VM:
```commandline
ignite exec o5s-up "apt-get install -y tcpdump"
ignite exec o5s-up "tcpdump -i ogstun -n"
```


# Cleanup

To clean all the VM(s) execute following

```commandline
ignite stop o5s-up
ignite stop o5s-cp
ignite stop gnb
ignite stop ue

ignite rm o5s-up
ignite rm o5s-cp
ignite rm gnb
ignite rm ue
```

To delete all images:

```commandline
ignite image rm open5gs-base ueransim-ue ueransim-gnb
```
