## How to setup Redfish with libvirt - Sushy tools 

#### Environment 
- Centos8 / RHEL8

#### Steps 

1. Install required package: 
```
# yum install -y python3 python3-libvirt
# pip3 install sushy-tools 
```

2. Create sushy service file:
```
# vim /usr/lib/systemd/system/sushy.service
[Unit]
Description=Sushy Libvirt emulator
After=syslog.target

[Service]
Type=simple
ExecStart=/usr/local/bin/sushy-emulator --config /etc/sushy.conf
StandardOutput=syslog
StandardError=syslog
```

3. Creat sushy service configuration file:
```
# vim /etc/sushy
SUSHY_EMULATOR_LISTEN_IP = '0.0.0.0'
SUSHY_EMULATOR_LISTEN_PORT = 8000
SUSHY_EMULATOR_SSL_CERT = None
SUSHY_EMULATOR_SSL_KEY = None
SUSHY_EMULATOR_OS_CLOUD = None
SUSHY_EMULATOR_LIBVIRT_URI = u'qemu+ssh://root@${libvirtd_host_ip}/system'
SUSHY_EMULATOR_IGNORE_BOOT_DEVICE = True
SUSHY_EMULATOR_BOOT_LOADER_MAP = {
    u'UEFI': {
        u'x86_64': u'/usr/share/OVMF/OVMF_CODE.secboot.fd',
        u'aarch64': u'/usr/share/AAVMF/AAVMF_CODE.fd'
    },
    u'Legacy': {
        u'x86_64': None,
        u'aarch64': None
    }
}
```

4. Start and enable sushy.service:
```
# systemctl enable --now sushy
```

5. Configure no password login to root@<libvirtd host>: 
```
# ssh-keygen 
# ssh-copy-id root@<libvirtd host>
```
  
6. Now you should be able to see your libvirt domain among the Redfish Systems:
```
# curl http://localhost:8000/redfish/v1/Systems/

{
    "@odata.type": "#ComputerSystemCollection.ComputerSystemCollection",
    "Name": "Computer System Collection",
    "Members@odata.count": 8,
    "Members": [
        
            {
                "@odata.id": "/redfish/v1/Systems/f091dec2-5660-4c71-b941-23134b6e1ea2"
            },
        
            {
                "@odata.id": "/redfish/v1/Systems/cd878746-6142-40a4-8b1a-5f1952fac4be"
            },
        
            {
                "@odata.id": "/redfish/v1/Systems/f3ddde60-1c6e-41fa-a877-3682808dc294"
            },
        
            {
                "@odata.id": "/redfish/v1/Systems/c4d0dc1f-1a40-493a-8193-432aaa0c6ff3"
            },
        
            {
                "@odata.id": "/redfish/v1/Systems/64459a24-d792-4e9c-85f2-7cad43a44d1a"
            },
        
            {
                "@odata.id": "/redfish/v1/Systems/d5f8d009-1da9-4e2e-b293-eab53d742b0b"
            },
        
            {
                "@odata.id": "/redfish/v1/Systems/5faef380-4272-49e0-b827-48484313bb77"
            },
        
            {
                "@odata.id": "/redfish/v1/Systems/63a9868c-134a-4d0c-b0c7-e1b26444f1b4"
            }
        
    ],
    "@odata.context": "/redfish/v1/$metadata#ComputerSystemCollection.ComputerSystemCollection",
    "@odata.id": "/redfish/v1/Systems",
    "@Redfish.Copyright": "Copyright 2014-2016 Distributed Management Task Force, Inc. (DMTF). For the full DMTF copyright policy, see http://www.dmtf.org/about/policies/copyright."
```
