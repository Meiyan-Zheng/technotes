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
