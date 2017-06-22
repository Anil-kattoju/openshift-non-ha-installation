# non ha install TL;DR

### On all nodes
```sh
subscription-manager register --username=${user_name} --password=${password}

subscription-manager list --available --matches '*OpenShift*'
subscription-manager attach --pool=${pool_id}


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

### Installer Helper
```yml
  openshift_master_default_subdomain: apps.osecloud.com
  openshift_docker_insecure_registries: 172.30.0.0/16
  openshift_master_api_port: 443
  openshift_master_console_port: 443
  openshift_hosted_manage_router: True
  openshift_hosted_manage_registry: True
  openshift_hosted_router_selector: 'region=infra'
  openshift_hosted_registry_selector: 'region=infra'
  openshift_hosted_metrics_deploy: True
  openshift_hosted_metrics_public_url: https://hawkular-metrics.apps.ck.osecloud.com/hawkular/metrics
  openshift_master_logging_public_url: https://kibana.apps.ck.osecloud.com
  openshift_hosted_logging_deploy: True
  openshift_master_identity_providers: [{'name': 'allow_all', 'login': 'true', 'challenge': 'true', 'kind': 'AllowAllPasswordIdentityProvider'}]
```
