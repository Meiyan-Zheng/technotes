# RHOCP4.9 Troubleshoot Guide

## Get basic information 
1. Get nodes status:
~~~
[root@ocpclient ~]# oc get nodes
NAME                   STATUS   ROLES           AGE     VERSION
all.ospd.example.com   Ready    master,worker   2d14h   v1.22.3+4dd1b5a
~~~

**Get detailed node information:**
~~~
[root@ocpclient ~]# oc describe node all.ospd.example.com
Name:               all.ospd.example.com
Roles:              master,worker
...
~~~

2. Get CPU/Memory usage for each node:
~~~
[root@ocpclient ~]# oc adm top nodes
NAME                   CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
all.ospd.example.com   2812m        8%     23162Mi         36%   
~~~

3. Get current cluster version:
~~~
[root@ocpclient ~]# oc get clusterversion
NAME      VERSION   AVAILABLE   PROGRESSING   SINCE   STATUS
version   4.9.9     True        False         2d14h   Cluster version is 4.9.9
~~~
