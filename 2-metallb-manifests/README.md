[LINK:](https://metallb.universe.tf/installation/)

```sh
wget https://raw.githubusercontent.com/metallb/metallb/v0.14.9/config/manifests/metallb-native.yaml
kubectl create ns metallb-system
kubectl apply -f metallb-native.yaml -n metallb-system
mkdir metallb-manifests
cd metallb-manifests/
kubectl apply -f advert.yaml -n metallb-system
kubectl apply -f ippool.yaml -n metallb-system
```