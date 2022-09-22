### Create On Premise K8 cluster

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
#  Install

