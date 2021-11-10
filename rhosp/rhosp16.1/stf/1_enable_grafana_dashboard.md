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

<img width="1440" alt="Screen Shot 2021-11-10 at 3 41 37 PM" src="https://user-images.githubusercontent.com/30589773/141070618-f37afe58-6c06-49b1-b1ef-6721b9589b1d.png">

12. You should able to see 5 Dashboards in `service-telemetry` folder: 

<img width="1440" alt="Screen Shot 2021-11-10 at 4 08 01 PM" src="https://user-images.githubusercontent.com/30589773/141074522-a67ff634-b993-4f59-a063-0776be5354d7.png">

13. And you should able to see the data in each dashboard: 

 <img width="1440" alt="Screen Shot 2021-11-10 at 4 10 03 PM" src="https://user-images.githubusercontent.com/30589773/141074679-e33e00be-6640-4622-a0cc-37e1eeeeb3ab.png">
