- [Install](#install)
- [Knative Serving](#knative-serving)
  - [YAML / GitOps](#yaml--gitops)
  - [`kn` CLI](#kn-cli)
    - [`kn service create` : Create a service from an image](#kn-service-create--create-a-service-from-an-image)
    - [`kn service list`](#kn-service-list)
    - [`kn revision list`](#kn-revision-list)
    - [`kn service update` : updates revision](#kn-service-update--updates-revision)
      - [Update revision but only forward 20% of traffic to new one](#update-revision-but-only-forward-20-of-traffic-to-new-one)
      - [Get route info](#get-route-info)
      - [Update traffic allocation:](#update-traffic-allocation)
    - [Delete](#delete)
      - [Delete any unused revisions (untagged and without traffic allocated)](#delete-any-unused-revisions-untagged-and-without-traffic-allocated)
      - [Delete a Knative Serving Service (will delete all revisions too)](#delete-a-knative-serving-service-will-delete-all-revisions-too)
- [knative Functions](#knative-functions)
  - [`func create -h` : List available programming languages and examples of creating functions](#func-create--h--list-available-programming-languages-and-examples-of-creating-functions)
  - [Setup function prerequisites if building Function(`func build`) from OpenShift.](#setup-function-prerequisites-if-building-functionfunc-build-from-openshift)
  - [`func` cli from localhost - Create sample, build, push to Image Registry and deploy as Knative Service](#func-cli-from-localhost---create-sample-build-push-to-image-registry-and-deploy-as-knative-service)
    - [`Python` - Create and deploy to remote registry (auth to Dockerhub and ocp/k8s)](#python---create-and-deploy-to-remote-registry-auth-to-dockerhub-and-ocpk8s)
    - [`Go` - local OCP image registry (requires auth to ocp/k8s)](#go---local-ocp-image-registry-requires-auth-to-ocpk8s)
    - [`Node` - local OCP image registry (requires auth to ocp/k8s)](#node---local-ocp-image-registry-requires-auth-to-ocpk8s)



# Install

[install/README.md](install/README.md)


---------------
---------------


# Knative Serving

## YAML / GitOps

[serverless-service.yaml](serverless-demo/ksvc/serverless-service.yaml)

1. New Knative Service: `oc apply -f serverless-demo/ksvc/serverless-service.yaml`
    - This will create a Knative Service with a Revision name `serverless-service-00001` and tag of `green`
2. Update [serverless-service.yaml](serverless-demo/ksvc/serverless-service.yaml) with comments provided in file:
    - Environment variable `value: v1` to `value: v2`. This is to create a new Revision (deployment).
    - Route partial traffic to new Revision(deployment)
3. Update Knative Service for New Revision with partial traffic: `oc apply -f serverless-demo/ksvc/serverless-service.yaml`


## `kn` CLI

### `kn service create` : Create a service from an image

`kn service create serverless-service --namespace serverless-demo --port 8080 --env TARGET=v1 --env FROM=examples --image knativesamples/helloworld` : create a Knative Service from an image

`--target .` : Output to yaml (you can oc apply or put it in ArgoCD, or etc.)

`--cluster-local` : No Route (accessible only within cluster)


### `kn service list`

```
NAME                 URL                                                                 LATEST                     AGE   CONDITIONS   READY   REASON
serverless-service   https://serverless-service-serverless-demo.apps.dev.openshift.com   serverless-service-00001   14m   3 OK / 3     True 
```

### `kn revision list`
```
NAME                       SERVICE              TRAFFIC   TAGS   GENERATION   AGE   CONDITIONS   READY   REASON   
serverless-service-00002   serverless-service    100%            2            15m   3 OK / 4     True    
serverless-service-00001   serverless-service                    1            58m   3 OK / 4     True    
```

### `kn service update` : updates revision

`--traffic` : traffic allocation

`--tag blue` : Tag to get a custom route that can be used to test traffic. This sticks the traffic by the image's digest SHA key, and not by a tag like "latest". The tag will move up as you create new revisions, however traffic will remain on the original revision provided.

`--tag red --untag blue` : Remove old tag and replace with new tag.


#### Update revision but only forward 20% of traffic to new one
`kn service update serverless-service --env TARGET=someNewEnvVarABCDEF --tag serverless-service-00001=green --tag @latest=blue --traffic @latest=20 --traffic green=80`

#### Get route info
```
$ kn routes describe serverless-service

Name:       serverless-service
Namespace:  serverless-demo
Age:        12m
URL:        https://serverless-service-serverless-demo.apps.dev.openshift.com
Service:    serverless-service

Traffic Targets:  
   20%  @latest (serverless-service-00002) #blue
        URL:  https://blue-serverless-service-serverless-demo.apps.dev.openshift.com
   80%  serverless-service-00001 #green
        URL:  https://green-serverless-service-serverless-demo.apps.dev.openshift.com
```

#### Update traffic allocation:

`kn service update serverless-service --traffic @latest=80 --traffic green=20`
 - Same can be done in YAML: https://knative.dev/docs/serving/traffic-management/



### Delete

#### Delete any unused revisions (untagged and without traffic allocated)
`kn revision delete --prune-all`

#### Delete a Knative Serving Service (will delete all revisions too)
`kn service delete serverless-service`



---------------
---------------
---------------

# knative Functions
Knative Functions provides a simple programming model for using functions on Knative, without requiring in-depth knowledge of Knative, Kubernetes, containers, or dockerfiles.

Auto detects runtime using [BuildPacks](https://buildpacks.io/docs/)


## `func create -h` : List available programming languages and examples of creating functions

## Setup function prerequisites if building Function(`func build`) from OpenShift.
1. `func tkn-tasks | oc apply -f -`

2. `oc apply -f https://raw.githubusercontent.com/openshift-knative/kn-plugin-func/serverless-1.34/pkg/pipelines/resources/tekton/pipeline/dev-console/0.1/nodejs-pipeline.yaml`
   


## `func` cli from localhost - Create sample, build, push to Image Registry and deploy as Knative Service

### `Python` - Create and deploy to remote registry (auth to Dockerhub and ocp/k8s)
1. `func create --language python ./functions/pythonapp`
2. `cd functions/pythonapp`
3. `func build pythonapp --registry docker.io/DockerHubUsername/serverless-py-function`
4. `func deploy --registry docker.io/DockerHubUsername/serverless-py-function --namespace serverless-demo`


### `Go` - local OCP image registry (requires auth to ocp/k8s)
1. `func create --language go ./functions/goapp`
2. `cd functions/goapp`
3. `func build`
4. `func deploy --namespace serverless-demo `

### `Node` - local OCP image registry (requires auth to ocp/k8s)
1. `func create --language node ./functions/nodeapp`
2. `cd functions/nodeapp`
3. `func build`
4. `func deploy --namespace serverless-demo`