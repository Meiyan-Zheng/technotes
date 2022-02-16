# RHOCP4.9 Troubleshoot Guide - Get basic information for cluster and node

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

**To get more detailed cluster status information:**
~~~
[root@ocpclient ~]# oc describe clusterversion
Name:         version
Namespace:    
Labels:       <none>
Annotations:  <none>
API Version:  config.openshift.io/v1
Kind:         ClusterVersion
...
Spec:
  Channel:     stable-4.9 <-----
  Cluster ID:  36fd8dd7-9a08-4535-b518-29e0435f3efa
...
  History:
    Completion Time:    2022-02-13T15:40:58Z <-----
    Image:              quay.io/openshift-release-dev/ocp-release@sha256:dc6d4d8b2f9264c0037ed0222285f19512f112cc85a355b14a66bd6b910a4940
    Started Time:       2022-02-13T14:46:42Z
    State:              Completed <-------
    Verified:           false
    Version:            4.9.9
  Observed Generation:  1
  Version Hash:         z9gGQmq8n88=
Events:                 <none>
~~~

4. Get list of cluster operator:
~~~
[root@ocpclient ~]# oc get clusteroperators
NAME                                       VERSION   AVAILABLE   PROGRESSING   DEGRADED   SINCE   MESSAGE
authentication                             4.9.9     True        False         False      39h     
baremetal                                  4.9.9     True        False         False      2d14h   
cloud-controller-manager                   4.9.9     True        False         False      2d14h   
cloud-credential                           4.9.9     True        False         False      2d14h   
...
~~~

5. Get logs of each node:
~~~
[root@ocpclient ~]# oc adm node-logs all.ospd.example.com
~~~

Or get journal logs for particular unit with `-u` option:
~~~
[root@ocpclient ~]# oc adm node-logs all.ospd.example.com -u kubelet
~~~

6. Open a shell prompt on an OpenShift node:
~~~
[root@ocpclient ~]# oc debug node/all.ospd.example.com
Starting pod/allospdexamplecom-debug ...
To use host binaries, run `chroot /host`

Pod IP: xx.xx.xx.xx
If you don't see a command prompt, try pressing enter.
sh-4.4# 
sh-4.4# chroot /host
sh-4.4# crictl ps
...
~~~
