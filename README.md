
# Install

[install/README.md](install/README.md)


---------------
---------------


# Knative Serving


## `kn service create` : Create a service from an image

`kn service create serverless-service --namespace serverless-demo --port 8080 --env TARGET=v1 --env FROM=examples --image knativesamples/helloworld` : create a Knative Service from an image

`--target .` : Output to yaml (you can oc apply or put it in ArgoCD, or etc.)

`--cluster-local` : No Route


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

`--tag blue` : Tagging for traffic allocation. This sticks the traffic by the image's digest SHA key, and not by a tag like "latest". The tag will move up as you create new revisions, however traffic will remain on the original tag provided.

`--tag red --untag blue` : Remove old tag and replace with new tag.

`--traffic` : traffic allocation


### Update revision but dont forward traffic to new revision
`kn service update serverless-service --env TARGET=someNewEnvVarABC --tag blue`

### Update traffic allocation:

`kn service update serverless-service --traffic serverless-service-00008=30,serverless-service-00003=70`
 - Same can be done in YAML: https://knative.dev/docs/serving/traffic-management/




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