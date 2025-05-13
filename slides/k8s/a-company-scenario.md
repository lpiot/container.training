# Using Kubernetes in an Enterprise-like scenario

- 💪🏼 Okay. The former training modules explain each subject in-depth, and each feature one at-a-time

- 🤯 One of the first complexity any `k8s` admin encounters resides in having to choose and assemble all these moving parts to build a _Production-ready_ `k8s` cluster

- 🎯 This module precisely aims to play a classic corporate scenario and see what we'll have to do to build such a Prod-ready Kubernetes cluster

- As we've done it before, we'll start to build our cluster from scratch and improve it step by step, by adding a feature one after another

---

## The plan

In a company, we have 3 different people, _**⚙️OPS**_, _**🎸ROCKY**_ and _**🎻CLASSY**_

- _**⚙️OPS**_ is the platform engineer building and configuring Kubernetes clusters

- Both _**🎸ROCKY**_ and _**🎻CLASSY**_ build Web apps that manage 💿 record collections

    - _**🎸ROCKY**_ builds a Web app managing _rock & pop_ collection
    - whereas _**🎻CLASSY**_ builds a Web app for _classical_ collection

- Each app is stored in its own git repository

- Both _**🎸ROCKY**_ and _**🎻CLASSY**_ want to code, build package and deploy their applications onto a `Kubernetes` _cluster_ _in an autonomous way_

---

## The whole cinematic

This is what our scenario looks like…

<pre class="mermaid">
%%{init:
    {
      "theme": "default",
      "gitGraph": {
        "mainBranchName": "ops",
        "mainBranchOrder": 0
      }
    }
}%%
gitGraph
    commit id:"0" tag:"start"
    branch ROCKY order:3
    branch CLASSY order:4

    checkout ops
    commit id:'BOOTSTRAP cluster creation'
    branch Bootstrap-cluster order:1
    commit id:'FLUX install on BOOTSTRAP'
    checkout ops
    commit id:'TEST cluster description'

    checkout Bootstrap-cluster
    merge ops id:'TEST cluster creation'
    branch TEST-cluster order:2
    commit id:'FLUX install on TEST'

    checkout ops
    commit id:'TEST cluster config.'
    checkout TEST-cluster
    merge ops id:'setup of TEST cluster'

    checkout ops
    commit id:'FLUX config for ROCKY deployment'
    checkout TEST-cluster
    merge ops id:'FLUX ready to deploy ROCKY' type: HIGHLIGHT

    checkout ROCKY
    commit id:'ROCKY' tag:'v1.0'
    checkout TEST-cluster
    merge ROCKY tag:'ROCKY v1.0 on TEST'
    
    checkout CLASSY
    commit id:'CLASSY' tag:'v1.0'

    checkout ops
    commit id:'FLUX config for CLASSY deployment'
    checkout TEST-cluster
    merge ops id:'FLUX ready to deploy CLASSY' type: HIGHLIGHT

    checkout CLASSY
    commit id:'CLASSY HELM chart'
    checkout TEST-cluster
    merge CLASSY tag:'CLASSY v1.0'
    
    checkout ROCKY
    commit id:'new color' tag:'v1.1'
    checkout TEST-cluster
    merge ROCKY tag:'ROCKY v1.1'

    checkout TEST-cluster
    commit id:'wrong namespace' type: REVERSE

    checkout ops
    commit id:'namespace isolation'
    checkout TEST-cluster
    merge ops type: HIGHLIGHT

    checkout ROCKY
    commit id:'fix namespace' tag:'v1.1.1'
    checkout TEST-cluster
    merge ROCKY tag:'ROCKY v1.1.1'

    checkout ROCKY
    commit id:'add a field' tag:'v1.2'
    checkout TEST-cluster
    merge ROCKY tag:'ROCKY v1.2'

    checkout ops
    commit id:'Kyverno install'
    commit id:'Kyverno rules'
    checkout TEST-cluster
    merge ops type: HIGHLIGHT

    checkout ops
    commit id:'Network policies'
    checkout TEST-cluster
    merge ops type: HIGHLIGHT

    checkout Bootstrap-cluster
    branch PROD-cluster order:5
    commit id:'FLUX install on PROD'
    commit id:'PROD cluster configuration'

    checkout ops
    commit id:'Add OpenEBS'
    checkout TEST-cluster
    merge ops id:'patch dedicated to PROD' type: REVERSE
    checkout PROD-cluster
    merge ops type: HIGHLIGHT
