# RHOCP4.9 Troubleshoot Guide - Get information for pods

1. Get pods status:
~~~
[root@ocpclient ~]# oc get pods -n openstack
NAME                                                              READY   STATUS      RESTARTS        AGE
a7ecbd81a15819d6ae486200f421f20ae33b672e0a0bfaa23b3289--1-pxxww   0/1     Completed   0               2d3h
network-resources-injector-hbdv4                                  1/1     Running     0               21h
operator-webhook-lghcj                                            1/1     Running     0               21h
osp-director-operator-controller-manager-5545756d8c-4hcs8         2/2     Running     330 (21h ago)   2d3h
osp-director-operator-index-7pbkl                                 1/1     Running     0               2d3h
sriov-network-config-daemon-q6wft                                 3/3     Running     0               21h
sriov-network-operator-8699d87676-cljn8                           1/1     Running     0               21h
~~~

2. Get log for particular pod:
~~~
[root@ocpclient ~]# oc logs -n openstack operator-webhook-lghcj   
~~~

If multiple containers in one pod, specify container name with `-c` option:
~~~
[root@ocpclient ~]# oc logs -n openstack osp-director-operator-controller-manager-5545756d8c-4hcs8 
error: a container name must be specified for pod osp-director-operator-controller-manager-5545756d8c-4hcs8, choose one of: [manager kube-rbac-proxy]

[root@ocpclient ~]# oc logs -n openstack osp-director-operator-controller-manager-5545756d8c-4hcs8 -c manager
...
~~~

3. Create a troubleshooting pod from deployment:
~~~
[root@ocpclient ~]# oc get deployment
NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
osp-director-operator-controller-manager   1/1     1            1           2d4h
sriov-network-operator                     1/1     1            1           22h

[root@ocpclient ~]# oc debug deployment/sriov-network-operator --as-root
Starting pod/sriov-network-operator-debug, command was: sriov-network-operator --leader-elect
Pod IP: 10.128.0.146
If you don't see a command prompt, try pressing enter.
sh-4.4# 
~~~

