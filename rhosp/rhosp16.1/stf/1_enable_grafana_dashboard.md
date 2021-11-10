# How to enable Grafana Dashboard for STF

## Environment:
- RHOSP 16.1
- RHOCP 4.7

## Steps: 
1. Change to the service-telemetry namespace:
~~~
$ oc project service-telemetry
~~~

2. Deploy the Grafana operator:
~~~
$ oc apply -f - <<EOF
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: grafana-operator
  namespace: service-telemetry
spec:
  channel: alpha
  installPlanApproval: Automatic
  name: grafana-operator
  source: operatorhubio-operators
  sourceNamespace: openshift-marketplace
EOF
~~~

3. Verify that the Operator launched successfully. If the  Operator launched successfully, the value of column `PHASE` should be Succeeded:
~~~
[root@bastion stf-working-dir]# oc get csv --selector operators.coreos.com/grafana-operator.service-telemetry
NAME                       DISPLAY            VERSION   REPLACES                   PHASE
grafana-operator.v3.10.3   Grafana Operator   3.10.3    grafana-operator.v3.10.2   Succeeded
~~~

4. To launch a Grafana instance, create or modify the ServiceTelemetry object. Set graphing.enabled and graphing.grafana.ingressEnabled to true:
~~~
[root@bastion stf-working-dir]# oc edit stf default
...
spec:
  backends:
    events:
      elasticsearch:
        enabled: true
  graphing:
    enabled: true
    grafana:
      ingressEnabled: true
~~~

5. Verify that the Grafana instance deployed:
~~~
[root@bastion stf-working-dir]# oc get pod -l app=grafana
NAME                                  READY   STATUS    RESTARTS   AGE
grafana-deployment-69c7849d64-n9bmw   1/1     Running   0          13d
~~~

6. Verify that the Grafana data sources installed correctly:
~~~
[root@bastion stf-working-dir]# oc get grafanadatasources
NAME                  AGE
default-datasources   13d
~~~

7. Verify that the Grafana route exists:
~~~
[root@bastion stf-working-dir]# oc get route grafana-route
NAME            HOST/PORT                                               PATH   SERVICES          PORT   TERMINATION   WILDCARD
grafana-route   grafana-route-service-telemetry.apps.ocp4.example.com          grafana-service   3000   edge          None
~~~

8. Importing dashboards
~~~
infrastructure dashboard:
$ oc apply -f https://raw.githubusercontent.com/infrawatch/dashboards/master/deploy/stf-1.3/rhos-dashboard.yaml

cloud dashboard:
$ oc apply -f https://raw.githubusercontent.com/infrawatch/dashboards/master/deploy/stf-1.3/rhos-cloud-dashboard.yaml

cloud events dashboard:
$ oc apply -f https://raw.githubusercontent.com/infrawatch/dashboards/master/deploy/stf-1.3/rhos-cloudevents-dashboard.yaml

virtual machine dashboard:
$ oc apply -f https://raw.githubusercontent.com/infrawatch/dashboards/master/deploy/stf-1.3/virtual-machine-view.yaml

memcached dashboard:
$ oc apply -f https://raw.githubusercontent.com/infrawatch/dashboards/master/deploy/stf-1.3/memcached-dashboard.yaml
~~~

9. Verify that the dashboards are available:
~~~
[root@bastion stf-working-dir]# oc get grafanadashboards
NAME                         AGE
memcached-dashboard-1.3      4s
rhos-cloud-dashboard-1.3     13d
rhos-cloudevents-dashboard   13d
rhos-dashboard-1.3           13d
virtual-machine-view-1.3     102s
~~~

10. Retrieve the Grafana route address:
~~~
[root@bastion stf-working-dir]# oc get route grafana-route -ojsonpath='{.spec.host}'
grafana-route-service-telemetry.apps.ocp4.example.com
~~~


11. In a web browser, navigate to https://grafana-route-service-telemetry.apps.ocp4.example.com
Sign in with **root/secret**:

