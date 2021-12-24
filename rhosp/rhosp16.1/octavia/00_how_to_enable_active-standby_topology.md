# How to deploy octavia with active-standby topology enabled

## Environment
- RHOSP 16.1
- OVN + geneve

## Steps
1. Create a custom YAML environment file:
~~~
$ cat templates/my-octavia-environment.yaml 
parameter_defaults:
    OctaviaLoadBalancerTopology: "ACTIVE_STANDBY"
~~~

2. Run `openstack overcloud deploy` command:
~~~
$ openstack overcloud deploy --templates \
-e [your-environment-files] \
-e /usr/share/openstack-tripleo-heat-templates/environments/services/octavia.yaml \
-e /home/stack/templates/my-octavia-environment.yaml
...
~~~

**NOTE:** A minimum of three Compute node hosts are required: two Compute node hosts to place the amphorae on different hosts (Compute anti-affinity), and a third host to successfully fail over an active-standby load balancer.

3. After deployment completed, you'll see network `lb-mgmt-net`, and interface created on controller nodes with name `o-hm0`:
~~~
$ openstack network list
+--------------------------------------+--------------+--------------------------------------+
| ID                                   | Name         | Subnets                              |
+--------------------------------------+--------------+--------------------------------------+
| 569cebc3-a460-4d58-8b9b-6077fb225570 | lb-mgmt-net  | 346212c0-da99-4483-9a10-f9b0094dc199 |
+--------------------------------------+--------------+--------------------------------------+

[root@overcloud-controller-0 ~]# ip a
...
5: o-hm0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1442 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether fa:16:3e:df:12:01 brd ff:ff:ff:ff:ff:ff
    inet 172.24.2.207/16 brd 172.24.255.255 scope global o-hm0
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fedf:1201/64 scope link 
       valid_lft forever preferred_lft forever
~~~

4. listing Load-balancing service provider capabilities:
~~~
$ source overcloudrc 
$ openstack loadbalancer provider capability list amphora
+-----------------------+-----------------------------------------------------------------------------------------------------------------------------+
| name                  | description                                                                                                                 |
+-----------------------+-----------------------------------------------------------------------------------------------------------------------------+
| loadbalancer_topology | The load balancer topology. One of: SINGLE - One amphora per load balancer. ACTIVE_STANDBY - Two amphora per load balancer. |
| compute_flavor        | The compute driver flavor ID.                                                                                               |
+-----------------------+-----------------------------------------------------------------------------------------------------------------------------+
~~~

5. Defining flavor profiles:
~~~
$ openstack loadbalancer flavorprofile create --name amphora-single-profile --provider amphora --flavor-data '{"loadbalancer_topology": "SINGLE"}'
+---------------+--------------------------------------+
| Field         | Value                                |
+---------------+--------------------------------------+
| id            | 50dea167-830b-450c-972b-a7360fe98841 |
| name          | amphora-single-profile               |
| provider_name | amphora                              |
| flavor_data   | {"loadbalancer_topology": "SINGLE"}  |
+---------------+--------------------------------------+
~~~

6. Creating Load-balancing service flavors:
~~~
$ openstack loadbalancer flavor create --name standalone-lb --flavorprofile amphora-single-profile --description "A non-high availability load balancer for testing."
+-------------------+----------------------------------------------------+
| Field             | Value                                              |
+-------------------+----------------------------------------------------+
| id                | c7084189-dfaf-44f3-ae9e-a3e2482ef9f7               |
| name              | standalone-lb                                      |
| flavor_profile_id | 50dea167-830b-450c-972b-a7360fe98841               |
| enabled           | True                                               |
| description       | A non-high availability load balancer for testing. |
+-------------------+----------------------------------------------------+
~~~

7. Creating private network:
~~~
$ openstack network create private
$ openstack subnet create --network private --subnet-range 192.0.1.0/24 privateSub
~~~

8. Creating non-secure loadbalancer:
~~~
$ openstack loadbalancer create --name lb1 --vip-subnet-id <privateSub_uuid>

