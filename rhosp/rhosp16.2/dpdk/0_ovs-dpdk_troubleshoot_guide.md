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

Show dpdk interface details
~~~
[root@overcloud-computeovsdpdksriov-0 ~]# ovs-vsctl list interface dpdk0 | egrep "name|mtu|options|status"
bfd_status          : {}
cfm_fault_status    : []
mtu                 : 1500
mtu_request         : []
name                : dpdk0
options             : {dpdk-devargs="0000:82:00.0"}
statistics          : {flow_director_filter_add_errors=0, flow_director_filter_remove_errors=0, mac_local_errors=4, mac_remote_errors=0, ovs_rx_qos_drops=0, ovs_tx_failure_drops=0, ovs_tx_invalid_hwol_drops=0, ovs_tx_mtu_exceeded_drops=0, ovs_tx_qos_drops=0, rx_128_to_255_packets=82, rx_1_to_64_packets=280, rx_256_to_511_packets=440, rx_512_to_1023_packets=1, rx_65_to_127_packets=72377, rx_broadcast_packets=25024, rx_bytes=5202426, rx_crc_errors=0, rx_dropped=0, rx_errors=0, rx_fcoe_crc_errors=0, rx_fcoe_dropped=0, rx_fcoe_mbuf_allocation_errors=0, rx_fragment_errors=0, rx_illegal_byte_errors=0, rx_jabber_errors=0, rx_length_errors=0, rx_mac_short_packet_dropped=0, rx_management_dropped=0, rx_management_packets=0, rx_mbuf_allocation_errors=0, rx_missed_errors=0, rx_oversize_errors=0, rx_packets=73181, rx_priority0_dropped=0, rx_priority0_mbuf_allocation_errors=0, rx_priority1_dropped=0, rx_priority1_mbuf_allocation_errors=0, rx_priority2_dropped=0, rx_priority2_mbuf_allocation_errors=0, rx_priority3_dropped=0, rx_priority3_mbuf_allocation_errors=0, rx_priority4_dropped=0, rx_priority4_mbuf_allocation_errors=0, rx_priority5_dropped=0, rx_priority5_mbuf_allocation_errors=0, rx_priority6_dropped=0, rx_priority6_mbuf_allocation_errors=0, rx_priority7_dropped=0, rx_priority7_mbuf_allocation_errors=0, rx_q0_errors=0, rx_undersize_errors=0, tx_128_to_255_packets=2, tx_1_to_64_packets=32, tx_256_to_511_packets=0, tx_512_to_1023_packets=1, tx_65_to_127_packets=106, tx_broadcast_packets=1, tx_bytes=12765, tx_dropped=0, tx_errors=0, tx_management_packets=0, tx_multicast_packets=12, tx_packets=142}
status              : {driver_name=net_ixgbe, if_descr="DPDK 20.11.1 net_ixgbe", if_type="6", link_speed="1Gbps", max_hash_mac_addrs="4096", max_mac_addrs="127", max_rx_pktlen="1518", max_rx_queues="128", max_tx_queues="64", max_vfs="0", max_vmdq_pools="64", min_rx_bufsize="1024", numa_id="1", pci-device_id="0x154d", pci-vendor_id="0x8086", port_no="0"}
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

### Check instance configuration

Check interface used by instance:
~~~
[root@overcloud-computeovsdpdksriov-0 ~]# podman exec nova_libvirt virsh dumpxml 7
...
  <memory unit='KiB'>8388608</memory>
  <currentMemory unit='KiB'>8388608</currentMemory>
  <memoryBacking>
    <hugepages>
      <page size='1048576' unit='KiB' nodeset='0'/>
    </hugepages>
  </memoryBacking>
  <vcpu placement='static'>4</vcpu>
  <cputune>
    <shares>4096</shares>
    <vcpupin vcpu='0' cpuset='8'/>
    <vcpupin vcpu='1' cpuset='24'/>
    <vcpupin vcpu='2' cpuset='14'/>
    <vcpupin vcpu='3' cpuset='30'/>
    <emulatorpin cpuset='12'/>
  </cputune>
  <numatune>
    <memory mode='strict' nodeset='0'/>
    <memnode cellid='0' mode='strict' nodeset='0'/>
  </numatune>
  ...
    <interface type='vhostuser'>
      <mac address='fa:16:3e:57:6e:10'/>
      <source type='unix' path='/var/lib/vhost_sockets/vhuf081e75a-e9' mode='server'/>
      <target dev='vhuf081e75a-e9
'/>
      <model type='virtio'/>
      <driver rx_queue_size='1024' tx_queue_size='1024'/>
      <alias name='net0'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x0'/>
    </interface>
~~~

Confirm that the receive queues for the instance are being serviced by a poll mode driver (PMD).
~~~
[root@overcloud-computeovsdpdksriov-0 ~]# ovs-appctl dpif-netdev/pmd-rxq-show
pmd thread numa_id 0 core_id 2:
  isolated : false
  port: vhuf081e75a-e9    queue-id:  0 (enabled)   pmd usage:  0 %
pmd thread numa_id 1 core_id 3:
  isolated : false
  port: dpdk0             queue-id:  0 (enabled)   pmd usage:  0 %
pmd thread numa_id 0 core_id 18:
  isolated : false
pmd thread numa_id 1 core_id 19:
  isolated : false
~~~

Show statistics for the PMDs:
~~~
[root@overcloud-computeovsdpdksriov-0 ~]# ovs-appctl dpif-netdev/pmd-stats-show
pmd thread numa_id 0 core_id 2:
  packets received: 206
  packet recirculations: 160
  avg. datapath passes per packet: 1.78
  emc hits: 0
  smc hits: 0
  megaflow hits: 146
  avg. subtable lookups per megaflow hit: 2.27
  miss with success upcall: 219
  miss with failed upcall: 1
  avg. packets per output batch: 1.00
  idle cycles: 18526538222838 (100.00%)
  processing cycles: 20806284 (0.00%)
  avg cycles per packet: 89934752568.55 (18526559029122/206)
  avg processing cycles per packet: 101001.38 (20806284/206)
pmd thread numa_id 1 core_id 3:
  packets received: 77212
  packet recirculations: 64
  avg. datapath passes per packet: 1.00
  emc hits: 34317
  smc hits: 0
  megaflow hits: 33676
  avg. subtable lookups per megaflow hit: 1.06
  miss with success upcall: 9283
  miss with failed upcall: 0
  avg. packets per output batch: 1.00
  idle cycles: 18521597218095 (100.00%)
  processing cycles: 923502096 (0.00%)
  avg cycles per packet: 239891736.00 (18522520720191/77212)
  avg processing cycles per packet: 11960.60 (923502096/77212)
pmd thread numa_id 0 core_id 18:
  packets received: 0
  packet recirculations: 0
  avg. datapath passes per packet: 0.00
  emc hits: 0
  smc hits: 0
  megaflow hits: 0
  avg. subtable lookups per megaflow hit: 0.00
  miss with success upcall: 0
  miss with failed upcall: 0
  avg. packets per output batch: 0.00
pmd thread numa_id 1 core_id 19:
  packets received: 0
  packet recirculations: 0
  avg. datapath passes per packet: 0.00
  emc hits: 0
  smc hits: 0
  megaflow hits: 0
  avg. subtable lookups per megaflow hit: 0.00
  miss with success upcall: 0
  miss with failed upcall: 0
  avg. packets per output batch: 0.00
main thread:
  packets received: 20
  packet recirculations: 0
  avg. datapath passes per packet: 1.00
  emc hits: 0
  smc hits: 0
  megaflow hits: 7
  avg. subtable lookups per megaflow hit: 1.00
  miss with success upcall: 13
  miss with failed upcall: 0
  avg. packets per output batch: 1.00
~~~
