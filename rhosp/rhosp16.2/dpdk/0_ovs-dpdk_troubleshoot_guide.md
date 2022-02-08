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
