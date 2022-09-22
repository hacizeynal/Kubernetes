# Create On Premise K8 cluster


## Install Container Runtime

The first step in setting up a new cluster is to install a container runtime such as Docker. We will be installing
Docker on our three servers in preparation for standing up a Kubernetes cluster. After completing this, we should have a cluster containing three servers, all with Docker up and running.

Here are the commands used for install Docker runtime:

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository \
"deb [arch=amd64] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) \
stable"
sudo apt-get update
sudo apt-get install -y docker-ce=18.06.1~ce~3-0~ubuntu
sudo apt-mark hold docker-ce

```

Following link can be used as a reference

```
https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository

```

## Install K8 components

Now that Docker is installed, we are ready to install the Kubernetes components. Installation includes process of installing Kubeadm, Kubelet, and Kubectl on all three playground servers. After completing this ,we should be ready for the next step, which is to bootstrap the cluster.

Here are the commands used to install the Kubernetes components in this lesson. Run these on all three servers.

```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

sudo apt-get update

sudo apt-get install -y kubelet=1.15.7-00 kubeadm=1.15.7-00 kubectl=1.15.7-00

sudo apt-mark hold kubelet kubeadm kubectl

```

After installing these components, verify that kubeadm is working by getting the version info.

```
kubeadm version
```

## Bootstrapping the Cluster


Now we are ready to get a real Kubernetes cluster up and running! We will bootstrap the cluster on the Kube master node. Then, we will join each of the two worker nodes to the cluster, forming an actual multi-node Kubernetes cluster.

On the Kube master node, initialize the cluster:

```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 
```

That command may take a few minutes to complete.

When it is done, set up the local kubeconfig:

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Verify that the cluster is responsive and that Kubectl is working:

```kubectl version```

You should get Server Version as well as Client Version . It should look something like this:

The kubeadm init command should output a kubeadm join command containing a token and hash. Copy that command and run it with sudo on both worker nodes. It should look something like this:

```
Client Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.7", GitCommit:"6c143d35bb11d74970e7bc0b6c45b6bfdffc0bd4", GitTreeState:"clean", BuildDate:"2019-12-11T12:42:56Z", GoVersion:"go1.12.12", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.12", GitCommit:"e2a822d9f3c2fdb5c9bfbe64313cf9f657f0a725", GitTreeState:"clean", BuildDate:"2020-05-06T05:09:48Z", GoVersion:"go1.12.17", Compiler:"gc", Platform:"linux/amd64"}
```

The kubeadm init command should output a kubeadm join command containing a token and hash. Copy that command and run it with sudo on both worker nodes. It should look something like this:

```
sudo kubeadm join $some_ip:6443 --token $some_token --discovery-token-ca-cert-hash $some_hash
sudo kubeadm join <IP_ADDRESS> --token <TOKEN> \
    --discovery-token-ca-cert-hash sha256:<HASH>

```

From your Kube Master node, verify that all nodes have successfully joined the cluster:

kubectl get nodes

You should see all three of your nodes listed. It should look something like this:

```
NAME                           STATUS     ROLES    AGE     VERSION
f8bbdd78c31c.mylabserver.com   NotReady   master   4m22s   v1.15.7
f8bbdd78c32c.mylabserver.com   NotReady   <none>   85s     v1.15.7
f8bbdd78c33c.mylabserver.com   NotReady   <none>   79s     v1.15.7
```

Note: The nodes are expected to have a STATUS of NotReady at this point



## Configuring Networking with Flannel

Once the Kubernetes cluster is set up, we still need to configure cluster networking in order to make the cluster fully functional.

In this lesson, we will walk through the process of configuring a cluster network using Flannel. You can find more information on Flannel at the official site: 

https://coreos.com/flannel/docs/latest/

Here are the commands used in this lesson:

On all three nodes, run the following:

```
echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

```

Install Flannel in the cluster by running this only on the Master node:

kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel-old.yaml

Verify that all the nodes now have a STATUS of "Ready":

kubectl get nodes

You should see all three of your servers listed, and all should have a STATUS of "Ready". It should look something like this:

NAME                           STATUS   ROLES    AGE   VERSION
f8bbdd78c31c.mylabserver.com   Ready    master   15m   v1.15.7
f8bbdd78c32c.mylabserver.com   Ready    <none>   12m   v1.15.7
f8bbdd78c33c.mylabserver.com   Ready    <none>   12m   v1.15.7

Note: It may take a few moments for all nodes to enter the "Ready" status, so if they are not all "Ready", wait a few moments and try again.

It is also a good idea to verify that the Flannel pods are up and running. Run this command to get a list of system pods:

kubectl get pods -n kube-system

# Expected output:

```
NAME                                                   READY   STATUS    RESTARTS   AGE
coredns-5d4dd4b4db-4hx4f                               1/1     Running   0          15m
coredns-5d4dd4b4db-jb6gz                               1/1     Running   0          15m
etcd-f8bbdd78c31c.mylabserver.com                      1/1     Running   0          14m
kube-apiserver-f8bbdd78c31c.mylabserver.com            1/1     Running   0          15m
kube-controller-manager-f8bbdd78c31c.mylabserver.com   1/1     Running   0          15m
kube-flannel-ds-amd64-mh2c7                            1/1     Running   0          71s
kube-flannel-ds-amd64-qk7zl                            1/1     Running   0          71s
kube-flannel-ds-amd64-srnb6                            1/1     Running   0          71s
kube-proxy-2g26k                                       1/1     Running   0          13m
kube-proxy-cqls4                                       1/1     Running   0          15m
kube-proxy-x7qwl                                       1/1     Running   0          12m
kube-scheduler-f8bbdd78c31c.mylabserver.com            1/1     Running   0          14m
```

We should have three pods with "flannel" in the name, and all three should have a status of "Running".





