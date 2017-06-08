# non ha install TL;DR

### On all nodes
```sh
subscription-manager register --username=${user_name} --password=${password}

subscription-manager list --available --matches '*OpenShift*'
subscription-manager attach --pool=${pool_id}
subscription-manager repos --disable="*"

subscription-manager repos  --disable=*
subscription-manager repos \
    --enable="rhel-7-server-rpms" \
    --enable="rhel-7-server-extras-rpms" \
    --enable="rhel-7-server-ose-3.5-rpms" \
    --enable="rhel-7-fast-datapath-rpms"
    
yum -y install wget git net-tools bind-utils iptables-services bridge-utils bash-completion vim
yum -y update
yum -y install atomic-openshift-utils
yum -y install atomic-openshift-excluder atomic-openshift-docker-excluder
atomic-openshift-excluder unexclude

yum -y install docker    
sed -i '/OPTIONS=.*/c\OPTIONS="--selinux-enabled --insecure-registry 172.30.0.0/16"' /etc/sysconfig/docker

```

### On master
```sh
root@master# ssh-keygen  #copy public keys to all machines
atomic-openshift-installer install
```
