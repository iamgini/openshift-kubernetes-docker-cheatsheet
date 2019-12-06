# 1. OpenShift-Kubernetes-Docker-Cheatsheet

Comprehensive CLI Cheatsheet for OpenShift, Kubernetes and Docker. 

Most of the time `oc` and `kubectl` shares the same command set but some cases we have some differences. 
- `oc` has support for logging to OpenShift cluster 
- with `kubectl` you need to create your kubeconfig file with credentials.

[iamgini.com](https://www.iamgini.com/) \| [LinkedIn](http://bit.ly/gineesh) \| [www.techbeatly.com](https://www.techbeatly.com)

<!-- TOC depthfrom:2 orderedlist:false -->

- [OpenShift CLI - Installation](#openshift-cli---installation)
- [Get Help](#get-help)
- [Build from image](#build-from-image)
- [Enable/Disable scheduling](#enabledisable-scheduling)
- [Resource quotas](#resource-quotas)
- [Labels & Annotations](#labels--annotations)
- [Limit ranges](#limit-ranges)
- [ClusterQuota or ClusterResourceQuota](#clusterquota-or-clusterresourcequota)
- [Config View](#config-view)
- [Managing Environment Variables](#managing-environment-variables)
- [Security Context Constraints](#security-context-constraints)
- [Services & Routes](#services--routes)
- [Scaling & AutoScaling of the pod - HorizontalPodAutoscaler](#scaling--autoscaling-of-the-pod---horizontalpodautoscaler)
- [Configuration Maps (ConfigMap)](#configuration-maps-configmap)
- [Creation of objects](#creation-of-objects)
- [Reading config maps](#reading-config-maps)
- [Dynamically change the config map](#dynamically-change-the-config-map)
- [Mounting config map as ENV](#mounting-config-map-as-env)
- [The Replication Controller](#the-replication-controller)
- [Create persistent volume](#create-persistent-volume)
- [Create volume claim](#create-volume-claim)
- [Deployments](#deployments)
- [Deployment strategies](#deployment-strategies)
- [Rolling](#rolling)
- [Triggers](#triggers)
- [Recreate](#recreate)
- [Custom](#custom)
- [Lifecycle hooks](#lifecycle-hooks)
- [Deployment Pod Resources](#deployment-pod-resources)
- [Blue-Green deployments](#blue-green-deployments)
- [A/B Deployments](#ab-deployments)
- [Canary Deployments](#canary-deployments)
- [Rollbacks](#rollbacks)
- [Pipelines](#pipelines)
- [Configuration Management](#configuration-management)
- [Secrets](#secrets)
- [Creation](#creation)
- [Using secrets in Pods](#using-secrets-in-pods)
- [ENV](#env)
- [Adding](#adding)
- [Removing](#removing)
- [Change triggers](#change-triggers)
- [OpenShift Builds](#openshift-builds)
  - [Build strategies](#build-strategies)
  - [Build sources](#build-sources)
  - [Build Configurations](#build-configurations)
  - [S2I](#s2i)
- [Troubleshooting](#troubleshooting)
- [Integrated logging](#integrated-logging)
- [Simple metrics](#simple-metrics)
- [Resource scheduling](#resource-scheduling)
- [Multiproject quota](#multiproject-quota)
- [Essential Docker Registry Commands](#essential-docker-registry-commands)
  - [Private Docker Registry and Access](#private-docker-registry-and-access)
- [Docker Commands](#docker-commands)
- [Basic Networking](#basic-networking)
- [Technical Jargons](#technical-jargons)

<!-- /TOC -->

## 1.1. OpenShift CLI - Installation
```oc``` command line tool will be installed on all master and node machines during cluster installation. You can also install oc utility on any other machines which is not part of openshift cluster. 
Download oc cli tool from : https://www.okd.io/download.html

On a RHEL system with valid subscription you can install with yum as below.
```
$ sudo yum install -y atomic-openshift-clients
```

Many common oc operations are invoked using the following syntax:
```
$ oc <action> <object_type> <object_name_or_id>
```

## Basic Structure of OpenShift/Kubernetes defenition file
```
apiVersion: v1
kind: Service
metadata: 
  name: my-service
spec:
  type: NodePort
  ports:
    - targetPort: 80
      port: 80
      nodePort: 30008
  selector:
    app: myapp
    type: front-end
```

## Login and Logout
```
oc login https://10.142.0.2:8443 -u admin -p openshift 
                              # Login to openshift cluster
oc whoami                     # identify the current login
oc login -u system:admin      # login to cluster from any master node without a password
oc logout                     # logout from cluster
```
## oc status
```
oc status -v                  # get oc cluster status
oc types                      # to list all concepts and types
```
## Managing Projects
```
oc get projects               # list Existing Projects
oc get project                # Display current project
oc project myproject          # switch to a project
oc new-project testlab --display-name='testlab' --description='testlab'        
                              # create a new project
oc adm new-project testlab --node-selector='project101=testlab'
                              # create a new project with node-selector. 
                              # Project pods will be created only those nodes with a label "project101=testlab"
oc delete project testlab     # delete a project
oc delete all --all           # delete all from a project
oc delete all -l app=web      # delete all where label app=web
```
## Resources
```
oc get all                    # list all resource items
                                -w  watches the result output in realtime.
oc process                    # process a template into list of resources.                              
```

## Taints and Tolerations
```
kubectl taint nodes node1 app=blue:NoSchedule
                              # Apply taint on node
kubectl taint nodes node1 app=blue:NoSchedule-                               
                              # untaint a node by using "-" at the end.
```

## Controlling Access & Managing Users
```
oc create user USER_NAME       # create a user
oc adm add-role-to-user ROLE_NAME USERNAME -n PROJECT_NAME
                              # add cluster role to a user
                              # add-role-to-group - to add role to a group
                              # add-cluster-role-to-user - to add cluster role to a user
                              # add-cluster-role-to-group - to add cluster role to a group
eg:
oc adm add-role-to-user edit demo-user -n demo-project
oc adm policy add-cluster-role-to-user cluster-admin develoer
                              # add cluster-admin role to the user developer                              
oc adm policy remove-cluster-role-from-group \
  self-provisioner \
  system:authenticated \
  system:authenticated:oauth
                              # remove role from a group
oc get sa                     # list all service accounts
oc get cluserrole             # list all cluster rolesrole  
oc get rolebinding -n PROJECT_NAME
                              # list all roles details for the project
oc describe policybindings :default -n PROJECT_NAME
                              # OCP 3.7 < show details of a project policy details 
oc describe rolebinding.rbac -n PROJECT_NAME
                              # OCP 3.7 > show details of a project policy details  
oc describe user USER_NAME    # details of a user    
oc adm policy who-can edit pod -n PROJECT_NAME
                              # list details of access
```

You can also create user with HTPasswdIdentityProvider module as below.
```
htpasswd -b /etc/origin/master/htpasswd user1 password1
                              # create user1
                              # -b used to take password from command line rather than promopting for it.
htpasswd -D /etc/origin/master/htpasswd user1
                              # -D deletes user1
```  

### Check Access

```
kubectl auth can-i create deployments
                                      # check access 
kubectl auth can-i create deployments \
  --as user1
                                      # check access for a user
kubectl auth can-i create deployments \
  --as user1 \
  --namespace dev                     
                                      # check access for a user in a namespace
```

                            

## oc describe 
```
oc describe node <node1>      # show deatils of a specific resource
oc describe pod POD_NAME      # pod details                               
oc describe svc SERVICE_NAME  # service details                               
oc describe route ROUTE_NAME  # route details                               
```
## oc export 
```
oc export RESOURCE_TYPE RESOURCE_NAME -o OUTPUT_FORMAT
                              # export a definition of a resource (creating a backup etc) in JSON or YAML format.
oc export pod mysql-1-p1d35 -o yaml
oc export svc/myapp -o json

```
## Managing pods
Get pods, Rollout, delete etc.

```
oc get pods                   # list running pods inside a project
oc get pods -o wide           # detailed listing of pods
oc get pod -o name            # for pod names
oc get pods -n PROJECT_NAME   # list running pods inside a project/name-space
oc get pods --show-labels     # show pod labels
oc get pods --selector env=dev
                              # list pods with env=dev
oc get po POD_NAME -o=jsonpath="{..image}"
                              # get othe pod image details
oc get po POD_NAME -o=jsonpath="{..uid}"
                              # get othe pod uid details
oc adm manage-node NODE_NAME --list-pods
                              # list all pods running on specific node
oc rollout history dc/<name>  # available revisions
oc rollout latest hello       # deploy a new version of app.
oc rollout undo dc/<name>     # rollback to the last successful 
                                deployed revision of your configuration
oc rollout cancel dc/hello    # cancel current depoyment 

oc delete pod POD_NAME -n PROJECT_NAME --grace-period=0 --force
                              # delete a pod forcefully
                                if pod still stays in Terminating state, 
                                try replace deletionTimestamp: null
                                as well as finalizers: null  
                                (it may contain an item foregroundDeletion, 
                                remove that)
kubectl get pods \
  --server kubesandbox:6443 \
  --client-key admin.key \
  --client-certificate admin.crt \
  --certificate-authority ca.cert
                              #  specify credential and certificate details.

kubectl get pods --kubeconfig config                              
                              # or put those info inside a file `config` \
                                and call --kubeconfig in command
```

### Static Pods
```
kubectl run --restart=Never --image=busybox static-busybox --dry-run -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml                                
                                # Create a static pod named static-busybox 
                                  that uses the busybox image and 
                                  the command sleep 1000
```
## Managing Nodes

```
oc get nodes                  # list nodes in a cluster
oc get node/NODE_NAME -o yaml
                              # to see a node’s current capacity and allocatable resources
oc get nodes --show-labels | grep -i "project101=testlab"
                              # show nodes info with lable and list only node with a lable "project101=testlab"
oc get nodes -L region -L env
                              # show nodes with "region" and "evn" labels

oadm manage-node compute-102 --schedulable=false
kubectl cordon node-2
                              # make a node unschedulable
oc adm drain compute-102                              
kubectl drain node-1          # drain node by evicting pods
                              -–force — force deletion of bare pods
                              –-delete-local-data — delete even if there are 
                                pods using emptyDir (local data that will be deleted
                                when the node is drained)
                              -–ignore-daemonsets — ignore daemonset-managed pods
                              
oadm manage-node compute-102 --schedulable=true
kubectl uncordon node-1       # enable scheduling on node
```


## PV & PVC - PersistentVolume & PersistentVolumeClaim
``` 
oc get pv                       # list all pv in the cluster
oc create -f mysqldb-pv.yml     # create a pv with template
oc get pvc -n PROJECT_NAME      # list all pvc in the project
oc set volume dc/mysqldb \
  --add --overwrite --name=mysqldb-volume-1 -t pvc \
  --claim-name=mysqldb-pvclaim \
  --claim-size=3Gi \
  --claim-mode='ReadWriteMany'
                                # Create volume claim for mysqldb-volume-1
kubectl get pv
kubectl get pvc
```

## oc exec - execute command inside a containe
```
oc exec  <pd> -i -t -- <command> 
                              # run command inside a container without login
                                eg: oc exec  my-php-app-1mmh1 -i -t -- curl -v http://dbserver:8076
```
## Events and Troubleshooting
```
oc get events                 # list events inside cluster
oc logs POD                   # get logs from pod
oc logs <pod> --timestamps    
oc logs -f bc/myappx           
oc rsh <pod>                  # login to a pod

kubectl logs -f POD_NAME CONTAINER_NAME
                              # mention container name if you have
                                more than one container inside pod
```

## Understand and Help
```
oc explain <resource>         # documentation of a resource and its fields
                                eg: oc explain pod
                                    oc explain pod.spec.volumes.configMap
```
## Applications
```oc new-app``` will create a,
- dc (deploynment configuration)
- is (image stream)
- svc (service)

```
oc new-app mysql MYSQL_USER=user MYSQL_PASSWORD=pass MYSQL_DATABASE=mydb -l db=mysql
                              # create a new application
oc new-app --docker-image=myregistry.example.com/dockers/myapp --name=myapp
                              # create a new application from private registry
oc new-app https://github.com/techbeatly/python-hello-world --name=python-hello
                              # create a new application from source code (s2i)
                              # -i or --image-stream=[] : Name of an image stream to use in the app
```

How to find registry ?

```
oc get route -n default       # you can see the registry url
```

## 1.2. Get Help

```
# oc help                     # list oc command help options
```

## 1.3. Build from image

```
oc new-build openshift/nodejs-010-centos7~https://github.com/openshift/nodejs-ex.git --name='newbuildtest'
```

## 1.4. Enable/Disable scheduling

```
oadm manage-node mycbjnode --schedulable=false 
                              # Disable scheduling on node
```

## 1.5. Resource quotas

Hard constraints how much memory/CPU your project can consume

```
oc create -f <YAML_FILE_with_kind: ResourceQuota> -n PROJECT_NAME
                              # create quota details with YAML tempalte where kind should ResourceQuota
                              # Sample : https://github.com/ginigangadharan/openshift-cli-cheatsheet/blob/master/quota-template-32Gi_no_limit.yaml
oc describe quota -n PROJECT_NAME
                              # describe the quota details
oc get quota -n PROJECT_NAME  
                              # get quota details of the project
oc delete quota -n PROJECT_NAME 
                              # delete a quota for the project                            
```

## 1.6. Labels & Annotations

1. Label examples: release, environment, relationship, dmzbased, tier, node type, user type
    - Identifying metadata consisting of key/value pairs attached to resources
2. Annotation examples: example.com/skipValidation=true, example.com/MD5checksum-1234ABC, example.com/BUILDDATE=20171217
    - Primarily concerned with attaching non-identifying information, which is used by other clients such as tools or libraries

```
oc label node1 region=us-west zone=power1a --overwrite
oc label node node2 region=apac-sg zone=power2b --overwrite
oc patch node NODE_NAME -p '{"metadata": {"labels": {"project101":"testlab"}}}'
                              # add label to node
oc patch dc myapp --patch '{"spec":{"template":{"nodeselector":{"env":"qa"}}}'
                              # modify dc to run pods only on nodes where label 'evn':'qa'
oc label secret ssl-secret env=test
                              # add label                              
```

## 1.7. Limit ranges

- mechanism for specifying default project CPU and memory limits and requests


```
oc get limits -n development
oc describe limits core-resource-imits -n development
```

## 1.8. ClusterQuota or ClusterResourceQuota

Ref: https://docs.openshift.com/container-platform/3.3/admin_guide/multiproject_quota.html

```
oc create clusterquota for-user-developer --project-annotation-selector openshift.io/requester=developer --hard pods=8
oc get clusterresourcequota |grep USER
                              # find the clusterresourcequota for USER
oc describe clusterresourcequota USER
```

## 1.9. Config View

```
oc config view                  # command to view your current, full CLI configuration
                                  also can see the cluster url, project url etc.

kubectl config view             # to view the config in ~/.kube/config

kubectl config view --kubeconfig=path-to-config-file
                                # to view the config

kubectl config use-context dev@singapore-cluster                              
                                # to change the current-context 
                                
kubectl config -h               # to list avaialbe options    
```

## 1.10. Managing Environment Variables

https://docs.openshift.com/enterprise/3.0/dev_guide/environment_variables.html

```
oc env rc/RC_NAME --list -n PROJECT
                                # list environment variable for the rc
oc env rc my-newapp MAX_HEAP_SIZE=128M
                                # set environment variable for the rc
```

## 1.11. Security Context Constraints

```
oc get scc                      # list all seven SCCs
                                      - anyuid
                                      - hostaccess
                                      - Hostmount-anyuid
                                      - hostnetwork    	
                                      - nonroot
                                      - privileged
                                      - restricted
oc describe scc SCC_NAME        # can see which all service account enabled.                                      
```

## 1.12. Services & Routes

```
oc expose service SERVICE_NAME route-name-project-name.default-domain
or
oc expose svc SERVICE_NAME
                                # create/expose a service route
eg:
oc expose service myapache --name=myapache --hostname=myapache.app.cloudapps.example.com
                                # if you don't mention the hostname, then
                                # it will create a hostname as route-name-project-name.default-domain
                                # if you don't mention the route name, then
                                # it will take the service name as route name

oc port-forward POD_NAME 3306:3306
                                # temporary port-forwarding to a port from local host.
```   

## 1.13. Scaling & AutoScaling of the pod - HorizontalPodAutoscaler

**OpenShift**
```
oc scale dc/APP_NAME --replicas=2                              
                                # scale application (increase or decrease replicas)
oc autoscale dc my-app --min 1 --max 4 --cpu-percent=75
                                # enable autoscaling for my-app
oc get hpa my-app               # list Horizontal Pod Autoscaler
oc describe hpa/my-app
```

**Kubernetes**
```
kubectl create -f replicaset-defenition.yml
                                # create replicaset
kubectl create -f replicaset-defenition.yml -namespace=YOUR_NAMESPACE
                              # create in a specific namespace                         
kubectl replace -f replicaset-defenition.yml
                                # change the replicas option in replicaset defenition
                                # and then run it
kubectl scale --replicas=6 -f replicaset-defenition.yml
kubectl scale --replicas=6 replicaset myapp-replicaset
                                # this one will not update replica details 
                                # in replicaset defenition file
kubectl delete replicaset myapp-replicaset
                                # delete replicaset
```

## 1.14. Configuration Maps (ConfigMap)

- Similar to secrets, but with non-sensitive text-based configuration

## 1.15. Creation of objects

```
oc create configmap test-config --from-literal=key1=config1 --from-literal=key2=config2 --from-file=filters.properties

oc volume dc/nodejs-ex --add -t configmap -m /etc/config --name=app-config --configmap-name=test-config
```

## 1.16. Reading config maps

```
oc rsh nodejs-ex-26-44kdm ls /etc/config
```

## 1.17. Dynamically change the config map

```
oc delete configmap test-config

<CREATE AGAIN WITH NEW VALUES>

<NO NEED FOR MOUNTING AS VOLUME AGAIN>
```

## 1.18. Mounting config map as ENV

```
oc set env dc/nodejs-ex --from=configmap/test-config
oc describe pod nodejs-ex-27-mqurr
```

## 1.19. The Replication Controller
*to be done*

```
oc describe RESOURCE RESOURCE_NAME
oc export
oc create
oc edit
oc exec POD_NAME <options> <command>
oc rsh POD_NAME <options>
oc delete RESOURCE_TYPE name
oc version
docker version

oc cluster up \
  --host-data-dir=... \
  --host-config-dir=...

oc cluster down

oc cluster up \
  --host-data-dir=... \
  --host-config-dir=... \
  --use-existing-config

oc project myproject
```

## 1.20. Create persistent volume

- Supports stateful applications
- Volumes backed by shared storage which are mounted into running pods
- iSCSI, AWS EBS, NFS etc.

## 1.21. Create volume claim

- Manifests that pods use to retreive and mount the volume into pod at initialization time

- Access modes: REadWriteOnce, REadOnlyMany, ReadWriteMany

## 1.22. Deployments
```
kubectl run blue --image=nginx --replicas=6
                                # Create a new deployment named blue
                                  with nginx image and 6 replicas
kubectl set image deployment/myapp-dc                                
                                # specify new image to deployment
kubectl apply -f DEFINITION.YML                                
                                # apply new config to existing deployment
kubectl rollout undo deployment/myapp-dc                                
                                # rollback a deployment
kubectl rollout status deployment/myapp-dc                                
                                # status of deployment
kubectl rollout history deployment/myapp-dc                                
                                # history of deployment
```

## 1.23. Deployment strategies

## 1.24. Rolling

## 1.25. Triggers

## 1.26. Recreate

## 1.27. Custom

## 1.28. Lifecycle hooks

## 1.29. Deployment Pod Resources

## 1.30. Blue-Green deployments

```
oc new-app https://github.com/devops-with-openshift/bluegreen#green --name=green

oc patch route/bluegreen -p '{"spec":{"to":{"name":"green"}}}'

oc patch route/bluegreen -p '{"spec":{"to":{"name":"blue"}}}'
```

## 1.31. A/B Deployments

```
oc annotate route/ab haproxy.router.openshift.io/balance=roundrobin
oc set route-backends ab cats=100 city=0
oc set route-backends ab --adjust city=+10%
```

## 1.32. Canary Deployments

## 1.33. Rollbacks

```
oc rollback cotd --to-version=1 --dry-run
                                # Dry run only
oc rollback cotd --to-version=1
oc describe dc cotd
```


## 1.34. Pipelines

```
oc new-app jenkins-pipeline-example
oc start-build sample-pipeline
```

- Customizing Jenkins:

```
vim openshift.local.config/master/master-confi.yaml

jenkinsPipelineConfig:
  autoProvisionEnabled: true
  parameters:
    JENKINS_IMAGE_STREAM_TAG: jenkins-2-rhel7:latest
    ENABLE_OAUTH: true
  serviceName: jenkins
  templateName: jenkins-ephemeral
  templateNamespace: openshift
```  
- Good resource for Jenkinsfiles: https://github.com/fabric8io/fabric8-jenkinsfile-library
  
## 1.35. Configuration Management

## 1.36. Secrets

## 1.37. Creation

- Maximum size 1MB

```
oc secret new test-secret cert.pem
oc secret new ssl-secret keys=key.pem certs=cert.pem
oc get secrets --show-labels=true
oc delete secret ssl-secret
```

## 1.38. Using secrets in Pods

- Mounting the secret as a volume

```
oc volume dc/nodejs-ex --add -t secret --secret-name=ssl-secret -m /etc/keys --name=ssl-keys deploymentconfigs/nodejs-ex
oc rsh nodejs-ex-22-8noey ls /etc/keys
```

- Injecting the secret as an env var

```
oc secret new env-secrets username=user-file password=password-file
oc set env dc/nodejs-ex --from=secret/env-secret
oc env dc/nodejs-ex --list
```

## 1.39. ENV

## 1.40. Adding

```
oc set env dc/nodejs-ex ENV=TEST DB_ENV=TEST1 AUTO_COMMIT=true
oc set env dc/nodejs-ex --list
```

## 1.41. Removing

```
oc set env dc/nodejs-ex DB_ENV-
```

## 1.42. Change triggers

1. `ImageChange` - when uderlying image stream changes

2. `ConfigChange` - when the config of the pod template changes



## 1.43. OpenShift Builds

### 1.43.1. Build strategies

- Source-to-Image (S2I): uses the opensource S2I tool to enable developers to reporducibly build images by layering the application's soure onto a container image

- Docker: using the Dockerfile

- Pipeline: uses Jenkins, developers provide Jenkinsfile containing the requisite build commands

- Custom: allows the developer to provide a customized builder image to build runtime image

### 1.43.2. Build sources

- Git
- Dockerfile
- Image
- Binary

### 1.43.3. Build Configurations

- contains the details of the chosen build strategy as well as the source

```
oc new-app https://github.com/openshift/nodejs-ex
oc get bc/nodejs-ex -o yaml
```

- unless specified otherwise, the `oc new-app` command will scan the supplied Git repo. If it finds a Dockerfile, the Docker build strategy will be used; otherwise source strategy will be used and an S2I builder will be configured


### 1.43.4. S2I

- Components:

1. Builder image - installation and runtime dependencies for the app
2. S2I script - assemble/run/usage/save-artifacts/test/run

- Process:

1. Start an instance of the builder image
2. Retreive the source artifacts from the specified repository
3. Place the source artifacts as well as the S2I scripts into the builder image (bundle into .tar and stream into builder image)
4. Execute assemble script
5. Commit the image and push to OCP registry

- Customize the build process:

1. Custom S2I scripts - their own assemble/run etc. by placing scripts in .s2i/bin at the base of the source code, can also contain environment file
2. Custom S2I builder - write your own custom builder

## 1.44. Troubleshooting

- Adding the --follow flag to the start-build command
- oc get builds
- oc logs build/test-app-3
- oc set env bc/test-app BUILD_LOGLEVEL=5 S2I_DEBUG=true

```
oc adm diagnostics
```
- `--dry-run`: To test your command; this will not create the resource, instead, tell you weather the resource can be created and if your command is right.
- `-o yaml`: to print the output in YAML (or JASON) format.


- Operational layers:

1. Operating system infrastructure operations - compute, network, storage, OS
2. Cluster operations - cluster managemebt OpenShift/Kubernetes
3. Application operations - deployments, telemetry, logging

## 1.45. Integrated logging

- the EFK (Elasticsearch/Fluentd/Kibana) stack aggregates logs from nodes and application pods

```
oc cluster up --logging=true
```

## 1.46. Simple metrics

- the Kubelet/Heapster/Cassandra and you can use Grafana to build dashboard

```
oc cluster up --metrics=true

kubectl top node                # memory and CPU usage on node 
kubectl top pod                 # memory and CPU usage by pods

# Enable metrics in minikube
minikube addons enable metrics-server
```

## 1.47. Resource scheduling

- default behavior:

1. best effor isolation = no primises what resources can be allocated for your project

2. might get defaulted values

3. out of memory killed randomly

4. might get CPU starved (wait to schedule your workload)



## 1.48. Multiproject quota

- you may use project labels or annotations when creating multiproject spanning quotas

```
oc login -u system:admin
oc login -u developer -p developer
oc describe AppliedClusterResourceQuota
```

## 1.49. Essential Docker Registry Commands
```
docker login -u USER_NAME -p TOKEN REGISTRY_URL
                                # before we push images, we need to 
                                  login to docker registry.
                                  
docker login -u developer -p ${TOKEN} \
  docker-registry-default.apps.lab.example.com                                
                                # TOKEN can be get as TOKEN=$(oc whoami)
                                
docker images --no-trunc --format '{{.ID}} {{.CreatedSince}}' --filter "dangling=true" --filter "before=IMAGE_ID"
                                # list image with format and 
                                # using multiple filters                                
```

### 1.49.1. Private Docker Registry and Access
```
kubectl create secret docker-registry private-docker-cred \
    --docker-server=myregistry
    --docker-username=registry-user
    --docker-password=registry-password
    --docker-email=registry-user@example.com
                                # Create a secret for docker-registry
```
Then specify the image pull secret under the `imagePullSecrets` of pod/deployment definition (same level of `container`)
```
    imagePullSecrets:
    - name: private-docker-cred
```        

## 1.50. Docker Commands

Image Handling
```
docker create [IMAGE]           # Create a new container from a particular image.
docker search [term]            # Search the Docker Hub repository for a particular term.
docker login                    # Log into the Docker Hub repository.
docker pull [IMAGE]             # Pull an image from the Docker Hub repository.
docker push [username/image]    # Push an image to the Docker Hub repository.
docker tag [source] [target]    # Create a target tag or alias that refers to a source image.
```

Running Containers
```
docker start [CONTAINER]        # Start a particular container.
docker stop [CONTAINER]         # Stop a particular container.
docker exec -ti [CONTAINER] [command]
                                # Run a shell command inside a particular container.
docker run -ti — image [IMAGE] [CONTAINER] [command]
                                # Create and start a container at the same time, and then run a command inside it.
docker run -ti — rm — image [IMAGE] [CONTAINER] [command]
                                # Create and start a container at the same time, 
                                # run a command inside it, and then remove the container 
                                # after executing the command.
docker pause [CONTAINER]        # Pause all processes running within a particular container.
```

Docker Utilities
```
docker history [IMAGE]          # Display the history of a particular image.
docker ps                       # List all of the containers that are currently running.
docker version                  # Display the version of Docker that is currently installed on the system.
docker images                   # List all of the images that are currently stored on the system.
docker inspect [object]         # Display low-level information about a particular Docker object.
```

Cleaning Docker Environment
```
docker kill [CONTAINER]         # Kill a particular container.
docker kill $(docker ps -q)     # Kill all containers that are currently running.
docker rm [CONTAINER]           # Delete a particular container that is not currently running.
docker rm $(docker ps -a -q)    # Delete all containers that are not currently running.
docker network ls               # list available networks
```

## 1.51. Basic Networking
(For docker/kubernetes/openshift operations)

```
ip link                         # show interface of host
ip addr add 10.1.10.10/24 dev eth0
                                # assign IP to an interface
ip route add 10.1.20.0/24 via 10.1.10.1
                                # add a route to another network 10.1.20.0/24
                                  via 10.1.10.1 which is our router/gateway.   
ip route add default via 10.1.10.1
                                # add defaulr route to any network; like internet
                                  you can also mention 0.0.0.0/0 instead of default
route                           # display kernel routing table

ip netns add newnamespace       # create a new network namespace
ip netns                        # list network namespaces
ip netns exec red ping IP_ADDRESS
ip netns exec newnamespace ip link
                                # display details inside namespace
ip link add veth-red type veth peer name veth-blue
                                # create a pipe or virtual eth (veth) 
ip link set veth-red netns red  
                                # attach the virtual interface to a namespace          
ip -n red addr add 10.1.10.1 dev veth-red
                                # assign ip for virtual interface (veth) 
                                  inside a namespace                                
ip -n red link set veth-red up
                                # make virtual interface up and running
ip link add v-net-0 type bridge 
                                # add linux bridge
                                                                
```

## 1.52. Technical Jargons
```
OSSM                OpenShift Service Mesh (OSSM)
                    Istio is the upstream project
                    - The upstream Istio community installation automatically 
                      injects the sidecar to namespaces you have labeled.
                    - Red Hat OpenShift Service Mesh does not automatically 
                      inject the sidecar to any namespaces, but requires you to 
                      specify the sidecar.istio.io/inject annotation as 
                      illustrated in the Automatic sidecar injection section.

CRI-O               Container Runtime Interface
OCI                 Open Container Initiative
cgroup              control group
Jaeger              Distributed Tracing System
kiali               observability console for Istio
                    Kiali answers the questions:
                    -  What microservices are part of my Istio service mesh?
                    -  How are they connected?
                    -  How are they performing?

runc                CLI tool for spawning and running 
                    containers according to the OCI specification.
FaaS                Function as a Service
CaaS                Containers as a service          
                    
```

# 2. Points to Remember
- Docker was started as a project by a company called **[dotCloud](https://www.docker.com/docker-news-and-press/dotcloud-inc-now-docker-inc)**, made available as open source in March 2013.
- Kubernetes surfaced from work at Google in 2014, and became the standard way of managing containers.


# 3. References
- https://cloudowski.com/articles/10-differences-between-openshift-and-kubernetes/