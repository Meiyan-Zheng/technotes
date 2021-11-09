# How to deploy STF 

## Environment:
- RHOSP 16.1 
- RHOCP 4.7 


## Steps: 

### On RHOCP side: 
1. Create a namespace to contain the STF components, for example, service-telemetry:
~~~
$ oc new-project service-telemetry
~~~
2. Create an OperatorGroup in the namespace so that you can schedule the Operator pods:
~~~
$ oc create -f - <<EOF
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: service-telemetry-operator-group
  namespace: service-telemetry
spec:
  targetNamespaces:
  - service-telemetry
EOF
~~~
3. Enable the OperatorHub.io Community Catalog Source to install data storage and visualization Operators:
~~~
$ oc create -f - <<EOF
apiVersion: operators.coreos.com/v1alpha1
kind: CatalogSource
metadata:
  name: operatorhubio-operators
  namespace: openshift-marketplace
spec:
  sourceType: grpc
  image: quay.io/operatorhubio/catalog:latest
  displayName: OperatorHub.io Operators
  publisher: OperatorHub.io
EOF
~~~
4. Subscribe to the AMQ Certificate Manager Operator by using the redhat-operators CatalogSource:
~~~
$ oc create -f - <<EOF
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: amq7-cert-manager-operator
  namespace: openshift-operators
spec:
  channel: 1.x
  installPlanApproval: Automatic
  name: amq7-cert-manager-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
EOF
~~~
5. Validate your ClusterServiceVersion. Ensure that amq7-cert-manager.v1.0.1 displays a phase of Succeeded:
~~~
# oc get --namespace openshift-operators csv
NAME                       DISPLAY                                         VERSION   REPLACES   PHASE
amq7-cert-manager.v1.0.1   Red Hat Integration - AMQ Certificate Manager   1.0.1                Succeeded
~~~
6. If you plan to store events in ElasticSearch, you must enable the Elastic Cloud on Kubernetes (ECK) Operator. 
~~~
$ oc create -f - <<EOF
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: elasticsearch-eck-operator-certified
  namespace: service-telemetry
spec:
  channel: stable
  installPlanApproval: Automatic
  name: elasticsearch-eck-operator-certified
  source: certified-operators
  sourceNamespace: openshift-marketplace
EOF
~~~
7. Verify that the ClusterServiceVersion for Elastic Cloud on Kubernetes Succeeded:
~~~
# oc get csv
NAME                                          DISPLAY                                         VERSION          REPLACES                                      PHASE
amq7-cert-manager.v1.0.1                      Red Hat Integration - AMQ Certificate Manager   1.0.1                                                          Succeeded
amq7-interconnect-operator.v1.10.2            Red Hat Integration - AMQ Interconnect          1.10.2                                                         Succeeded
elasticsearch-eck-operator-certified.v1.8.0   Elasticsearch (ECK) Operator                    1.8.0            elasticsearch-eck-operator-certified.v1.7.1   Succeeded
...
~~~
Or use below command to check pod status: 
~~~
# oc get pods
NAME                                                      READY   STATUS    RESTARTS   AGE
...
elastic-operator-7f7566c9cc-85dtc                         1/1     Running   13         17d
elasticsearch-es-default-0                                1/1     Running   0          14d
~~~
8. Create the Service Telemetry Operator subscription to manage the STF instances:
~~~
$ oc create -f - <<EOF
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: service-telemetry-operator
  namespace: service-telemetry
spec:
  channel: stable-1.3
  installPlanApproval: Automatic
  name: service-telemetry-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
