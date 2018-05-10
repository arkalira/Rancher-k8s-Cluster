# Rancher-k8s-Cluster

Lets configure a Kubernetes cluster with three nodes DEBIAN 9.4 and on top of it we will configure Rancher to manage services, images, etc.

## Prerequisites

- 3 nodes Debian 9.4 with full network visibility between them.
- 1 FW or HAPROXY to make SSL Offloading with and interface in the same subnet as the 3 nodes.
- Basic installation of: wget, curl, etc

```
apt-get install -y vim mc uml-utilities ntp qemu-guest-agent \
htop sudo curl git git-core etckeeper zsh apt-transport-https ca-certificates \
bridge-utils gettext-base
usermod root -s /bin/zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
sed -ri 's/ZSH_THEME="robbyrussell"/ZSH_THEME="pygmalion"/g' .zshrc
sed -ri 's/plugins=\(git\)/plugins=\(debian apt systemd docker zsh-navigation-tools\)/g' .zshrc
echo 'export VTYSH_PAGER=more' >> /etc/zsh/zshenv
source .zshrc
```

## Install docker

- Install only one of the supported versions of Docker-CE. You can review the list in the following link: https://rancher.com/quick-start/ (For Rancher 2.0)

- Lets start installing Docker: https://docs.docker.com/install/linux/docker-ce/ubuntu/

```
apt-get remove docker docker-engine docker.io && apt-get update
apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
apt-get update && apt-cache madison docker-ce
apt-get install -y docker-ce=17.03.2~ce-0~debian-stretch
```
## Launch Rancher 2.0

- This method is not recommended for PRODUCTION ENVIRONMENTS.

### Launching with a single container and a bind mount MySQL Volume

```
mkdir -p /opt/mysql-rancher
docker run -d -v /opt/mysql-rancher:/var/lib/mysql --restart=unless-stopped -p 8080:8080 rancher/server
```

- See more launching methods here: https://rancher.com/docs/rancher/v1.6/en/installing-rancher/installing-server/#single-container-bind-mount

#### Access to Rancher

- In your browser: http://localhost:8080 or set rancher.ironshared.com to the public IP of the firewall or HAProxy that you put in front of Rancher in your etc/hosts or local DNS.

#### Add a Host to Rancher

Click in **Infrastructure - Host - Custom** and then put as many labels as you need and add the same host where you are running Rancher.

This will generate a script to register the new host:

```
sudo docker run -e CATTLE_AGENT_IP="10.200.1.10"  -e CATTLE_HOST_LABELS='hostname=rancher201' -e CATTLE_URL='http://rancher.ironshared.com/v1'  --rm --privileged -v /var/run/docker.sock:/var/run/docker.sock -v /var/lib/rancher:/var/lib/rancher rancher/agent:v1.2.10-rc5 http://rancher.ironshared.com/v1/scripts/4D5A8637A59D1FBA9FE5:1514678400000:WQ2rkACuz6TIlStbXbpEQtJtB4
```

**IMPORTANT:** be sure that rancher.ironshared.com is correctly resolved by the host before executing this line. Once this is launched, it will pull Rancher Agent and install in the designated server.

- At this point we detect some kind of bug related with CATTLE_URL: https://github.com/rancher/rancher/issues/12028

- As we dont have a valid DNS resolution to generate true SSL Certs, we would like to test it via HTTP, but CATTLE_URL is not accepted to be set as an environment variable at this moment.
