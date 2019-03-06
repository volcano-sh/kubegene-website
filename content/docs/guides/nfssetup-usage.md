+++
title = "Network File System(NFS),Its Setup & Usage in KubeGene"
description = "Information on how to setup NFS server,client and different provisioners in k8s"
weight = 10
toc = true
bref= "command."
aliases = ["/docs/guides/"]
[menu.main]
  parent = "Guides"
  weight = 1
+++

# General Information related to NFS

## *NFS Basics & How to install and configure NFS mounts*

NFS, or Network File System, is a distributed file system protocol that allows you to mount remote directories on your server. This lets you manage storage space in a different location and write to that space from multiple clients. NFS provides a relatively quick and easy way to access remote systems over a network and works well in situations where the shared resources will be accessed regularly.

You need to use the following commands to find out if nfs is running or not on the server / clinet machines.

* $ ps aux | grep nfsd  (Linux / Unix users) 
* $ /etc/init.d/nfs-kernel-server status  or service nfs-kernel-server status   (Debian / Ubuntu Linux user)

On older system (NFSv3 and older), you also need to make sure portmap service is running:

* $ ps aux | grep 'portman' (Linux / Unix users)

* $ /etc/init.d/portmap status or service portmap status (Debian / Ubuntu Linux user)


Assume, we’re going to be using two systems:

Server Computer with IP address 192.168.71.131
Client Computer with IP address 192.168.71.133
Share Resource Name:  publicdata

### Installing NFS Server packages on the Host computer
To get NFS server working you must install the server packages. To do that run the commands below:

* $ sudo apt-get update
* $ sudo apt-get install nfs-kernel-server

verify that the NFS server is running or not by using the above commands

When the server packages are installed, switch to the client system to install the client package.

### Installing NFS client packages on the client systems
To access NFS mount points on the server, you must install NFS client packages. To do that, run the commands below

* $ sudo apt-get update
* $ sudo apt-get install nfs-common

After installing the client packages, switch to the server to configure a mount point to export to the client.


### Creating the folder/directory to export (share) to the NFS clients
At this point the server and client components of NFS have been installed, now go and create the folder or directory you which to export to the clients.

For Example creating a folder called publicdata in the /mnt/ directory. To create the folder, run the command below.

* $ sudo mkdir -p /mnt/publicdata

Since we want this location to be viewed by all clients, we’re going to remove the restrictive permissions. To do that, change the folder permission to be owned by nobody in no group.

* $ sudo chown nobody:nogroup /mnt/publicdata
* $ sudo chmod 777 /mnt/publicdata

Now the folder is ready to be exported so the client can access it. Sub folders can be created there as well.

### Configuring NFS Exports file
Now that the location is created on the host system, open NFS export file and define the client access.

Access can be granted to a single client or entire network subnet. For Example, we’re allowing access to the single client mentioned above.

NFS export file is at /etc/exports

In that file is where you define client access. The configuration format is:

/server_share_resource          client_IP(share option1, ..... share_optionN)

So, permit access to only the client with IP 192.168.71.133. To do that, open the export file by running the commands below:

* $ sudo vim /etc/exports

Then add the line below:

`/mnt/publicdata   192.168.71.133(rw,sync,no_subtree_check)`

and save the file. Only client with IP address defined above will access that location remotely.

To allow an entire subnet

/mnt/publicdata   192.168.71.0/24(rw,sync,no_root_squash,no_subtree_check)


rw -> This will allow client computers to read and write to the share.

sync -> This will allow the data to be written in the NFS before it applies to queries and It also increase consistent environment and will be stable.

no_subtree_check -> This will prevent subtree checking, where if we enable this option, it will cause many problems if the client has opened the file.

no_root_squash -> This will makes the NFS translation request from the root user for the client into a not –privileged users of the server, 
where it will also prevent the root account on the client from using the file system of the server as root.

Export the shares by running the commands below (Debian / Ubuntu Linux user)

* $ sudo exportfs -a

Restart the NFS server by running the commands below.

* sudo systemctl restart nfs-kernel-server

### Mounting the NFS share on the client
Next, switch to the client to mount the NFS directory defined on the server. To do that, create a mount point on the client where the host mount will mounted. This can be anywhere.

* $ sudo mkdir -p /mnt/publicdata

After creating the mounting point, use the command below to mount the server NFS folder on the client

The format to mount directories on the client is as shown below:

* $ sudo mount server_IP:/NFS_directory_on_server   /client_mount_point

* $ sudo mount 192.168.71.131:/mnt/publicdata /mnt/publicdata

This command above mounts the directory on the client computer.

You can automatically mount the directory by editing the /etc/fstab file by adding the lines below and saving the file.

* $ sudo vim /etc/fstab