EOF
~~~
9. Validate the Service Telemetry Operator and the dependent operators:
~~~
# oc get csv --namespace service-telemetry
NAME                                          DISPLAY                                         VERSION          REPLACES                                      PHASE
amq7-cert-manager.v1.0.1                      Red Hat Integration - AMQ Certificate Manager   1.0.1                                                          Succeeded
amq7-interconnect-operator.v1.10.2            Red Hat Integration - AMQ Interconnect          1.10.2                                                         Succeeded
elasticsearch-eck-operator-certified.v1.8.0   Elasticsearch (ECK) Operator                    1.8.0            elasticsearch-eck-operator-certified.v1.7.1   Succeeded
grafana-operator.v3.10.3                      Grafana Operator                                3.10.3           grafana-operator.v3.10.2                      Succeeded
prometheusoperator.0.47.0                     Prometheus Operator                             0.47.0           prometheusoperator.0.37.0                     Succeeded
service-telemetry-operator.v1.3.1632277596    Service Telemetry Operator                      1.3.1632277596                                                 Succeeded
smart-gateway-operator.v3.0.1632277599        Smart Gateway Operator                          3.0.1632277599                                                 Succeeded
~~~
10. Create a ServiceTelemetry object that results in an STF deployment that uses the default values, create a ServiceTelemetry object with an empty spec parameter:
~~~
$ oc apply -f - <<EOF
apiVersion: infra.watch/v1beta1
kind: ServiceTelemetry
metadata:
  name: default
  namespace: service-telemetry
spec: {}
EOF
~~~
11. View the STF deployment logs in the Service Telemetry Operator:
~~~
# oc logs --selector name=service-telemetry-operator
{"level":"info","ts":1635389831.9003289,"logger":"logging_event_handler","msg":"[playbook task]","name":"default","namespace":"service-telemetry","gvk":"infra.watch/v1beta1, Kind=ServiceTelemetry","event_type":"playbook_on_task_start","job":"4561862115305449611","EventData.Name":"servicetelemetry : Remove unlisted Smart Gateway"}
{"level":"info","ts":1635389832.6810424,"logger":"runner","msg":"Ansible-runner exited successfully","job":"4561862115305449611","name":"default","namespace":"service-telemetry"}

--------------------------- Ansible Task Status Event StdOut  -----------------

PLAY RECAP *********************************************************************
localhost                  : ok=69   changed=0    unreachable=0    failed=0    skipped=13   rescued=0    ignored=0   


-------------------------------------------------------------------------------
~~~
12. To determine that all workloads are operating correctly, view the pods and the status of each pod.
~~~
# oc get pods
NAME                                                      READY   STATUS    RESTARTS   AGE
alertmanager-default-0                                    2/2     Running   0          14d
default-cloud1-ceil-event-smartgateway-6fbb9f4895-8wlqm   2/2     Running   11         14d
default-cloud1-ceil-meter-smartgateway-69c9885b6-czfm8    2/2     Running   15         14d
default-cloud1-coll-event-smartgateway-99c75cc86-q9qzc    2/2     Running   11         14d
default-cloud1-coll-meter-smartgateway-6f46c8b6f6-fz452   2/2     Running   2          14d
default-cloud1-sens-meter-smartgateway-57fc47759b-fkqjs   2/2     Running   14         14d
default-interconnect-668d5bbcd6-htl8b                     1/1     Running   0          14d
elastic-operator-7f7566c9cc-85dtc                         1/1     Running   13         17d
elasticsearch-es-default-0                                1/1     Running   0          14d
grafana-deployment-69c7849d64-n9bmw                       1/1     Running   0          12d
grafana-operator-79d4c5c4d8-zzvlw                         1/1     Running   0          12d
interconnect-operator-67ccf5bfcc-5lwtn                    1/1     Running   0          16d
prometheus-default-0                                      2/2     Running   1          14d
prometheus-operator-6d88cff766-wx8jk                      1/1     Running   0          16d
service-telemetry-operator-67999cbd84-lbjfn               1/1     Running   0          16d
smart-gateway-operator-5b88b86ddf-wkd99                   1/1     Running   0          16d
~~~
13. To update the `ServiceTelemetry` configuration:
~~~
# oc edit ServiceTelemetry
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: infra.watch/v1beta1
kind: ServiceTelemetry
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"infra.watch/v1beta1","kind":"ServiceTelemetry","metadata":{"annotations":{},"name":"default","namespace":"service-telemetry"},"spec":{"backends":{"events":{"elasticsearch":{"enabled":true}}}}}
  creationTimestamp: "2021-10-25T10:59:25Z"
  generation: 3
  name: default
  namespace: service-telemetry
  resourceVersion: "17821375"
  selfLink: /apis/infra.watch/v1beta1/namespaces/service-telemetry/servicetelemetrys/default
  uid: acba1931-b7e5-40e5-a99d-dcfebed98dcd
