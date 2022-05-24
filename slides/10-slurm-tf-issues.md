---
author: Colin Gross
title: BRAVO Slurm
date: 2022-05-23
---

#
<h3>SchedMD</h3>

> SchedMD® is the core company behind the Slurm workload manager software, a free open-source workload manager designed specifically to satisfy the demanding needs of high performance computing.

##
### Provider's Terraform
- SchedMD slurm-gcp [project on github](https://github.com/SchedMD/slurm-gcp/tree/e82eba4336f05e31618810ae2df0074c10339ec7)
- Has a terraform directory (`tf`)
    - Has modules!
    - Has examples!

##
### Only *Slightly* Outdated
Current terraform version is 1.2.1
```tf
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = ">= 3.54"
    }
  }

  required_version = ">= 0.12.20"
}
```

# 
<h3>First yak: Update Terraform Version</h3>

- Terraform version 0.12.2 to 1.1.7

##
### Variables

Variables in child modules are no longer automagically present.

<pre>
<font color="#C4A000"><b>Warning: </b></font><b>Value for undeclared variable</b>
<font color="#C4A000"><b>Warning: </b></font><b>Value for undeclared variable</b>
<font color="#C4A000"><b>Warning: </b></font><b>Value for undeclared variable</b>
<font color="#C4A000"><b>Warning: </b></font><b>Value for undeclared variable</b>
</pre>

##
### Solution: Placeholders
- Add placeholders to `variables.tf` in top level module.
```tf
# Placeholders for vars of modules passed verbatim
#   See io.tf of modules for full description.
variable "controller_machine_type" {}
variable "controller_image"        {}
variable "controller_disk_type"    {}
variable "controller_disk_size_gb" {}
```

#
<h3>Terraform Cloud</h3>

> Terraform Cloud is an application that helps teams use Terraform together. 

- Manages Terraform runs in a consistent and reliable environment
- Easy access to shared state
- Securely stores secret data

##
### Shared Variables
![](assets/tf_cloud_vars.png){ width=600px }

##
### Shared Environment
```txt
∫ workspace/slurm_vms (deb_img)⮞ terraform plan
```
Runs on terraform cloud remote VM:
```txt
Running plan in Terraform Cloud. Output will stream here. Pressing Ctrl-C
will stop streaming the logs, but will not stop the plan running remotely.

Preparing the remote plan...

To view this run in a browser, visit:
https://app.terraform.io/app/statgen/bravo-slurm/runs/run-tbnYAVwbcp9W2R5s

Waiting for the plan to start...

Terraform v1.1.7
on linux_amd64
Initializing plugins and modules...
```
##
### Shared Environment Scope

TF top level directory + cloud Workspace Vars
```txt
slurm_vms/
├── main.tf
├── modules/
├── outputs.tf
├── slurm.tfvars
├── variables.tf
└── versions.tf
```

# 
<h3>Second yak: Script references</h3>

In provider modules, found relative paths to outside project directory.

```tf
metadata_startup_script =
  file("${path.module}/../../../scripts/startup.sh")
```

Only one directory above modules is available!
```tree
slurm_vms/
├── main.tf
├── modules
│   ├── compute
│   ├── controller
│   └── login
├── variables.tf
└── versions.tf
```

##
### Solution: Common Scripts Module
```tree
└── modules
    ├── common
    │   └── scripts
    │       ├── startup.sh
    │       └── util.py
    ├── compute
    ├── controller
    └── login
```

Provide scripts from sibling module
```
metadata_startup_script = 
  module.common.metadata_startup_script
```
