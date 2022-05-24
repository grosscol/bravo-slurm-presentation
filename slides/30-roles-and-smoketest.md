---
author: Colin Gross
title: BRAVO Slurm
date: 2022-05-23
---

# 
<h3>Ansible Roles</h3>
Project organization where related tasks can be grouped together, and run independently.
```
roles/
├── basis_data
├── data_prep
├── gcc
├── htslib
├── libstatgen
├── loftee
├── nextflow
├── packages
├── ref_data
├── samtools
├── test_data
├── vep
└── vep_cache
```

##
### Group Roles in top level playbook
```yaml
- hosts: login
  become: yes
  tags: 
    - base
  roles:
    - packages
    - shared_dir_setup
```
```yaml
- hosts: login
  tags: 
    - backing_data
  roles:
    - ref_data
    - basis_data
    - test_data
```

# 
<h3>Test that Cluster is Working</h3>
```
grosscol_umich_edu@dpclust-login0:
  ~/nf_smoke_test$ nextflow run smoke_test.nf -profile slurm
```
```
N E X T F L O W  ~  version 21.10.6
Launching `smoke_test.nf` [high_cajal] - revision: e42f5cc55c
executor >  slurm (51)
[65/e2c998] process > calc_md5s (49) [100%] 50 of 50 ✔
[f1/71365e] process > collect_md5s   [100%] 1 of 1 ✔
Completed at: 24-May-2022 15:10:13
Duration    : 1m 51s
CPU hours   : (a few seconds)
Succeeded   : 51
```
