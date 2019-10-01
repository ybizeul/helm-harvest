## Introduction

This chart deploys a full Graphite/Grafana/Harvest stack on a kubernetes cluster

## Requirement

To use this chart, you must have NetApp Harvest available
as a container in a private registry.

### Running Harvest in Docker

To run NetApp Harvest in Docker, you must build your own image that includes NetApp Manageability SDK files in the `lib/` directory.

Once `NaServer.pm` and other files are in the `lib/` directory, you can build and push the container to your registry.

#### Example

```bash
$ cp Dockerfile /path/to/netapp-harvest
$ cd /path/to/netapp-harvest

# Check NMSDK files are in lib/
$ ls lib
DfmErrno.pm		NaErrno.pm		OCUMAPI.pm		ONTAPILogParser.pm	Ontap7ModeAPI.pm	README.txt		Test.pm
NaElement.pm		NaServer.pm		OCUMClassicAPI.pm	ONTAPITestContainer.pm	OntapClusterAPI.pm	SdkEnv.pm

# Build the image
$ docker build .
Sending build context to Docker daemon  21.47MB
Step 1/8 : FROM alpine
[...]
Successfully built 7febc8cb6a26

# Untag latest (if necessary) and tag the new build
$ docker rmi netapp-harvest:latest
$ docker tag 7febc8cb6a26 netapp-harvest:1.4.2
$ docker tag 7febc8cb6a26 netapp-harvest:latest

# Push to a local registry
docker tag netapp-harvest:1.4.2 myregistry.local.lan/netapp-harvest:1.4.2
docker push myregistry.local.lan/netapp-harvest:1.4.2
```

## Usage

Sample `values.yaml` file :
```yaml
# Default values for netapp-harvest.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

grafana:
  enabled: true

graphite:
  enabled: true

harvest:
  - name: cluster_name
    hostname: my.cdot.system
    group: Group
    username: admin
    password: myp@ssWd
  - name: cluster2_name
    hostname: my.cdot2.system
    group: Group
    username: admin
    password: myp@ssWd

nameOverride: ""
fullnameOverride: ""

resources: {}
  # We usually recommend not to specify default resources and to leave this as a conscious
  # choice for the user. This also increases chances charts run on environments with little
  # resources, such as Minikube. If you do want to specify resources, uncomment the following
  # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
  # limits:
  #  cpu: 100m
  #  memory: 128Mi
  # requests:
  #  cpu: 100m
  #  memory: 128Mi

nodeSelector: {}

tolerations: []

affinity: {}
```

    $ helm install -f values.yaml netapp-harvest
    NAME:   riotous-hydra
    LAST DEPLOYED: Mon Sep 30 22:37:03 2019
    NAMESPACE: test-harvest
    STATUS: DEPLOYED

    RESOURCES:
    ==> v1beta1/RoleBinding
    NAME                               AGE
    riotous-hydra-add-secrets-default  <invalid>

    ==> v1/Service
    NAME                        TYPE          CLUSTER-IP     EXTERNAL-IP    PORT(S)       AGE
    riotous-hydra-grafana       LoadBalancer  10.100.38.92   10.65.176.152  80:30416/TCP  <invalid>
    riotous-hydra-graphite-web  ClusterIP     10.98.245.108  <none>         80/TCP        <invalid>
    riotous-hydra-graphite-tcp  ClusterIP     10.108.88.232  <none>         2003/TCP      <invalid>

    ==> v1beta1/Deployment
    NAME                            DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
    riotous-hydra-grafana           1        1        1           0          <invalid>
    riotous-hydra-harvest-cluster1  1        1        1           0          <invalid>
    riotous-hydra-harvest-cluster2  1        1        1           0          <invalid>

    ==> v1/StatefulSet
    NAME                    DESIRED  CURRENT  AGE
    riotous-hydra-graphite  1        1        <invalid>

    ==> v1/Pod(related)
    NAME                                             READY  STATUS             RESTARTS  AGE
    riotous-hydra-grafana-76bb89c9fb-rpwlb           0/1    ContainerCreating  0         <invalid>
    riotous-hydra-harvest-cluster1-7b489b764b-pz5qs  0/1    Init:0/1           0         <invalid>
    riotous-hydra-harvest-cluster2-6677987dd5-xgzhn  0/1    Init:0/1           0         <invalid>
    riotous-hydra-graphite-0                         0/1    Pending            0         <invalid>

    ==> v1/ConfigMap
    NAME                          DATA  AGE
    riotous-hydra-grafana-config  1     <invalid>

    ==> v1beta1/Role
    NAME                       AGE
    riotous-hydra-add-secrets  <invalid>

You can connect to grafana EXTERNAL-IP with username `admin`, password `Netapp01`