spec:
  backends:
    events:
      elasticsearch:
        enabled: true #<--- enable elasticsearch
  graphing:
    enabled: true
    grafana:
      ingressEnabled: true #<--- enable grafana 
status:
  conditions:
  - ansibleResult:
      changed: 0
      completion: 2021-10-28T02:57:12.256053
      failures: 0
      ok: 69
      skipped: 13
    lastTransitionTime: "2021-10-25T10:59:25Z"
    message: Awaiting next reconciliation
    reason: Successful
    status: "True"
    type: Running
~~~
14. In the service-telemetry project, retrieve the AMQ Interconnect route address:
~~~
# oc get routes -ogo-template='{{ range .items }}{{printf "%s\n" .spec.host }}{{ end }}' | grep "\-5671"
default-interconnect-5671-service-telemetry.apps.ocp4.example.com
~~~
15. Create a tripleo template file called `enable-stf.yaml`:
~~~
$ cat enable-stf.yaml 
parameter_defaults:
    # only send to STF, not other publishers
    EventPipelinePublishers: []
    PipelinePublishers: []

    # manage the polling and pipeline configuration files for Ceilometer agents
    ManagePolling: true
    ManagePipeline: true

    # enable Ceilometer metrics and events
    CeilometerQdrPublishMetrics: true
    CeilometerQdrPublishEvents: true

    # enable collection of API status
    CollectdEnableSensubility: true
    CollectdSensubilityTransport: amqp1
    CollectdSensubilityResultsChannel: sensubility/telemetry

    # enable collection of containerized service metrics
    CollectdEnableLibpodstats: true

    # set collectd overrides for higher telemetry resolution and extra plugins
    # to load
    CollectdConnectionType: amqp1
    CollectdAmqpInterval: 60
    CollectdDefaultPollingInterval: 60
    CollectdExtraPlugins:
    - vmem
    - aggregation
    - connectivity
    - conntrack
    - df

    # set standard prefixes for where metrics and events are published to QDR
    MetricsQdrAddresses:
    - prefix: 'collectd'
      distribution: multicast
    - prefix: 'anycast/ceilometer'
      distribution: multicast

    ExtraConfig:
        ceilometer::agent::polling::polling_interval: 60
        ceilometer::agent::polling::polling_meters:
        - cpu
        - disk.*
        - ip.*
        - image.*
        - memory
        - memory.*
        - network.*
        - perf.*
        - port
        - port.*
        - switch
        - switch.*
        - storage.*
        - volume.*

        # to avoid filling the memory buffers if disconnected from the message bus
        collectd::plugin::amqp1::send_queue_limit: 50

        # memory
        collectd::plugin::memory::valuesabsolute: true
        collectd::plugin::memory::valuespercentage: true

        # receive extra information about virtual memory
        collectd::plugin::vmem::verbose: true

        # provide name and uuid in addition to hostname for better correlation
        # to ceilometer data
        collectd::plugin::virt::hostname_format: "name uuid hostname"

        # provide the human-friendly name of the virtual instance
        collectd::plugin::virt::plugin_instance_format: metadata

        # set memcached collectd plugin to report its metrics by hostname
        # rather than host IP, ensuring metrics in the dashboard remain uniform
        collectd::plugin::memcached::instances:
          local:
            host: "%{hiera('fqdn_canonical')}"
            port: 11211
