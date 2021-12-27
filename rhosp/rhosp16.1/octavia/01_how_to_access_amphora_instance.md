# How to access amphora instances via SSH
## Environment:
- RHOSP 16.1
## Steps 
1. On any of the controller nodes, check `o-hm0` device exists: 
~~~
[root@overcloud-controller-0 ~]# ip addr show dev o-hm0
5: o-hm0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1442 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether fa:16:3e:df:12:01 brd ff:ff:ff:ff:ff:ff
    inet 172.24.2.207/16 brd 172.24.255.255 scope global o-hm0
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fedf:1201/64 scope link 
       valid_lft forever preferred_lft forever
~~~

2. Check lb-mgmt-net IP address allocated to amphora instance:
~~~
$ source overcloudrc
$ openstack server list --all | grep amphora
| 1974c635-8bee-4680-a029-da3b66612616 | amphora-a7bebaa4-bc72-4708-a285-8e8b00bb4897 | ACTIVE  | adminPrivate=192.0.1.109; lb-mgmt-net=172.24.1.56  | octavia-amphora-16.1-20211111.1.x86_64 |        |
| 02bd2329-b7cd-439e-b0ff-76af2df5d550 | amphora-f74df421-b342-42ca-8f24-079af66a64df | ACTIVE  | adminPrivate=192.0.1.209; lb-mgmt-net=172.24.0.167 | octavia-amphora-16.1-20211111.1.x86_64 |        |
~~~

3. Check ctlplane IP address on the controller node of step 1: 
~~~
$ source ~/stackrc
$ openstack server list | grep controller-0
| 11b371a0-abfa-4c5c-b6ed-757d82662107 | overcloud-controller-0  | ACTIVE | ctlplane=192.168.24.15 | overcloud-full | control |
~~~

4. Create a tunnel to the amphora instance on undercloud node: 
~~~
$ ssh -N -f -L 127.0.0.1:54322:172.24.1.56:22 heat-admin@192.168.24.15
~~~

5. Login to amphora instance:
~~~
$ ssh cloud-user@127.0.0.1 -p 54322
~~~

**NOTE:** `id_rsa.pub` will be uploaded as `octavia-ssh-key` when deploying octavia. 
