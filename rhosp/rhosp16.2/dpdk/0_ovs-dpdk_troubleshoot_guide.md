## OVS-DPDK troulbeshoot guide 

### Related template files

`nic-config`
~~~
$ vim templates/nic-configs/computedpdksriov.yaml
...
              # Used as a provider network with external DHCP
              - type: ovs_user_bridge
                name: br-dpdk
                use_dhcp: false
                members:
                  - type: ovs_dpdk_port
                    name: dpdk0
                    members:
                      - type: interface
                        name: enp130s0f0
~~~

`network-environment.yaml`
~~~
$ vim templates/network-environment.yaml
...
  NeutronNetworkVLANRanges: 'datacentre:183:186,dpdk:188:189,sriov:190:191'
  NeutronBridgeMappings: "datacentre:br-ex,dpdk:br-dpdk"
  ...
  ComputeOvsDpdkSriovParameters:
    IsolCpusList: 2,18,4,20,6,22,8,24,10,26,12,28,14,30,3,19,5,21,7,23,9,25,11,27,13,29,15,31
    KernelArgs: default_hugepagesz=1GB hugepagesz=1G hugepages=32 iommu=pt intel_iommu=on
      isolcpus=2,18,4,20,6,22,8,24,10,26,12,28,14,30,3,19,5,21,7,23,9,25,11,27,13,29,15,31
    TunedProfileName: "cpu-partitioning"
    NovaComputeCpuDedicatedSet: 4,20,6,22,8,24,10,26,12,28,14,30,5,21,7,23,9,25,11,27,13,29,15,31
    NovaReservedHostMemory: 4096
    OvsDpdkSocketMemory: "1024,4096"
    OvsDpdkMemoryChannels: "4"
    OvsDpdkCoreList: 0,16,1,17
    OvsPmdCoreList: 2,18,3,19
    NovaComputeCpuSharedSet: [0,16,1,17]
~~~

### Check dpdk configuration after deployment

Check dpdk interface
~~~
[root@overcloud-computeovsdpdksriov-0 ~]# cat /var/lib/os-net-config/dpdk_mapping.yaml
- driver: vfio-pci
  mac_address: a0:36:9f:e4:0d:ac
  name: enp130s0f0
  pci_address: 0000:82:00.0
~~~

Check `ovs-vsctl show`
~~~
[root@overcloud-computeovsdpdksriov-0 ~]# ovs-vsctl show
1de4330f-815e-4fd8-be3b-cfbb82d62adc
    Manager "ptcp:6640:127.0.0.1"
        is_connected: true
        ...
     Bridge br-dpdk
        Controller "tcp:127.0.0.1:6633"
            is_connected: true
        fail_mode: secure
        datapath_type: netdev
        Port phy-br-dpdk
            Interface phy-br-dpdk
                type: patch
                options: {peer=int-br-dpdk}
        Port br-dpdk
            Interface br-dpdk
                type: internal
        Port dpdk0
            Interface dpdk0
                type: dpdk
                options: {dpdk-devargs="0000:82:00.0"}
~~~

### Create dpdk network after deployment

Create dpdk provider network
~~~
$ openstack network show dpdk188
...
| id                        | 770d5cef-e8dd-4146-b251-d7b9ef738a1d  
| mtu                       | 1500 
| name                      | dpdk188  
| provider:network_type     | vlan  
| provider:physical_network | dpdk    
| provider:segmentation_id  | 188 
| shared                    | True 
| subnets                   | 087260e2-1c2c-4ffd-b9a6-888b71bafdd2    
~~~

Create subnet for dpdk188
~~~
| allocation_pools  | 10.72.51.162-10.72.51.189   
| cidr              | 10.72.51.160/27 
| dns_nameservers   | 10.72.51.93   
| enable_dhcp       | True     
| gateway_ip        | 10.72.51.190    
~~~
