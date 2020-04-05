---
title: Get started
tags: [formatting]
keywords: notes,
summary: "play with some definition on k8s like pod, deployment, services, networking and other"
sidebar: mydoc_sidebar
permalink: mydoc_k8s.html
folder: mydoc
toc: false
---

## Learn K8s
---
catatan selama ngoprek k8s di baremetal menggunakan 4 vm. Sebelum mulai perlu setup 4 vm tersebut seperti:
- ubuntu 18.04
- kubeadm
- docker
- spec:
-- master: 2 core 4 GB
-- worker: 1 core 2

### Master and Worker

saya menggunakan satu master dan 3 worker sebagai berikut

| VM Type | IP Address | Domain |
|-------|--------|---------|
| master | 192.168.0.100 | master.ridwan.com |
| worker-1 | 192.168.0.101 | worker-1.ridwan.com |
| worker-2 | 192.168.0.102 | worker-2.ridwan.com |
| worker-3 | 192.168.0.103 | worker-3.ridwan.com |
