# RHOCP4.9 Troubleshoot Guide - Get information for project

1. Check project resource status:
~~~
[root@ocpclient ~]# oc project openstack
Already on project "openstack" on server "https://api.ospd.example.com:6443".

[root@ocpclient ~]# oc status
In project openstack on server https://api.ospd.example.com:6443

svc/network-resources-injector-service - 172.30.146.156:443 -> 6443
  daemonset/network-resources-injector manages registry.redhat.io/openshift4/ose-sriov-dp-admission-controller@sha256:544e149a78525abf934d0c2d82ecd321f4f14d56b790e13edb1be60e0ddc79ff
    generation #1 running for 23 hours - 1 pod

svc/operator-webhook-service - 172.30.146.24:443 -> 6443
  daemonset/operator-webhook manages registry.redhat.io/openshift4/ose-sriov-network-webhook@sha256:da3e115e52fbe5f0c39065e1a4e8df6418da293064f0b037b184f3c03ac45c80
    generation #1 running for 23 hours - 1 pod
...
~~~

2. Get error logs in this project:
~~~
[root@ocpclient ~]# oc get events
...
58m         Warning   FailedKillPod          pod/sriov-network-operator-debug           error killing pod: failed to "KillContainer" for "sriov-network-operator" with KillContainerError: "rpc error: code = NotFound desc = could not find container \"bc784924959281931d8802fb435f2ee39c709049f4ea8674989f6ad54b659755\": container with ID starting with bc784924959281931d8802fb435f2ee39c709049f4ea8674989f6ad54b659755 not found: ID does not exist"
~~~