Then add the line below and save.

`192.168.71.131:/mnt/publicdata /mnt/publicdata auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0`

To un-mount the folder, use the umount command.

* $ sudo umount /mnt/publicdata

If for some reasons the client can’t access the host folder, open Ubuntu firewall to the client on the host computer.

* $ sudo ufw allow from 192.168.71.133 to any port nfs

[For More Info](https://www.tutorialspoint.com/articles/how-to-set-up-and-configure-nfs-on-ubuntu-16-04)

#  **NFS Provisioners in k8s**

## *NFS Server Provisioner SetUp*

Download the project into your GOPATH directory by using go get or cloning it manually

* $ go get github.com/kubernetes-incubator/external-storage

Now build the project and the Docker image by checking out the latest release and running make container in the project directory.

* $ `cd  $GOPATH/src/github.com/kubernetes-incubator/external-storage/nfs`

* $ make  or  make container 

provide a unique name to the provisioner that follows the naming scheme <vendor name>/<provisioner name> where <vendor name> cannot be "kubernetes.io." 
The provisioner will only provision volumes for claims that request a StorageClass with a provisioner field set equal to this name. 
For example, the names of the in-tree GCE and AWS provisioners are kubernetes.io/gce-pd and kubernetes.io/aws-ebs.

Admin can deploy the nfs-provisioner different ways

* 1) StatefulSet in Kubernetes Cluster 
* 2) ReplicaSet in Kubernetes Cluster
* 3) a Docker Container outside k8s
* 4) standalone process outside k8s

### Brining up as standalone process

Running nfs-provisioner as standalone  process allows to manipulate exports directly on the NFS-Server host machine. It will create & store all its data at /export so ensure the directory exists and is available for use. 

Run nfs-provisioner with provisioner equal to the name you decided on(example-nfs), one of master or kubeconfig set, run-server set false, and use-ganesha set according to how the NFS server is running on the host. It probably needs to be run as root.

We may want to specify the hostname the NFS server exports from, i.e. the server IP to put on PVs, by setting the server-hostname flag.

bash-shell open in the nfs directory.



* $ sudo ./nfs-provisioner `-provisioner=example.com/nfs` \
`-kubeconfig=$HOME/.kube/config` \
`-run-server=false `\
`-use-ganesha=false`

or

* $ sudo ./nfs-provisioner `-provisioner=example.com/nfs` \
`-master=http://0.0.0.0:8080 \`
`-run-server=false` \
`-use-ganesha=false`

We may want to enable per-PV quota enforcement. It is based on xfs project level quotas and so requires that the volume mounted at /export be xfs mounted with the prjquota/pquota option. 
Add the -enable-xfs-quota=true argument to enable it.


* $ sudo ./nfs-provisioner `-provisioner=example.com/nfs` \
`-kubeconfig=$HOME/.kube/config `\
`-run-server=false` \
`-use-ganesha=false` \
`-enable-xfs-quota=true`



* Arguments

provisioner - Name of the provisioner. The provisioner will only provision volumes for claims that request a StorageClass with a provisioner field set equal to this name.

master      - Master URL to build a client config from. Either this or kubeconfig needs to be set if the provisioner is being run out of cluster.

kubeconfig  - Absolute path to the kubeconfig file. Either this or master needs to be set if the provisioner is being run out of cluster.

run-server  - If the provisioner is responsible for running the NFS server, i.e. starting and stopping NFS Ganesha. Default true.

use-ganesha - If the provisioner will create volumes using NFS Ganesha (D-Bus method calls) as opposed to using the kernel NFS server ('exportfs'). If run-server is true, this must be true. Default true.

grace-period - NFS Ganesha grace period to use in seconds, from 0-180. If the server is not expected to survive restarts, i.e. it is running as a pod & its export directory is not persisted, this can be set to 0. Can only be set if both run-server and use-ganesha are true. Default 90.

enable-xfs-quota - If the provisioner will set xfs quotas for each volume it provisions. Requires that the directory it creates volumes in ('/export') is xfs mounted with option prjquota/pquota, and that it has the privilege to run xfs_quota. Default false.

failed-retry-threshold - If the number of retries on provisioning failure need to be limited to a set number of attempts. Default 10

server-hostname - The hostname for the NFS server to export from. Only applicable when running out-of-cluster i.e. it can only be set if either master or kubeconfig are set. 
If unset, the first IP output by hostname -i is used.


The nfs-provisioner has been deployed and is now watching for claims it should provision volumes for. No such claims can exist until a properly configured StorageClass for claims to request is created.



A StorageClass provides a way for administrators to describe the “classes” of storage they offer. Different classes might map to quality-of-service levels, or to backup policies, or to arbitrary policies determined by the cluster administrators.

Each StorageClass contains the fields provisioner, parameters, and reclaimPolicy, which are used when a PersistentVolume belonging to the class needs to be dynamically provisioned.
The name of a StorageClass object is significant, and is how users can request a particular class. Administrators set the name and other parameters of a class when first creating StorageClass objects, and the objects cannot be updated once they are created.


Example: 

#### step1 (Creating the StorageClass)

the below text as part of class.yaml

```yaml

kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: example-nfs
provisioner: example.com/nfs
mountOptions:
  - vers=4.1

```

* $ kubectl create -f class.yaml

#### step2 (Creating the Claim)

when we create a claim requesting the above class which is just created, the provisioner will automatically create a volume.

the below text as part of claim.yaml

```yaml

kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: nfs
  annotations:
    volume.beta.kubernetes.io/storage-class: "example-nfs"
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Mi

```
* $ kubectl create -f claim.yaml


#### step3(Using the Claim in POD)

```yaml

kind: Pod
apiVersion: v1
metadata:
  name: write-pod
spec:
  containers:
  - name: write-pod
    image: gcr.io/google_containers/busybox:1.24
    command:
      - "/bin/sh"
    args:
      - "-c"
      - "touch /mnt/SUCCESS && exit 0 || exit 1"
    volumeMounts:
      - name: nfs-pvc
        mountPath: "/mnt"
  restartPolicy: "Never"
  volumes:
    - name: nfs-pvc
      persistentVolumeClaim:
        claimName: nfs

```

Note: Brought up the NFS-provisioner on the VM where k8s is deployed (may be using the hack/local-up-cluster.sh ). if on different VMs then need to take care of the 
static address configurations, communication among VMs, then we may need to mount the nfs-server shared path on to the VM where we are deploying the POD.

* [For More Info on Setup](https://github.com/kubernetes-incubator/external-storage/blob/master/nfs/docs/deployment.md)
* [For More Info on Usage](https://github.com/kubernetes-incubator/external-storage/blob/master/nfs/docs/usage.md)

## *NFS Client Provisioner SetUp*

nfs-client is an automatic provisioner that use your existing and already configured NFS server to support dynamic provisioning of Kubernetes Persistent Volumes via Persistent Volume Claims. 

Download the project into your GOPATH directory by using go get or cloning it manually

* $ go get github.com/kubernetes-incubator/external-storage

* $ git clone `https://github.com/kubernetes-incubator/external-storage.git`

bash-shell open in the nfs-client directory.Edit the below files for required parameters like NFS SERVER HOSTNAME

* $ kubectl create -f deploy/rbac.yaml

* $ kubectl create -f deploy/deployment.yaml

the below text as part of the deploy/class.yaml

```yaml

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: managed-nfs-storage
provisioner: fuseim.pri/ifs # or choose another name, must match deployment's env PROVISIONER_NAME'
parameters:
  archiveOnDelete: "false"
```


* $ kubectl create -f deploy/class.yaml

* $ kubectl create -f deploy/test-claim.yaml -f deploy/test-pod.yaml

Now check your NFS Server for the file SUCCESS. [For More Info](https://github.com/kubernetes-incubator/external-storage/tree/master/nfs-client)

#  Simple SetUp and Usage in Kubegene

## pre requisites

* kubernetes is up with minikube on a machine/laptop
* nfs-kernel-server is installed

## Steps to Setup nfs-provisioner & create pv/pvc & use them

### just clone  and build the project

* $ `cd  $GOPATH/src/github.com/kubernetes-incubator`
* $ `git clone https://github.com/kubernetes-incubator/external-storage.git`
* $ `cd  $GOPATH/src/github.com/kubernetes-incubator/external-storage/nfs`
* $ make

### brought up the nfs-provisioner as shown below (supposed to create  /export folder in that machine where all the data is stored)

* $ sudo ./nfs-provisioner `-provisioner=example.com/nfs` \
`-kubeconfig=$HOME/.kube/config` \
`-run-server=false `\
`-use-ganesha=false`

### create the StorageClass 

the below text as part of class.yaml

```yaml

kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: example-nfs
provisioner: example.com/nfs
mountOptions:
  - vers=4.1

```

* $ kubectl create -f class.yaml

### create the PV and PVC 

when we create a claim requesting the above class which is just created, the provisioner will automatically create a volume.

the below text as part of claim.yaml

```yaml

kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: nfs
  annotations:
    volume.beta.kubernetes.io/storage-class: "example-nfs"
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Mi

```
* $ kubectl create -f claim.yaml

so then we can use the nfs instead of the execution-pvc in example/execution samples.

Note: When we follow above steps then we need not required to create pvc/pv as explanied in [README](https://github.com/kubegene/kubegene/blob/master/example/execution/README.md)


