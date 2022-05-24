---
author: Colin Gross
title: BRAVO Slurm
date: 2022-05-23
---

#
<h3>Data Prep Deployment</h3>
- Data backing
- Installing Tools
- Making Slurm Modules

##
### Four More Yaks
![Slow downloads, Dependencies, Configuration, Compiling](assets/four_yak_crop.jpg)

#
<h3>Ansible</h3>
Does most of the things a remote shell script would do, but safer.

- Data acquisition
- Dependency management
- Build tools
- Copying files
- Idempotency
- Run commands (if no module available)

##
### Caching Backing Data 

- Download from Ensembl is slow.
- Download from GCP buckets is fast.
- Ansible block-rescue idiom lets us failback to Ensembl

```yaml
- name: Retrieve GERP scores for GRCh38
  block:
    - name: Copy GERP scores from cache bucket
      ansible.builtin.command:
        cmd: "gsutil cp {{bucket_gerp}} {{loftee_data_gerp}}"
        creates: "{{loftee_data_gerp}}"
  rescue:
    - name: Copy GERP scores from Ensembl
      ansible.builtin.get_url:
        url: "{{gerp_scores_url}}"
        dest: "{{loftee_data_gerp}}"
```

##
### Managing Dependencies
```yaml
- name: Install dependencies for pipeline tooling
  ansible.builtin.apt:
    pkg:
      - openjdk-11-jre-headless
      - cmake
      - zlib1g-dev
      - libbz2-dev
      - liblzma-dev
      - perl
      - cpanminus
      - libminizip1
      - minizip
      - unzip
    state: present
    update_cache: yes
```

##
### Building Tools
```yaml
- name: Clone lib StatGen dependency 
  ansible.builtin.git:
    depth: 1
    dest: "{{libstatgen_dir}}"
    repo: "https://github.com/statgen/libStatGen.git"

- name: Build lib StatGen dependency 
  community.general.make:
    chdir: "{{libstatgen_dir}}"
```
```yaml
- name: Clone StatGen bamUtil 
  ansible.builtin.git:
    depth: 1
    dest: "{{bamutil_dir}}"
    repo: "https://github.com/statgen/bamUtil.git"

- name: Build StatGen bamUtil 
  community.general.make:
    chdir: "{{bamutil_dir}}"
```

##
### Idempotency
Many ansible modules specify how they avoid unneccesary work in their attrs.  

- **creates**: If the specified absolute path (file or directory) already exists, this step will not be run.

- **force**: Influence whether the remote file must always be replaced. If yes, the remote file will be replaced when contents are different than the source.

##
### Idempotency using Lockfiles
When `creates` attribute not available, lockfile idiom is easy to implement.
```yaml
- name: Check if VEP install lockfile present
  ansible.builtin.stat:
    path: "{{vep_dir}}/.ansible_lock"
  register: vep_install_lock
```
```yaml
- name: Install VEP
  ansible.builtin.command:
    chdir: "{{vep_dir}}"
    cmd: "perl INSTALL.pl --CACHEDIR {{vep_cache_dir}} --PLUGINS all --AUTO ap -q -n"
  when: vep_install_lock.stat.exists == false

- name: Create VEP install lockfile after install
  ansible.builtin.copy:
    dest: "{{vep_dir}}/.ansible_lock"
    content: "VEP installed via Ansible"
  when: vep_install.changed
```

##
### Cluster Uses Service Account

Don't have to handle passing around credentials.
```txt
grosscol_umich_edu@dpclust-login0:~$ gsutil ls gs://bravo-deploy-cache
```
```txt
gs://bravo-deploy-cache/gerp_conservation_scores.homo_sapiens.GRCh38.bw
gs://bravo-deploy-cache/homo_sapiens_vep_106_GRCh38.tar.gz
gs://bravo-deploy-cache/hs38DH.fa
gs://bravo-deploy-cache/hs38DH.fa.fai
gs://bravo-deploy-cache/human_ancestor.fa.gz
gs://bravo-deploy-cache/human_ancestor.fa.gz.fai
gs://bravo-deploy-cache/human_ancestor.fa.gz.gzi
gs://bravo-deploy-cache/loftee.sql.gz
```
