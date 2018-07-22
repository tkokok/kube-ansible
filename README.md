[![Build Status](https://travis-ci.org/kairen/kube-ansible.svg?branch=master)](https://travis-ci.org/kairen/kube-ansible) [![pipeline status](https://gitlab.com/inwinstack/kube-ansible/badges/pr-94/pipeline.svg)](https://gitlab.com/inwinstack/kube-ansible/commits/pr-94)
# Ansible playbooks to build Kubernetes


Forked from [https://github.com/kairen/kube-ansible](https://github.com/kairen/kube-ansible)


Kubernetes version support:
- [x] v1.8.x, upto 1.8.14
- [x] v1.9.x, upto 1.9.9
- [x] v1.10.x, upto 1.10.5
- [x] v1.11.1,

CNI plugins support:
- [x] overlay: flannel, calico(IPIP)
- [x] underlay: calico #Need GBP support or edit static routing manually

DNS plugins support:
- [x] kube-dns
- [x] coredns #GA from v1.11.0

Monitor plugins support:
- [x] heapster #[Deprecation Timeline](https://github.com/kubernetes/heapster/blob/master/docs/deprecation.md)
- [x] prometheus
- [x] weave-scope

Cluster network mechanism:
- [x] iptables
- [x] ipvs #GA from v1.11.0

## Manual deployment
In this section you will manually deploy a cluster on your machines.

##### Prerequisites:
* *Ansible version: v2.4.0 (or newer)*.
* *Linux distributions*: Ubuntu 16+/CentOS 7.x.
* All Master/Node should have password-less access from `Deploy` node.

```sh
(ansible2.4) tkokok@suigintou ~/deploy/kube-ansible $ ansible --version
ansible 2.4.2.0
```

##### For machine example:

| IP Address      |   Role           |   CPU    |   Memory   |
|-----------------|------------------|----------|------------|
| 10.0.100.100    | vip              |    -     |     -      |
| 10.0.10.1       | master1          |    2     |     4G     |
| 10.0.10.2       | master2          |    2     |     4G     |
| 10.0.10.3       | master3          |    2     |     4G     |
| 10.0.20.1       | etcd1            |    2     |     2G     |
| 10.0.20.2       | etcd2            |    2     |     2G     |
| 10.0.20.3       | etcd3            |    2     |     2G     |
| 10.0.20.3       | node1            |    4     |     8G     |
| 10.0.20.3       | node2            |    4     |     8G     |


##### For inventory example:
Add the machine info gathered above into a file called `inventory`.
```
[etcds]
10.0.20.[1:3]
[masters]
10.0.10.[1:3]
[nodes]
10.0.20.1[1:8]

[certs:children]
masters
etcds
[kube-cluster:children]
masters
etcds
nodes
[kube-addon:children]
masters
[versions:children]
kube-cluster

# IF_NOT_SSH_KEY
# apt-get -y install sshpass
# yum install -y install sshpass
[all:vars]
ansible_user=root
ansible_password=SSH_PASSWD
```


##### For vars example:
Set the variables in `group_vars/all.yml` to reflect you need options.
```yml
# Kubenrtes version,
#major_version:11
#minor_version:1

# CNI plugin,
# Supported network: flannel, calico, canal, weave or router.
network: calico
pod_network_cidr: 10.244.0.0/16

# CNI opts: flannel(--iface=enp0s8), calico(interface=enp0s8), canal(enp0s8).
#cni_iface: interface=eth1

lb_vip_address: 172.16.35.9

# Extra addons
kube_dashboard: true
kube_monitoring: true
```

### Deploy a Kubernetes cluster
If everything is ready, just run `cluster.yml` to deploy cluster:
```sh
$ ansible-playbook cluster.yml
```
```
 ____________
< PLAY RECAP >
 ------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

10.0.10.1                  : ok=140  changed=68   unreachable=0    failed=0
10.0.10.2                  : ok=110  changed=50   unreachable=0    failed=0
10.0.10.3                  : ok=106  changed=46   unreachable=0    failed=0
10.0.20.1                  : ok=31   changed=1    unreachable=0    failed=0
10.0.20.11                 : ok=52   changed=34   unreachable=0    failed=0
10.0.20.12                 : ok=51   changed=33   unreachable=0    failed=0
10.0.20.2                  : ok=31   changed=1    unreachable=0    failed=0
10.0.20.3                  : ok=31   changed=1    unreachable=0    failed=0

Sunday 22 July 2018  12:54:52 +0800 (0:00:00.829)       0:10:23.113 ***********
===============================================================================
kubernetes/master : Wait for Kubernetes core component start ------------------------------------------------------------------------------------------------- 93.05s
commons/packages : Downloading Kubernetes kubelet component binaries ----------------------------------------------------------------------------------------- 78.59s
commons/packages : Downloading Kubernetes kubelet component binaries ----------------------------------------------------------------------------------------- 59.50s
commons/packages : Downloading Kubernetes CLI tool binaries -------------------------------------------------------------------------------------------------- 23.67s
commons/container-images : Pull kube-apiserver-amd64 image --------------------------------------------------------------------------------------------------- 20.05s
Gathering Facts ---------------------------------------------------------------------------------------------------------------------------------------------- 18.37s
Gathering Facts ---------------------------------------------------------------------------------------------------------------------------------------------- 17.10s
kubernetes/gen-configs : Copy Kubernetes certificate from ansible host --------------------------------------------------------------------------------------- 16.22s
commons/packages : Downloading Calicoctl network plugin binaries --------------------------------------------------------------------------------------------- 15.03s
Gathering Facts ---------------------------------------------------------------------------------------------------------------------------------------------- 13.10s
commons/packages : Downloading Docker engine archives -------------------------------------------------------------------------------------------------------- 12.82s
gen-certs : Fetch files from remote host --------------------------------------------------------------------------------------------------------------------- 12.21s
etcd : Enable and restart Etcd -------------------------------------------------------------------------------------------------------------------------------- 9.00s
commons/packages : Downloading Container network interface archives ------------------------------------------------------------------------------------------- 8.53s
commons/packages : Downloading Kubernetes package manager archives -------------------------------------------------------------------------------------------- 6.30s
Gathering Facts ----------------------------------------------------------------------------------------------------------------------------------------------- 6.03s
commons/packages : Extract Docker engine file ----------------------------------------------------------------------------------------------------------------- 5.22s
commons/packages : Downloading Container network interface archives ------------------------------------------------------------------------------------------- 4.97s
gen-certs : Check SSL certificate json files ------------------------------------------------------------------------------------------------------------------ 4.62s
kubernetes/master : Create bootstrap clusterrole binding ------------------------------------------------------------------------------------------------------ 4.58s
...
```
And then run `addons.yml` to create addons:
```sh
$ ansible-playbook addons.yml
```
```
 ____________
< PLAY RECAP >
 ------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

10.0.10.1                  : ok=20   changed=17   unreachable=0    failed=0
10.0.10.2                  : ok=1    changed=0    unreachable=0    failed=0
10.0.10.3                  : ok=1    changed=0    unreachable=0    failed=0

Sunday 22 July 2018  12:57:34 +0800 (0:00:04.102)       0:00:54.103 ***********
===============================================================================
Gathering Facts ---------------------------------------------------------------------------------------------------------------------------------------------- 22.20s
kubernetes/addon : Copy kube-logging yml files ---------------------------------------------------------------------------------------------------------------- 5.25s
kubernetes/addon : Apply nginx-ingress addon ------------------------------------------------------------------------------------------------------------------ 4.10s
kubernetes/addon : Apply kube-logging addon ------------------------------------------------------------------------------------------------------------------- 3.94s
kubernetes/addon : Copy nginx-ingress yml files --------------------------------------------------------------------------------------------------------------- 3.32s
kubernetes/addon : Copy kube-monitoring yml files ------------------------------------------------------------------------------------------------------------- 2.69s
kubernetes/addon : Apply kube-monitoring addon ---------------------------------------------------------------------------------------------------------------- 2.16s
kubernetes/addon : Copy kube-dashboard yml files -------------------------------------------------------------------------------------------------------------- 1.88s
kubernetes/addon : Create dashboard certificates -------------------------------------------------------------------------------------------------------------- 1.80s
kubernetes/addon : Apply kube-dashboard addon ----------------------------------------------------------------------------------------------------------------- 1.59s
kubernetes/addon : Check dashboard secret already exists ------------------------------------------------------------------------------------------------------ 0.63s
kubernetes/addon : Create Kubernetes dashboard secret --------------------------------------------------------------------------------------------------------- 0.60s
kubernetes/addon : Create Kubernetes addon directory ---------------------------------------------------------------------------------------------------------- 0.60s
kubernetes/addon : Create Kubernetes addon directory ---------------------------------------------------------------------------------------------------------- 0.54s
kubernetes/addon : Create Kubernetes addon directory ---------------------------------------------------------------------------------------------------------- 0.48s
kubernetes/addon : Create Kubernetes addon directory ---------------------------------------------------------------------------------------------------------- 0.47s
kubernetes/addon : Check dashboard certificates already exists ------------------------------------------------------------------------------------------------ 0.47s
kubernetes/addon : Delete dashboard pass key ------------------------------------------------------------------------------------------------------------------ 0.44s
kubernetes/addon : Create dashboard certificates directory ---------------------------------------------------------------------------------------------------- 0.44s
kubernetes/addon : Config Kubernetes dashboard ---------------------------------------------------------------------------------------------------------------- 0.13s
```
### Reset cluster
You can reset cluster with the `reset.yml` playbook:
```sh
$ ansible-playbook reset.yml
```

## Verify cluster
Now, check the service as follow:
```sh
$ sshpass -e ssh root@10.0.100.100 "kubectl get node -nkube-system -owide"
NAME       STATUS    ROLES     AGE       VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION               CONTAINER-RUNTIME
master-1   Ready     master    18m       v1.11.0   10.0.10.1     <none>        CentOS Linux 7 (Core)   3.10.0-514.26.2.el7.x86_64   docker://18.3.1
master-2   Ready     master    18m       v1.11.0   10.0.10.2     <none>        CentOS Linux 7 (Core)   3.10.0-514.26.2.el7.x86_64   docker://18.3.1
master-3   Ready     master    18m       v1.11.0   10.0.10.3     <none>        CentOS Linux 7 (Core)   3.10.0-514.26.2.el7.x86_64   docker://18.3.1
node-11    Ready     <none>    14m       v1.11.0   10.0.20.11    <none>        CentOS Linux 7 (Core)   3.10.0-862.3.3.el7.x86_64    docker://18.3.1
node-12    Ready     <none>    14m       v1.11.0   10.0.20.12    <none>        CentOS Linux 7 (Core)   3.10.0-862.3.3.el7.x86_64    docker://18.3.1
```

## Config dashboard

As of release 1.7 Dashboard no longer has full admin privileges granted by default, so you need to create a token to access the resources:
```sh
$ kubectl -n kube-system create sa dashboard
$ kubectl create clusterrolebinding dashboard --clusterrole cluster-admin --serviceaccount=kube-system:dashboard
$ kubectl -n kube-system get sa dashboard -o yaml
$ sleep 5
$ kubectl describe secret -nkube-system $(kubectl get secret -n kube-system | awk '{if ($1 ~ /^dashboard-token-/) {print $1}}') | awk 'END{print $NF}'
```

```sh
eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkYXNoYm9hcmQtdG9rZW4tc2o3dnciLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGFzaGJvYXJkIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiNzk4MWNiMTgtOGQ2Zi0xMWU4LWE1ZjUtMDAwYzI5NmZjMTVlIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOmRhc2hib2FyZCJ9.audnd_-b-bMESjKcGLZDtaeMojtT-AfbopQrCmg16kYlZyS5RFC_a8kmFTul2Ch3P1hifghgax6z75dImM1xCGG0NpUCqwf0Q3iF0JQw0UErMW1_nlsoFz7lhTZw_DZyAfW4f4iCzGbgG___-NovqiWku1esQ6bIJBmg-c0fKlGpoAIICTjBbqTsTNt7X0JaYVY-NNl3ATkjFcwSdhkYLpV4_vYn2k8HO9ETBOP7a2x77x6UDtRyxMo7Sky2VIFwgVGvVZAoFhXmnpzqYhWnCTIaMTF5VvInvmxuExjFWI4L8A5nk5Zxyx_BRC2dgEe1paM3Bz_OMgU6baR3uXHk_g
```

##### Login the addon's dashboard:
- Dashboard: [https://API_SERVER:6443/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/](https://API_SERVER:6443/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/)
- Logging: [https://API_SERVER:6443/api/v1/namespaces/kube-system/services/kibana-logging/proxy](https://API_SERVER:6443/api/v1/namespaces/kube-system/services/kibana-logging/proxy)
- Monitor: [https://API_SERVER:6443/api/v1/namespaces/kube-system/services/monitoring-grafana/proxy/](https://API_SERVER:6443/api/v1/namespaces/kube-system/services/monitoring-grafana/proxy/)

> Copy and paste the `token` to dashboard.


## Thanks
Thanks to `Kyle Bai` `@github` https://github.com/kairen