~~~
**Note**: 
There are a list of plugins defined in `CollectdDefaultPlugins` , don't specify those default plugins in `CollectdExtraPlugins`: 
~~~
deployment/metrics/collectd-container-puppet.yaml:
...
  CollectdDefaultPlugins:
    default:
      - cpu
      - df
      - disk
      - hugepages
      - interface
      - load
      - memory
      - processes
      - unixsock
      - uptime
~~~

16. Create a tripleo template file called `stf-connectors.yaml`:
~~~
parameter_defaults:
  CeilometerQdrPublishEvents: true
  MetricsQdrConnectors:
  - host: default-interconnect-5671-service-telemetry.apps.ocp4.example.com
    port: 443
    role: edge
    sslProfile: sslProfile
    verifyHostname: false

  MetricsQdrSSLProfiles:
  - name: sslProfile
~~~

17. Including `stf-connectors.yaml` and `enable-stf.yaml` when deploy overcloud: 
~~~
openstack overcloud deploy --override-ansible-cfg /home/stack/ansible.cfg \
 --templates \
 -r /home/stack/ovs-sriov/templates/roles_data.yaml \
 -n /home/stack/ovs-sriov/templates/network_data.yaml \
 ...
 -e /usr/share/openstack-tripleo-heat-templates/environments/metrics/ceilometer-write-qdr.yaml  \
 -e /usr/share/openstack-tripleo-heat-templates/environments/metrics/collectd-write-qdr.yaml \
 -e /usr/share/openstack-tripleo-heat-templates/environments/metrics/qdr-edge-only.yaml \
 ...
 -e $CUSTOM/templates/enable-stf.yaml \
 -e $CUSTOM/templates/stf-connectors.yaml \
 --ntp-server 192.168.24.1 --libvirt-type kvm  --timeout 240 \
 --log-file overcloud-deployment.log
~~~

18. Validating installation on RHOSP side: 
18.1 Ensure that the `metrics_qdr` container is running on the node:
~~~
[root@overcloud-controller-0 ~]# sudo podman container inspect --format '{{.State.Status}}' metrics_qdr
running
~~~

18.2 Return the internal network address on which AMQ Interconnect is running:
~~~
[root@overcloud-controller-0 ~]# sudo podman exec -it metrics_qdr cat /etc/qpid-dispatch/qdrouterd.conf

listener {
    host: 10.72.51.227
    port: 5666
    authenticatePeer: no
    saslMechanisms: ANONYMOUS
}
~~~

18.3 Return a list of connections to the local AMQ Interconnect:
~~~
[root@overcloud-controller-0 ~]# sudo podman exec -it metrics_qdr qdstat --bus=10.72.51.227:5666 --connections
Connections
  id    host                                                                   container                                                                                                            role    dir  security                            authentication  tenant
  ===========================================================================================================================================================================================================================================================================
  17    10.72.51.227:46062                                                     openstack.org/om/container/overcloud-controller-0/ceilometer-agent-notification/26/c0821a4f886a4c71956bdda6f6278b5c  normal  in   no-security                         no-auth         
  21    10.72.51.227:60578                                                     metrics                                                                                                              normal  in   no-security                         anonymous-user  
  23    10.72.51.227:60610                                                     overcloud-controller-0.internalapi.localdomain-infrawatch-out-1635763800                                             normal  in   no-security                         no-auth         
  22    10.72.51.227:60608                                                     overcloud-controller-0.internalapi.localdomain-infrawatch-in-1635763800                                              normal  in   no-security                         anonymous-user  
  2980  default-interconnect-5671-service-telemetry.apps.ocp4.example.com:443  default-interconnect-668d5bbcd6-htl8b                                                                                edge    out  TLSv1.2(DHE-RSA-AES256-GCM-SHA384)  anonymous-user  
  8179  10.72.51.227:57148                                                     314e715f-520e-4b3d-858d-87787ec767bd                                                                                 normal  in   no-security                         no-auth         
