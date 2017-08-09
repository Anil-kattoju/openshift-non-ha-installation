# non ha install TL;DR

### List of nodes with a specific label
> with type=cluster and Values=Chak labels   
```sh
aws ec2 describe-instances   --filters "Name=tag:cluster,Values=chak" |   jq -j '.Reservations[].Instances[] | .PrivateIpAddress, "  ", .PublicIpAddress, "\n"'  
```

### Update route 53 rules
> Choose one of them as master and one of them as router.
```sh
export record_name=apps.ck.osecloud.com
export record_value=13.59.37.234
export ttl=60
export action=UPSERT
export record_type=A

export zone_id=$(aws route53 list-hosted-zones | jq -r ".HostedZones[] | select(.Name == \"ck.osecloud.com.\") | .Id" | cut -d'/' -f3)


function change_batch() {
	jq -c -n "{\"Changes\": [{\"Action\": \"$action\", \"ResourceRecordSet\": {\"Name\": \"$record_name\", \"Type\": \"$record_type\", \"TTL\": $ttl, \"ResourceRecords\": [{\"Value\": \"$record_value\"} ] } } ] }"
}

aws route53 change-resource-record-sets --hosted-zone-id ${zone_id} --change-batch $(change_batch) | jq -r '.ChangeInfo.Id' | cut -d'/' -f3
```
> Repeat it for *.apps and also ck.osecloud.com  


### On all nodes
```sh
sudo subscription-manager register --username=${user_name} --password=${password}

sudo subscription-manager list --available --matches '*OpenShift*'
sudo subscription-manager attach --pool=${pool_id}


sudo subscription-manager repos  --disable=*
sudo subscription-manager repos \
    --enable="rhel-7-server-rpms" \
    --enable="rhel-7-server-extras-rpms" \
    --enable="rhel-7-server-ose-3.5-rpms" \
    --enable="rhel-7-fast-datapath-rpms"
    
sudo yum -y install wget git net-tools bind-utils iptables-services bridge-utils bash-completion vim
sudo yum -y update
sudo yum -y install atomic-openshift-utils
sudo yum -y install atomic-openshift-excluder atomic-openshift-docker-excluder
sudo atomic-openshift-excluder unexclude

sudo yum -y install docker    
sudo sed -i '/OPTIONS=.*/c\OPTIONS="--selinux-enabled --insecure-registry 172.30.0.0/16"' /etc/sysconfig/docker

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
  openshift_master_named_certificates: [{"certfile": "/home/ec2-user/vault/ck.osecloud.com/fullchain.pem", "keyfile": "/home/ec2-user/vault/ck.osecloud.com/privkey.pem", "names":["ck.osecloud.com"]}]
  openshift_master_overwrite_named_certificates: true
  #openshift_master_identity_providers=[{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider', 'filename': '/etc/origin/master/htpasswd'}]
  #openshift_master_htpasswd_users={'admin': 'Admin@1'}
```