Wait until loadbalancer become ONLINE and ACTIVE:
$ openstack loadbalancer show lb1
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| admin_state_up      | True                                 |
| created_at          | 2021-12-23T15:26:12                  |
| description         |                                      |
| flavor_id           | None                                 |
| id                  | 2bf1304e-3346-4148-b019-dea3348b08e7 |
| listeners           | 44b30f90-08e4-41e3-850d-e8e711a5b97b |
| name                | lb1                                  |
| operating_status    | ONLINE                               |
| pools               | 347f6b46-8caa-4f3f-8e02-a739db1e26fb |
| project_id          | 33a7ec8e0b05411abea31a8dac8b9802     |
| provider            | amphora                              |
| provisioning_status | ACTIVE                               |
| updated_at          | 2021-12-24T02:04:08                  |
| vip_address         | 192.0.1.144                          |
| vip_network_id      | 250cd0b4-72f7-44c8-8950-017e388f7916 |
| vip_port_id         | 4fd6c291-6cc5-46f0-87d7-693f5fece64a |
| vip_qos_policy_id   | None                                 |
| vip_subnet_id       | e9c115a1-185c-4507-927b-33127471f77b |
+---------------------+--------------------------------------+

$ openstack loadbalancer amphora list
+--------------------------------------+--------------------------------------+-----------+--------+---------------+-------------+
| id                                   | loadbalancer_id                      | status    | role   | lb_network_ip | ha_ip       |
+--------------------------------------+--------------------------------------+-----------+--------+---------------+-------------+
| f74df421-b342-42ca-8f24-079af66a64df | 2bf1304e-3346-4148-b019-dea3348b08e7 | ALLOCATED | BACKUP | 172.24.0.167  | 192.0.1.144 |
| a7bebaa4-bc72-4708-a285-8e8b00bb4897 | 2bf1304e-3346-4148-b019-dea3348b08e7 | ALLOCATED | MASTER | 172.24.1.56   | 192.0.1.144 |
+--------------------------------------+--------------------------------------+-----------+--------+---------------+-------------+
~~~

9. Create a listener (listener1) on a port (80) and verify the state of the listener.
~~~
$ openstack loadbalancer listener create --name listener1 --protocol HTTP --protocol-port 80 lb1
$ openstack loadbalancer listener show listener1
+-----------------------------+--------------------------------------+
| Field                       | Value                                |
+-----------------------------+--------------------------------------+
| admin_state_up              | True                                 |
| connection_limit            | -1                                   |
| created_at                  | 2021-12-23T15:44:22                  |
| default_pool_id             | 347f6b46-8caa-4f3f-8e02-a739db1e26fb |
| default_tls_container_ref   | None                                 |
| description                 |                                      |
| id                          | 44b30f90-08e4-41e3-850d-e8e711a5b97b |
| insert_headers              | None                                 |
| l7policies                  |                                      |
| loadbalancers               | 2bf1304e-3346-4148-b019-dea3348b08e7 |
| name                        | listener1                            |
| operating_status            | ONLINE                               |
| project_id                  | 33a7ec8e0b05411abea31a8dac8b9802     |
| protocol                    | HTTP                                 |
| protocol_port               | 80                                   |
| provisioning_status         | ACTIVE                               |
| sni_container_refs          | []                                   |
| timeout_client_data         | 50000                                |
| timeout_member_connect      | 5000                                 |
| timeout_member_data         | 50000                                |
| timeout_tcp_inspect         | 0                                    |
| updated_at                  | 2021-12-24T01:58:06                  |
| client_ca_tls_container_ref | None                                 |
| client_authentication       | NONE                                 |
| client_crl_container_ref    | None                                 |
| allowed_cidrs               | None                                 |
+-----------------------------+--------------------------------------+
~~~

10. Create the listener default pool (pool1).
~~~
$ openstack loadbalancer pool create --name pool1 --lb-algorithm ROUND_ROBIN --listener listener1 --protocol HTTP
~~~

11. Create a health monitor on the pool (pool1) that connects to the back end servers and tests the path (/).
~~~
$ openstack loadbalancer healthmonitor create --delay 5 --max-retries 4 --timeout 10 --type HTTP --url-path / pool1
~~~

12. Add load balancer members on the private subnet (privateSub) to the default pool.
~~~
$ openstack loadbalancer member create --subnet-id privateSub --address 192.0.1.10 --protocol-port 80 pool1
$ openstack loadbalancer member create --subnet-id privateSub --address 192.0.1.11 --protocol-port 80 pool1
~~~