~~~

18.4 To ensure that messages are delivered, list the links, and view the _edge address in the deliv column for delivery of messages:
~~~
[root@overcloud-controller-0 ~]# sudo podman exec -it metrics_qdr qdstat --bus=10.72.51.227:5666 --links
Router Links
  type      dir  conn id  id     peer  class   addr                                phs  cap  pri  undel  unsett  deliv    presett  psdrop  acc      rej  rel  mod  delay  rate
  ==============================================================================================================================================================================
  endpoint  out  17       43           local   temp.SzZ54FAjNPFu3l1                     250  0    0      0       0        0        0       0        0    0    0    0      0
  endpoint  in   17       48           mobile  anycast/ceilometer/metering.sample  0    250  0    0      0       28401    0        0       28401    0    1    0    0      0
  endpoint  in   21       54                                                            250  0    0      0       3813407  0        0       3813407  0    0    0    0      0
  endpoint  in   23       55           mobile  sensubility/telemetry               0    250  0    0      0       1        0        0       1        0    0    0    0      0
  endpoint  in   23       57           mobile  sensubility/telemetry               0    250  0    0      0       1        0        0       1        0    0    0    0      0
  endpoint  in   23       60           mobile  sensubility/telemetry               0    250  0    0      0       1        0        0       1        0    0    0    0      0
  endpoint  in   23       61           mobile  sensubility/telemetry               0    250  0    0      0       1        0        0       1        0    0    0    0      0
...

19. Validating installation on RHOCP side
19.1 List the available AMQ Interconnect pods:
~~~
[root@bastion stf-working-dir]# oc get pods -l application=default-interconnect
NAME                                    READY   STATUS    RESTARTS   AGE
default-interconnect-668d5bbcd6-htl8b   1/1     Running   0          14d
~~~

19.2 Connect to the pod and list the known connections. We can see rhosp nodes connected with id 26,30,31,32,33
~~~
[root@bastion stf-working-dir]# oc exec -it default-interconnect-668d5bbcd6-htl8b -- qdstat --connections
2021-11-09 09:31:48.183429 UTC
default-interconnect-668d5bbcd6-htl8b

Connections
  id  host                container                                    role    dir  security                                authentication  tenant  last dlv      uptime
  ==============================================================================================================================================================================
  1   10.129.1.71:57908   bridge-32b                                   edge    in   no-security                             anonymous-user          000:00:00:00  014:20:05:42
  5   10.128.0.40:48750   bridge-3c4                                   edge    in   no-security                             anonymous-user          000:00:00:13  014:19:02:54
  6   10.129.1.70:33188   bridge-189                                   edge    in   no-security                             anonymous-user          000:00:00:07  014:09:12:29
  7   10.129.1.183:55292  bridge-e2                                    edge    in   no-security                             anonymous-user          001:07:32:14  014:00:25:01
  12  10.129.1.184:38414  bridge-95                                    edge    in   no-security                             anonymous-user          000:00:30:30  012:23:43:35
  26  10.130.0.1:52800    Router.overcloud-controller-0.localdomain    edge    in   TLSv1/SSLv3(DHE-RSA-AES256-GCM-SHA384)  anonymous-user          000:00:00:07  005:00:53:21
  30  10.130.0.1:38646    Router.overcloud-controller-1.localdomain    edge    in   TLSv1/SSLv3(DHE-RSA-AES256-GCM-SHA384)  anonymous-user          000:00:00:00  002:01:55:24
  31  10.129.0.1:44794    Router.overcloud-computesriov-0.localdomain  edge    in   TLSv1/SSLv3(DHE-RSA-AES256-GCM-SHA384)  anonymous-user          000:00:00:02  000:22:33:38
  32  10.129.0.1:44862    Router.overcloud-controller-2.localdomain    edge    in   TLSv1/SSLv3(DHE-RSA-AES256-GCM-SHA384)  anonymous-user          000:00:00:00  000:22:33:37
  33  10.129.0.1:44896    Router.overcloud-computesriov-1.localdomain  edge    in   TLSv1/SSLv3(DHE-RSA-AES256-GCM-SHA384)  anonymous-user          000:00:00:00  000:22:33:36
  34  127.0.0.1:59070     0575aaf2-f97a-4c7a-b8fe-9f24aeac0cdd         normal  in   no-security                             no-auth                 000:00:00:00  000:00:00:00
