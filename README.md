
# Install

[install/README.md](install/README.md)


---------------
---------------


# Knative Serving


## `kn service create` : Create a service from an image

`kn service create serverless-service --namespace serverless-demo --port 8080 --env TARGET=v1 --env FROM=examples --image knativesamples/helloworld` : create a Knative Service from an image

`--target .` : Output to yaml (you can oc apply or put it in ArgoCD, or etc.)

`--cluster-local` : No Route (accessible only within cluster)


## `kn service list`

```
NAME                 URL                                                                 LATEST                     AGE   CONDITIONS   READY   REASON
serverless-service   https://serverless-service-serverless-demo.apps.dev.openshift.com   serverless-service-00001   14m   3 OK / 3     True 
```

## `kn revision list`
```
NAME                       SERVICE              TRAFFIC   TAGS   GENERATION   AGE   CONDITIONS   READY   REASON   
serverless-service-00002   serverless-service    100%            2            15m   3 OK / 4     True    
serverless-service-00001   serverless-service                    1            58m   3 OK / 4     True    
```

## `kn service update` : updates revision

`--traffic` : traffic allocation

`--tag blue` : Tag to get a custom route that can be used to test traffic. This sticks the traffic by the image's digest SHA key, and not by a tag like "latest". The tag will move up as you create new revisions, however traffic will remain on the original revision provided.

`--tag red --untag blue` : Remove old tag and replace with new tag.


### Update revision but only forward 20% of traffic to new one
`kn service update serverless-service --env TARGET=someNewEnvVarABCDEF --tag serverless-service-00001=green --tag @latest=blue --traffic @latest=20 --traffic green=80`

### Get route info
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

### Update traffic allocation:

`kn service update serverless-service --traffic @latest=80 --traffic green=20`
 - Same can be done in YAML: https://knative.dev/docs/serving/traffic-management/



## Delete

### Delete any unused revisions (untagged and without traffic allocated)
`kn revision delete --prune-all`

### Delete a Knative Serving Service (will delete all revisions too)
`kn service delete serverless-service`



---------------
---------------
---------------

# knative Functions
Knative Functions provides a simple programming model for using functions on Knative, without requiring in-depth knowledge of Knative, Kubernetes, containers, or dockerfiles.

Auto detects runtime using [BuildPacks](https://buildpacks.io/docs/)


## `func create -h` : List available programming languages and examples of creating functions

## Setup function prerequisites if using Web Console:
1. `func tkn-tasks | oc apply -f -`

2. `oc apply -f https://raw.githubusercontent.com/openshift-knative/kn-plugin-func/serverless-1.34/pkg/pipelines/resources/tekton/pipeline/dev-console/0.1/nodejs-pipeline.yaml`
   



## Create a python function
1. `func create --language python --template http pythonapp`
2. `cd pythonapp`
3. `func build pythonapp --registry docker.io/mdbell47/serverless-py-function`
4. `func deploy --registry docker.io/mdbell47/serverless-py-function --namespace serverless-demo`