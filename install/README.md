# Install

## 1. Install OpenShift Serverless

1. `oc apply -f install/1-serverless-operator.yaml`
2. `oc apply -f install/2-knative-serving.yaml`
3. `oc apply -f install/3-knative-eventing.yaml`

### 2. Install `kn` cli

1. `wget https://storage.googleapis.com/knative-nightly/client/latest/kn-linux-amd64`
    - This is the nightly binary that is considered newer/experimental mode.
    - Stable releases can be found at: https://github.com/knative/client/releases
2. `sudo cp kn-linux-amd64 /usr/local/bin/kn`
3. `sudo chmod 755 /usr/local/bin/kn`
4. `rm kn-linux-amd64`
5. Verify install: `kn version`


### 3. Install `func` cli

1. `wget https://github.com/knative/func/releases/download/knative-v1.16.1/func_linux_amd64`
2. `sudo cp func_linux_amd64 /usr/local/bin/func`
3. `sudo chmod 755 /usr/local/bin/func`
4. `rm func_linux_amd64`
5. Verify install: `func version`


------------
------------
------------

## 1. Optional - Install Serverless Logic Operator

1. Do this in UI as it's in alpha.
   - `Operators -> OperatorHub -> Search for "logic" -> Install "OpenShift Serverless Logic Operator"`


### 2. Optional - Install `kn-workflow` plugin (for Serverless Logic)

1. `wget https://developers.redhat.com/content-gateway/file/pub/cgw/serverless-logic/1.34.0/kn-workflow-linux-amd64`
2. `sudo cp kn-workflow-linux-amd64 /usr/local/bin/kn-workflow`
3. `sudo chmod 755 /usr/local/bin/kn-workflow`
4. `rm kn-workflow-linux-amd64`
5. Verify install: `kn plugin list ; kn workflow version`