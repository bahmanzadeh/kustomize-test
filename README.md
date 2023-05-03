# kustomize-test
Testing Kustomization in Kubernetes

## Link
https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/

## Instructions
Kustomize has the concepts of bases and overlays. A base is a directory with a kustomization.yaml, which contains a set of resources and associated customization. A base could be either a local directory or a directory from a remote repo, as long as a kustomization.yaml is present inside. An overlay is a directory with a kustomization.yaml that refers to other kustomization directories as its bases. A base has no knowledge of an overlay and can be used in multiple overlays. An overlay may have multiple bases and it composes all resources from bases and may also have customization on top of them.
Use --kustomize or -k in kubectl commands to recognize Resources managed by kustomization.yaml. Note that -k should point to a kustomization directory, such as

kubectl apply -k <kustomization directory>/

## Folder Structure

├── base
  │   ├── deployment.yaml
  │   ├── hpa.yaml
  │   ├── kustomization.yaml
  │   └── service.yaml
  └── overlays
      ├── dev
      │   ├── hpa.yaml
      │   └── kustomization.yaml
      ├── production
      │   ├── hpa.yaml
      │   ├── kustomization.yaml
      │   ├── rollout-replica.yaml
      │   └── service-loadbalancer.yaml
      └── staging
          ├── hpa.yaml
          ├── kustomization.yaml
          └── service-nodeport.yaml

base/kustomization.yaml
The kustmization.yaml file is the most important file in the base folder and it describes what resources you use.

apiVersion: kustomize.config.k8s.io/v1beta1
  kind: Kustomization

  resources:
    - service.yaml
    - deployment.yaml
    - hpa.yaml

Define Dev Overlay Files
The overlays folder houses environment-specific overlays. It has 3 sub-folders (one for each environment).

dev/kustomization.yaml
This file defines which base configuration to reference and patch using patchesStrategicMerge, which allows partial YAML files to be defined and overlaid on top of the base.

apiVersion: kustomize.config.k8s.io/v1beta1
  kind: Kustomization
  bases:
  - ../../base
  patchesStrategicMerge:
  - hpa.yaml
dev/hpa.yaml

This file has the same resource name as the one located in the base file. This helps in matching the file for patching. This file also contains important values, such as min/max replicas, for the dev environment.

Review Patches
To confirm that your patch config file changes are correct before applying to the cluster, you can run 
kustomize build overlays/dev

Apply Patches
Once you have confirmed that your overlays are correct, use the this command to apply the the settings to your cluster:
kubectl apply -k overlays/dev


Note:
before pushing to git, run this command to format the file and set the indentation right.
kubectl kustomize cfg fmt file_name 


## Kustomize Feature List
Field	Type	Explanation
namespace	string	add namespace to all resources
namePrefix	string	value of this field is prepended to the names of all resources
nameSuffix	string	value of this field is appended to the names of all resources
commonLabels	map[string]string	labels to add to all resources and selectors
commonAnnotations	map[string]string	annotations to add to all resources
resources	[]string	each entry in this list must resolve to an existing resource configuration file
configMapGenerator	[]ConfigMapArgs	Each entry in this list generates a ConfigMap
secretGenerator	[]SecretArgs	Each entry in this list generates a Secret
generatorOptions	GeneratorOptions	Modify behaviors of all ConfigMap and Secret generator
bases	[]string	Each entry in this list should resolve to a directory containing a kustomization.yaml file
patchesStrategicMerge	[]string	Each entry in this list should resolve a strategic merge patch of a Kubernetes object
patchesJson6902	[]Patch	Each entry in this list should resolve to a Kubernetes object and a Json Patch
vars	[]Var	Each entry is to capture text from one resource's field
images	[]Image	Each entry is to modify the name, tags and/or digest for one image without creating patches
configurations	[]string	Each entry in this list should resolve to a file containing Kustomize transformer configurations
crds	[]string	Each entry in this list should resolve to an OpenAPI definition file for Kubernetes types
