# Vela Server
## example

In this Demo, Application application-sample will be converted to appconfig and component

The fields in the application spec come from the parametes defined in the definition template
, so we must install Definition at first

Step 1: Install Workload Definition & Trait Definition
```
kubectl apply -f template.yaml
```
Step 2: Create a sample application in the cluster
```
kubectl apply -f application-sample.yaml
```
Step 3: View the application status
```
kubectl get -f application-sample.yaml -oyaml

// You can see the following
apiVersion: core.oam.dev/v1alpha2
kind: Application
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"core.oam.dev/v1alpha2","kind":"Application","metadata":{"annotations":{},"name":"application-sample","namespace":"oam-test"},"spec":{"template":"services:\n  myweb:\n    type: worker\n    image: \"busybox\"\n    cmd:\n    - sleep\n    - \"1000\"\n    scaler:\n      replicas: 10"}}
  name: application-sample
  namespace: oam-test
spec:
  services:
    myweb:
      cmd:
      - sleep
      - "1000"
      image: busybox
      scaler:
        replicas: 10
      type: worker
status:
  conditions:
  - lastTransitionTime: "2020-12-02T12:12:52Z"
    reason: Available
    status: "True"
    type: Parsed
  - lastTransitionTime: "2020-12-02T12:12:52Z"
    reason: Available
    status: "True"
    type: Built
  - lastTransitionTime: "2020-12-02T12:12:52Z"
    reason: Available
    status: "True"
    type: Applied
  status: running

```

Step 4: View the oam CR generated by application

```
kubectl get appconfig/application-sample -oyaml

// appconfig is as follows
apiVersion: core.oam.dev/v1alpha2
kind: ApplicationConfiguration
metadata:
  labels:
    application.oam.dev: application-sample
  name: application-sample
  namespace: oam-test
  ownerReferences:
  - apiVersion: core.oam.dev/v1alpha2
    controller: true
    kind: Application
    name: application-sample
    uid: dca7acc3-664c-422b-aa52-4fe012e37974
spec:
  components:
  - componentName: myweb
    traits:
    - trait:
        apiVersion: core.oam.dev/v1alpha2
        kind: ManualScalerTrait
        spec:
          replicaCount: 10
status:
  conditions:
  - lastTransitionTime: "2020-12-02T12:12:52Z"
    reason: Successfully reconciled resource
    status: "True"
    type: Synced
  dependency: {}
  workloads:
  - componentName: myweb
    componentRevisionName: myweb-v1
    traits:
    - traitRef:
        apiVersion: core.oam.dev/v1alpha2
        kind: ManualScalerTrait
        name: myweb-trait-78fdd467d6
    workloadRef:
      apiVersion: apps/v1
      kind: Deployment
      name: myweb

kubectl get component/myweb -oyaml

// component is as follows
apiVersion: core.oam.dev/v1alpha2
kind: Component
metadata:
  labels:
    application.oam.dev: application-sample
  name: myweb
  namespace: oam-test
  ownerReferences:
  - apiVersion: core.oam.dev/v1alpha2
    controller: true
    kind: Application
    name: application-sample
    uid: dca7acc3-664c-422b-aa52-4fe012e37974
spec:
  workload:
    apiVersion: apps/v1
    kind: Deployment
    spec:
      selector:
        matchLabels:
          app.oam.dev/component: myweb
      template:
        metadata:
          labels:
            app.oam.dev/component: myweb
        spec:
          containers:
          - command:
            - sleep
            - "1000"
            image: busybox
            name: myweb
status:
  latestRevision:
    name: myweb-v1
    revision: 1      
```