</pre>

---

# Creating the Kubernetes clusters
<!-- TODO: choose how to deploy k8s cluster -->

We want to have 2 main `Kubernetes` clusters, one for **testing** and one for **production**

--

- we should use tools to **industrialise creation** of both clusters

> 💻 we'll use `xxx` to configure and deploy

--

- each cluster has it's **own lifecycle** (the addition or configuration of extra components/features may be done on one cluster and not the other)
- yet, configurations of these clusters must be as centralized as possible (to avoid inconsistency and to limit configuration code expansion and maintainance)

> 💻 we'll use `flux` to configure and deploy new resources onto the clusters

---

## Multi-tenancy

Both _**🎸ROCKY**_ and _**🎻CLASSY**_ should use a **dedicated _"tenant"_** on each cluster

- _**🎸ROCKY**_ should be able to deploy, upgrade and configure its own app in its dedicated **namespace** without anybody else involved
- and the same for _**🎻CLASSY**_
- neither conflict nor collision should be allowed between the 2 apps or the 2 teams.

---

### Creating the bootstrap cluster

First we have to create a bootstrap cluster that will hold our ClusterAPI / k0smotron coonfiguration and operators…

blabla

<!-- TODO: complete the k0smotron setup -->

---

# 1st development team  _**🎸ROCKY**_

Our first development team is developping an app to manage **rock & pop** music records.

Here is a what it looks like :

<!-- TODO: include screenshot -->

---

## The code

Here is the code…

<!-- TODO: include screenshot -->


---

## How to deploy?

- The app is to be deployed on Kubernetes cluster, in a dedicated `rocky` namespace.  
- So the _**🎸ROCKY**_ team produces a YAML file to deploy the whole needed Kubernetes resources to run this application.

.lab[

- Review the deployment manifest:
  ```bash
  cd apps/rocky/
  cat ./deployment.yaml
  ```

]

--

Then, the _**🎸ROCKY**_ team will have to apply its deployment using this command  

```
kubectl apply -f ./deployment.yaml
```

---

## Deploying in an autonomous way…


To do so, the _**🎸ROCKY**_ team requires

- ⚠️ a **network connection** between its workstation (where `kubectl` is executed) and both Kubernetes clusters

--

- ⚠️ an enabled **account** to operate onto the Kubernetes clusters

--

- ⚠️ an always _up-to-date_ `kubectl` config with **valid credentials**

--

- ⚠️ a `kubectl` CLI in a version compatible with both Kubernetes clusters

--

- 🔒 if this command is executed by a _CI/CD pipeline_, it will need the same requirements

--

💡 There might be a better way. With GitOps! 🍾

---

## incoming Flux

💡 We'll use `Flux` so that deployments will be directly executed from inside the Kubernetes clusters.

--

The _**⚙️OPS**_ team will proceed to configure GitOps for the _**🎸ROCKY**_ team.

---

## Configuring Flux for _**🎸ROCKY**_ team

What the _**⚙️OPS**_ team has to do:

- 🔧 Creating a dedicated `rocky` _tenant_ on `staging` cluster
- 🔧 Creating the `flux` Github source pointing to the _**rocky**_ app source code repository
- 🔧 Adding a `kustomize` patch into the global `flux` config to include the `flux` configuration for the _**rocky**_ app

What the _**🎸rocky**_ team has to do:

- 👨‍💻 Creating the kustomize file in the _**rocky**_ app source code repository

---

## creating the dedicated `rocky` tenant

- Using the `flux` _CLI_, we create the file configuring the tenant for the _**🎸ROCKY**_ team
- This is done in the global mutualized `base` configuration for both Kubernetes clusters

.lab[

- Review the deployment manifest:
  ```bash
  
$ mkdir -p ./tenants/base/dev1
$ flux create tenant dev1            \
    --with-namespace=dev1-ns         \
    --cluster-role=dev1-full-access  \
    --export > ./tenants/base/dev1/rbac.yaml  ```

]

---

