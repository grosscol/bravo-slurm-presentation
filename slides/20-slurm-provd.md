---
author: Colin Gross
title: BRAVO Slurm
date: 2022-05-23
---

# 
<h3>Easy Provisioning</h3>
```
∫ workspace/slurm_net (main)⮞ terraform apply
```
```txt
filestore_ip_app = "10.195.88.34"
filestore_ip_home = "10.8.189.242"
filestore_name_app = "slurm_app"
filestore_name_home = "slurm_home"
network_name = "bravo-slurm-vpc"
subnet_name = "bravo-slurm-subnet"
```

```
∫ workspace/slurm_vms (deb_img)⮞ terraform apply
```
```
login_nat_ips = [
  "34.72.34.241",
]
```

##
### Slurm Provisioned

![](assets/slurm_login.svg){ width=650px }

##
### Check Cluster
Single partition (queue) with 3 compute nodes
```txt
grosscol_umich_edu@dpclust-login0:~$ sinfo
```
```txt
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
debug*       up   infinite      3   idle dpclust-compute-0-[0-2]
```
