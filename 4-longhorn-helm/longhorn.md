## https://longhorn.io/docs/1.8.1/deploy/install/install-with-kubectl/



```sh
helm repo add longhorn https://charts.longhorn.io
helm repo update
helm pull  longhorn/longhorn --untar=true
```

## IMPORTANT: update the following values in the values.yaml file:
## This is to  guarantee full control over where your persistent data lives
############################ update from ###############################
longhornManager:
  nodeSelector: {}
  tolerations: []

longhornDriver:
  nodeSelector: {}
  tolerations: []

longhornUI:
  nodeSelector: {}
  tolerations: []

defaultSettings:
  defaultDataPath: ~

#########################################################################
#####################  update to ########################################
longhornManager:
  nodeSelector:
    longhorn-node: "true"
  tolerations:
    - key: "dedicated"
      operator: "Equal"
      value: "longhorn"
      effect: "NoSchedule"

longhornDriver:
  nodeSelector:
    longhorn-node: "true"
  tolerations:
    - key: "dedicated"
      operator: "Equal"
      value: "longhorn"
      effect: "NoSchedule"

longhornUI:
  nodeSelector:
    longhorn-node: "true"
  tolerations:
    - key: "dedicated"
      operator: "Equal"
      value: "longhorn"
      effect: "NoSchedule"

defaultSettings:
  defaultDataPath: /var/lib/longhorn     ## you may likely give this path permission
##################################################################################

### Label and taing the One Dedicated Node for Longhorn
```sh
## Label the Node for Longhorn Only
kubectl label node pixiecloud-worker-longhorn node.longhorn.io/create-default-disk=true

kubectl get nodes --show-labels | grep longhorn
## node to run only Longhorn and no other workloads
kubectl taint nodes pixiecloud-worker-longhorn dedicated=longhorn:NoSchedule
kubectl describe node pixiecloud-worker-longhorn | grep -i taint
```

## Label the rest  of my other nodes as such
- Because: Even though you want only pixiecloud-worker-longhorn to run the Longhorn components, Longhorn still needs to recognize all nodes that will use volumes — even if they don’t host volumes.

```sh
kubectl label node pixiecloud-worker1 longhorn-node=true
kubectl label node pixiecloud-worker2 longhorn-node=true
kubectl label node pixiecloud-worker3 longhorn-node=true
kubectl label node pixiecloud-worker4 longhorn-node=true
kubectl label node pixiecloud-worker5 longhorn-node=true
```

## Install Longhorn with Helm (Recommended) and Use nodeSelector

```sh
helm install longhorn longhorn/longhorn --namespace longhorn-system --create-namespace -f values.yaml
kubectl -n longhorn-system get po -o wide
```

## makesure the path is well provisioned for the dedicated server
## login to that server and get an assign 
```sh
sudo vgdisplay
## Create A dedicated 160GB volume for Longhorn mounted at /var/lib/
sudo lvextend -L 160G /dev/ubuntu-vg/longhorn-disk
## Make sure the mount is formatted, and made persistent via /etc/fstab
sudo mkfs.ext4 /dev/ubuntu-vg/longhorn-disk
sudo mkdir -p /var/lib/longhorn
sudo mount /dev/ubuntu-vg/longhorn-disk /var/lib/longhorn
## Make It Persistent
echo '/dev/ubuntu-vg/longhorn-disk /var/lib/longhorn ext4 defaults 0 2' | sudo tee -a /etc/fstab
## Verify It
df -h /var/lib/longhorn
```

```sh
lsblk
```

## If you need to delete
```sh
kubectl delete pods --all -n longhorn-system --grace-period=0 --force
kubectl delete job longhorn-uninstall -n longhorn-system
helm uninstall longhorn -n longhorn-system --no-hooks
kubectl delete all --all -n longhorn-system
kubectl delete crd $(kubectl get crds | grep longhorn | awk '{print $1}')
kubectl delete ns longhorn-system
kubectl get pods -A | grep longhorn
kubectl get crds | grep longhorn
kubectl delete ns longhorn-system
```

## if you have an issue with deleteting the crt
```sh
## 1. Backup & Edit the CRD to remove finalizers
kubectl get crd backuptargets.longhorn.io -o json | jq 'del(.metadata.finalizers)' | kubectl replace --raw "/apis/apiextensions.k8s.io/v1/customresourcedefinitions/backuptargets.longhorn.io" -f -

kubectl get crd engineimages.longhorn.io -o json | jq 'del(.metadata.finalizers)' | kubectl replace --raw "/apis/apiextensions.k8s.io/v1/customresourcedefinitions/engineimages.longhorn.io" -f -

kubectl get crd nodes.longhorn.io -o json | jq 'del(.metadata.finalizers)' | kubectl replace --raw "/apis/apiextensions.k8s.io/v1/customresourcedefinitions/nodes.longhorn.io" -f -

kubectl delete crd backuptargets.longhorn.io
kubectl delete crd engineimages.longhorn.io
kubectl delete crd nodes.longhorn.io
```


### note: I faced some issues and had to patch longhorn
```sh
## get the ds
kubectl -n longhorn-system get ds 
## patch the ds
kubectl -n longhorn-system patch ds engine-image-ei-db6c2b6f \
  --type='merge' \
  -p '{
    "spec": {
      "template": {
        "spec": {
          "nodeSelector": {
            "longhorn-node": "true"
          },
          "tolerations": [
            {
              "key": "dedicated",
              "operator": "Equal",
              "value": "longhorn",
              "effect": "NoSchedule"
            }
          ]
        }
      }
    }
  }'
```

## Restart the DaemonSet Pods
```sh
kubectl -n longhorn-system delete pods -l longhorn.io/engine-image=ei-db6c2b6f
kubectl -n longhorn-system get pods -l longhorn.io/engine-image=ei-db6c2b6f -o wide
```