~~~

19.2 To view the number of messages delivered by the network, use each address with the oc exec command:
~~~
[root@bastion stf-working-dir]# oc exec -it default-interconnect-668d5bbcd6-htl8b -- qdstat --address
2021-11-09 09:33:52.496591 UTC
default-interconnect-668d5bbcd6-htl8b

Router Addresses
  class   addr                                         phs  distrib    pri  local  remote  in          out         thru  fallback
  =================================================================================================================================
  local   $_management_internal                             closest    -    0      0       0           0           0     0
  mobile  $management                                  0    closest    -    0      0       11          0           0     0
  local   $management                                       closest    -    0      0       0           0           0     0
  edge    Router.overcloud-computesriov-0.localdomain       balanced   -    1      0       0           0           0     0
  edge    Router.overcloud-computesriov-1.localdomain       balanced   -    1      0       0           0           0     0
  edge    Router.overcloud-controller-0.localdomain         balanced   -    1      0       0           0           0     0
  edge    Router.overcloud-controller-1.localdomain         balanced   -    1      0       0           0           0     0
  edge    Router.overcloud-controller-2.localdomain         balanced   -    1      0       0           0           0     0
  mobile  _$qd.addr_lookup                             0    balanced   -    0      0       0           0           0     0
  mobile  _$qd.edge_addr_tracking                      0    balanced   -    0      0       0           0           0     0
  mobile  anycast/ceilometer/event.sample              0    balanced   -    1      0       5,427       5,427       0     0
  mobile  anycast/ceilometer/metering.sample           0    balanced   -    1      0       192,339     192,339     0     0
  mobile  collectd/notify                              0    multicast  -    1      0       70          70          0     0
  mobile  collectd/telemetry                           0    multicast  -    1      0       31,561,302  31,561,302  0     0
  local   qdhello                                           flood      -    0      0       0           0           0     0
  local   qdrouter                                          flood      -    0      0       0           0           0     0
  topo    qdrouter                                          flood      -    0      0       0           0           0     0
  local   qdrouter.ma                                       multicast  -    0      0       0           0           0     0
  topo    qdrouter.ma                                       multicast  -    0      0       0           0           0     0
  mobile  sensubility/telemetry                        0    balanced   -    1      0       512,844     512,844     0     0
  local   temp.3zinYItNtTSpxlP                              balanced   -    1      0       0           0           0     0
  local   temp.5kjOe33bCYJYj2c                              balanced   -    1      0       0           0           0     0
  local   temp.JmzMVo5GrqBU1Um                              balanced   -    1      0       0           0           0     0
  local   temp.NCUyxkorxN5wU4q                              balanced   -    1      0       0           1           0     0
  local   temp.Ovvd_52XI4MnZb6                              balanced   -    1      0       0           9,141       0     0
  local   temp.ShUXUCkgaL3nzZJ                              balanced   -    1      0       0           8,946       0     0
  local   temp.U8loqCegiD8iP_+                              balanced   -    1      0       0           48,923      0     0
  local   temp.mQpXy_50fbNG28E                              balanced   -    1      0       0           0           0     0
  local   temp.rdtR5MsAUNTh21H                              balanced   -    1      0       0           8,946       0     0
  local   temp.s2Ml4CGizZj10v0                              balanced   -    1      0       0           20,221      0     0
  local   temp.tzYJFl5c62AqmkJ                              balanced   -    1      0       0           0           0     0
~~~
